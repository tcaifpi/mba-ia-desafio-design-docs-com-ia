# RFC: Sistema de Webhooks para Notificacao de Pedidos

## Metadados
* **Titulo:** Sistema de Notificacao Externa de Transicoes de Pedidos via Webhooks
* **Autor:** Tiago Cavalcante Aragao
* **Status:** Em Revisao (Proposed)
* **Data:** 2026-09-16
* **Revisores:** Diego (Tech Lead), Larissa (Product Manager), Sofia (Seguranca da Informacao), Bruno (Engenharia de Software), Marcos (Operacoes / Engenharia)

---
## 1. Resumo Executivo (TL;DR)
Esta RFC propoe a implementacao de um subsistema assincrono e resiliente de notificacoes externas via Webhooks para o Order Management System (OMS).
A solucao adota o padrao Transactional Outbox integrado ao MySQL existente via Prisma, com entrega desacoplada processada por um Worker independente em polling de 2 segundos.
O mecanismo conta com assinatura criptografica HMAC-SHA256, politica escalonada de 5 retries com backoff exponencial, desvio para Dead Letter Queue (DLQ) persistida e garantia de entrega at-least-once idempotente via header X-Event-Id.

---
## 2. Contexto e Declaracao do Problema
O OMS gerencia operacoes de e-commerce e seu ciclo de pedidos e regulado por uma maquina de estados finita (OrderService.changeStatus).
Clientes empresariais integradores (com destaque para o cliente piloto Atlas) necessitam receber atualizacoes operacionais imediatas sempre que um pedido transicionar entre estados (ex: PAID, PACKED, SHIPPED, DELIVERED, CANCELED).
Atualmente o OMS nao possui barramento de mensageria. Realizar disparos HTTP sincronos dentro da transacao do pedido traria acoplamento temporal critico, latencia descontrolada e risco de falhas operacionais em cascata.

---
## 3. Proposta Tecnica
A arquitetura proposta estrutura-se em quatro pilares fundamentais:
1. Transactional Outbox no MySQL: Na mesma transacao que comita a alteracao do pedido ($transaction), grava-se um registro na tabela webhook_events caso o cliente tenha webhook cadastrado para o evento.
2. Worker Desacoplado via Polling: Processo Node.js em background executa polling a cada 2 segundos, selecionando eventos PENDING ordenados por created_at ASC com controle de concorrencia.
3. Resiliencia e DLQ: Politica de 5 retries em backoff exponencial (1m, 5m, 30m, 2h, 12h). Esgotadas as tentativas, o registro e transferido para a tabela webhook_dead_letter (DLQ).
4. Seguranca e Idempotencia: Assinatura obrigatoria no header X-Hub-Signature-256 via HMAC-SHA256 (secret unico por endpoint com rotacao de 24h). O header X-Event-Id garante idempotencia no cliente.

---
## 4. Alternativas Consideradas e Descartadas
* Redis Streams / RabbitMQ / Kafka: Descartado por ser considerado overengineering para o tamanho do time e volume atual, adicionando overhead de infraestrutura e complexidade de dual-write.
* Disparo HTTP Sincrono no OrderService: Descartado pelo risco de falha na transacao do pedido caso o endpoint receptor apresente instabilidade ou timeout.
* Segredo Global da Plataforma: Rejeitado pela Seguranca da Informacao por risco de comprometimento sistemico caso uma secret seja vazada por um unico cliente.

---
## 5. Questoes em Aberto e Pontos Adiados
1. Fallback por E-mail: Adiado para proximas fases para preservar a entrega nas 3 sprints planejadas.
2. Rate Limiting de Saida: Sera observado o comportamento em producao com a Atlas antes de estipular quotas rigidas.
3. Expurgo de Eventos Entregues: Rotina de arquivamento apos 30 dias considerada fora do escopo inicial da feature.

---
## 6. Impacto e Riscos
* Impacto no Banco: Mitigado por criacao de indices compostos em (status, created_at).
* Seguranca de Rede: Validacao obrigatoria de URLs HTTPS via Zod e limite rigido de payload em 500 KB.
* Retencao na DLQ: Mitigado por endpoint administrativo de replay restrito a usuarios com role ADMIN.

