# ADR 0008 — Service Worker do PWA: Workbox standalone (`injectManifest`)

- **Status:** Aprovado
- **Data:** 30/04/2026
- **Tema:** Frontend / PWA offline-first
- **Autoria:** escrito pelo autor deste portfólio, como parte da equipe

## Contexto

Um dos ciclos de desenvolvimento transformou o portal LESER em um **PWA offline-first** para técnicos em campo. A camada de Service Worker precisava:

- pré-cachear ícones e favicon com revisão controlada;
- cachear assets estáticos (`CacheFirst`);
- cachear a shell HTML (`StaleWhileRevalidate`);
- cachear GETs de API com fallback offline (`NetworkFirst`);
- atualizar de versão de forma controlada, com um toast "Nova versão disponível — Recarregar";
- **nunca** cachear endpoints sensíveis (autenticação) nem mutações.

O frontend roda em **Next.js 16** com `output: 'standalone'`. As entregas seguintes do mesmo ciclo trariam **Background Sync** e **Push Notifications**, que exigem lógica customizada dentro do Service Worker. Esse requisito pesou na escolha.

## Decisão

Usar **Workbox standalone com `injectManifest`** e um Service Worker escrito à mão em TypeScript (`sw.ts`).

- O `sw.ts` é empacotado com **esbuild**, e o `workbox-build.injectManifest` injeta o manifesto de pré-cache. O resultado é um `sw.js` autocontido, gerado num passo `postbuild` depois do `next build`.
- O `next.config.ts` não usa nenhum wrapper de terceiros.

Estratégias de cache:

| Rota | Estratégia | Motivo |
| --- | --- | --- |
| Ícones e favicon | Pré-cache com revisão | Ativos fixos do PWA |
| Assets com hash (`/_next/static`) | CacheFirst | O nome muda a cada build, então o conteúdo nunca muda |
| Shell HTML | StaleWhileRevalidate | Entrega imediata e atualização em segundo plano |
| GETs de API | NetworkFirst | Online preferencial, cache como fallback |
| Autenticação | Sem cache | Credenciais nunca são cacheadas |
| POST / PUT / PATCH / DELETE | Sem cache | Mutações nunca são cacheadas |

**Atualização controlada (sem `skipWaiting` automático):** o novo SW fica em *waiting*. O `workbox-window` detecta isso e um hook React exibe o toast. Quando o usuário confirma, o SW recebe `SKIP_WAITING`, ativa, assume as abas (`clientsClaim`) e a página recarrega no evento `controlling`. O listener de `controlling` só é registrado **depois** do evento `waiting`, o que evita um loop de recarregamento na primeira instalação do SW.

## Alternativas consideradas

| Opção | Prós | Contras |
| --- | --- | --- |
| `@ducanh2912/next-pwa` | Configuração mínima, integração direta com Next.js | Suporte ao Next.js 16 não confirmado; wrappers de SW historicamente conflitam com `output: 'standalone'` |
| `@serwist/next` | Projeto ativo, suporta App Router | Camada extra de abstração sem benefício concreto neste cenário |
| `next-pwa` original | — | Abandonado desde 2022 |
| Workbox `generateSW` | Configuração mínima, sem SW manual | Não suporta Background Sync nem Push, o que forçaria uma migração no meio do ciclo |
| **Workbox `injectManifest` + `sw.ts`** (escolhida) | Controle total; extensível para Background Sync e Push sem migração; SW auditável em TypeScript | Exige um script de build com esbuild; `sw.ts` fica fora do `tsconfig` principal |

## Consequências

- **Positivas:** nenhuma dependência de wrapper de terceiros, então atualizações do Next.js não quebram o SW; SW tipado com os tipos do Workbox; extensível sem trocar a infraestrutura de build; atualização de versão sob controle do usuário.
- **Negativas e riscos:**
  - `sw.ts` fica fora do `tsconfig` principal por conflito entre as libs `dom` e `webworker`. Mitigação: diretiva `/// <reference lib="webworker" />`.
  - `sw.js` é um artefato gerado e ignorado pelo git, então o CI precisa de um job de build do frontend para evitar *drift* silencioso.
  - A rota *catch-all* `StaleWhileRevalidate` pode cachear arquivos novos de `public/` sem intenção. Mitigação: avaliar uma rota específica sempre que um novo tipo de arquivo estático for adicionado.
- **Confirmação posterior:** a decisão se pagou. O mesmo `sw.ts` ganhou depois o handler de **Background Sync** para a fila de uploads e para a *outbox* de mutações offline, além de uma página `/offline` pré-cacheada no install, sem nenhuma mudança na infraestrutura de build.
