# LAB 05 — Python + Banco de Dados (PostgreSQL)

**Ferramenta:** Jupyter Notebook
**Banco de dados:** PostgreSQL (`jjbike`)
**Pré-requisito:** LAB04 concluído, psycopg2 instalado.

Este lab conecta Python com o banco de dados. É o que você vai fazer toda vez que precisar automatizar consultas, gerar relatórios ou alimentar sistemas.

---

## Parte 1 — Primeira conexão

Crie um novo notebook `lab05_python_banco.ipynb` e execute:

```python
import psycopg2

# Parâmetros de conexão — ajuste a senha para a sua
conn = psycopg2.connect(
    host="localhost",
    database="jjbike",
    user="postgres",
    password="SUA_SENHA_AQUI"
)

print("Conexão estabelecida com sucesso!")
conn.close()
```

Se aparecer "Conexão estabelecida", funcionou.

**Erros comuns:**
- `password authentication failed` → senha errada
- `could not connect to server` → PostgreSQL não está rodando

---

## Parte 2 — Executando uma query e lendo o resultado

```python
import psycopg2

conn = psycopg2.connect(
    host="localhost", database="jjbike",
    user="postgres", password="SUA_SENHA_AQUI"
)

cursor = conn.cursor()

# Executar a query
cursor.execute("SELECT * FROM relacional.clientes LIMIT 5;")

# Buscar todos os resultados
resultados = cursor.fetchall()

for linha in resultados:
    print(linha)

cursor.close()
conn.close()
```

Observe: cada linha é uma tupla. O `fetchall()` retorna uma lista de tuplas.

---

## Parte 3 — Nomes das colunas

```python
conn = psycopg2.connect(
    host="localhost", database="jjbike",
    user="postgres", password="SUA_SENHA_AQUI"
)
cursor = conn.cursor()

cursor.execute("SELECT * FROM relacional.clientes LIMIT 5;")

# Pegar os nomes das colunas
colunas = [desc[0] for desc in cursor.description]
print("Colunas:", colunas)

resultados = cursor.fetchall()
for linha in resultados:
    # Criar dicionário combinando coluna + valor
    registro = dict(zip(colunas, linha))
    print(registro)

cursor.close()
conn.close()
```

**Exercício:** Adapte o código acima para consultar a tabela `relacional.vendas` e imprima cada venda no formato: `"Venda ID X: R$ Y em DD/MM/AAAA"`.

---

## Parte 4 — Pandas + banco de dados (o jeito profissional)

Na prática, você vai usar Pandas para transformar resultados em DataFrames.

```python
import psycopg2
import pandas as pd

conn = psycopg2.connect(
    host="localhost", database="jjbike",
    user="postgres", password="SUA_SENHA_AQUI"
)

# pd.read_sql executa a query e já retorna um DataFrame
query = """
    SELECT
        v.nome AS vendedor,
        SUM(vn.total) AS total_vendas,
        COUNT(vn.idvenda) AS qtd_vendas
    FROM relacional.vendas vn
    INNER JOIN relacional.vendedores v ON vn.idvendedor = v.idvendedor
    GROUP BY v.nome
    ORDER BY total_vendas DESC
"""

df = pd.read_sql(query, conn)
print(df)

conn.close()
```

**Exercício:** Usando `pd.read_sql`, traga os 10 produtos mais vendidos (por quantidade) com o nome do produto e a quantidade total. Salve o resultado em um DataFrame e exiba.

---

## Parte 5 — Inserindo dados com Python

```python
conn = psycopg2.connect(
    host="localhost", database="jjbike",
    user="postgres", password="SUA_SENHA_AQUI"
)
conn.autocommit = False  # Controle manual de transação
cursor = conn.cursor()

try:
    cursor.execute("""
        INSERT INTO relacional.clientes (cliente, estado, sexo, status)
        VALUES (%s, %s, %s, %s)
    """, ("Gabriel Barcelos", "SP", "M", "Gold"))

    conn.commit()
    print("Cliente inserido com sucesso!")

except Exception as e:
    conn.rollback()
    print(f"Erro: {e}")

finally:
    cursor.close()
    conn.close()
```

