# Setup — Ferramentas que você vai precisar

Configure tudo antes de começar os labs. Estimativa: 1-2 horas.

---

## 1. PostgreSQL (banco de dados relacional)

É o banco que você vai usar nos primeiros labs.

**Instalar:**
1. Acesse: https://www.postgresql.org/download/windows/
2. Baixe o instalador do PostgreSQL 16
3. Execute e siga o instalador:
   - Senha do superusuário: anote bem, você vai precisar sempre
   - Porta padrão: 5432 (não mude)
   - Marque "pgAdmin 4" na lista de componentes

**Verificar se funcionou:**
Abra o pgAdmin 4 (instalou junto), conecte com a senha que você definiu. Se aparecer a tela principal, funcionou.

---

## 2. Criar o banco JJBike (usado nos LAB01 e LAB02)

1. No pgAdmin 4, clique com botão direito em "Databases" → Create → Database
2. Nome: `jjbike` → Save
3. Clique com botão direito em `jjbike` → Query Tool
4. Abra o arquivo: `5.Modelo Relacional e SQL - Postgres e EC2\Projeto Prático\1.CreateTable.sql`
5. Cole o conteúdo no Query Tool e execute (F5)
6. Repita para os arquivos 2, 3, 4, 5 e 6 (nessa ordem — são os inserts de dados)

**Verificar:** No painel esquerdo, expanda jjbike → Schemas → relacional → Tables. Deve aparecer: clientes, produtos, vendas, vendedores, itensvenda.

---

## 3. Criar o banco Northwind (usado no LAB03)

1. Crie um novo database chamado `northwind`
2. Abra o Query Tool do northwind
3. Execute o arquivo: `17.Projeto Final I\scripts\northwindddl.sql`
4. Depois execute cada arquivo CSV usando o script `copy.sql` (adapte os caminhos)

**Atalho:** Você pode importar os CSVs pelo pgAdmin: clique com botão direito na tabela → Import/Export Data.

---

## 4. Python (Anaconda — recomendado para iniciantes)

Anaconda instala Python + Jupyter + todas as bibliotecas de dados de uma vez.

**Instalar:**
1. Acesse: https://www.anaconda.com/download
2. Baixe a versão para Windows (64-bit)
3. Execute o instalador — opções padrão estão corretas
4. Ao final, abra o "Anaconda Navigator"

**Verificar:** Abra o "Anaconda Prompt" e digite:
```
python --version
```
Deve aparecer Python 3.x.x.

---

## 5. Jupyter Notebook (vem com Anaconda)

É onde você vai escrever e executar código Python.

**Abrir:**
1. Anaconda Navigator → Jupyter Notebook → Launch
2. Vai abrir no navegador
3. Navegue até a pasta do curso e crie um novo notebook (.ipynb)

**Alternativa mais moderna:** VS Code com extensão Python + Jupyter. Se preferir, instale o VS Code (https://code.visualstudio.com/) e adicione as extensões "Python" e "Jupyter".

---

## 6. Bibliotecas Python que você vai precisar

Abra o Anaconda Prompt e execute:

```
pip install psycopg2-binary
pip install boto3
pip install pandas
pip install sqlalchemy
```

**Verificar:** No Jupyter, execute:
```python
import psycopg2
import boto3
import pandas as pd
print("Tudo instalado!")
```

---

## 7. DBeaver (opcional — editor SQL mais confortável que pgAdmin)

Se o pgAdmin parecer pesado, o DBeaver é uma alternativa mais leve e moderna.

**Instalar:** https://dbeaver.io/download/ — versão Community (gratuita)

---

## Checklist de setup

- [ ] PostgreSQL instalado e rodando
- [ ] pgAdmin 4 abrindo com sucesso
- [ ] Banco `jjbike` criado com as 5 tabelas
- [ ] Banco `northwind` criado com as 8 tabelas
- [ ] Python (Anaconda) instalado
- [ ] Jupyter Notebook abrindo no navegador
- [ ] Bibliotecas psycopg2, boto3, pandas instaladas

**Só avance para o LAB01 depois de marcar todos os itens acima.**
