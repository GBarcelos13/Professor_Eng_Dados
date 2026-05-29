# LAB 01 — SQL Básico

**Banco de dados:** `jjbike` (schema `relacional`)
**Pré-requisito:** Setup concluído, banco jjbike criado com dados.

---

## O banco que você vai usar

A JJBike é uma empresa fictícia de vendas. O banco tem 5 tabelas:

```
Vendedores       Clientes
----------       --------
IDVendedor (PK)  IDCliente (PK)
Nome             Cliente (nome)
                 Estado (ex: SP, RJ, RS)
                 Sexo (M ou F)
                 Status (Bronze, Silver, Gold, Platinum)

Produtos         Vendas                    ItensVenda
--------         ------                    ----------
IDProduto (PK)   IDVenda (PK)              IDProduto (FK)
Produto          IDVendedor (FK)           IDVenda (FK)
Preco            IDCliente (FK)            Quantidade
                 Data                      ValorUnitario
                 Total                     ValorTotal
                                           Desconto
```

**Abra o pgAdmin 4, selecione o banco `jjbike` e abra o Query Tool.**

---

## Exercício 1 — Veja todos os clientes

Execute a query abaixo e observe o resultado:

```sql
SELECT * FROM relacional.clientes;
```

Quantos clientes existem? Anote o número.

Agora altere para trazer só as colunas `cliente`, `estado` e `status`:

```sql
SELECT cliente, estado, status
FROM relacional.clientes;
```

---

## Exercício 2 — Filtrando com WHERE

Traga apenas os clientes do estado de São Paulo (SP):

```sql
SELECT cliente, estado, status
FROM relacional.clientes
WHERE estado = 'SP';
```

Agora escreva você mesmo uma query para trazer clientes do **Rio Grande do Sul (RS)**. Não olhe o gabarito antes de tentar.

---

## Exercício 3 — Múltiplos filtros

Traga clientes que são do estado SP **ou** RS:

```sql
-- Usando OR
SELECT cliente, estado
FROM relacional.clientes
WHERE estado = 'SP' OR estado = 'RS';

-- Mesma coisa com IN (mais elegante quando são vários valores)
SELECT cliente, estado
FROM relacional.clientes
WHERE estado IN ('SP', 'RS');
```

**Desafio:** Traga clientes do status `Gold` **e** do sexo `F`. (Dica: use AND)

---

## Exercício 4 — Ordenando resultados

Traga todos os produtos ordenados do mais caro para o mais barato:

```sql
SELECT produto, preco
FROM relacional.produtos
ORDER BY preco DESC;
```

Agora escreva uma query que traz os produtos do mais **barato** para o mais **caro**.

---

## Exercício 5 — Limitando resultados

Traga apenas os 5 produtos mais caros:

```sql
SELECT produto, preco
FROM relacional.produtos
ORDER BY preco DESC
LIMIT 5;
```

---

## Exercício 6 — Funções de agregação

Execute cada uma e observe o que cada função faz:

```sql
-- Quantas vendas existem no total?
SELECT COUNT(*) FROM relacional.vendas;

-- Qual é o valor total de todas as vendas somadas?
SELECT SUM(total) FROM relacional.vendas;

-- Qual é o valor médio das vendas?
SELECT AVG(total) FROM relacional.vendas;

-- Qual foi a maior venda?
SELECT MAX(total) FROM relacional.vendas;

-- Qual foi a menor venda?
SELECT MIN(total) FROM relacional.vendas;
```

---

## Exercício 7 — Filtrando com valores numéricos

Traga todas as vendas com valor acima de R$ 6.000:

```sql
SELECT *
FROM relacional.vendas
WHERE total > 6000;
```

**Desafio:** Traga vendas entre R$ 5.000 e R$ 8.000. (Dica: use BETWEEN)

---

## Exercício 8 — Valores distintos

Quais são os status possíveis de clientes? Use DISTINCT para não repetir:

```sql
SELECT DISTINCT status
FROM relacional.clientes;
```

---

## Exercício 9 — Agrupando dados

Quantas vendas cada vendedor fez?

```sql
SELECT idvendedor, COUNT(*) AS total_vendas
FROM relacional.vendas
GROUP BY idvendedor;
```

Quanto cada vendedor vendeu em valor total?

```sql
SELECT idvendedor, SUM(total) AS valor_total
FROM relacional.vendas
GROUP BY idvendedor
ORDER BY valor_total DESC;
```

---

## Desafio Final do LAB01

Sem olhar o gabarito, escreva uma query que responde:

> **"Quantos clientes do sexo masculino existem por estado? Ordene do estado com mais clientes para o com menos."**

---

---
---

## GABARITO (só olhe depois de tentar)

---

**Exercício 2 — clientes do RS:**
```sql
SELECT cliente, estado, status
FROM relacional.clientes
WHERE estado = 'RS';
```

---

**Exercício 3 — clientes Gold e femininas:**
```sql
SELECT cliente, estado, status, sexo
FROM relacional.clientes
WHERE status = 'Gold' AND sexo = 'F';
```

---

**Exercício 4 — mais baratos para mais caros:**
```sql
SELECT produto, preco
FROM relacional.produtos
ORDER BY preco ASC;
-- ASC é o padrão, mas escrevê-lo deixa o código mais claro
```

---

**Exercício 7 — BETWEEN:**
```sql
SELECT *
FROM relacional.vendas
WHERE total BETWEEN 5000 AND 8000;
```

---

**Desafio Final:**
```sql
SELECT estado, COUNT(*) AS total_clientes
FROM relacional.clientes
WHERE sexo = 'M'
GROUP BY estado
ORDER BY total_clientes DESC;
```

---

**Quando terminar este lab, marque LAB01 como concluído no PROGRESSO.md e avance para o LAB02.**
