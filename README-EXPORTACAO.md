# Exportação do painel

Este pacote contém o código-fonte do painel e da API.

## Configuração

O arquivo exports/configuracao-painel.json contém o retrato das configurações escolhidas para esta versão do ZIP. No ZIP limpo, ele está vazio. O arquivo não é carregado automaticamente pelo painel.

Para restaurar as configurações exportadas em uma instalação nova:

1. Configure DATABASE_URL para um banco novo e vazio.
2. Execute pnpm install.
3. Crie as tabelas com pnpm --filter @workspace/db run push.
4. Execute pnpm --filter @workspace/db run import-config.
5. Configure seus próprios segredos fora do repositório: DISCORD_BOT_TOKEN e SESSION_SECRET.

O importador interrompe a operação se encontrar dados nas tabelas de configuração; ele não apaga nem substitui dados existentes. Tokens de bots, perfis de bot com credenciais e URLs de webhook não são transferidos. O nome dos perfis de bot pode aparecer no JSON salvo, sem token.

## Antes de publicar no GitHub

O ZIP com configurações salvas contém textos dos comandos, IDs do Discord, regras de whitelist e configurações de mensagens. Revise o JSON antes de publicar em um repositório público. Não publique tokens, arquivos .env nem segredos.

Este painel não tem login próprio; quem conseguir acessar o endereço do painel pode controlar o bot configurado. Não exponha uma instalação publicamente sem proteção de acesso.
