# LAB 08 — Apache Spark com PySpark

**Plataforma:** Databricks Community Edition (gratuito) ou PySpark local
**Pré-requisito:** LAB06 concluído.

Spark é a principal ferramenta de processamento de grandes volumes de dados. Você vai usá-lo quando os dados são grandes demais para Pandas ou quando precisa de processamento distribuído.

---

## Setup

**Opção recomendada — Databricks Community (gratuita, sem instalar nada):**
1. Acesse: https://community.cloud.databricks.com
2. Crie uma conta gratuita
3. Crie um Cluster (All-Purpose, runtime 13+)
4. Crie um notebook Python

**Os notebooks do curso estão em:** `12.Spark - Databricks\Projeto Prático\`
Use-os como referência, mas execute os exercícios deste lab.

---

## Por que Spark?

| Situação | Ferramenta certa |
|----------|-----------------|
| Dataset cabe na memória (< 1 GB) | Pandas |
| Dataset grande (> 1 GB) | Spark |
| Processamento em tempo real | Spark Streaming |
| SQL em data lake | Spark SQL |

---

## Parte 1 — SparkSession e primeiros DataFrames

No Databricks, o SparkSession já existe como `spark`. Em ambiente local você cria assim:

```python
# Em ambiente local
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("LabSpark") \
    .getOrCreate()

print(spark.version)
```

```python
# Criar um DataFrame simples
from pyspark.sql import Row

dados = [
    Row(nome="Ana", estado="SP", valor=8500.0),
    Row(nome="Carlos", estado="RJ", valor=4200.0),
    Row(nome="Beatriz", estado="SP", valor=12300.0),
    Row(nome="Daniel", estado="MG", valor=3100.0),
]

df = spark.createDataFrame(dados)
df.show()
df.printSchema()
```

---

## Parte 2 — Lendo arquivos CSV

No Databricks, faça upload dos CSVs do curso para o DBFS (File System do Databricks).

```python
# Ler CSV
df_orders = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("/FileStore/tables/orders.csv")

df_orders.show(5)
df_orders.printSchema()
df_orders.count()
```

---

## Parte 3 — Transformações básicas

```python
from pyspark.sql.functions import col, sum, count, avg, round

# Selecionar colunas
df_orders.select("order_id", "customer_id", "order_date").show(5)

# Filtrar
df_orders.filter(col("ship_country") == "France").show()

# Equivalente com where
df_orders.where(col("freight") > 100).show()

# Adicionar coluna calculada
from pyspark.sql.functions import lit
df_orders.withColumn("fonte", lit("csv")).show(3)
```

---

## Parte 4 — Aggregações

```python
from pyspark.sql.functions import sum, count, avg, round as spark_round

# Total de pedidos por país
df_orders.groupBy("ship_country") \
    .agg(count("order_id").alias("total_pedidos")) \
    .orderBy("total_pedidos", ascending=False) \
    .show(10)
```

```python
# Ler order_details
df_details = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("/FileStore/tables/order_details.csv")

# Calcular valor por item
from pyspark.sql.functions import col, expr

df_details = df_details.withColumn(
    "valor_liquido",
    spark_round(col("unit_price") * col("quantity") * (1 - col("discount")), 2)
)

# Agrupar por pedido
df_total_pedido = df_details.groupBy("order_id") \
    .agg(spark_round(sum("valor_liquido"), 2).alias("valor_total"))

df_total_pedido.show(5)
```

**Exercício:** Encontre os 5 países com maior valor total de pedidos. Vai precisar fazer JOIN entre `orders` e o `df_total_pedido` calculado acima.

---

## Parte 5 — SQL no Spark

Spark permite escrever SQL puro em cima de DataFrames. Em dados, isso é extremamente comum.

```python
# Registrar como view temporária (para usar SQL)
df_orders.createOrReplaceTempView("orders")
df_details.createOrReplaceTempView("order_details")
df_total_pedido.createOrReplaceTempView("total_pedido")

