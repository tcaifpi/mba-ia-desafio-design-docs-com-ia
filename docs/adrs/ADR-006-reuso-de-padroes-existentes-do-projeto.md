# ADR-006: Reuso dos Padroes Arquiteturais e Tratamento de Erros Existentes

## Status
Aceito

## Contexto
A nova funcionalidade de webhooks precisa se integrar de forma harmonica e padronizada a arquitetura do monolito existente, evitando criar abstracoes concorrentes de infraestrutura ou tratamento disperso de falhas.

## Decisao
1. **Banco de Dados:** Utilizar o client Prisma central instanciado e exportado diretamente em `src/config/database.ts` para transacoes e operacoes de dados, sem criar wrappers desnecessarios.
2. **Tratamento de Excecoes:** Estender a classe central de dominio `AppError` presente no projeto, mapeando o prefixo obrigatorio `WEBHOOK_*` e respectivos status HTTP.
3. **Logs Estruturados:** Utilizar o logger central baseado no Pino ja configurado na aplicacao.
4. **Localizacao dos Modulos:** Isolar a regra de negocios e rotas em `src/modules/webhooks/` e integrar o worker no script de background dedicado.

## Consequencias
Garante coesao total com a base legada de codigo, curva de aprendizado nula para mantenedores do OMS e facilidade de manutencao.
