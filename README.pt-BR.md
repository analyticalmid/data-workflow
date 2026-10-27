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
| Skill `sql` | Revisa, verifica e corrige queries: corretude (fan-out, NULL, granularidade, janelas, tempo), performance pelo plano e segurança. Referências para PostgreSQL, T-SQL (SQL Server, Azure SQL, Fabric), Spark SQL / Databricks, BigQuery e Snowflake |
| `/data-workflow:sql-review` | Roda essa revisão numa query, num arquivo, num objeto ou no SQL do diff atual, e verifica a correção contra a original |
| Agente `sql-reviewer` | Segundo revisor, só de leitura, para queries longas, migrations e comandos que escrevem |
| Skill `python-data` | pandas e numpy: carga enxuta, dados faltantes, merges validados, vetorização e memória, datas e fusos, reshape, reconciliação; referências das mudanças do pandas 3 e do NumPy 2 |
| `/data-workflow:data-profile` | Faz o perfil de um arquivo, DataFrame ou tabela do warehouse: estatísticas por coluna, valores suspeitos, chaves candidatas e granularidade, frescor, dados pessoais sinalizados |
| Skill `etl-pipelines` | Extração (APIs, watermarks, CDC, arquivos, streams), cargas idempotentes e backfills, dbt incremental, orquestração; referências de medalhão e armazenamento (lake, warehouse, lakehouse, Delta, Iceberg, Parquet) e de modelagem dimensional (granularidade, fatos, SCD) |
| Skill `data-quality` | Quais testes escrever e o que uma falha faz (falhar, avisar, quarentena) em dbt, Lakeflow, Snowflake ou Great Expectations; dados pessoais sob LGPD e GDPR, prazos de incidente e mascaramento por plataforma |
| `/data-workflow:pipeline` | Desenha ou revisa um pipeline de ponta a ponta: saídas, fontes, camadas, cargas idempotentes, backfills, testes, agenda, dados pessoais |
| Agente `data-model-reviewer` | Revisão só de leitura de granularidade, chaves, SCD, cargas, backfills, testes e dados pessoais num modelo ou pipeline |

Skills, comandos e revisores entram a cada release; esta tabela lista o que já está na atual.

## Licença

[MIT](LICENSE)
