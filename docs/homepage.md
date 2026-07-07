{% docs __overview__ %}
# 🌤️ ClimaBR

Pipeline de engenharia e transformação de dados climáticos das **27 capitais brasileiras**, construído com **dbt Core** rodando sobre o **Amazon Athena**, a partir de dados coletados da **Visual Crossing Weather API**.

Este projeto documenta a camada de transformação (dbt) de um pipeline maior, que também envolve ingestão via AWS Lambda, armazenamento em S3, catalogação com Glue e orquestração com Step Functions + EventBridge. Aqui, o foco é o que acontece **depois** que o dado bruto já está no catálogo: como ele é limpo, enriquecido e transformado em modelos analíticos.

---

## O que esse projeto resolve

A API retorna, por cidade, um conjunto de registros vindos de fontes diferentes — previsão (`fcst`), observação (`obs`) e dados combinados (`comb`). Sem tratamento, isso é só um JSON aninhado difícil de consultar. O dbt entra aqui para:

- Achatar e tipar os dados brutos (`staging`)
- Calcular métricas derivadas — amplitude térmica, precipitação esperada, horas de sol, fase da lua — e aplicar regras de negócio (`intermediate`)
- Consolidar indicadores mensais por cidade e medir o desvio entre o que foi previsto e o que de fato ocorreu (`marts`)

---

## Arquitetura em camadas

models/
├── staging/
│   └── stg_clima.sql              # dados brutos tipados e achatados (1 linha por cidade/dia/fonte)
├── intermediate/
│   └── int_clima_diario.sql       # métricas derivadas + fase da lua classificada
└── marts/
├── fct_analise_mensal.sql     # agregados mensais por cidade (fonte 'comb')
└── fct_desvio_previsao.sql    # desvio entre previsão (fcst) e combinado (comb)

| Camada | Materialização | Papel |
|---|---|---|
| `staging` | view | Espelha a fonte, só com tipagem e nomes padronizados |
| `intermediate` | view | Enriquecimento: métricas calculadas e classificações |
| `marts` | table | Modelos finais, prontos para consumo em BI |

---

## Fonte de dados

[Visual Crossing Weather API](https://www.visualcrossing.com/weather-data-editions) — dados diários por cidade, incluindo temperatura, umidade, precipitação, vento, índice UV, fase da lua e a fonte de cada registro (`fcst` / `obs` / `comb`).

---

## Métricas principais

- Temperatura média, mínima e máxima, sensação térmica
- Amplitude térmica diária
- Precipitação registrada vs. precipitação esperada (ponderada pela probabilidade)
- Horas de sol (calculadas a partir do nascer/pôr do sol)
- Fase da lua classificada (Nova, Crescente, Cheia, Minguante)
- Condição climática predominante no mês
- Desvio entre dado previsto e dado combinado (temperatura, precipitação, sensação, umidade)

---

## Qualidade de dados

Testes automatizados em toda a camada `staging`: `not_null` nas colunas críticas, `accepted_values` para validar as fontes (`fcst`/`comb`/`obs`), e testes de intervalo via [`dbt_expectations`](https://github.com/metaplane/dbt-expectations) para garantir que `fase_lua_valor` está sempre entre 0 e 1.

---

## Stack

- **dbt Core** 1.11
- **dbt-athena** (adapter)
- **Amazon Athena** como engine de query
- **AWS Glue Data Catalog** como catálogo de metadados
- **Amazon S3** como camada de armazenamento físico

---

## Autor

**Joel de Jesus da Cruz**
Engenheiro de Dados em formação — pipelines ETL/ELT, arquitetura de dados e Analytics Engineering em ambientes cloud (AWS, dbt, Spark, Databricks).

[LinkedIn](https://www.linkedin.com/in/joel-cruz-444976247/) · [GitHub](https://github.com/JjesusCrz)
{% enddocs %}