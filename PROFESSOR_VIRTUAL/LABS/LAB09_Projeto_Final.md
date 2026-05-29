# LAB 09 — Projeto Final Integrador

**Todas as ferramentas aprendidas**
**Pré-requisito:** Todos os labs anteriores concluídos.

Este projeto simula um trabalho real de engenharia de dados. Não há instruções passo a passo — você decide como resolver. Use os labs anteriores como referência.

---

## Contexto

Você foi contratado como Engenheiro de Dados na Northwind Traders. No primeiro dia, o gestor de dados apresenta os problemas:

> "Temos dados espalhados em CSVs, sem nenhuma organização. Preciso de um pipeline que limpe e consolide tudo, um banco analítico estruturado e alguns relatórios automatizados. Você tem 1 semana."

---

## Entregáveis

### Entregável 1 — Pipeline ETL completo

Crie um script Python (`pipeline_northwind.py` ou notebook) que:

1. Lê todos os CSVs do projeto final (`customers`, `orders`, `order_details`, `products`, `employees`, `categories`, `suppliers`, `shippers`)
2. Aplica as seguintes transformações:
   - Converter todas as datas para `datetime`
   - Calcular `valor_liquido` em `order_details` (`unit_price * quantity * (1 - discount)`)
   - Adicionar coluna `valor_total` em `orders` (soma dos itens)
   - Adicionar coluna `atrasado` em `orders` (boolean)
   - Adicionar colunas `ano` e `mes` baseadas em `order_date`
3. Grava as tabelas transformadas no PostgreSQL (banco `northwind`) nas tabelas:
   - `fato_pedidos` (pedidos enriquecidos)
   - `dim_clientes` (dimensão de clientes)
   - `dim_produtos` (dimensão de produtos)
   - `dim_funcionarios` (dimensão de funcionários)

---

### Entregável 2 — Camada analítica (SQL)

Com o banco carregado, crie as seguintes views ou queries SQL (salve num arquivo `.sql`):

**View 1 — Receita mensal:**
```
ano | mes | receita_total | total_pedidos | pedidos_atrasados | pct_atrasados
```

**View 2 — Performance de funcionários:**
```
funcionario | total_pedidos | receita_total | ticket_medio | ranking
```

**View 3 — Top produtos:**
```
produto | categoria | total_vendido_qtd | receita_total | preco_medio
```

**View 4 — Análise por país:**
```
pais | total_clientes | total_pedidos | receita_total | receita_por_cliente
```

---

### Entregável 3 — Relatório automático em Python

Crie um script que gera um arquivo `relatorio_northwind.csv` com uma aba por análise, usando Pandas. O relatório deve conter as 4 análises do Entregável 2.

*(Dica: use `pd.ExcelWriter` para múltiplas abas se preferir Excel)*

```python
with pd.ExcelWriter("relatorio_northwind.xlsx") as writer:
    df_receita_mensal.to_excel(writer, sheet_name="Receita Mensal", index=False)
    df_funcionarios.to_excel(writer, sheet_name="Funcionários", index=False)
    df_produtos.to_excel(writer, sheet_name="Produtos", index=False)
    df_paises.to_excel(writer, sheet_name="Países", index=False)
```

---

### Entregável 4 — MongoDB (opcional, bônus)

Crie uma coleção `pedidos_mongo` no MongoDB onde cada documento representa um pedido completo com seus itens embutidos:

```json
{
  "order_id": 10248,
  "cliente": {
    "id": "VINET",
    "nome": "Vins et alcools Chevalier",
    "pais": "France"
  },
  "funcionario": "Steven Buchanan",
  "data": "1996-07-04",
  "valor_total": 440.00,
  "atrasado": false,
  "itens": [
    { "produto": "Queso Cabrales", "quantidade": 12, "valor": 168.00 },
    { "produto": "Singaporean Hokkien Fried Mee", "quantidade": 10, "valor": 98.00 }
  ]
}
```

Use Python (PyMongo + Pandas) para construir essa estrutura e inserir todos os pedidos.

---

## Critérios de avaliação (autoavaliação)

Ao concluir, responda honestamente:

| Critério | Sim | Parcial | Não |
|----------|-----|---------|-----|
| O pipeline roda sem erros | | | |
| Todas as tabelas foram criadas no banco | | | |
| As queries SQL retornam os resultados esperados | | | |
| O relatório Excel é gerado automaticamente | | | |
| O código está organizado em funções | | | |
| Eu consigo explicar cada etapa do que fiz | | | |

---

## Dica de estrutura do projeto

Organize seu projeto assim:

```
projeto_northwind/
├── config.py            # credenciais e caminhos (nunca commite com senha real)
├── extract.py           # funções de leitura dos CSVs
├── transform.py         # funções de transformação
├── load.py              # funções de carga no banco
├── pipeline.py          # orquestra extract → transform → load
├── relatorio.py         # gera os relatórios
├── queries/
│   ├── view_receita_mensal.sql
│   ├── view_funcionarios.sql
│   ├── view_produtos.sql
│   └── view_paises.sql
└── output/
    └── relatorio_northwind.xlsx
```

---

## Referências dos labs

- Leitura de CSV com Pandas → LAB04, Parte 7-8
- Transformações com Pandas → LAB06, Parte 2
- Conexão com PostgreSQL → LAB05
- Carga com SQLAlchemy → LAB06, Parte 3
- Queries SQL analíticas → LAB03
- MongoDB com Python → LAB07, Parte 8

---

## Quando terminar

1. Marque LAB09 no PROGRESSO.md
2. Me mostre o que você fez: cole qualquer parte do seu código ou o erro que travar
3. Farei uma revisão do seu pipeline e darei feedback

**Parabéns por chegar até aqui. Quem conclui este projeto está pronto para candidaturas a vagas júnior de engenharia de dados.**
