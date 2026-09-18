# Matriz de Rastreabilidade e Decisoes (Tracker)

| ID | Topico Discutido | Decisao / Direcionamento | ADR / Documento | Fonte / Participantes | Localizacao (Timestamp) |
|---|---|---|---|---|---|
| DEC-01 | Persistencia de Eventos | Outbox Pattern no MySQL existente via Prisma | ADR-001, RFC Secao 3, FDD Secao 2 | TRANSCRICAO.md (Diego, Larissa, Bruno) | [00:04:15 - 00:08:30] |
| DEC-02 | Resiliencia e Falhas | 5 retries com backoff exponencial (1m, 5m, 30m, 2h, 12h) + DLQ persistida | ADR-002, RFC Secao 3, PRD RNF-05 | TRANSCRICAO.md (Diego, Larissa) | [00:11:40 - 00:16:10] |
| DEC-03 | Seguranca de Autenticacao | HMAC-SHA256 sobre raw body, secret unico por endpoint e rotacao com grace period de 24h | ADR-003, RFC Secao 3, FDD Secao 3.3 | TRANSCRICAO.md (Sofia, Bruno) | [00:18:25 - 00:23:45] |
| DEC-04 | Garantia de Entrega | Semantica at-least-once com header de idempotencia X-Event-Id | ADR-004, RFC Secao 3, PRD RF-02 | TRANSCRICAO.md (Larissa, Diego) | [00:25:10 - 00:28:30] |
| DEC-05 | Mecanismo de Envio | Processo Node.js isolado com polling de 2s e SKIP LOCKED | ADR-005, RFC Secao 3, FDD Secao 5 | TRANSCRICAO.md (Diego, Larissa) | [00:30:15 - 00:34:50] |
| DEC-06 | Padroes de Codigo | Reuso de AppError (WEBHOOK_*), Pino e client Prisma em src/config/database.ts | ADR-006, FDD Secao 4 e 5 | TRANSCRICAO.md (Diego, Larissa) | [00:36:20 - 00:39:10] |
| DEC-07 | Seguranca de Rede | HTTPS mandatorio (Zod) e limite de payload em 500 KB (WEBHOOK_PAYLOAD_TOO_LARGE) | FDD Secao 4, RFC Secao 6 | TRANSCRICAO.md (Sofia) | [00:41:00 - 00:44:20] |
| DEC-08 | Reprocessamento DLQ | Endpoint administrativo protegido por JWT role: ADMIN com log de auditoria | FDD Secao 3.4, PRD RF-05 | TRANSCRICAO.md (Diego, Sofia, Larissa) | [00:46:15 - 00:49:50] |
| DEC-09 | Historico de Entregas | Consulta das ultimas tentativas via GET /webhooks/:id/deliveries | FDD Secao 3.2, PRD RF-04 | TRANSCRICAO.md (Marcos, Larissa) | [00:51:30 - 00:54:15] |
| DEC-10 | Itens Fora de Escopo | Fallback por e-mail, rate limiting inicial, dashboard visual e expurgo de 30 dias | RFC Secao 5, PRD Secao 4 | TRANSCRICAO.md (Larissa, Diego) | [00:56:00 - 01:00:20] |
