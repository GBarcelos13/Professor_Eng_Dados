# LAB 04 — Python: Fundamentos para Engenharia de Dados

**Ferramenta:** Jupyter Notebook
**Pré-requisito:** Python/Anaconda instalado.

Abra o Jupyter Notebook e crie um arquivo novo chamado `lab04_python.ipynb`.
Execute cada célula conforme avança nos exercícios.

---

## Parte 1 — Variáveis e tipos de dados

Em Python, você não precisa declarar o tipo. O Python descobre sozinho.

Execute no Jupyter:

```python
# Texto (string)
nome = "Gabriel"
cidade = "São Paulo"

# Número inteiro
registros = 1500

# Número decimal
taxa = 3.14

# Booleano
ativo = True

print(nome, cidade, registros, taxa, ativo)
print(type(nome))      # <class 'str'>
print(type(registros)) # <class 'int'>
print(type(taxa))      # <class 'float'>
```

**Exercício:** Crie variáveis para representar um produto de e-commerce: nome do produto, preço, quantidade em estoque, disponível (True/False). Imprima todas.

---

## Parte 2 — Operações e f-strings

```python
preco = 49.90
quantidade = 3
total = preco * quantidade

# f-string: forma moderna de formatar texto
print(f"Total da compra: R$ {total:.2f}")
# O :.2f formata com 2 casas decimais
```

**Exercício:** Calcule o desconto de 15% sobre o total acima e imprima uma mensagem formatada mostrando o preço original, o desconto e o preço final.

---

## Parte 3 — Listas

Listas armazenam múltiplos valores em ordem. São o tipo mais usado em dados.

```python
estados = ["SP", "RJ", "MG", "RS", "PR"]

# Acessar por posição (começa em 0)
print(estados[0])   # SP
print(estados[-1])  # PR (último)

# Adicionar um elemento
estados.append("SC")

# Tamanho da lista
print(len(estados))

# Percorrer a lista
for estado in estados:
    print(estado)
```

**Exercício:** Crie uma lista com 5 produtos. Adicione um produto. Imprima o total de produtos. Percorra a lista imprimindo cada produto com seu número de posição.

*(Dica para o número de posição: use `enumerate`)*
```python
for i, produto in enumerate(lista):
    print(i, produto)
```

---

## Parte 4 — Dicionários

Dicionários armazenam pares chave-valor. Muito usados para representar um registro.

```python
cliente = {
    "id": 101,
    "nome": "Ana Silva",
    "estado": "SP",
    "status": "Gold"
}

# Acessar um valor
print(cliente["nome"])
print(cliente.get("status"))

# Adicionar ou atualizar um campo
cliente["email"] = "ana@email.com"
cliente["status"] = "Platinum"

print(cliente)
```

**Exercício:** Crie um dicionário representando uma venda com: id_venda, id_cliente, data, valor_total, status (pode ser "pendente", "aprovado", "cancelado"). Imprima cada campo com sua chave formatado assim: `id_venda: 1001`.

---

## Parte 5 — Lista de dicionários (a estrutura mais usada em dados)

Imagine uma tabela: cada linha é um dicionário, a tabela inteira é uma lista de dicionários.

```python
vendas = [
    {"id": 1, "vendedor": "Carlos", "valor": 8500.00, "estado": "SP"},
    {"id": 2, "vendedor": "Ana",    "valor": 4200.00, "estado": "RJ"},
    {"id": 3, "vendedor": "Carlos", "valor": 6100.00, "estado": "MG"},
    {"id": 4, "vendedor": "Beatriz","valor": 9300.00, "estado": "SP"},
    {"id": 5, "vendedor": "Ana",    "valor": 7800.00, "estado": "SP"},
]

# Percorrer e imprimir cada venda
for venda in vendas:
    print(f"Venda {venda['id']}: {venda['vendedor']} - R$ {venda['valor']:.2f}")
```

**Exercício 1:** Calcule o valor total de todas as vendas somando o campo `valor` de cada dicionário.

**Exercício 2:** Filtre e imprima apenas as vendas do estado `SP`.

**Exercício 3:** Encontre e imprima a maior venda (maior valor).

---

## Parte 6 — Funções

Funções evitam repetição de código. Em dados você vai criar muitas funções de transformação.

