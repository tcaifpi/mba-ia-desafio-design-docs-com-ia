# Product Requirements Document (PRD): Subsistema de Webhooks OMS

## 1. Visao Geral e Objetivos
O Order Management System (OMS) necessita fornecer notificacoes automaticas, seguras e em tempo real sobre mudancas de status de pedidos para integradores externos, viabilizando a operacao com o cliente piloto Atlas.

---
## 2. Requisitos Funcionais (RF)
* **RF-01 (Cadastro e Gestao):** O cliente deve cadastrar, atualizar e remover endpoints HTTPS, especificando os eventos de interesse (ex: order.shipped, order.delivered).
* **RF-02 (Disparo At-Least-Once):** O sistema deve despachar cada evento para os endpoints cadastrados com garantia at-least-once e header X-Event-Id.
* **RF-03 (Rotacao de Segredos):** O sistema deve suportar rotacao de secret via API, garantindo grace period de 24 horas para o segredo antigo.
* **RF-04 (Historico de Entregas):** O integrador deve consultar as ultimas tentativas de envio com status HTTP, latencia e timestamps.
* **RF-05 (Replay de DLQ):** Administradores (role ADMIN) podem reprocessar eventos retidos na Dead Letter Queue com log de auditoria.

---
## 3. Requisitos Nao-Funcionais (RNF)
* **RNF-01 (Atomicidade):** A gravacao do evento outbox deve ocorrer dentro da transacao de mudanca de status do pedido ($transaction).
* **RNF-02 (Seguranca Criptografica):** Assinatura HMAC-SHA256 obrigatoria no header X-Hub-Signature-256 e recusa de URLs HTTP (nao-seguras).
* **RNF-03 (Latencia de Polling):** Worker operando a cada 2 segundos sem degradar a API HTTP principal.
* **RNF-04 (Limite de Carga):** Rejeicao estrita com erro WEBHOOK_PAYLOAD_TOO_LARGE para payloads que excedam 500 KB.
* **RNF-05 (Resiliencia):** Politica de 5 retries com backoff exponencial (1m, 5m, 30m, 2h, 12h) antes de transferir para a DLQ.

---
## 4. Fora de Escopo
* Envio de e-mail como fallback em falhas persistentes.
* Rate limiting de saida (egress throttling) inicial.
* Painel/Dashboard visual para gestao de webhooks.
* Rotina automatica de expurgo de dados entregues apos 30 dias.