---
## 7. Decisoes Relacionadas (ADRs)
* [ADR-001: Outbox Pattern no MySQL](adrs/ADR-001-padrao-outbox-no-mysql.md)
* [ADR-002: Politica de Retry e DLQ](adrs/ADR-002-politica-de-retry-e-dead-letter-queue.md)
* [ADR-003: Autenticacao HMAC-SHA256](adrs/ADR-003-autenticacao-hmac-sha256-com-secret-por-endpoint.md)
* [ADR-004: Garantia At-Least-Once com X-Event-Id](adrs/ADR-004-garantia-at-least-once-com-x-event-id.md)
* [ADR-005: Worker em Processo Separado](adrs/ADR-005-worker-em-processo-separado-com-polling.md)
* [ADR-006: Reuso de Padroes Existentes](adrs/ADR-006-reuso-de-padroes-existentes-do-projeto.md)
## 5. Questoes em Aberto e Pontos Adiados
1. Fallback por E-mail: Adiado para proximas fases para preservar o foco nas 3 sprints.
2. Rate Limiting de Saida: Observar primeiro a volumetria em producao com a Atlas.
3. Expurgo de Eventos Entregues: Rotina de arquivamento apos 30 dias fora do escopo inicial.

---
## 6. Impacto e Riscos
* Impacto no Banco: Mitigado por criacao de indices em (status, created_at) e polling ordenado.
* Seguranca de Rede: HTTPS obrigatorio via Zod e limite de payload em 500 KB.
* Retencao na DLQ: Mitigado por endpoint administrativo de replay restrito a role ADMIN.

---
## 7. Decisoes Relacionadas (ADRs)
* [ADR-001: Outbox Pattern no MySQL](adrs/ADR-001-padrao-outbox-no-mysql.md)
* [ADR-002: Politica de Retry e DLQ](adrs/ADR-002-politica-de-retry-e-dead-letter-queue.md)
* [ADR-003: Autenticacao HMAC-SHA256](adrs/ADR-003-autenticacao-hmac-sha256-com-secret-por-endpoint.md)
* [ADR-004: Garantia At-Least-Once com X-Event-Id](adrs/ADR-004-garantia-at-least-once-com-x-event-id.md)
* [ADR-005: Worker em Processo Separado](adrs/ADR-005-worker-em-processo-separado-com-polling.md)
* [ADR-006: Reuso de Padroes Existentes](adrs/ADR-006-reuso-de-padroes-existentes-do-projeto.md)
## 5. Questoes em Aberto e Pontos Adiados
1. Fallback por E-mail: Adiado para proximas fases para preservar o foco nas 3 sprints.
2. Rate Limiting de Saida: Observar primeiro a volumetria em producao com a Atlas.
3. Expurgo de Eventos Entregues: Rotina de arquivamento apos 30 dias fora do escopo inicial.

---
## 6. Impacto e Riscos
* Impacto no Banco: Mitigado por criacao de indices em (status, created_at) e polling ordenado.
* Seguranca de Rede: HTTPS obrigatorio via Zod e limite de payload em 500 KB.
* Retencao na DLQ: Mitigado por endpoint administrativo de replay restrito a role ADMIN.

---
## 7. Decisoes Relacionadas (ADRs)
* [ADR-001: Outbox Pattern no MySQL](adrs/ADR-001-padrao-outbox-no-mysql.md)
* [ADR-002: Politica de Retry e DLQ](adrs/ADR-002-politica-de-retry-e-dead-letter-queue.md)
* [ADR-003: Autenticacao HMAC-SHA256](adrs/ADR-003-autenticacao-hmac-sha256-com-secret-por-endpoint.md)
* [ADR-004: Garantia At-Least-Once com X-Event-Id](adrs/ADR-004-garantia-at-least-once-com-x-event-id.md)
* [ADR-005: Worker em Processo Separado](adrs/ADR-005-worker-em-processo-separado-com-polling.md)
* [ADR-006: Reuso de Padroes Existentes](adrs/ADR-006-reuso-de-padroes-existentes-do-projeto.md)
