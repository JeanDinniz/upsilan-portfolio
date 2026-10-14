# ADR 0010 — Geração do PDF do certificado no backend

- **Status:** Aceito
- **Data:** 14/05/2026
- **Tema:** Backend / documentos regulatórios
- **Substitui:** a entrega anterior, em que o PDF era gerado no navegador

## Contexto

Até esta decisão, o "download do PDF" do certificado acontecia no navegador: o frontend abria uma rota de impressão, renderizava o HTML e chamava `window.print()`, e o usuário escolhia "Salvar como PDF".

Para um sistema de **certificação regulatória** (ASME I/VIII, NR-13), isso era dívida técnica:

1. **Resultado não determinístico:** margens, fontes e DPI dependem do navegador, do sistema operacional e da impressora virtual, então dois usuários geram PDFs diferentes byte a byte.
2. **Sem hash de referência:** um auditor externo não consegue provar que o PDF recebido é o mesmo emitido pelo sistema, e uma adulteração visual passaria despercebida.
3. **Volume inviável:** 50 certificados significam 50 cliques e 50 nomes de arquivo digitados à mão, o que bloqueava a exportação em lote.
4. **Sem API binária:** integrações programáticas (ERP do cliente, robôs regulatórios, e-mail automático) eram impossíveis.

## Decisão

Gerar o PDF **no backend** com **`@react-pdf/renderer`**. O layout é escrito em React (o mesmo modelo mental do frontend) e renderizado em Node, **sem navegador e sem Chromium**.

Implementação:

- Um template React define o layout do certificado, e um renderer devolve um `Buffer` com o PDF e o QR Code gerado no servidor (biblioteca `qrcode`).
- Helpers puros (hash de integridade, nome do arquivo) ficam separados para poderem ser testados sem carregar a biblioteca de PDF, que é ESM e deixa o boot do Jest mais lento.
- O service faz **import preguiçoso** (*lazy import*) do renderer, para não carregar a biblioteca no boot da API nem nos testes unitários.
- O endpoint `GET /certificates/:id/pdf` tem `@Auditable` (ação de download) e segue as mesmas regras de acesso da leitura do certificado. Existe também uma variante pública, protegida pelo token do QR Code.

## Alternativas consideradas

| Opção | Prós | Contras |
| --- | --- | --- |
| **`@react-pdf/renderer`** (escolhida) | Biblioteca Node pura, determinística, modelo mental React, footprint de ~3 MB | Layout reescrito em código (não reaproveita o HTML existente) |
| Puppeteer / Playwright headless | Reaproveita o HTML da rota de impressão | Chromium de ~150–200 MB no container e ~50 MB por processo; propenso a OOM (o ambiente de staging já tinha histórico de incidentes de memória); timeouts e travamentos |
| PDFKit puro | Ainda mais leve | API imperativa; difícil manter consistência com o design system |
| Serviço externo de conversão | Zero infraestrutura própria | Dependência externa, custo recorrente, latência e dados sensíveis trafegando para fora |
| CSV / dados estruturados | Simples | Não atende ao requisito de certificado oficial em PDF para auditoria |

## Consequências

- **Positivas:**
  - **Determinismo:** a mesma entrada gera os mesmos bytes (exceto o timestamp visível no rodapé).
  - **Hash de integridade no rodapé:** os 12 primeiros caracteres do SHA-256 dos campos imutáveis, que um auditor pode comparar com o hash devolvido pelo backend.
  - **QR Code sempre íntegro**, porque é gerado no servidor.
  - **Metadados XMP** (título, autor, assunto, produtor) pesquisáveis em ferramentas de auditoria documental.
  - **Primitiva reaproveitável** para exportação em lote e para o e-mail de "certificado publicado" com o PDF anexado.
  - **Sem Chromium:** o tamanho do container praticamente não muda e não surgem novos vetores de OOM.
- **Negativas:**
  - Layout duplicado entre o template do PDF e a rota de impressão, enquanto a rota antiga não é descontinuada.
  - A API do `@react-pdf` (`<Page>`, `<View>`, `<Text>`, StyleSheet) é diferente da do React DOM e do Tailwind.
  - Fontes customizadas exigem registro explícito; o MVP usa as fontes embutidas.
- **Próximos passos registrados:** descontinuar a rota de impressão depois de um período de coexistência; exportação em lote via job BullMQ; assinatura digital ICP-Brasil quando a regulação exigir; PDF bilíngue (PT/EN).
