# ADR 0007 — QR Code do certificado com token JWT assinado

- **Status:** Aprovado
- **Data:** 21/04/2026
- **Tema:** Segurança / verificação pública

## Contexto

Cada certificado emitido precisa de um QR Code imprimível. Ao ser escaneado por qualquer pessoa, **inclusive sem login**, ele leva a uma página que mostra:

- número do certificado;
- dados básicos da válvula (identificação, fabricante, modelo);
- empresa cliente;
- datas de emissão e validade;
- status atual (vigente, expirado ou revogado).

Com isso, fiscais da NR-13, auditores e equipes de campo validam um certificado na hora, sem precisar de conta no sistema. Na prática, o QR Code fica impresso ou colado na própria válvula.

## Decisão

A URL embutida no QR Code aponta para a rota pública do certificado e leva, na query string, um **JWT assinado** com payload mínimo: o identificador do certificado (`sub`), a data de emissão (`iat`) e a data de expiração (`exp`, 1 ano).

Regras:

1. **Chave de assinatura dedicada**, separada da chave que autentica usuários. Se uma vazar, a outra não é afetada.
2. **Algoritmo HS256** por enquanto. A migração para **RS256** está prevista para quando for preciso distribuir a chave pública a terceiros (por exemplo, fiscais validando offline).
3. **Expiração de 1 ano**, alinhada à validade de 12 meses do certificado. Revogação e expiração **não** dependem do `exp` do token: o status é sempre consultado no banco.
4. **Payload mínimo:** só o identificador do certificado, sem nenhum dado sensível. O token apenas prova que o link foi emitido pelo sistema.
5. **Rate limit** no endpoint público: **10 requisições por minuto por IP** (`@nestjs/throttler`), para mitigar a enumeração de certificados.
6. **Sem autenticação de usuário:** o endpoint é realmente público, e a única credencial é o token do link.
7. **Página pública** renderizada pelo frontend (Next.js). Ela chama o backend com o token, e o backend valida a assinatura, consulta o status e devolve apenas os dados permitidos.
8. **Revogação:** o QR Code antigo continua funcionando, mas passa a exibir "revogado", porque a validação sempre consulta o banco.

## Alternativas consideradas

| Opção | Prós | Contras |
| --- | --- | --- |
| **JWT assinado no link** (escolhida) | Prova de que o link foi emitido pelo sistema; URL não pode ser alterada sem invalidar a assinatura; stateless (sem Redis no caminho crítico) | Token de vida longa: se a chave vazar, todos os QR Codes precisam ser reemitidos |
| UUID puro no QR Code | Simples | Quem tem o UUID acessa os dados; não há camada de "link emitido por nós" |
| OAuth / token de sessão | Modelo de autenticação padrão | Exige login e quebra o caso de uso do fiscal externo |
| URL com hash HMAC | Funcionalmente equivalente | Menos suporte em bibliotecas e sem metadados de expiração |
| Token salvo no banco e validado por lookup | Revogação por token | I/O extra no caminho quente; o status já é consultado de qualquer forma |

## Consequências

- **Positivas:** endpoint público resistente à enumeração (payload mínimo + rate limit + assinatura); risco da verificação pública isolado do risco da autenticação de usuários; nenhuma dependência de sessão, o que permite escalar horizontalmente; migração futura para RS256 simples.
- **Negativas e riscos:**
  - Se a chave de assinatura vazar, todos os tokens e etiquetas impressas precisam ser reemitidos. Mitigação: chave em cofre de segredos, rotação documentada no runbook e auditoria de acesso.
  - O tamanho do QR Code fica em ~250–300 caracteres, que cabem confortavelmente em um QR versão 10 com correção de erro L.
  - O rate limit por IP não protege contra bots distribuídos. Se isso virar um problema real, a equipe adiciona um CAPTCHA invisível.
- **Plano de rotação de chave:** gerar a nova chave → aceitar as duas (nova e antiga) por um período → reemitir os tokens dos certificados vigentes com a nova chave → reimprimir as etiquetas em uso → remover a chave antiga depois de 30 dias.
