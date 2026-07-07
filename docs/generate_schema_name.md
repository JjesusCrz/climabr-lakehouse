{% docs generate_schema_name %}
# Macro: `generate_schema_name`

Sobrescreve o macro padrão do dbt (`generate_schema_name.sql` do core) para controlar como os schemas são nomeados no Athena/Glue.

## Por que isso existe

Por padrão, o dbt concatena o schema do target com o `custom_schema_name` definido no `dbt_project.yml` — então `+schema: marts` viraria algo como `dev_marts` no banco. Isso não faz sentido pra esse projeto: aqui os schemas (`staging`, `intermediate`, `marts`) representam camadas fixas da arquitetura, e não devem variar conforme o ambiente/target. Essa macro remove a concatenação e usa o nome definido diretamente.

## Lógica

{% raw %}
```sql
{% if custom_schema_name is none %}
    {{ node.schema }}
{% else %}
    {{ custom_schema_name }}
{% endif %}
```
{% endraw %}

- **`custom_schema_name` não definido** → usa `node.schema`, ou seja, o schema padrão do profile (`target.schema` em `profiles.yml`).
- **`custom_schema_name` definido** (via `+schema:` no `dbt_project.yml`) → usa esse valor diretamente, sem concatenar com o target.

## Assinatura

{% raw %}
```sql
{{ generate_schema_name(custom_schema_name, node) }}
```
{% endraw %}

Não é chamada manualmente nos modelos — o dbt invoca esse macro automaticamente sempre que precisa resolver o schema de um model. Basta o nome do arquivo ser `generate_schema_name.sql` na pasta `macros/` que o dbt substitui a versão padrão pela sua.

## Uso no projeto

Configurado por camada no `dbt_project.yml`:

```yaml
models:
  climabr:
    staging:
      +schema: staging
    intermediate:
      +schema: intermediate
    marts:
      +schema: marts
```

Resultado: os modelos são materializados exatamente em `staging`, `intermediate` e `marts` no Glue Data Catalog — independente de qual target (`dev`, `prod`, etc.) estiver rodando.

## Cuidado

Se você tiver múltiplos targets (dev/prod) rodando no mesmo Glue Catalog, essa abordagem faz com que ambos escrevam no **mesmo schema físico** — não há isolamento por ambiente. Pra um projeto de portfólio single-target isso é irrelevante, mas é o tipo de trade-off que vale mencionar se alguém perguntar em entrevista.
{% enddocs %}