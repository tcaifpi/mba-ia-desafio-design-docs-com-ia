# ADR-001: Implementação do Padrão Transactional Outbox no MySQL Existente

## Status
Aceito

## Contexto
O Order Management System (OMS) gerencia transições críticas de status de pedidos (`OrderService.changeStatus`). Para notificar sistemas externos sobre essas alterações sem risco de inconsistência, é necessário desacoplar o disparo HTTP da transação de banco. Um disparo síncrono direto durante a transação adicionaria latência inaceitável e vulnerabilidade a falhas de rede.

## Decisão
Adotar o padrão **Transactional Outbox** utilizando o banco relacional MySQL existente gerenciado via Prisma ORM. O registro do evento será gravado na mesma transação atômica da alteração do pedido, garantindo consistência estrita sem introduzir novos componentes de infraestrutura neste estágio.

## Alternativas Consideradas
* **Redis Streams / RabbitMQ / SQS:** Descartado por introduzir overhead operacional desnecessário (overengineering) e risco de inconsistência de duas fases (dual write) para o tamanho da equipe e volume atual.
* **Disparo HTTP Síncrono direto no OrderService:** Descartado por comprometer a disponibilidade do fluxo de pedidos caso endpoints externos estejam instáveis.

## Consequências
* **Positivas:** Atomicidade garantida pela transação do MySQL; ausência de novos custos de infraestrutura; reuso do pool do Prisma.
* **Negativas / Trade-offs:** Tabela de outbox requer manutenção de índices adequados (`status`, `created_at`) e tráfego adicional de leitura e escrita no banco relacional.
