# ADR-003: Assinatura de Webhook via HMAC-SHA256 com Secret Único e Rotação

## Status
Aceito

## Contexto
O receptor do webhook precisa verificar a autenticidade e a integridade da notificação para garantir que ela se originou no OMS e que o payload não foi adulterado durante o trânsito.

## Decisão
Adotar autenticação por assinatura calculada com **HMAC-SHA256** sobre o corpo bruto (`raw body`) da requisição, injetada no cabeçalho `X-Hub-Signature-256`. Cada endpoint de cliente possuirá um segredo criptográfico dedicado gerado pelo sistema. O mecanismo permitirá rotação de segredo com período de transição (*grace period*) de 24 horas, durante o qual assinaturas com a chave antiga e a nova serão aceitas simultaneamente.

## Alternativas Consideradas
* **Segredo Global da Plataforma:** Descartado por representar risco sistêmico de segurança (vazamento único compromete todos os clientes).
* **Autenticação via Basic Auth ou API Key estática no Header:** Descartado por não proteger a integridade do payload contra ataques de interceptação ou adulteração.

## Consequências
* **Positivas:** Conformidade com padrões consolidados de mercado (GitHub, Stripe); isolamento total de risco entre endpoints e janelas seguras de rotação sem downtime.
* **Negativas / Trade-offs:** Exige que a aplicação receptora implemente a validação criptográfica e obriga o OMS a gerenciar múltiplas chaves ativas durante o período de transição.
