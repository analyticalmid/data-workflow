# data-workflow

[English](README.md) · Português

Um workflow de dados para o [Claude Code](https://code.claude.com), da query ao dashboard. Ele carrega no
agente o jeito certo de fazer cada tarefa de dados: revisar e corrigir SQL em vários dialetos, tratar dados
com pandas e numpy, desenhar ETL e armazenamento (medalhão, lake, warehouse, lakehouse), trabalhar no
Databricks e no Spark, montar modelos do Power BI em DAX e Power Query, desenhar dashboards e testar a
qualidade dos dados.

## Instalação

```sh
claude plugin marketplace add analyticalmid/data-workflow
claude plugin install data-workflow@data-workflow
```

Dentro de uma sessão: `/plugin marketplace add analyticalmid/data-workflow` e depois
`/plugin install data-workflow@data-workflow`.

## O contrato

Toda sessão começa com o [`contract.md`](contract.md): leitura por padrão, aprovação antes de qualquer
comando que altere dados num alvo compartilhado, granularidade declarada antes de todo join, toda
transformação reconciliada, credenciais e dados pessoais fora do output. As instruções do próprio
repositório se somam a ele e vencem quando discordam.

## O que vem no plugin

| Peça | O que faz |
| --- | --- |
| Contrato | As regras acima, sempre ativas, impressas no início da sessão |

Skills, comandos e revisores entram a cada release; esta tabela lista o que já está na atual.

## Licença

[MIT](LICENSE)
