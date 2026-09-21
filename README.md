# OMS Webhooks — Documentação de Design e Arquitetura

Este repositório documenta a especificação técnica e arquitetural para o subsistema assíncrono e resiliente de notificações externas via Webhooks do Order Management System (OMS), atendendo aos requisitos operacionais do cliente integrador Atlas.

---

## 1. O Processo de Engenharia de Contexto e Design

A elaboração desta documentação partiu da análise aprofundada da reunião técnica multidisciplinar entre Engenharia, Produto, Segurança e Operações (`TRANSCRICAO.md`). O objetivo central foi evitar ambiguidades e desenhar uma solução estritamente aderente ao monólito existente, sem introduzir overengineering.

### Metodologia de Tomada de Decisão:
1. **Identificação de Restrições:** Constatou-se a ausência de um barramento distribuído (Kafka/RabbitMQ) e a restrição de capacidade operacional do time. A necessidade de transacionalidade atômica com o ciclo de vida dos pedidos direcionou a escolha pelo **Transactional Outbox Pattern** no MySQL existente.
2. **Mapeamento de Riscos de Segurança:** A Segurança da Informação exigiu validação estrita de URLs (apenas HTTPS via Zod), payloads com limite máximo de 500 KB e isolamento de credenciais. Adotou-se HMAC-SHA256 calculado sobre o payload bruto (raw body) com chave simétrica exclusiva por endpoint e janela de rotação segura (grace period de 24h).
3. **Resiliência e Desacoplamento:** Para não degradar o event-loop da API Node.js durante requisições HTTP externas de latência imprevisível, isolou-se o envio em um processo worker independente em polling ordenado (`created_at ASC`) com concorrência mitigada via `SKIP LOCKED`, combinado a 5 tentativas com backoff exponencial e persistência final em Dead Letter Queue (DLQ).
4. **Rastreabilidade Fiel:** Cada direcionamento técnico foi mapeado de forma explícita na matriz `docs/TRACKER.md` (DEC-01 a DEC-10), registrando os timestamps exatos da discussão para garantir total auditabilidade e clareza das justificativas.

---

## 2. Decisões Arquiteturais e Padrões Adotados

* **ADR-001 (Transactional Outbox no MySQL):** Persistência no mesmo banco relacional dentro de `prisma.$transaction`, garantindo consistência estrita sem risco de dual-write.
* **ADR-002 (Retry Exponencial e DLQ):** Política de 5 tentativas espaçadas (1m, 5m, 30m, 2h, 12h) e desvio de mensagens esgotadas para inspeção administrativa e replay auditado.
* **ADR-003 (Assinatura HMAC-SHA256):** Autenticação ponto a ponto com assinatura no cabeçalho `X-Hub-Signature-256`, suportando migração suave de chaves via secret anterior temporário.
* **ADR-004 (Semântica At-Least-Once e Idempotência):** Inclusão mandante do cabeçalho `X-Event-Id` único por transição de estado, permitindo que receptores operem de forma idempotente frente a retentativas de rede.
* **ADR-005 (Worker Desacoplado via Polling):** Processo background isolado com ciclo de 2 segundos, preservando a disponibilidade do tráfego transacional de pedidos do checkout.
* **ADR-006 (Aderência ao Monólito Existente):** Integração mandatória com o cliente central exportado em `src/config/database.ts`, extensão de `AppError` com prefixo `WEBHOOK_*` e reutilização do logger central Pino.

---

## 3. Estrutura dos Documentos Técnicos

* [PRD (Product Requirements Document)](docs/PRD.md): Requisitos funcionais (RF-01 a RF-05), não-funcionais (RNF-01 a RNF-05) e fronteiras de escopo acordadas.
* [RFC (Request for Comments)](docs/RFC.md): Proposta técnica detalhada, justificativa de design, impacto de infraestrutura e análise comparativa de alternativas descartadas.
* [FDD (Feature Design Document)](docs/FDD.md): Modelagem Prisma dos novos esquemas, contratos JSON dos endpoints, catálogo padronizado de erros e pontos de integração com o código existente.
* [Matriz de Rastreabilidade (Tracker)](docs/TRACKER.md): Tabela de rastreamento com tópicos, decisões, participantes e timestamps exatos da transcrição.
* [ADRs (Architecture Decision Records)](docs/adrs/): Registro formal e imutável das 6 decisões arquiteturais que balizam a implementação.
