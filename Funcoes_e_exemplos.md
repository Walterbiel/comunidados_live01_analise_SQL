# Guia Rápido de Funções SQL – Aula de Análises (AdventureWorksDW2022)

Este documento apresenta as principais funções e conceitos de SQL que serão utilizados durante a aula de análises.
Cada função contém uma explicação curta e um exemplo simples, utilizando o **modelo dimensional (Data Warehouse)**.

Banco de referência: **AdventureWorksDW2022 (SQL Server)**  
Fato principal: `dbo.FactInternetSales`  
Dimensão de datas: `dbo.DimDate`

---

## 1. WHERE
Utilizado para filtrar registros **antes de qualquer agregação**.

```sql
SELECT *
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
WHERE d.CalendarYear >= 2013;
```

---

## 2. JOIN
Utilizado para relacionar fatos e dimensões por meio de chaves.

```sql
SELECT
    f.SalesOrderNumber,
    f.SalesAmount,
    d.FullDateAlternateKey
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey;
```

---

## 3. GROUP BY
Utilizado para agregar dados por categorias.

```sql
SELECT
    d.CalendarYear AS ano,
    COUNT(DISTINCT f.SalesOrderNumber) AS total_pedidos
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
GROUP BY d.CalendarYear;
```

---

## 4. SUM()
Função de agregação utilizada para somar valores numéricos.

```sql
SELECT
    SUM(SalesAmount) AS faturamento_total
FROM dbo.FactInternetSales;
```

---

## 5. AVG()
Função de agregação utilizada para calcular médias.

```sql
SELECT
    AVG(SalesAmount) AS valor_medio_venda
FROM dbo.FactInternetSales;
```

---

## 6. COUNT()
Utilizada para contar registros.

```sql
SELECT
    COUNT(*) AS total_linhas_fato
FROM dbo.FactInternetSales;
```

---

## 7. HAVING
Utilizado para filtrar dados **após a agregação**.

```sql
SELECT
    ProductKey,
    SUM(SalesAmount) AS faturamento
FROM dbo.FactInternetSales
GROUP BY ProductKey
HAVING SUM(SalesAmount) > 100000;
```

---

## 8. CASE WHEN
Utilizado para criar regras de negócio e classificações.

```sql
SELECT
    SalesOrderNumber,
    SalesAmount,
    CASE
        WHEN SalesAmount >= 5000 THEN 'Venda Alta'
        ELSE 'Venda Normal'
    END AS classificacao_venda
FROM dbo.FactInternetSales;
```

---

## 9. CTE (WITH)
Utilizada para organizar consultas complexas em etapas lógicas.

```sql
WITH vendas_por_pedido AS (
    SELECT
        SalesOrderNumber,
        SUM(SalesAmount) AS valor_pedido
    FROM dbo.FactInternetSales
    GROUP BY SalesOrderNumber
)
SELECT *
FROM vendas_por_pedido;
```

---

## 10. VIEW
Utilizada para criar consultas reutilizáveis.

```sql
CREATE VIEW vw_faturamento_anual AS
SELECT
    d.CalendarYear AS ano,
    SUM(f.SalesAmount) AS faturamento_total
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
GROUP BY d.CalendarYear;
```

---

## 11. LAG()
Função analítica utilizada para acessar valores de linhas anteriores.

```sql
SELECT
    d.CalendarYear,
    d.MonthNumberOfYear,
    SUM(f.SalesAmount) AS faturamento,
    LAG(SUM(f.SalesAmount)) OVER (
        ORDER BY d.CalendarYear, d.MonthNumberOfYear
    ) AS faturamento_mes_anterior
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
GROUP BY d.CalendarYear, d.MonthNumberOfYear;
```

---

## 12. RANK()
Utilizada para criar rankings dentro de um conjunto de dados.

```sql
SELECT
    ProductKey,
    SUM(SalesAmount) AS faturamento,
    RANK() OVER (ORDER BY SUM(SalesAmount) DESC) AS ranking_produto
FROM dbo.FactInternetSales
GROUP BY ProductKey;
```

---

## 13. PARTITION BY
Utilizada para segmentar análises dentro de uma window function.

```sql
SELECT
    d.CalendarYear,
    ProductKey,
    SUM(SalesAmount) AS faturamento,
    RANK() OVER (
        PARTITION BY d.CalendarYear
        ORDER BY SUM(SalesAmount) DESC
    ) AS ranking_por_ano
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
GROUP BY d.CalendarYear, ProductKey;
```

---

## 14. Média Móvel (Window Function)
Utilizada para analisar tendência ao longo do tempo.

```sql
SELECT
    d.CalendarYear,
    d.MonthNumberOfYear,
    SUM(f.SalesAmount) AS faturamento,
    AVG(SUM(f.SalesAmount)) OVER (
        ORDER BY d.CalendarYear, d.MonthNumberOfYear
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS media_movel_3_meses
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
GROUP BY d.CalendarYear, d.MonthNumberOfYear;
```

---

## Observação Final
Este guia serve como **nivelamento técnico**, garantindo que todos os alunos compreendam as funções
antes de aplicá-las em análises de negócio reais sobre um **Data Warehouse em estrela**.
