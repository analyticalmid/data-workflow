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

Toda sessão começa com o [`contract.md`](contract.md): carregar a skill do assunto antes de responder, leitura por
padrão, aprovação antes de qualquer comando que altere dados num alvo compartilhado, granularidade declarada antes de
todo join, toda transformação reconciliada, credenciais e dados pessoais fora do output. As instruções do próprio
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
| Skill `databricks-spark` | Execução e shuffles do Spark, funções nativas vs UDFs, partições e skew, cache, layout Delta (limites de particionamento, OPTIMIZE, VACUUM, deletion vectors), Unity Catalog, compute e custo, Lakeflow Jobs e pipelines, bundles, secrets, Structured Streaming, testes de PySpark |
| Skill `power-bi` | Modelo estrela, direção de filtro, muitos-para-muitos, tabela de datas, modos de armazenamento (Import, DirectQuery, Dual, Direct Lake), RLS, memória do VertiPaq; referências de DAX (contextos, CALCULATE, DIVIDE, variáveis, inteligência de tempo, formatação) e Power Query M (folding, staging, atualização incremental, privacidade, erros) |
| Skill `dashboard-design` | Leitor e decisões primeiro, KPIs com comparação, layout por importância e celular primeiro, gráfico por pergunta, contraste WCAG e cor nunca sozinha, segmentações e drill, performance da página, acessibilidade e layout mobile do Power BI |
| `/data-workflow:dashboard` | Transforma uma necessidade de negócio numa especificação de dashboard pronta para construir (perguntas, dicionário de KPIs com DAX, wireframes, interação, paleta, dados necessários), ou critica um relatório existente |
| Agente `data-model-reviewer` | Revisão só de leitura de granularidade, chaves, SCD, cargas, backfills, testes e dados pessoais num modelo ou pipeline |

Cada fato numa skill vem da documentação do fornecedor e traz o link, para ser conferido na versão que você usa.

## Evals

A pasta `evals/` tem um caso por área, rodados com `claude plugin eval .` na raiz do repositório. Cada caso dá ao agente
uma tarefa realista com problemas plantados, e um juiz (LLM) confere se a resposta encontra e corrige cada um; quando o
caso nomeia uma skill, um segundo grader confere se ela foi carregada.

Resultados da v0.1.0 (uma rodada por caso, Claude Code 2.1.311):

| Caso | Com o plugin | Sem |
| --- | --- | --- |
| `sql-fanout` | 1,00 | 0,00 |
| `pandas-merge`, `pandas-3-upgrade`, `pandas-method` | 1,00 | 1,00 |
| `etl-idempotency`, `quality-tests` | 1,00 | 1,00 |
| `spark-job` | 1,00 | 1,00 |
| `dax-measures`, `dashboard-spec` | 1,00 | 1,00 |

O caso de SQL é o que o modelo base erra sozinho. Nos outros, o modelo atual já responde bem; os casos ficam como teste
de regressão, e as skills garantem que o método (granularidade primeiro, reconciliação, verificação) não dependa da
rodada.

## Licença

[MIT](LICENSE)
