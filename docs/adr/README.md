# Architecture Decision Records (ADRs)

Decisões de arquitetura do LESER, a plataforma de certificação de válvulas de segurança (PSV).

> **Sobre estes documentos:** os ADRs foram escritos e aprovados **pela equipe do projeto** (arquiteto de solução, dev sênior/líder técnico e desenvolvedores) ao longo do desenvolvimento. Aqui eles aparecem reescritos em português, no formato MADR enxuto, com a mesma substância técnica dos originais. Foram removidos dados internos (nomes, ambientes, IDs de tarefas, credenciais). O ADR 0008 foi escrito pelo autor deste portfólio.
>
> Notas de "evolução" ou "nota de implementação" registram como a decisão se materializou no código, conforme verificado no repositório.

| Nº | Título | Tema | Resumo |
| --- | --- | --- | --- |
| [0001](0001-framework-backend-nestjs.md) | Framework de backend: NestJS | Backend | NestJS pela modularidade, DI testável e guards/interceptors/decorators prontos para RBAC e auditoria. |
| [0002](0002-postgresql-e-redis.md) | Banco de dados: PostgreSQL + Redis | Dados | PostgreSQL 15 (ACID, JSONB, índice único parcial para "uma inspeção ativa por válvula") + Redis 7 para tokens, cache e filas. |
| [0003](0003-autenticacao-jwt-access-refresh.md) | Autenticação: JWT access + refresh | Segurança | Access token de 15 min stateless + refresh token de 7 dias no Redis, revogável na hora. |
| [0004](0004-storage-de-midia-s3.md) | Storage de mídia: S3 | Infraestrutura | Object storage compatível com S3, URLs assinadas e o mesmo código em dev e produção; fotos obrigatórias por categoria. |
| [0005](0005-frontend-design-system-shadcn.md) | Frontend: shadcn/ui + Tailwind | Frontend / UI | Componentes no repositório (Radix + Tailwind), design system touch-first alinhado à identidade visual da LESER. |
| [0006](0006-padroes-de-testes.md) | Padrões de testes e qualidade | Qualidade | Pirâmide 70/20/10, cobertura global ≥ 80% bloqueante no CI, AAA + factories e cenários obrigatórios por módulo crítico. |
| [0007](0007-qr-code-com-jwt-assinado.md) | QR Code com JWT assinado | Segurança | Verificação pública sem login via JWT de payload mínimo, chave dedicada, rate limit e status sempre consultado no banco. |
| [0008](0008-pwa-service-worker-workbox.md) | Service Worker com Workbox `injectManifest` | Frontend / PWA | SW em TypeScript, sem wrapper, extensível para Background Sync/Push e com atualização de versão controlada pelo usuário. |
| [0009](0009-rota-clients-vs-admin-companies.md) | Rota `/clients` vs `/admin/companies` | Frontend / domínio | Mantém a rota existente, evitando uma refatoração destrutiva sem ganho funcional; registra a linguagem de domínio. |
| [0010](0010-geracao-de-pdf-no-backend.md) | PDF do certificado no backend | Backend / documentos | `@react-pdf/renderer` em Node, sem Chromium: PDF determinístico com hash de integridade, QR Code e metadados. |

Veja também a [visão de arquitetura](../architecture.md).
