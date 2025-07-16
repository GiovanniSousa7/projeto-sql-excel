<h1 align="center">📊 Projeto: Análise de Dados com SQL + Excel</h1>

<p align="center">
  <b>Consultas SQL com PostgreSQL + Dashboards no Excel</b><br>
  Projeto desenvolvido como parte do curso <i>SQL para Análise de Dados</i> da Midori Toyota (Udemy).
</p>

---

## 🚀 Objetivo

> Realizar extração de dados com SQL em banco PostgreSQL, analisar resultados e construir **dashboards interativos no Excel** a partir de dados reais.

---

## 🧰 Tecnologias e Ferramentas Utilizadas

- 🐘 **PostgreSQL** – Banco de dados relacional
- 🧠 **SQL** – Consultas para análise e extração de dados
- 📈 **Microsoft Excel** – Organização de dados e criação de dashboards
- 📄 **Power Query & Tabelas Dinâmicas**

---

## 📁 Estrutura do Projeto

| Arquivo | Descrição |
|--------|-----------|
| `Projeto SQL - Dashboard de Vendas.xlsx` | Excel com queries, resultados e dashboards |
| `projeto.sql` | Script SQL contendo a criação das tabelas e inserção de dados |
| `README.md` | Documentação do projeto |

---

## 🧪 Consultas SQL utilizadas

Abaixo, alguns exemplos reais de queries executadas no projeto:

---

### 📅 Leads, Vendas, Receita, Conversão e Ticket Médio por Mês

```sql
WITH leads AS (
  SELECT 
    date_trunc('month', visit_page_date)::date AS visit_page_month,
    COUNT(*) AS visit_page_count
  FROM sales.funnel
  GROUP BY visit_page_month
  ORDER BY visit_page_month
),
payments AS (
  SELECT 
    date_trunc('month', fun.paid_date)::date AS paid_month,
    COUNT(fun.paid_date) AS paid_count,
    SUM(pro.price * (1 + fun.discount)) AS receita
  FROM sales.funnel AS fun
  LEFT JOIN sales.products AS pro
    ON fun.product_id = pro.product_id
  WHERE paid_date IS NOT NULL
  GROUP BY paid_month
  ORDER BY paid_month
)
SELECT 
  leads.visit_page_month AS "mês",
  leads.visit_page_count AS "leads (#)",
  payments.paid_count AS "vendas (#)",
  (payments.receita / 1000) AS "receita (k, R$)",
  (payments.paid_count::float / leads.visit_page_count::float) AS "conversão (%)",
  (payments.receita / payments.paid_count / 1000) AS "ticket médio (k, R$)"
FROM leads
LEFT JOIN payments ON leads.visit_page_month = payments.paid_month;

### 🌎 Top 5 Estados com Mais Vendas no Brasil

```sql

SELECT 
  'Brazil' AS país,
  cus.state AS estado,
  COUNT(fun.paid_date) AS "vendas (#)"
FROM sales.funnel AS fun
LEFT JOIN sales.customers AS cus
  ON fun.customer_id = cus.customer_id
WHERE paid_date BETWEEN '2021-08-01' AND '2021-08-31'
GROUP BY país, estado
ORDER BY "vendas (#)" DESC
LIMIT 5;


### 📆 Visitas por Dia da Semana

```sql

SELECT
  EXTRACT('dow' FROM visit_page_date) AS dia_semana,
  CASE
    WHEN EXTRACT('dow' FROM visit_page_date) = 0 THEN 'domingo'
    WHEN EXTRACT('dow' FROM visit_page_date) = 1 THEN 'segunda'
    WHEN EXTRACT('dow' FROM visit_page_date) = 2 THEN 'terça'
    WHEN EXTRACT('dow' FROM visit_page_date) = 3 THEN 'quarta'
    WHEN EXTRACT('dow' FROM visit_page_date) = 4 THEN 'quinta'
    WHEN EXTRACT('dow' FROM visit_page_date) = 5 THEN 'sexta'
    WHEN EXTRACT('dow' FROM visit_page_date) = 6 THEN 'sábado'
    ELSE NULL
  END AS "dia da semana",
  COUNT(*) AS "visitas (#)"
FROM sales.funnel
WHERE visit_page_date BETWEEN '2021-08-01' AND '2021-08-31'
GROUP BY dia_semana
ORDER BY dia_semana;
