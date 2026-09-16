# ADR-002: Política de Retry com Backoff Exponencial e Dead Letter Queue (DLQ)

## Status
Aceito

## Contexto
Endpoints clientes podem apresentar indisponibilidades temporárias (erros 5xx, timeouts de rede). O sistema precisa lidar com falhas transitórias sem sobrecarregar os servidores de destino nem reter indefinidamente mensagens irrecuperáveis.

## Decisão
Implementar uma política de 5 tentativas de entrega com intervalos de backoff exponencial nos seguintes degraus: **1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas**. Esgotadas as 5 tentativas sem sucesso (respostas >= 500 ou timeout), a mensagem é transferida para uma tabela persistida de Dead Letter Queue (DLQ), cessando o polling automático.

## Alternativas Consideradas
* **Fallback com Notificação por E-mail:** Descartado e adiado para fases futuras para não desviar o foco da entrega principal.
* **Tentativas Infinitas em Fila:** Descartado pelo risco de entupimento do worker e degradação progressiva do banco de dados.

## Consequências
* **Positivas:** Resiliência contra indisponibilidades momentâneas sem saturação do worker; retenção estruturada de falhas definitivas para análise operacional.
* **Negativas / Trade-offs:** Eventos com falhas definitivas exigem intervenção manual por meio de endpoints administrativos autenticados para replay.
