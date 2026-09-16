# ADR-005: Worker em Processo Separado com Polling Otimizado

## Status
Aceito

## Contexto
O processamento da outbox envolve chamadas HTTP de I/O externo com latências variáveis. Executar esse loop de entrega no mesmo processo da API principal do OMS poderia saturar o event-loop do Node.js, degradando o tempo de resposta das operações de checkout.

## Decisão
Executar o worker de entrega de webhooks como um **processo Node.js independente** da API HTTP, operando em polling contínuo com ciclo de **2 segundos**. O worker fará leituras em lote ordenadas cronologicamente (`created_at ASC`) selecionando eventos com status `PENDING` com controle de concorrência no banco (`SKIP LOCKED` ou marcação atômica para `PROCESSING`).

## Alternativas Consideradas
* **Worker em Background Threads / In-Process via setInterval:** Descartado para evitar concorrência de recursos e impedir que instabilidades do worker afetem a API principal.
* **Notificação Reativa via Triggers:** Inviabilizado pela ausência de suporte nativo equivalente com a mesma maturidade no MySQL existente.

## Consequências
* **Positivas:** Isolamento de falhas e recursos; escalabilidade independente do componente de envio; ausência de impacto no throughput das rotas HTTP do OMS.
* **Negativas / Trade-offs:** Necessidade de gerenciar um processo adicional em runtime e latência mínima inerente ao intervalo do polling (até 2 segundos).
