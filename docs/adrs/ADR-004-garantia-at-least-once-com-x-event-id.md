# ADR-004: Garantia de Entrega At-Least-Once com Header de Idempotência X-Event-Id

## Status
Aceito

## Contexto
Falhas de rede ou timeouts podem ocorrer mesmo após o receptor ter processado com sucesso uma notificação, impedindo a confirmação HTTP de retorno. O sistema emissor precisa garantir que nenhuma notificação seja perdida.

## Decisão
Adotar a semântica de entrega **at-least-once**, injetando um identificador imutável e único via cabeçalho HTTP `X-Event-Id` (UUIDv4 gerado na criação do evento na tabela outbox). A responsabilidade pela deduplicação e garantia de idempotência no processamento é transferida explicitamente para a aplicação receptora.

## Alternativas Consideradas
* **Garantia Exactly-Once via Two-Phase Commit (2PC):** Descartada por inviabilidade técnica sobre protocolo HTTP com múltiplos sistemas heterogêneos externos.
* **At-Most-Once (Disparo único sem reenvio):** Descartada pelo risco de perda de eventos operacionais críticos.

## Consequências
* **Positivas:** Tolerância a falhas de rede; simplificação da lógica do worker; rastreabilidade de ponta a ponta correlacionando logs do emissor e do receptor.
* **Negativas / Trade-offs:** Exige que os clientes receptores armazenem os IDs de eventos já processados para descartar entregas duplicadas.
