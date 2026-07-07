{% docs classify_moon_phase %}
# Macro: `classify_moon_phase`

Traduz o valor numérico de fase da lua (`moonphase`), retornado pela API da Visual Crossing, em um rótulo textual em português. Sem isso, a coluna vem como um float de 0 a 1 — pouco útil pra quem for ler o dado num dashboard.

## Como funciona

A API entrega a fase lunar como uma fração de ciclo (0 = lua nova, 1 = ciclo completo de volta à lua nova). A macro faz um `CASE WHEN` simples pra mapear essa fração num nome:

| Faixa de `moonphase` | Fase                |
|-----------------------|----------------------|
| `= 0`                 | Lua Nova             |
| `0 – 0.25` (exclusive) | Crescente Côncava    |
| `= 0.25`              | Quarto Crescente     |
| `0.25 – 0.5`          | Gibosa Crescente     |
| `= 0.5`               | Lua Cheia            |
| `0.5 – 0.75`          | Gibosa Minguante     |
| `= 0.75`              | Quarto Minguante     |
| `0.75 – 1`            | Crescente Minguante  |
| qualquer outro valor  | Fase Desconhecida    |

## Assinatura

{% raw %}
```sql
{{ classify_moon_phase(moonphase_value) }}
```
{% endraw %}

**Parâmetro:**
- `moonphase_value` — nome da coluna (ou expressão) com o valor de fase da lua, esperado como `FLOAT`/`DOUBLE` entre 0 e 1.

**Retorno:** expressão SQL `CASE WHEN` que resolve para `VARCHAR` com o nome da fase.

## Uso no projeto

Aplicada em `int_clima_diario.sql`, sobre a coluna `fase_lua_valor` vinda de `stg_clima`:

{% raw %}
```sql
SELECT
    data,
    fase_lua_valor,
    {{ classify_moon_phase('fase_lua_valor') }} AS fase_lua_nome
FROM {{ ref('stg_clima') }}
```
{% endraw %}

## Observação

Os limites são tratados como intervalos abertos nas pontas (`>` e `<`, não `>=`/`<=`), então os valores exatos 0, 0.25, 0.5 e 0.75 caem nas próprias fases principais e não nas fases intermediárias — é assim que a Visual Crossing documenta o campo.

Fonte: [documentação da Visual Crossing sobre `moonphase`](https://www.visualcrossing.com/resources/documentation/weather-data/weather-data-documentation/#moonphase).
{% enddocs %}