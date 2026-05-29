# LAB 03 — SQL com Northwind (banco real)

**Banco de dados:** `northwind`
**Pré-requisito:** LAB02 concluído, banco northwind criado.

O Northwind é um banco clássico de treinamento — representa uma empresa real de importação/exportação de alimentos. É usado em entrevistas e avaliações técnicas da área de dados.

---

## Mapa do banco Northwind

```
customers          orders              order_details
---------          ------              -------------
customer_id (PK)   order_id (PK)       order_id (FK)
company_name       customer_id (FK)    product_id (FK)
contact_name       employee_id (FK)    unit_price
city               order_date          quantity
country            shipped_date        discount
                   freight

employees          products            categories
---------          --------            ----------
employee_id (PK)   product_id (PK)     category_id (PK)
first_name         product_name        category_name
last_name          category_id (FK)
title              unit_price
                   units_in_stock
                   discontinued

suppliers          shippers
---------          --------
supplier_id (PK)   shipper_id (PK)
company_name       company_name
country
```

---

## Aquecimento — Explore o banco

Execute cada query e observe os dados:

```sql
SELECT * FROM customers LIMIT 10;
SELECT * FROM products LIMIT 10;
SELECT * FROM orders LIMIT 10;
SELECT * FROM order_details LIMIT 10;
SELECT COUNT(*) FROM orders;
SELECT COUNT(*) FROM customers;
```

---

## Desafio 1 — Produtos por categoria

Liste o nome de cada categoria e quantos produtos ela tem. Ordene da categoria com mais produtos para a com menos.

*(Dica: você vai precisar de JOIN entre `products` e `categories`, e GROUP BY)*

---

## Desafio 2 — Faturamento por pedido

Cada pedido tem vários itens em `order_details`. O faturamento de um item é:
```
(unit_price * quantity) * (1 - discount)
```

Calcule o valor total de cada pedido. Mostre `order_id` e o valor total, ordenado do maior para o menor. Traga apenas os 10 maiores pedidos.

*(Dica: GROUP BY order_id, SUM do cálculo acima)*

---

## Desafio 3 — Funcionários e suas vendas

Liste o nome completo de cada funcionário (`first_name || ' ' || last_name`), o número de pedidos que ele processou e o valor total vendido. Ordene pelo valor total descendente.

*(Dica: JOIN entre orders, order_details e employees)*

---

## Desafio 4 — Clientes que nunca compraram

Encontre clientes que não têm nenhum pedido na tabela `orders`.

*(Dica: LEFT JOIN ou NOT IN com subquery)*

---

## Desafio 5 — Top 5 produtos mais vendidos

Por quantidade vendida (soma de `quantity` em order_details). Mostre o nome do produto e a quantidade total.

---

## Desafio 6 — Análise por país

Quantos clientes existem por país? Mostre apenas países com mais de 5 clientes, ordenado do maior para o menor.

---

## Desafio 7 — Pedidos atrasados

Um pedido está "atrasado" quando `shipped_date > required_date`.

Liste os pedidos atrasados com: `order_id`, `customer_id`, `order_date`, `required_date`, `shipped_date` e quantos dias de atraso (`shipped_date - required_date`).

---

## Desafio 8 — Receita mensal

Agrupe o faturamento total por ano e mês. Use as funções `EXTRACT(YEAR FROM order_date)` e `EXTRACT(MONTH FROM order_date)`. Ordene cronologicamente.

---

## Desafio Final — Relatório de performance de funcionários

Crie um relatório único com:
- Nome completo do funcionário
- Quantidade de pedidos processados
- Valor total vendido
- Ticket médio por pedido
- Ranking de performance (use RANK())

Ordene pelo ranking.

---

---
---

## GABARITO (só olhe depois de tentar)

---

**Desafio 1:**
```sql
SELECT
    c.category_name,
    COUNT(p.product_id) AS total_produtos
FROM categories c
INNER JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_name
ORDER BY total_produtos DESC;
```

---

**Desafio 2:**
```sql
SELECT
    order_id,
    ROUND(SUM(unit_price * quantity * (1 - discount))::numeric, 2) AS valor_total
FROM order_details
GROUP BY order_id
ORDER BY valor_total DESC
LIMIT 10;
```

---

**Desafio 3:**
```sql
SELECT
    e.first_name || ' ' || e.last_name AS funcionario,
    COUNT(DISTINCT o.order_id) AS total_pedidos,
    ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric, 2) AS valor_total
FROM employees e
INNER JOIN orders o ON e.employee_id = o.employee_id
INNER JOIN order_details od ON o.order_id = od.order_id
GROUP BY e.employee_id, funcionario
ORDER BY valor_total DESC;
```

---

**Desafio 4:**
```sql
-- Usando LEFT JOIN
SELECT c.customer_id, c.company_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;

-- Usando NOT IN
SELECT customer_id, company_name
FROM customers
WHERE customer_id NOT IN (SELECT customer_id FROM orders WHERE customer_id IS NOT NULL);
```

---

**Desafio 5:**
```sql
SELECT
    p.product_name,
    SUM(od.quantity) AS total_vendido
FROM order_details od
INNER JOIN products p ON od.product_id = p.product_id
GROUP BY p.product_name
ORDER BY total_vendido DESC
LIMIT 5;
```

---

**Desafio 6:**
```sql
SELECT country, COUNT(*) AS total_clientes
FROM customers
GROUP BY country
HAVING COUNT(*) > 5
ORDER BY total_clientes DESC;
```

---

**Desafio 7:**
```sql
SELECT
    order_id,
    customer_id,
    order_date,
    required_date,
    shipped_date,
    (shipped_date - required_date) AS dias_atraso
FROM orders
WHERE shipped_date > required_date
ORDER BY dias_atraso DESC;
```

---

**Desafio 8:**
```sql
SELECT
    EXTRACT(YEAR FROM o.order_date) AS ano,
    EXTRACT(MONTH FROM o.order_date) AS mes,
    ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric, 2) AS receita
FROM orders o
INNER JOIN order_details od ON o.order_id = od.order_id
GROUP BY ano, mes
ORDER BY ano, mes;
```

---

**Desafio Final:**
```sql
SELECT
    e.first_name || ' ' || e.last_name AS funcionario,
    COUNT(DISTINCT o.order_id) AS total_pedidos,
    ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric, 2) AS valor_total,
    ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric / COUNT(DISTINCT o.order_id), 2) AS ticket_medio,
    RANK() OVER (ORDER BY SUM(od.unit_price * od.quantity * (1 - od.discount)) DESC) AS ranking
FROM employees e
INNER JOIN orders o ON e.employee_id = o.employee_id
INNER JOIN order_details od ON o.order_id = od.order_id
GROUP BY e.employee_id, funcionario
ORDER BY ranking;
```

---

**Quando terminar, marque LAB03 no PROGRESSO.md e avance para o LAB04.**
