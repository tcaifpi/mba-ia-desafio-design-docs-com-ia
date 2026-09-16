# ADR-006: Reuso dos Padrões Arquiteturais e Tratamento de Erros Existentes

## Status
Aceito

## Contexto
A base de código do OMS já possui padrões estabelecidos para manipulação de banco de dados via Prisma, injeção de dependências modular em TypeScript, logging com Pino e um middleware centralizado para tratamento de erros HTTP.

## Decisão
Estruturar o novo domínio como um módulo isolado em `src/modules/webhooks/`, estendendo diretamente os padrões existentes:
1. Utilizar a classe base `AppError` para todas as falhas de domínio e validação, adotando o prefixo obrigatório `WEBHOOK_*` nos códigos de erro.
2. Integrar a emissão do evento diretamente no método `OrderService.changeStatus` (`src/modules/orders/order.service.ts`), compartilhando a transação atômica gerenciada pelo `PrismaService` (`src/infra/database/prisma.service.ts`).
3. Reutilizar o middleware centralizado de erros para tratamento consistente de respostas HTTP.

## Alternativas Consideradas
* **Microserviço Separado:** Descartado por complexidade desproporcional e quebra de consistência com o monólito modular existente.
* **Erros Genéricos HTTP:** Descartado para manter a padronização do catálogo de erros e facilitar a integração para os consumidores da API.

## Consequências
* **Positivas:** Curva de aprendizado nula para a equipe; padronização de contratos de erro; consistência em auditoria e testes.
* **Negativas / Trade-offs:** Dependência do ciclo de vida das classes centrais do monólito existente (`AppError` e `PrismaService`).
