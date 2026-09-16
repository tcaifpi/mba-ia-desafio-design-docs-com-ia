# Matriz de Rastreabilidade e Decisoes (Tracker)

| ID | Topico Discutido | Decisao / Direcionamento | ADR / Documento | Participantes Relevantes |
|---|---|---|---|---|
| DEC-01 | Persistencia de Eventos | Outbox Pattern no MySQL existente via Prisma | ADR-001, RFC Secao 3, FDD Secao 2 | Diego, Larissa, Bruno |
| DEC-02 | Resiliencia e Falhas | 5 retries com backoff exponencial (1m, 5m, 30m, 2h, 12h) + DLQ persistida | ADR-002, RFC Secao 3, PRD RNF-05 | Diego, Larissa |
| DEC-03 | Seguranca de Autenticacao | HMAC-SHA256 sobre raw body, secret unico por endpoint e rotacao com grace period de 24h | ADR-003, RFC Secao 3, FDD Secao 3.3 | Sofia, Bruno |
| DEC-04 | Garantia de Entrega | Semantica at-least-once com header de idempotencia X-Event-Id | ADR-004, RFC Secao 3, PRD RF-02 | Larissa, Diego |
| DEC-05 | Mecanismo de Envio | Processo Node.js isolado com polling de 2s e SKIP LOCKED | ADR-005, RFC Secao 3, FDD Secao 5 | Diego, Larissa |
| DEC-06 | Padroes de Codigo | Reuso de AppError (WEBHOOK_*), Pino, PrismaService e modulo em src/modules/webhooks | ADR-006, FDD Secao 4 e 5 | Diego, Larissa |
| DEC-07 | Seguranca de Rede | HTTPS mandatorio (Zod) e limite de payload em 500 KB (WEBHOOK_PAYLOAD_TOO_LARGE) | FDD Secao 4, RFC Secao 6 | Sofia |
| DEC-08 | Reprocessamento DLQ | Endpoint administrativo protegido por JWT role: ADMIN com log de auditoria | FDD Secao 3.4, PRD RF-05 | Diego, Sofia, Larissa |
| DEC-09 | Historico de Entregas | Consulta das ultimas tentativas via GET /webhooks/:id/deliveries | FDD Secao 3.2, PRD RF-04 | Marcos, Larissa |
| DEC-10 | Itens Fora de Escopo | Fallback por e-mail, rate limiting inicial, dashboard visual e expurgo de 30 dias | RFC Secao 5, PRD Secao 4 | Larissa, Diego |
