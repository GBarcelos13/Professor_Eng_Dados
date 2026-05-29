# LAB 06 — Pipeline ETL: CSV → Transformação → Banco

**Ferramenta:** Jupyter Notebook
**Pré-requisito:** LAB05 concluído.

ETL é a atividade central do engenheiro de dados:
- **E**xtract — lê dados de uma fonte (CSV, API, banco legado)
- **T**ransform — limpa, padroniza, enriquece os dados
- **L**oad — grava no destino (data warehouse, banco analítico)

Neste lab você vai construir um pipeline ETL real usando os dados do curso.

---

## Cenário

Você trabalha na Northwind. O time de vendas envia um CSV com pedidos toda semana. Sua tarefa é:
1. Ler o CSV
2. Aplicar transformações de limpeza e enriquecimento
3. Gravar no PostgreSQL para análise

---

## Parte 1 — Extract (leitura dos dados)

Crie o notebook `lab06_etl.ipynb`.

```python
import pandas as pd

# Leia os CSVs do projeto final
caminho_base = r"C:\Users\Gabriel Barcelos\Desktop\Curso Eng. Dados\17.Projeto Final I\scripts"

orders = pd.read_csv(f"{caminho_base}\\orders.csv")
order_details = pd.read_csv(f"{caminho_base}\\order_details.csv")
customers = pd.read_csv(f"{caminho_base}\\customers.csv")
products = pd.read_csv(f"{caminho_base}\\products.csv")

# Verifique o que chegou
print("=== ORDERS ===")
print(orders.shape)
orders.head()
```

```python
# Cheque valores ausentes
print(orders.isnull().sum())
```

**Exercício:** Execute o mesmo para `order_details`, `customers` e `products`. Anote quais colunas têm valores ausentes.

---

## Parte 2 — Transform (transformações)

### 2.1 — Converter tipos de dados

```python
# Datas costumam chegar como string — converta
orders["order_date"] = pd.to_datetime(orders["order_date"])
orders["shipped_date"] = pd.to_datetime(orders["shipped_date"])
orders["required_date"] = pd.to_datetime(orders["required_date"])

print(orders.dtypes)
```

### 2.2 — Criar colunas calculadas

```python
# Valor bruto de cada item (sem desconto)
order_details["valor_bruto"] = order_details["unit_price"] * order_details["quantity"]

# Valor com desconto aplicado
order_details["valor_liquido"] = order_details["valor_bruto"] * (1 - order_details["discount"])

# Arredondar para 2 casas
order_details["valor_liquido"] = order_details["valor_liquido"].round(2)

order_details.head()
```

### 2.3 — Calcular valor total por pedido

```python
total_por_pedido = order_details.groupby("order_id")["valor_liquido"].sum().reset_index()
total_por_pedido.columns = ["order_id", "valor_total"]
total_por_pedido.head()
```

### 2.4 — Enriquecer os pedidos com informações do cliente

```python
# Juntar pedidos com clientes
orders_enriched = orders.merge(
    customers[["customer_id", "company_name", "country"]],
    on="customer_id",
    how="left"
)

# Juntar com total calculado
orders_enriched = orders_enriched.merge(
    total_por_pedido,
    on="order_id",
    how="left"
)

orders_enriched.head()
```

### 2.5 — Criar flag de pedido atrasado

```python
orders_enriched["atrasado"] = (
    orders_enriched["shipped_date"] > orders_enriched["required_date"]
)

orders_enriched["dias_atraso"] = (
    orders_enriched["shipped_date"] - orders_enriched["required_date"]
).dt.days.clip(lower=0)  # clip(lower=0) garante que não aparece negativo
```

### 2.6 — Extrair ano e mês

```python
orders_enriched["ano"] = orders_enriched["order_date"].dt.year
orders_enriched["mes"] = orders_enriched["order_date"].dt.month
```

**Exercício:** Após todas as transformações, mostre um resumo:
- Total de pedidos
- Total de pedidos atrasados
- Valor total de todos os pedidos
- País com mais pedidos

---

## Parte 3 — Load (carga no banco)

Agora grave o DataFrame transformado no PostgreSQL.

### 3.1 — Criar a tabela de destino

Execute no pgAdmin no banco `northwind`:

```sql
CREATE TABLE IF NOT EXISTS fato_pedidos (
    order_id      SMALLINT,
    customer_id   VARCHAR(10),
    company_name  VARCHAR(100),
    country       VARCHAR(50),
    order_date    DATE,
    required_date DATE,
    shipped_date  DATE,
    valor_total   NUMERIC(10,2),
    atrasado      BOOLEAN,
    dias_atraso   INTEGER,
    ano           INTEGER,
    mes           INTEGER
);
```

