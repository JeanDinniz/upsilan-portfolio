# ADR 0002 — Banco de dados: PostgreSQL + Redis

- **Status:** Aprovado
- **Data:** 24/02/2026 (regras de modelagem confirmadas com o cliente em março de 2026)
- **Tema:** Dados / persistência

## Contexto

O sistema armazena dados estruturados com relacionamentos importantes (empresas, usuários, válvulas, ordens de serviço, certificados, trilha de auditoria). Também precisa de armazenamento rápido para sessões, cache e filas de tarefas assíncronas.

## Decisão

- **PostgreSQL 15** como banco principal.
- **Redis 7** para refresh tokens (com expiração automática), cache e filas de jobs assíncronos (upload e e-mail).

Por que PostgreSQL:

- Banco relacional maduro, com transações ACID.
- Suporte a JSON/JSONB para campos flexíveis, como os metadados da auditoria e os dados técnicos da inspeção.
- Busca textual nativa e grande variedade de índices.
- Bom suporte nos provedores de nuvem usados pelo projeto.

Por que Redis:

- Refresh tokens com TTL automático.
- Cache de dados acessados com frequência.
- Fila de jobs assíncronos.

### Regras de negócio que influenciaram a modelagem

Confirmadas com o cliente em março de 2026:

1. **Identidade da válvula:** a válvula é identificada de forma única pela combinação **número de série + fabricante**, com constraint `UNIQUE` no banco. A TAG da válvula **não** é única: pode se repetir até dentro do mesmo cliente, então não há constraint nela.
2. **Uma inspeção ativa por válvula:** só pode existir uma ordem de serviço ativa (rascunho, pendente ou em revisão) por válvula. A regra foi implementada como **índice único parcial** no PostgreSQL, e não apenas como validação na camada de serviço, porque o banco garante a integridade mesmo em condições de corrida (*race condition*).
3. **Traveller digital:** o checklist de acompanhamento do processo é uma entidade própria, vinculada à ordem de serviço.
4. **Instrumentos de medição:** um cadastro próprio, com alertas de vencimento de calibração, ficou acordado como melhoria futura. No escopo atual, os dados do instrumento são registrados como campos simples dentro de cada inspeção.

## Alternativas consideradas

| Opção | Prós | Contras |
| --- | --- | --- |
| **PostgreSQL + Redis** (escolhida) | Responsabilidades bem separadas (dados relacionais × dados efêmeros e filas); stack conhecida pela equipe; bom suporte em nuvem | Dois armazenamentos para operar |
| Regra "uma inspeção ativa" só na camada de serviço | Mais simples de implementar e de testar unitariamente | Não protege contra duas requisições simultâneas (*race condition*) |
| Regra "uma inspeção ativa" com índice único parcial (escolhida) | Integridade garantida pelo banco, mesmo sob concorrência | Regra de negócio passa a existir também no schema e precisa de migration |

## Consequências

- **Positivas:** stack conhecida pela equipe e bem suportada nos ambientes de nuvem; integridade das regras críticas garantida no próprio banco.
- **Negativas:** dois armazenamentos para gerenciar, embora com responsabilidades bem separadas.
- **Nota de implementação:** as filas assíncronas acabaram implementadas com **BullMQ** sobre o Redis (filas de e-mail, de processamento de uploads e de jobs de certificados), e o schema é versionado com migrations do TypeORM (`synchronize` desligado).