```python
def calcular_desconto(preco, percentual):
    desconto = preco * (percentual / 100)
    preco_final = preco - desconto
    return preco_final

resultado = calcular_desconto(100.00, 15)
print(f"Preço com desconto: R$ {resultado:.2f}")
```

**Exercício:** Crie uma função `classificar_venda(valor)` que retorna:
- `"Alta"` se valor > 8000
- `"Média"` se valor entre 4000 e 8000
- `"Baixa"` se valor < 4000

Aplique a função em cada venda da lista do Exercício 5 e imprima: `"Venda X: classificação Y"`.

---

## Parte 7 — Lendo um arquivo CSV

Em engenharia de dados você vai ler arquivos o tempo todo.

```python
import csv

# O CSV do curso está em:
# 4.Armazenamentos de dados distribuidos - S3\Projeto Prático\vendas.csv

# Ajuste o caminho abaixo para onde está no seu computador
caminho = r"C:\Users\Gabriel Barcelos\Desktop\Curso Eng. Dados\4.Armazenamentos de dados distribuidos - S3\Projeto Prático\vendas.csv"

with open(caminho, encoding='utf-8') as arquivo:
    leitor = csv.DictReader(arquivo)
    for linha in leitor:
        print(linha)
```

Execute e observe as colunas disponíveis. Depois:

**Exercício:** Leia o CSV e calcule o valor total somando a coluna de valor. Se o CSV tiver valores como string, converta com `float(linha['coluna'])`.

---

## Parte 8 — Pandas (biblioteca essencial de dados)

Pandas transforma um CSV em uma tabela que você pode manipular com uma linha de código.

```python
import pandas as pd

caminho = r"C:\Users\Gabriel Barcelos\Desktop\Curso Eng. Dados\17.Projeto Final I\scripts\customers.csv"

df = pd.read_csv(caminho)

# Visualizar as primeiras linhas
df.head()
```

```python
# Informações sobre o dataset
df.info()
df.describe()

# Filtrar
df_brasil = df[df['country'] == 'Brazil']
print(df_brasil)

# Contar por país
print(df['country'].value_counts())
```

**Exercício:** Leia o arquivo `orders.csv` do projeto final. Quantos pedidos existem? Quantos pedidos existem por `ship_country`? Quais os 5 países que mais receberam pedidos?

---

## Desafio Final do LAB04

Usando Pandas, leia o arquivo `products.csv` do projeto final e responda:
1. Quantos produtos existem?
2. Qual o produto mais caro?
3. Qual a média de preço?
4. Quantos produtos estão descontinuados (`discontinued = 1`)?

---

---
---

## GABARITO (só olhe depois de tentar)

---

**Parte 5 — Exercício 1 (total de vendas):**
```python
total = sum(venda["valor"] for venda in vendas)
print(f"Total: R$ {total:.2f}")
```

**Parte 5 — Exercício 2 (filtro por SP):**
```python
vendas_sp = [v for v in vendas if v["estado"] == "SP"]
for v in vendas_sp:
    print(v)
```

**Parte 5 — Exercício 3 (maior venda):**
```python
maior = max(vendas, key=lambda v: v["valor"])
print(f"Maior venda: {maior}")
```

---

**Parte 6 — classificar_venda:**
```python
def classificar_venda(valor):
    if valor > 8000:
        return "Alta"
    elif valor >= 4000:
        return "Média"
    else:
        return "Baixa"

for venda in vendas:
    classificacao = classificar_venda(venda["valor"])
    print(f"Venda {venda['id']}: {classificacao}")
```

---

**Desafio Final (Pandas):**
```python
import pandas as pd

df = pd.read_csv(r"C:\Users\Gabriel Barcelos\Desktop\Curso Eng. Dados\17.Projeto Final I\scripts\products.csv")

print(f"Total de produtos: {len(df)}")
print(f"Produto mais caro: {df.loc[df['unit_price'].idxmax(), 'product_name']}")
print(f"Preço médio: R$ {df['unit_price'].mean():.2f}")
print(f"Produtos descontinuados: {df[df['discontinued'] == 1].shape[0]}")
```

---

**Quando terminar, marque LAB04 no PROGRESSO.md e avance para o LAB05.**
