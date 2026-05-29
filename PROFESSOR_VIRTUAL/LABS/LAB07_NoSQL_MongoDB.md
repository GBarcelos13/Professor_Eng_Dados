# LAB 07 — NoSQL com MongoDB

**Ferramenta:** MongoDB (local ou Atlas) + MongoDB Shell ou Compass
**Pré-requisito:** LAB06 concluído. MongoDB instalado ou conta gratuita no Atlas.

---

## Setup rápido

**Opção 1 — MongoDB local:**
1. Baixe em: https://www.mongodb.com/try/download/community
2. Instale com as opções padrão
3. Instale também o MongoDB Compass (interface gráfica) — vem na mesma página

**Opção 2 — MongoDB Atlas (gratuito, sem instalar nada):**
1. Acesse: https://www.mongodb.com/cloud/atlas
2. Crie conta gratuita
3. Crie um cluster Free (M0)
4. Em "Connect" escolha "Connect with MongoDB Shell"

---

## Por que NoSQL?

MongoDB armazena **documentos JSON** em vez de linhas de tabelas. Ideal quando:
- Os dados não têm estrutura fixa
- Você precisa aninhar objetos (ex: um pedido com seus itens dentro)
- A estrutura muda com frequência

---

## Parte 1 — Primeiros comandos no shell

Abra o MongoDB Shell (mongosh) e execute:

```javascript
// Ver bancos existentes
show dbs

// Criar/usar um banco
use engdados

// Ver coleções (equivalente a tabelas)
show collections
```

---

## Parte 2 — Inserindo documentos

```javascript
// Inserir um documento
db.clientes.insertOne({
  nome: "Ana Silva",
  email: "ana@email.com",
  estado: "SP",
  status: "Gold",
  compras: 5,
  valor_total: 12500.00
})

// Inserir vários de uma vez
db.clientes.insertMany([
  { nome: "Carlos Souza", email: "carlos@email.com", estado: "RJ", status: "Silver", compras: 2, valor_total: 3400.00 },
  { nome: "Beatriz Lima", email: "bia@email.com", estado: "MG", status: "Platinum", compras: 12, valor_total: 45000.00 },
  { nome: "Daniel Rocha", email: "daniel@email.com", estado: "SP", status: "Bronze", compras: 1, valor_total: 890.00 },
  { nome: "Fernanda Costa", email: "fe@email.com", estado: "RS", status: "Gold", compras: 7, valor_total: 18700.00 }
])
```

**Exercício:** Insira mais 3 clientes com os dados que você quiser.

---

## Parte 3 — Consultando documentos

```javascript
// Todos os clientes
db.clientes.find()

// Formato mais legível
db.clientes.find().pretty()

// Filtrar por campo
db.clientes.find({ estado: "SP" })

// Filtrar com operador (maior que)
db.clientes.find({ valor_total: { $gt: 10000 } })

// Filtrar com múltiplas condições (AND implícito)
db.clientes.find({ estado: "SP", status: "Gold" })

// Projeção: mostrar apenas alguns campos (1=incluir, 0=excluir)
db.clientes.find({}, { nome: 1, status: 1, _id: 0 })
```

**Exercício:** 
1. Encontre clientes com mais de 5 compras
2. Encontre clientes do estado SP com status Silver ou Gold (use `$in`)
3. Mostre apenas nome e valor_total de todos os clientes

*(Dica para o $in: `{ status: { $in: ["Silver", "Gold"] } }`)*

---

## Parte 4 — Atualizando documentos

```javascript
// Atualizar um campo de um documento
db.clientes.updateOne(
  { nome: "Ana Silva" },           // filtro
  { $set: { status: "Platinum" } } // o que muda
)

// Atualizar campo numérico (incrementar)
db.clientes.updateOne(
  { nome: "Ana Silva" },
  { $inc: { compras: 1, valor_total: 2500 } }
)

// Atualizar múltiplos documentos
db.clientes.updateMany(
  { estado: "SP" },
  { $set: { regiao: "Sudeste" } }
)
```

**Exercício:** Adicione um campo `ultima_compra` com a data de hoje em todos os clientes de SP. Use `new Date()` como valor.

---

## Parte 5 — Deletando documentos

```javascript
// Deletar um
db.clientes.deleteOne({ nome: "Daniel Rocha" })

// Deletar vários
db.clientes.deleteMany({ status: "Bronze" })
```

---

## Parte 6 — Documentos aninhados (o poder do MongoDB)

Aqui fica claro por que MongoDB é diferente de SQL. Um pedido pode conter seus itens dentro do próprio documento:

