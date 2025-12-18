# Respostas – Análises de Negócio com SQL (AdventureWorksDW2022)

Este documento contém as respostas em SQL para as solicitações de negócio baseadas no **Data Warehouse AdventureWorksDW2022**.
As queries foram desenvolvidas para **SQL Server (SSMS)** e seguem boas práticas para uso didático em aulas de SQL analítico.

---

## 1. Pedidos realizados a partir de um determinado ano e valor total por pedido

```sql
SELECT
    f.SalesOrderNumber,
    d.FullDateAlternateKey AS data_pedido,
    SUM(f.SalesAmount) AS valor_total_pedido
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
WHERE d.CalendarYear >= 2013
GROUP BY
    f.SalesOrderNumber,
    d.FullDateAlternateKey
ORDER BY data_pedido;
```

---

## 2. Faturamento total por mês e ano

```sql
SELECT
    d.CalendarYear AS ano,
    d.MonthNumberOfYear AS mes,
    SUM(f.SalesAmount) AS faturamento_total
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
GROUP BY
    d.CalendarYear,
    d.MonthNumberOfYear
ORDER BY ano, mes;
```

---

## 3. Quantidade de pedidos por tipo de venda

```sql
SELECT
    'Internet Sales' AS tipo_venda,
    COUNT(DISTINCT SalesOrderNumber) AS quantidade_pedidos
FROM dbo.FactInternetSales

UNION ALL

SELECT
    'Reseller Sales' AS tipo_venda,
    COUNT(DISTINCT SalesOrderNumber) AS quantidade_pedidos
FROM dbo.FactResellerSales;
```

---

## 4. Produtos com maior faturamento total

```sql
SELECT
    p.EnglishProductName AS produto,
    SUM(f.SalesAmount) AS faturamento_total
FROM dbo.FactInternetSales f
JOIN dbo.DimProduct p
    ON p.ProductKey = f.ProductKey
GROUP BY p.EnglishProductName
ORDER BY faturamento_total DESC;
```

---

## 5. Produtos com maior quantidade vendida

```sql
SELECT
    p.EnglishProductName AS produto,
    SUM(f.OrderQuantity) AS quantidade_total_vendida
FROM dbo.FactInternetSales f
JOIN dbo.DimProduct p
    ON p.ProductKey = f.ProductKey
GROUP BY p.EnglishProductName
ORDER BY quantidade_total_vendida DESC;
```

---

## 6. Faturamento total por território

```sql
SELECT
    st.SalesTerritoryCountry AS pais,
    st.SalesTerritoryRegion  AS regiao,
    SUM(f.SalesAmount) AS faturamento_total
FROM dbo.FactInternetSales f
JOIN dbo.DimSalesTerritory st
    ON st.SalesTerritoryKey = f.SalesTerritoryKey
GROUP BY
    st.SalesTerritoryCountry,
    st.SalesTerritoryRegion
ORDER BY faturamento_total DESC;
```

---

## 7. Faturamento total por revendedor

```sql
SELECT
    r.ResellerName,
    SUM(f.SalesAmount) AS faturamento_total
FROM dbo.FactResellerSales f
JOIN dbo.DimReseller r
    ON r.ResellerKey = f.ResellerKey
GROUP BY r.ResellerName
ORDER BY faturamento_total DESC;
```

---

## 8. Ticket médio dos pedidos por ano

```sql
WITH pedidos_ano AS (
    SELECT
        d.CalendarYear AS ano,
        f.SalesOrderNumber,
        SUM(f.SalesAmount) AS valor_pedido
    FROM dbo.FactInternetSales f
    JOIN dbo.DimDate d
        ON d.DateKey = f.OrderDateKey
    GROUP BY
        d.CalendarYear,
        f.SalesOrderNumber
)
SELECT
    ano,
    AVG(valor_pedido) AS ticket_medio
FROM pedidos_ano
GROUP BY ano
ORDER BY ano;
```

---

## 9. Produtos com faturamento acima de um valor mínimo

