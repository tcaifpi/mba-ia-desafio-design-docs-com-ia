# OMS Webhooks - Design Docs & Arquitetura

Projeto de especificacao arquitetural e documentacao tecnica para o subsistema assincrono de notificacoes externas via Webhooks do Order Management System (OMS).

## Documentacao Tecnica
* [PRD (Product Requirements Document)](docs/PRD.md): Requisitos de negocio, RFs, RNFs e definicao de escopo.
* [RFC (Request for Comments)](docs/RFC.md): Proposta tecnica estruturada, trade-offs e alternativas descartadas.
* [FDD (Feature Design Document)](docs/FDD.md): Modelagem Prisma, contratos JSON, integracao em `src/config/database.ts` e catalogo `WEBHOOK_*`.
* [Matriz de Rastreabilidade](docs/TRACKER.md): Rastreio completo das decisoes tecnicas com timestamps e fontes da reuniao.
* [ADRs (Architecture Decision Records)](docs/adrs/): Decisoes formais de ADR-001 a ADR-006.

## Principais Padroes Adotados
* **Transactional Outbox:** MySQL com Prisma Client (`src/config/database.ts`).
* **Worker Desacoplado:** Polling a cada 2s com `SKIP LOCKED`.
* **Seguranca:** Assinatura HMAC-SHA256 com secret rotacionavel e grace period de 24h.
* **Resiliencia:** Politica de 5 retries com backoff exponencial e persistencia em DLQ.
* **Idempotencia:** Header obrigatorio `X-Event-Id` garantindo semantica at-least-once.
