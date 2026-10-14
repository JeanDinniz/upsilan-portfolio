# ADR 0006 — Padrões de testes e qualidade de código

- **Status:** Em revisão formal, mas já aplicado na prática (thresholds ativos no CI)
- **Data:** 03/03/2026 (revisão em 15/05/2026)
- **Tema:** Qualidade / testes

## Contexto

O LESER certifica válvulas de segurança industriais. Uma falha no software pode resultar em:

- certificados inválidos emitidos para equipamentos de segurança;
- perda de rastreabilidade e de auditoria;
- impacto na segurança operacional de plantas industriais;
- responsabilidade legal para o fabricante e seus clientes.

O diagnóstico de março de 2026 mostrou que a base de testes ainda era insuficiente:

| Métrica | Valor na época | Meta |
| --- | --- | --- |
| Arquivos testados | 4 de ~15 | 100% |
| Cobertura de linhas | ~45% | ≥ 80% |
| Testes de integração | 0 | Obrigatórios |
| Testes E2E | 0 | Obrigatórios |

## Decisão

Estabelecer padrões **obrigatórios** de teste:

1. **Pirâmide de testes**, com cerca de 70% unitários, 20% de integração e 10% E2E.
   - **Unitários:** services, guards, pipes e utilitários, sem I/O e com mocks. Todo service novo precisa de `.spec.ts`.
   - **Integração:** service + repositório + **PostgreSQL real em container**, com queries TypeORM reais, transações e rollbacks.
   - **E2E:** fluxos completos via HTTP (`supertest`), cobrindo autenticação, autorização e validação.
2. **Cobertura mínima global de 80%** (statements, branches, functions, lines). O threshold fica no `jest.coverageThreshold` do `package.json` e o CI falha se qualquer métrica ficar abaixo dele.
3. **Padrão de escrita:** estrutura AAA (Arrange, Act, Assert), nomes no formato `should <comportamento> when <condição>` e **factories** reutilizáveis para mocks, em vez de objetos inline repetidos.
4. **Cenários obrigatórios por módulo crítico**, por exemplo:
   - **Auth:** login válido/inválido, usuário inativo, refresh expirado ou já rotacionado, logout, fluxo login → refresh → logout e rate limit.
   - **Certificados:** aprovação/rejeição por papel, geração de PDF, QR Code, verificação pública e trilha de auditoria.
   - **Traveller:** impedir que uma etapa obrigatória seja pulada e sincronização offline → online.
   - **Tolerâncias:** faixas de tolerância de pressão de abertura (CDTP) segundo ASME I e ASME VIII, pressão de teste de estanqueidade, "uma inspeção ativa por válvula" e fotos obrigatórias.
5. **Checklist de code review:** o PR é reprovado se faltar `.spec.ts` em service novo, se a cobertura ficar abaixo de 80%, se houver testes dependentes de ordem, testes sem assertion ou mocks inline repetidos.

## Alternativas consideradas

| Opção | Prós | Contras |
| --- | --- | --- |
| **Padrões obrigatórios + gate de cobertura no CI** (escolhida) | Confiança para refatorar; regressões detectadas cedo; testes servem como documentação viva; onboarding mais fácil | Aumento estimado de 20% a 30% no tempo inicial de desenvolvimento; testes precisam ser mantidos |
| Manter o status quo (testes opcionais) | Entregas iniciais mais rápidas | Cobertura de ~45% e nenhum teste de integração ou E2E, inaceitável para um domínio regulatório |

## Consequências

- **Positivas:** mais confiança para evoluir o código, detecção precoce de regressões e menos tempo de depuração em produção.
- **Negativas:** mais tempo de desenvolvimento no início e curva de aprendizado para quem tem menos experiência com testes.
- **Mitigações:** pair programming, templates de testes e o CI bloqueando merge sem cobertura mínima.
- **Evolução registrada:** em maio de 2026, o threshold de branches subiu de 70% para 80%, depois que a cobertura real passou de 84%. A suíte de backend hoje tem specs unitários junto ao código, testes de integração e E2E em `test/`. No frontend, o projeto ganhou testes com Vitest e **E2E com Playwright** (Chromium/Firefox/WebKit, mais um projeto dedicado a PWA/offline), executados em workflows próprios do CI.
