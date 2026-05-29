# LAB 02 — SQL Avançado

**Banco de dados:** `jjbike` (schema `relacional`)
**Pré-requisito:** LAB01 concluído.

---

## Parte 1 — GROUP BY com HAVING

`WHERE` filtra linhas antes de agrupar. `HAVING` filtra depois de agrupar.

Execute e observe a diferença:

```sql
-- Valor total por vendedor (todos)
SELECT idvendedor, SUM(total) AS total_vendas
FROM relacional.vendas
GROUP BY idvendedor
ORDER BY total_vendas DESC;
```

```sql
-- Só os vendedores que venderam mais de R$ 50.000 no total
SELECT idvendedor, SUM(total) AS total_vendas
FROM relacional.vendas
GROUP BY idvendedor
HAVING SUM(total) > 50000
ORDER BY total_vendas DESC;
```

**Exercício:** Quantos clientes existem por estado? Mostre apenas os estados com mais de 30 clientes.

---

## Parte 2 — INNER JOIN

JOIN combina linhas de duas tabelas com base em uma coluna em comum.

O INNER JOIN só traz linhas que têm correspondência nos dois lados.

```sql
-- Vendas com nome do vendedor (em vez do ID)
SELECT
    vendedores.nome,
    vendas.data,
    vendas.total
FROM relacional.vendas
INNER JOIN relacional.vendedores
    ON vendas.idvendedor = vendedores.idvendedor;
```

**Exercício:** Traga as vendas com o nome do **cliente** (em vez do ID do cliente). Use INNER JOIN com a tabela `clientes`.

---

## Parte 3 — Múltiplos JOINs

Você pode encadear vários JOINs numa mesma query.

```sql
-- Vendas com nome do vendedor E nome do cliente
SELECT
    vendedores.nome AS vendedor,
    clientes.cliente,
    clientes.estado,
    vendas.data,
    vendas.total
FROM relacional.vendas
INNER JOIN relacional.vendedores
    ON vendas.idvendedor = vendedores.idvendedor
INNER JOIN relacional.clientes
    ON vendas.idcliente = clientes.idcliente
ORDER BY vendas.data DESC;
```

**Exercício:** Usando a query acima como base, filtre apenas as vendas de clientes do estado **SP** com valor acima de R$ 5.000.

---

## Parte 4 — LEFT JOIN

LEFT JOIN traz todos os registros da tabela da esquerda, mesmo que não tenham correspondência na direita.

```sql
-- Todos os vendedores, mesmo os que nunca fizeram uma venda
SELECT
    vendedores.nome,
    COUNT(vendas.idvenda) AS total_vendas
FROM relacional.vendedores
LEFT JOIN relacional.vendas
    ON vendedores.idvendedor = vendas.idvendedor
GROUP BY vendedores.nome
ORDER BY total_vendas DESC;
```

Observe: vendedores sem vendas aparecem com `0`. Com INNER JOIN eles desapareceriam.

---

## Parte 5 — Subqueries

Uma query dentro de outra. Útil quando você precisa de um resultado intermediário.

```sql
-- Clientes que fizeram pelo menos uma compra acima de R$ 8.000
SELECT cliente, estado
FROM relacional.clientes
WHERE idcliente IN (
    SELECT idcliente
    FROM relacional.vendas
    WHERE total > 8000
);
```

**Exercício:** Encontre os produtos que já foram vendidos (que aparecem na tabela `itensvenda`). Use uma subquery com `IN`.

---

## Parte 6 — Aliases e cálculos nas colunas

```sql
-- Calcular desconto médio por produto
SELECT
    produtos.produto,
    ROUND(AVG(iv.desconto), 2) AS desconto_medio,
    SUM(iv.quantidade) AS total_vendido
FROM relacional.itensvenda AS iv
INNER JOIN relacional.produtos
    ON iv.idproduto = produtos.idproduto
GROUP BY produtos.produto
ORDER BY total_vendido DESC;
```

---

## Parte 7 — Window Functions (funções de janela)

Window functions calculam valores considerando um conjunto de linhas relacionadas, sem colapsar o resultado em grupos.

```sql
-- Ranking de vendedores por valor total vendido
SELECT
    vendedores.nome,
    SUM(vendas.total) AS total_vendas,
    RANK() OVER (ORDER BY SUM(vendas.total) DESC) AS posicao_ranking
FROM relacional.vendas
INNER JOIN relacional.vendedores
    ON vendas.idvendedor = vendedores.idvendedor
GROUP BY vendedores.nome;
```

```sql
-- Valor acumulado de vendas por data
SELECT
    data,
    total,
    SUM(total) OVER (ORDER BY data) AS total_acumulado
FROM relacional.vendas
ORDER BY data;
```

---

## Desafio Final do LAB02

Escreva **uma única query** que responde:

> **"Qual o top 3 de vendedores por valor total vendido? Mostre o nome do vendedor, o total vendido, a quantidade de vendas e o ticket médio (total / quantidade). Ordene do maior para o menor."**

---

---
---

## GABARITO (só olhe depois de tentar)

---

**Parte 1 — estados com mais de 30 clientes:**
```sql
SELECT estado, COUNT(*) AS total_clientes
FROM relacional.clientes
GROUP BY estado
HAVING COUNT(*) > 30
ORDER BY total_clientes DESC;
```

---

**Parte 2 — JOIN com clientes:**
```sql
SELECT
    clientes.cliente,
    vendas.data,
    vendas.total
FROM relacional.vendas
INNER JOIN relacional.clientes
    ON vendas.idcliente = clientes.idcliente;
```

---

**Parte 3 — SP com total acima de 5000:**
```sql
SELECT
    vendedores.nome AS vendedor,
    clientes.cliente,
    clientes.estado,
    vendas.data,
    vendas.total
FROM relacional.vendas
INNER JOIN relacional.vendedores
    ON vendas.idvendedor = vendedores.idvendedor
INNER JOIN relacional.clientes
    ON vendas.idcliente = clientes.idcliente
WHERE clientes.estado = 'SP'
  AND vendas.total > 5000
ORDER BY vendas.data DESC;
```

---

**Parte 5 — produtos vendidos:**
```sql
SELECT produto
FROM relacional.produtos
WHERE idproduto IN (
    SELECT idproduto
    FROM relacional.itensvenda
);
```

---

**Desafio Final:**
```sql
SELECT
    vendedores.nome,
    SUM(vendas.total) AS total_vendas,
    COUNT(vendas.idvenda) AS qtd_vendas,
    ROUND(SUM(vendas.total) / COUNT(vendas.idvenda), 2) AS ticket_medio
FROM relacional.vendas
INNER JOIN relacional.vendedores
    ON vendas.idvendedor = vendedores.idvendedor
GROUP BY vendedores.nome
ORDER BY total_vendas DESC
LIMIT 3;
```

---

**Quando terminar, marque LAB02 no PROGRESSO.md e avance para o LAB03.**