```sql
SELECT
    p.EnglishProductName AS produto,
    SUM(f.SalesAmount) AS faturamento_total
FROM dbo.FactInternetSales f
JOIN dbo.DimProduct p
    ON p.ProductKey = f.ProductKey
GROUP BY p.EnglishProductName
HAVING SUM(f.SalesAmount) > 1000000
ORDER BY faturamento_total DESC;
```

---

## 10. Clientes com maior faturamento total

```sql
SELECT
    c.CustomerKey,
    c.FirstName,
    c.LastName,
    SUM(f.SalesAmount) AS faturamento_total
FROM dbo.FactInternetSales f
JOIN dbo.DimCustomer c
    ON c.CustomerKey = f.CustomerKey
GROUP BY
    c.CustomerKey,
    c.FirstName,
    c.LastName
ORDER BY faturamento_total DESC;
```

---

## 11. Ranking de clientes por faturamento

```sql
WITH faturamento_cliente AS (
    SELECT
        c.CustomerKey,
        c.FirstName,
        c.LastName,
        SUM(f.SalesAmount) AS faturamento_total
    FROM dbo.FactInternetSales f
    JOIN dbo.DimCustomer c
        ON c.CustomerKey = f.CustomerKey
    GROUP BY
        c.CustomerKey,
        c.FirstName,
        c.LastName
)
SELECT
    CustomerKey,
    FirstName,
    LastName,
    faturamento_total,
    RANK() OVER (ORDER BY faturamento_total DESC) AS ranking_cliente
FROM faturamento_cliente;
```

---

## 12. Classificação de clientes por faixa de valor

```sql
WITH faturamento_cliente AS (
    SELECT
        c.CustomerKey,
        SUM(f.SalesAmount) AS faturamento_total
    FROM dbo.FactInternetSales f
    JOIN dbo.DimCustomer c
        ON c.CustomerKey = f.CustomerKey
    GROUP BY c.CustomerKey
)
SELECT
    CustomerKey,
    faturamento_total,
    CASE
        WHEN faturamento_total >= 500000 THEN 'Alto Valor'
        WHEN faturamento_total >= 100000 THEN 'Médio Valor'
        ELSE 'Baixo Valor'
    END AS classificacao_cliente
FROM faturamento_cliente;
```

---

## 13. Análise de sazonalidade (faturamento por mês)

```sql
SELECT
    d.MonthNumberOfYear AS mes,
    SUM(f.SalesAmount) AS faturamento_total
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
GROUP BY d.MonthNumberOfYear
ORDER BY mes;
```

---

## 14. Evolução do faturamento por território e ano

```sql
SELECT
    d.CalendarYear AS ano,
    st.SalesTerritoryCountry AS pais,
    SUM(f.SalesAmount) AS faturamento_total
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
JOIN dbo.DimSalesTerritory st
    ON st.SalesTerritoryKey = f.SalesTerritoryKey
GROUP BY
    d.CalendarYear,
    st.SalesTerritoryCountry
ORDER BY pais, ano;
```

---

## 15. View consolidada de vendas

```sql
CREATE VIEW vw_vendas_consolidada_dw AS
SELECT
    f.SalesOrderNumber,
    d.FullDateAlternateKey AS data_pedido,
    p.EnglishProductName AS produto,
    c.FirstName + ' ' + c.LastName AS cliente,
    st.SalesTerritoryCountry AS pais,
    f.OrderQuantity,
    f.SalesAmount
FROM dbo.FactInternetSales f
JOIN dbo.DimDate d
    ON d.DateKey = f.OrderDateKey
JOIN dbo.DimProduct p
    ON p.ProductKey = f.ProductKey
JOIN dbo.DimCustomer c
    ON c.CustomerKey = f.CustomerKey
JOIN dbo.DimSalesTerritory st
    ON st.SalesTerritoryKey = f.SalesTerritoryKey;
```

---

Documento de apoio para aulas e treinamentos práticos com SQL analítico sobre Data Warehouse.
