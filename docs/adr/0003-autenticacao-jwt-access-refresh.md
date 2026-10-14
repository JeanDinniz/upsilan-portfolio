# ADR 0003 — Autenticação: JWT com access token + refresh token

- **Status:** Aprovado
- **Data:** 24/02/2026
- **Tema:** Segurança / autenticação

## Contexto

O sistema tem quatro perfis de acesso (administrador, revisor, técnico e cliente). A autenticação precisa ser segura para o portal web e para o uso em campo pelos técnicos, inclusive com suporte a operação offline.

## Decisão

Usar **JWT** com duas camadas:

- **Access token** de curta duração (**15 minutos**), stateless, validado sem consulta ao banco a cada requisição.
- **Refresh token** de longa duração (**7 dias**), armazenado no **Redis**, o que permite revogação imediata.

Fluxo:

1. Login: a API devolve o access token (15 min) e o refresh token (7 dias).
2. O cliente envia o access token em cada requisição.
3. Quando o access token expira, o cliente chama `POST /auth/refresh` com o refresh token.
4. No logout, o refresh token é invalidado no Redis.

A implementação usa `@nestjs/jwt` + `passport`. O JWT é validado por um guard global, e só as rotas marcadas como públicas ficam de fora.

## Alternativas consideradas

| Opção | Prós | Contras |
| --- | --- | --- |
| **JWT access curto + refresh no Redis** (escolhida) | Janela de exposição pequena se um token vazar; revogação imediata via Redis; sem consulta ao banco por requisição; funciona para web e campo | Um pouco mais complexo que uma sessão simples |
| Sessão simples no servidor | Modelo mais simples | Menos adequada ao contexto multiplataforma e ao uso offline em campo |

## Consequências

- **Positivas:** equilíbrio entre segurança e experiência de uso; acesso revogável na hora (logout ou bloqueio de usuário).
- **Negativas:** complexidade um pouco maior que a de uma sessão simples, justificada pelo contexto multiplataforma.
- **Evolução posterior:** o backend passou a emitir os tokens também como **cookies `httpOnly`**. O refresh token fica restrito ao caminho de refresh. Uma estratégia de transição permitiu que Bearer e cookie convivessem sem logout em massa. Os refresh tokens são guardados por usuário + sessão, o que permite invalidar uma sessão específica ou todas de uma vez. O login tem rate limiting.