### 3.2 — Gravar com SQLAlchemy (mais eficiente para Pandas)

```python
from sqlalchemy import create_engine

# Crie a engine de conexão
engine = create_engine("postgresql://postgres:SUA_SENHA@localhost:5432/northwind")

# Selecione as colunas que vão para o banco
colunas_destino = [
    "order_id", "customer_id", "company_name", "country",
    "order_date", "required_date", "shipped_date",
    "valor_total", "atrasado", "dias_atraso", "ano", "mes"
]

df_para_gravar = orders_enriched[colunas_destino]

# Gravar no banco (if_exists='replace' apaga e recria a tabela; 'append' adiciona)
df_para_gravar.to_sql(
    name="fato_pedidos",
    con=engine,
    if_exists="replace",
    index=False
)

print(f"{len(df_para_gravar)} registros gravados!")
```

### 3.3 — Verificar no banco

```python
import pandas as pd

df_verificacao = pd.read_sql("SELECT * FROM fato_pedidos LIMIT 5", engine)
df_verificacao
```

---

## Parte 4 — Encapsular em funções (pipeline completo)

Um pipeline profissional separa cada etapa em funções:

```python
def extract():
    caminho_base = r"C:\Users\Gabriel Barcelos\Desktop\Curso Eng. Dados\17.Projeto Final I\scripts"
    orders = pd.read_csv(f"{caminho_base}\\orders.csv")
    order_details = pd.read_csv(f"{caminho_base}\\order_details.csv")
    customers = pd.read_csv(f"{caminho_base}\\customers.csv")
    return orders, order_details, customers


def transform(orders, order_details, customers):
    # Converter datas
    for col in ["order_date", "shipped_date", "required_date"]:
        orders[col] = pd.to_datetime(orders[col])

    # Calcular valor por pedido
    order_details["valor_liquido"] = (
        order_details["unit_price"] * order_details["quantity"] * (1 - order_details["discount"])
    ).round(2)

    total_por_pedido = order_details.groupby("order_id")["valor_liquido"].sum().reset_index()
    total_por_pedido.columns = ["order_id", "valor_total"]

    # Enriquecer
    df = orders.merge(customers[["customer_id", "company_name", "country"]], on="customer_id", how="left")
    df = df.merge(total_por_pedido, on="order_id", how="left")

    # Flags
    df["atrasado"] = df["shipped_date"] > df["required_date"]
    df["dias_atraso"] = (df["shipped_date"] - df["required_date"]).dt.days.clip(lower=0)
    df["ano"] = df["order_date"].dt.year
    df["mes"] = df["order_date"].dt.month

    return df


def load(df, engine):
    colunas = ["order_id","customer_id","company_name","country","order_date",
               "required_date","shipped_date","valor_total","atrasado","dias_atraso","ano","mes"]
    df[colunas].to_sql("fato_pedidos", con=engine, if_exists="replace", index=False)
    print(f"Pipeline concluído: {len(df)} registros gravados.")


# Executar o pipeline
engine = create_engine("postgresql://postgres:SUA_SENHA@localhost:5432/northwind")
orders, order_details, customers = extract()
df_transformado = transform(orders, order_details, customers)
load(df_transformado, engine)
```

---

## Desafio Final do LAB06

Adicione uma nova etapa ao pipeline: após o `load`, crie um segundo DataFrame com o **resumo mensal** (ano, mês, total de pedidos, valor total, percentual de pedidos atrasados) e grave-o numa tabela chamada `resumo_mensal` no mesmo banco.

---

---
---

## GABARITO (só olhe depois de tentar)

---

**Parte 2 — Exercício (resumo):**
```python
print(f"Total de pedidos: {len(orders_enriched)}")
print(f"Pedidos atrasados: {orders_enriched['atrasado'].sum()}")
print(f"Valor total: R$ {orders_enriched['valor_total'].sum():,.2f}")
print(f"País com mais pedidos: {orders_enriched['country'].value_counts().index[0]}")
```

---

**Desafio Final:**
```python
resumo = df_transformado.groupby(["ano", "mes"]).agg(
    total_pedidos=("order_id", "count"),
    valor_total=("valor_total", "sum"),
    pedidos_atrasados=("atrasado", "sum")
).reset_index()

resumo["pct_atrasados"] = (resumo["pedidos_atrasados"] / resumo["total_pedidos"] * 100).round(1)

resumo.to_sql("resumo_mensal", con=engine, if_exists="replace", index=False)
print("Resumo mensal gravado!")
print(resumo)
```

---

**Quando terminar, marque LAB06 no PROGRESSO.md e avance para o LAB07.**