```javascript
db.pedidos.insertMany([
  {
    order_id: 1001,
    cliente: "Ana Silva",
    data: new Date("2024-03-15"),
    status: "entregue",
    itens: [
      { produto: "Notebook", quantidade: 1, preco: 4500.00 },
      { produto: "Mouse", quantidade: 2, preco: 89.90 }
    ],
    total: 4679.80
  },
  {
    order_id: 1002,
    cliente: "Beatriz Lima",
    data: new Date("2024-03-16"),
    status: "em_transito",
    itens: [
      { produto: "Teclado", quantidade: 1, preco: 350.00 },
      { produto: "Monitor", quantidade: 1, preco: 1200.00 }
    ],
    total: 1550.00
  }
])

// Buscar pedidos que contêm um produto específico
db.pedidos.find({ "itens.produto": "Monitor" })
```

---

## Parte 7 — Aggregation Pipeline

É o equivalente do GROUP BY do SQL. Você encadeia estágios de transformação.

```javascript
// Total de compras e valor médio por estado
db.clientes.aggregate([
  {
    $group: {
      _id: "$estado",
      total_clientes: { $sum: 1 },
      valor_medio: { $avg: "$valor_total" },
      valor_total: { $sum: "$valor_total" }
    }
  },
  {
    $sort: { valor_total: -1 }
  }
])
```

```javascript
// Clientes com mais de R$ 10.000 agrupados por status
db.clientes.aggregate([
  { $match: { valor_total: { $gt: 10000 } } },
  { $group: { _id: "$status", contagem: { $sum: 1 } } },
  { $sort: { contagem: -1 } }
])
```

**Exercício:** Use aggregate para descobrir:
1. Quantos pedidos existem por status (na coleção `pedidos`)
2. Qual o valor total de pedidos por status

---

## Parte 8 — MongoDB com Python (PyMongo)

```python
# No Jupyter Notebook
# pip install pymongo

from pymongo import MongoClient
import pandas as pd

# Conectar
client = MongoClient("mongodb://localhost:27017/")
db = client["engdados"]
colecao = db["clientes"]

# Inserir
colecao.insert_one({"nome": "Teste Python", "estado": "SC", "valor_total": 999.00})

# Buscar como lista de dicionários
resultados = list(colecao.find({ "estado": "SP" }, { "_id": 0 }))
print(resultados)

# Transformar em DataFrame
df = pd.DataFrame(resultados)
print(df)
```

**Exercício:** Use PyMongo para:
1. Inserir 5 clientes novos via Python
2. Buscar todos os clientes com valor_total > 5000
3. Converter o resultado em um DataFrame Pandas

---

## Desafio Final do LAB07

Use PyMongo para importar os dados do arquivo `posts.json` do módulo 9 do curso (`9. Orientado a Documento - Mongodb e EC2\Projeto Prático\posts.json`) para uma coleção chamada `posts` no MongoDB. Depois:
1. Conte quantos posts existem
2. Mostre os campos disponíveis (keys) do primeiro documento
3. Filtre e exiba posts que têm mais de 100 "likes" (se o campo existir) ou qualquer filtro relevante que você descobrir nos dados

---

---
---

## GABARITO (só olhe depois de tentar)

---

**Parte 3 — Exercícios:**
```javascript
// Mais de 5 compras
db.clientes.find({ compras: { $gt: 5 } })

// SP com Silver ou Gold
db.clientes.find({ estado: "SP", status: { $in: ["Silver", "Gold"] } })

// Apenas nome e valor_total
db.clientes.find({}, { nome: 1, valor_total: 1, _id: 0 })
```

---

**Parte 4 — Exercício (ultima_compra):**
```javascript
db.clientes.updateMany(
  { estado: "SP" },
  { $set: { ultima_compra: new Date() } }
)
```

---

**Parte 7 — Exercício aggregate pedidos:**
```javascript
db.pedidos.aggregate([
  { $group: { _id: "$status", total_pedidos: { $sum: 1 }, valor_total: { $sum: "$total" } } },
  { $sort: { valor_total: -1 } }
])
```

---

**Desafio Final (PyMongo + posts.json):**
```python
import json
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017/")
db = client["engdados"]
colecao = db["posts"]

caminho = r"C:\Users\Gabriel Barcelos\Desktop\Curso Eng. Dados\9. Orientado a Documento - Mongodb e EC2\Projeto Prático\posts.json"

with open(caminho, encoding="utf-8") as f:
    dados = json.load(f)

if isinstance(dados, list):
    colecao.insert_many(dados)
else:
    colecao.insert_one(dados)

print(f"Total de posts: {colecao.count_documents({})}")
primeiro = colecao.find_one()
print(f"Campos disponíveis: {list(primeiro.keys())}")
```

---

**Quando terminar, marque LAB07 no PROGRESSO.md e avance para o LAB08.**