# Escrever SQL
resultado = spark.sql("""
    SELECT
        o.ship_country AS pais,
        COUNT(o.order_id) AS total_pedidos,
        ROUND(SUM(tp.valor_total), 2) AS receita_total
    FROM orders o
    INNER JOIN total_pedido tp ON o.order_id = tp.order_id
    GROUP BY o.ship_country
    ORDER BY receita_total DESC
    LIMIT 10
""")

resultado.show()
```

**Exercício:** Usando Spark SQL, escreva a query que encontra os 5 produtos mais vendidos por quantidade. (Vai precisar do CSV de products também.)

---

## Parte 6 — Delta Lake (formato moderno de data lake)

Delta Lake adiciona transações ACID ao data lake. É o padrão no Databricks.

```python
# Salvar como Delta Table
df_total_pedido.write \
    .format("delta") \
    .mode("overwrite") \
    .save("/delta/total_pedidos")

# Ler de volta
df_delta = spark.read.format("delta").load("/delta/total_pedidos")
df_delta.show(5)
```

```python
# Versionar e fazer time travel
df_orders.write \
    .format("delta") \
    .mode("overwrite") \
    .save("/delta/orders")

# Ver histórico de modificações
from delta.tables import DeltaTable
dt = DeltaTable.forPath(spark, "/delta/orders")
dt.history().show()
```

---

## Parte 7 — Comparação Pandas vs Spark

Execute ambas e compare o resultado:

```python
# Pandas
import pandas as pd
df_pandas = pd.read_csv("/path/orders.csv")
resultado_pandas = df_pandas.groupby("ship_country")["order_id"].count().sort_values(ascending=False).head(5)
print(resultado_pandas)
```

```python
# Spark (mesma operação)
resultado_spark = df_orders.groupBy("ship_country") \
    .count() \
    .orderBy("count", ascending=False) \
    .limit(5)
resultado_spark.show()
```

O resultado é o mesmo, mas Spark processa em paralelo em múltiplas máquinas.

---

## Desafio Final do LAB08

Usando Spark SQL, crie uma análise completa que responde:

> "Quais os 3 funcionários que mais geraram receita? Mostre nome completo, total de pedidos e receita total."

Você vai precisar dos CSVs: `orders.csv`, `order_details.csv` e `employees.csv`. Faça tudo via Spark SQL (registre cada CSV como uma view e escreva a query).

---

---
---

## GABARITO (só olhe depois de tentar)

---

**Parte 4 — Exercício (5 países com maior receita):**
```python
df_com_pais = df_total_pedido.join(
    df_orders.select("order_id", "ship_country"),
    on="order_id",
    how="inner"
)

df_com_pais.groupBy("ship_country") \
    .agg(spark_round(sum("valor_total"), 2).alias("receita_total")) \
    .orderBy("receita_total", ascending=False) \
    .show(5)
```

---

**Parte 5 — Exercício (produtos mais vendidos):**
```python
# Assumindo que df_products está registrado como view "products"
spark.sql("""
    SELECT
        p.product_name,
        SUM(od.quantity) AS total_vendido
    FROM order_details od
    INNER JOIN products p ON od.product_id = p.product_id
    GROUP BY p.product_name
    ORDER BY total_vendido DESC
    LIMIT 5
""").show()
```

---

**Desafio Final:**
```python
# Registrar employees
df_employees = spark.read.option("header","true").option("inferSchema","true").csv("/FileStore/tables/employees.csv")
df_employees.createOrReplaceTempView("employees")

spark.sql("""
    SELECT
        e.first_name || ' ' || e.last_name AS funcionario,
        COUNT(DISTINCT o.order_id) AS total_pedidos,
        ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount)), 2) AS receita_total
    FROM employees e
    INNER JOIN orders o ON e.employee_id = o.employee_id
    INNER JOIN order_details od ON o.order_id = od.order_id
    GROUP BY funcionario
    ORDER BY receita_total DESC
    LIMIT 3
""").show()
```

---

**Quando terminar, marque LAB08 no PROGRESSO.md e avance para o LAB09.**