**Importante:** Sempre use `%s` como placeholder — NUNCA construa a query com f-string usando dados externos. Isso previne SQL Injection.

**Exercício:** Insira 3 clientes novos usando um loop. Após inserir, faça uma query para confirmar que os 3 aparecem no banco.

---

## Parte 6 — Executando múltiplas queries (batch insert)

```python
import psycopg2

conn = psycopg2.connect(
    host="localhost", database="jjbike",
    user="postgres", password="SUA_SENHA_AQUI"
)
cursor = conn.cursor()

novos_clientes = [
    ("Maria Souza", "RJ", "F", "Silver"),
    ("João Lima", "MG", "M", "Bronze"),
    ("Carla Ferreira", "RS", "F", "Platinum"),
]

cursor.executemany("""
    INSERT INTO relacional.clientes (cliente, estado, sexo, status)
    VALUES (%s, %s, %s, %s)
""", novos_clientes)

conn.commit()
print(f"{cursor.rowcount} clientes inseridos!")

cursor.close()
conn.close()
```

---

## Parte 7 — Função reutilizável de conexão

Na prática, você não vai repetir o bloco de conexão em todo lugar. Crie uma função:

```python
import psycopg2
import pandas as pd
from contextlib import contextmanager

@contextmanager
def get_connection():
    conn = psycopg2.connect(
        host="localhost",
        database="jjbike",
        user="postgres",
        password="SUA_SENHA_AQUI"
    )
    try:
        yield conn
        conn.commit()
    except Exception as e:
        conn.rollback()
        raise e
    finally:
        conn.close()

def query_para_dataframe(sql):
    with get_connection() as conn:
        return pd.read_sql(sql, conn)

def executar_comando(sql, params=None):
    with get_connection() as conn:
        cursor = conn.cursor()
        cursor.execute(sql, params)
        return cursor.rowcount
```

**Exercício:** Use as funções acima para:
1. Criar um relatório de vendas por vendedor (usando `query_para_dataframe`)
2. Inserir um novo vendedor (usando `executar_comando`)
3. Confirmar que o vendedor foi inserido com uma nova query

---

## Desafio Final do LAB05

Crie um script Python que:
1. Conecta ao banco `jjbike`
2. Busca o total de vendas por estado do cliente
3. Salva o resultado em um arquivo CSV chamado `relatorio_vendas_estado.csv`

*(Dica: use `df.to_csv('nome.csv', index=False)`)*

---

---
---

## GABARITO (só olhe depois de tentar)

---

**Parte 3 — Exercício (vendas formatadas):**
```python
cursor.execute("SELECT idvenda, total, data FROM relacional.vendas;")
colunas = [desc[0] for desc in cursor.description]
for linha in cursor.fetchall():
    reg = dict(zip(colunas, linha))
    print(f"Venda ID {reg['idvenda']}: R$ {reg['total']:.2f} em {reg['data'].strftime('%d/%m/%Y')}")
```

---

**Parte 4 — Exercício (10 produtos mais vendidos):**
```python
query = """
    SELECT
        p.produto,
        SUM(iv.quantidade) AS total_vendido
    FROM relacional.itensvenda iv
    INNER JOIN relacional.produtos p ON iv.idproduto = p.idproduto
    GROUP BY p.produto
    ORDER BY total_vendido DESC
    LIMIT 10
"""
df = pd.read_sql(query, conn)
print(df)
```

---

**Desafio Final:**
```python
def gerar_relatorio_por_estado():
    query = """
        SELECT
            c.estado,
            COUNT(v.idvenda) AS total_pedidos,
            SUM(v.total) AS valor_total
        FROM relacional.vendas v
        INNER JOIN relacional.clientes c ON v.idcliente = c.idcliente
        GROUP BY c.estado
        ORDER BY valor_total DESC
    """
    with get_connection() as conn:
        df = pd.read_sql(query, conn)

    df.to_csv("relatorio_vendas_estado.csv", index=False)
    print("Relatório salvo!")
    print(df)

gerar_relatorio_por_estado()
```

---

**Quando terminar, marque LAB05 no PROGRESSO.md e avance para o LAB06.**
