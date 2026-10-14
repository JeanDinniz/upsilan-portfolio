# ADR 0004 — Storage de mídia: S3 em produção, storage compatível com S3 em dev/staging

- **Status:** Aprovado
- **Data:** 24/02/2026 (regras de evidência fotográfica confirmadas com o cliente em março de 2026)
- **Tema:** Infraestrutura / armazenamento

## Contexto

Em campo, os técnicos enviam fotos e vídeos vinculados às ordens de serviço, e o volume pode ser grande. O acesso a esses arquivos precisa ser seguro e ter prazo controlado.

## Decisão

- **Produção:** AWS S3 com CDN (CloudFront) e **URLs assinadas** (*presigned URLs*).
- **Dev/staging:** um object storage com **API compatível com S3** (DigitalOcean Spaces), sem custo adicional no provedor de staging.

Como as duas APIs são compatíveis, **o mesmo código roda nos dois ambientes**. Só as variáveis de ambiente mudam.

Tipos e limites de arquivo:

| Tipo | Formatos | Tamanho máximo |
| --- | --- | --- |
| Imagens | JPG, PNG | 10 MB por arquivo |
| Vídeos | MP4 | 500 MB por arquivo |
| PDFs | certificados gerados pelo sistema | — |

### Evidências fotográficas obrigatórias

Uma ordem de serviço **não pode ser enviada para aprovação** sem as duas fotos obrigatórias: a **plaqueta do fabricante** (`NAMEPLATE`) e uma **vista ampla** da válvula (`OVERVIEW`). Cada upload leva um metadado de categoria:

| Categoria | Descrição | Obrigatória |
| --- | --- | --- |
| `NAMEPLATE` | Plaqueta do fabricante | Sim |
| `OVERVIEW` | Vista ampla da válvula | Sim |
| `FRONT` / `REAR` / `LEFT` / `RIGHT` | Vistas frontal, traseira e laterais | Não |
| `DAMAGE` | Detalhes de avarias/anomalias | Condicional |
| `INTERNAL` | Internos após desmontagem | Condicional |

A validação acontece **no backend (camada de serviço) e no frontend** (formulário de envio).

## Alternativas consideradas

| Opção | Prós | Contras |
| --- | --- | --- |
| **Object storage S3 / compatível com S3** (escolhida) | Escalável, durável e barato; URLs assinadas dão acesso temporário sem expor arquivos publicamente; mesmo código em dev e produção | Custo variável de armazenamento e transferência |
| Guardar arquivos no disco do servidor da API | Sem dependência externa | Risco de perda por falha de disco; não escala horizontalmente; acesso controlado precisaria ser implementado à mão |

## Consequências

- **Positivas:** nenhum arquivo fica no servidor da aplicação, então não há risco de perda por falha de disco; o código é o mesmo em dev e produção; arquivos nunca ficam públicos.
- **Negativas:** custo variável por armazenamento e transferência, previsto no orçamento de infraestrutura.
- **Nota de implementação:** o upload passa por uma fila assíncrona (BullMQ). Um processador extrai metadados e gera miniaturas de imagens (sharp) e vídeos (ffmpeg) antes de persistir a referência do arquivo.
