# ADR 0009 — Rota de gestão de empresas: manter `/clients` em vez de `/admin/companies`

- **Status:** Aprovado
- **Data:** 12/05/2026
- **Tema:** Frontend / convenção de rotas e linguagem de domínio

## Contexto

Uma tarefa pedia um "CRUD de empresas/clientes no frontend em `/admin/companies`". Durante a implementação, a equipe percebeu que **a funcionalidade já existia na rota `/clients`**, com todos os critérios de aceite cumpridos:

| Critério | Implementação existente |
| --- | --- |
| Rota protegida por papel | `RoleGuard` permite só administradores; botões de ação também condicionados ao papel (defesa em camadas sobre o guard de autenticação do layout) |
| Tabela com busca, filtro e paginação no servidor | Hook de listagem com página, busca e filtro de ativo |
| Modais de criar, editar e desativar | Diálogo de formulário + diálogo de confirmação |
| Usuários vinculados | Página de detalhe com cards de usuários, válvulas e certificados da empresa |
| Auditoria | `@Auditable()` nas operações do controller de empresas |
| Validação de CNPJ | Validação com dígitos verificadores no schema Zod |

A única diferença era o **nome da rota**. Outros pontos relevantes:

- O portal admin usa um *route group* do Next.js, `(admin)`, que **não aparece na URL**. Nenhuma rota do projeto começa com `/admin`.
- No domínio, "cliente" e "empresa cliente" são sinônimos: o banco usa `companies` e a UI sempre exibiu "Clientes".
- O backend expõe `/companies/*`, e o frontend apresenta isso como "Clientes".

## Decisão

**Manter a rota em `/clients`**, sem renomear e sem criar redirect. Futuras tarefas e documentos devem usar `/clients` como referência para a gestão de empresas.

## Alternativas consideradas

| Opção | Prós | Contras |
| --- | --- | --- |
| **Manter `/clients`** (escolhida) | Nada quebra; a tarefa é fechada como "já entregue"; vocabulário da UI preservado | Inconsistência de nomes entre backend (`companies`) e frontend (`clients`) |
| Renomear para `/admin/companies` | Cumpre literalmente a tarefa | Quebra URLs salvas e deep links internos; refatoração em ~20 arquivos sem nenhum ganho funcional; contradiz o *route group* sem prefixo |
| Criar redirect `/admin/companies` → `/clients` | Atende ao nome pedido sem quebrar nada | Duas URLs canônicas para a mesma tela (atrapalha analytics e favoritos); perpetua a convenção errada em tarefas futuras |

## Consequências

- **Positivas:** nenhum trabalho destrutivo; referência clara para corrigir tarefas futuras; backend e frontend mantêm seus vocabulários.
- **Negativas (aceitas):** o backend usa `/companies` e o frontend usa `/clients`. A diferença foi aceita porque o usuário final só vê "Clientes", o termo `Company` já está consolidado no ORM e renomear o backend seria uma mudança muito maior.
- **Não objetivos explícitos:** não criar `/admin/companies`, não adicionar redirect, não renomear `/clients` no frontend e não renomear `/companies` no backend.
- **Lição de processo:** o nome pedido pela tarefa vinha de uma falha de descoberta no planejamento, não de um requisito de produto. Verificar o que já existe antes de implementar evitou retrabalho.
