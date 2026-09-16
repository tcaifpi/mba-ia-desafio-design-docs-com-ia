# Feature Design Document (FDD): Subsistema de Webhooks

## 1. Visao Geral
Este documento especifica os detalhes tecnicos de implementacao do sistema de webhooks do OMS, detalhando modelos Prisma, contratos JSON, catalogo de erros AppError e pontos de integracao no codigo existente.

---
## 2. Modelagem de Dados (Prisma Schema)
A solucao adiciona tres modelos ao schema Prisma:

* **WebhookEndpoint:** Armazena URL de destino, secret ativo, secret anterior (grace period de 24h), customer_id, array de eventos assinados e status ativo.
* **WebhookEvent (Outbox):** Armazena event_id (UUID), customer_id, event_type, payload JSON, status (PENDING, PROCESSING, DELIVERED, FAILED), retry_count e proxima tentativa (next_retry_at). Indices obrigatorios em (status, created_at).
* **WebhookDeadLetter (DLQ):** Armazena eventos que esgotaram 5 tentativas de entrega, contendo payload original, ultimo erro HTTP/timeout, response body do cliente e timestamp de envio para DLQ.

---
## 3. Contratos de API (Endpoints)

### 3.1 Cadastrar Endpoint: POST /webhooks
* **Auth:** Bearer Token JWT (autenticado comum)
* **Request Body:**
```json
{"customerId": "cust_123", "url": "[https://api.atlas.com/wh](https://api.atlas.com/wh)", "events": ["order.shipped", "order.delivered"]}
```
* **Response (201 Created):**
```json
{"id": "wh_01J", "customerId": "cust_123", "url": "[https://api.atlas.com/wh](https://api.atlas.com/wh)", "secret": "whsec_live_abc123", "events": ["order.shipped", "order.delivered"], "active": true}
```

### 3.2 Listar Historico de Entregas: GET /webhooks/:id/deliveries
* **Auth:** Bearer Token JWT
* **Response (200 OK):**
```json
{"deliveries": [{"id": "del_01", "eventId": "evt_uuid", "status": "DELIVERED", "statusCode": 200, "responseTimeMs": 142, "createdAt": "2026-09-16T13:00:00Z"}]}
```

---
### 3.3 Rotacionar Segredo: POST /webhooks/:id/rotate-secret
* **Auth:** Bearer Token JWT
* **Regra:** Gera nova secret. A secret anterior permanece valida por exatamente 24 horas (grace period).
* **Response (200 OK):**
```json
{"id": "wh_01J", "newSecret": "whsec_live_new456", "expiresAt": "2026-09-17T13:00:00Z"}
```

### 3.4 Reprocessar DLQ (Admin): POST /admin/webhooks/dead-letter/:id/replay
* **Auth:** Bearer Token JWT (Requer claim role: ADMIN)
* **Regra:** Registra log obrigatorio de auditoria com ID do usuario operador e reinjeta evento na tabela outbox.
* **Response (200 OK):**
```json
{"success": true, "replayedEventId": "evt_uuid", "operatorId": "usr_admin_99"}
```

---
## 4. Catalogo de Erros (AppError)
Todos os erros do modulo herdam de AppError e utilizam o prefixo WEBHOOK_:

* **WEBHOOK_INVALID_URL (400):** URL invalida ou nao-HTTPS.
* **WEBHOOK_PAYLOAD_TOO_LARGE (413):** Payload de disparo excede 500 KB.
* **WEBHOOK_NOT_FOUND (404):** Webhook ou mensagem de DLQ inexistente.
* **WEBHOOK_ROTATION_IN_PROGRESS (409):** Tentativa de rotacionar durante janela de grace period ativa.
* **WEBHOOK_UNAUTHORIZED_ADMIN (403):** Usuario sem permissao ADMIN ao tentar replay de DLQ.

---
## 5. Integracao com Codigo Existente
* **src/modules/orders/order.service.ts:** No metodo changeStatus, verificar se ha webhooks para o cliente e gravar na outbox dentro do prisma.$transaction.
* **src/modules/webhooks/webhook.worker.ts:** Loop de polling a cada 2s lendo eventos PENDING com SKIP LOCKED e disparando HTTP com HMAC-SHA256 e X-Event-Id.
* **src/modules/webhooks/webhook.service.ts:** Metodos de negocio (registerEndpoint, rotateSecret, listDeliveries, replayDeadLetter).
* **src/infra/database/prisma.service.ts:** Reuso do client Prisma existente para gerenciar transacoes e consultas.
