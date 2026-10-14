# ADR 0005 — Frontend: shadcn/ui + Tailwind e design system próprio

- **Status:** Aprovado
- **Data:** 24/02/2026 (revisão em 24/03/2026 para alinhar com o manual de identidade visual oficial da LESER)
- **Tema:** Frontend / UI

## Contexto

O LESER tem duas experiências web com perfis bem diferentes:

- **Portal administrativo:** usado pela equipe interna (revisores, técnicos, administradores). É uma interface densa, com tabelas, formulários e fluxos de trabalho.
- **Portal do cliente:** usado pelas empresas clientes. É mais simples e focado em consultar válvulas e certificados.

Com vários desenvolvedores entregando telas em paralelo, padronizar o visual e o código era essencial para manter a consistência.

## Decisão

Adotar **shadcn/ui** como base de componentes para os dois portais, construída sobre **Radix UI** (acessibilidade) e **Tailwind CSS** (estilização).

O shadcn/ui não é instalado como dependência: os componentes são **copiados para o repositório** e passam a ser código do projeto, o que dá controle total e customização sem limitações.

Bibliotecas complementares: `lucide-react` (ícones), `recharts` (gráficos), `react-hook-form` + `zod` (formulários e validação), `@tanstack/react-table` (tabelas), `next-themes` (tema claro/escuro), `date-fns` (datas) e `@dnd-kit` (drag and drop *touch-friendly*).

### Principais regras do design system

- **Identidade visual:** a paleta segue o manual oficial da LESER (azul institucional como cor primária e o vermelho oficial como cor destrutiva). As cores são definidas como variáveis CSS semânticas, e cor *hardcoded* em componente é proibida. O itálico é proibido em toda a interface, por regra do manual da marca.
- **Touch-first:** uso primário em tablets em campo, com área clicável mínima de 44×44 px, sidebar recolhível e Kanban com arrastar e soltar.
- **Tema:** claro como padrão, escuro opcional.
- **Padrões de interação:** `Dialog` para formulários curtos, `Sheet` para detalhes e formulários médios, página própria para fluxos complexos, `AlertDialog` obrigatório antes de ações destrutivas, nunca modal dentro de modal e nunca `alert()` nativo.
- **Acessibilidade mínima:** `alt` em imagens, `aria-label` em botões só com ícone, foco gerenciado pelo Radix e contraste mínimo de 4,5:1.
- **Código:** TypeScript sem `any`, Server Components por padrão e `'use client'` só quando necessário.

## Alternativas consideradas

| Opção | Prós | Contras |
| --- | --- | --- |
| **shadcn/ui + Radix + Tailwind** (escolhida) | Código dos componentes vive no repositório; customização total; acessibilidade nativa (Radix); visual limpo, sem "cara" de framework | Componentes complexos (tabelas avançadas, gráficos) exigem bibliotecas complementares |
| Ant Design | Catálogo amplo e pronto | Visual muito opinionado, difícil de adaptar à identidade da LESER |
| MUI (Material UI) | Ecossistema grande | Estética Material muito reconhecível; bundle pesado |
| PrimeReact | Muitos componentes | Comunidade menor, menos material de apoio |

## Consequências

- **Positivas:** liberdade de design sem depender de versões de biblioteca; o time entende e modifica os componentes com facilidade; consistência entre portal admin e portal do cliente.
- **Negativas:** componentes mais complexos precisam de bibliotecas complementares.
- **Evolução:** o plano inicial previa um app nativo separado para os técnicos. O projeto acabou adotando um **PWA offline-first** no mesmo frontend (ver [ADR 0008](0008-pwa-service-worker-workbox.md)), que hoje roda em Next.js 16 / React 19 e herda este design system.
