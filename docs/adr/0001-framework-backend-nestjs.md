# ADR 0001 — Framework de backend: NestJS

- **Status:** Aprovado
- **Data:** 24/02/2026
- **Tema:** Backend / arquitetura

## Contexto

O backend do LESER precisava, desde o início, de autenticação, controle de permissões por perfil, um fluxo de aprovação (ordem de serviço → revisão → certificado), upload de mídia e geração de certificados. A equipe já tinha experiência com Node.js e queria uma estrutura que se mantivesse organizada com vários desenvolvedores trabalhando em paralelo.

## Decisão

Adotar **NestJS** (TypeScript) como framework de backend.

Motivos registrados pela equipe:

- Arquitetura modular, com o código organizado por domínio (autenticação, válvulas, ordens de serviço, certificados, traveller etc.).
- Injeção de dependência nativa, o que facilita testes unitários com mocks.
- Guards, interceptors e decorators nativos, adequados para controle de acesso por papel (RBAC) e auditoria.
- TypeScript obrigatório, o que reduz erros em tempo de execução.
- Integração nativa com TypeORM (entidades e migrations).
- Familiaridade da equipe com o ecossistema Node.js.

## Alternativas consideradas

> O registro original não comparou frameworks concorrentes de forma explícita. A tabela resume os prós e contras que a própria equipe registrou para a opção escolhida.

| Opção | Prós | Contras |
| --- | --- | --- |
| **NestJS** (escolhida) | Estrutura padronizada por módulos; DI facilita testes; guards/interceptors/decorators prontos para RBAC e auditoria; integração com TypeORM | Curva de aprendizado inicial para quem nunca usou NestJS |

## Consequências

- **Positivas:** estrutura padronizada desde o primeiro módulo; testes unitários mais simples de escrever, já que as dependências são injetadas e substituíveis por mocks.
- **Negativas:** curva de aprendizado inicial para os desenvolvedores sem experiência prévia com o framework.
- **Na prática:** o backend cresceu para mais de 15 módulos de domínio (auth, users, companies, valves, service-orders, tolerances, traveller, certificates, uploads, storage, audit, mail, sync, metrics, entre outros), todos registrados no módulo raiz, com guards e interceptors globais (ver [arquitetura](../architecture.md)).
