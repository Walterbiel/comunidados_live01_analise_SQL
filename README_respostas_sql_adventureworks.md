# Respostas – Análises de Negócio com SQL (AdventureWorks)

Este documento contém as respostas em SQL para as solicitações de negócio baseadas no Data Warehouse AdventureWorks.
As queries foram desenvolvidas para SQL Server (SSMS) e seguem boas práticas para uso didático em aulas de SQL.

---

## 1. Pedidos realizados a partir de um determinado ano e valor total por pedido

```sql
SELECT
    soh.SalesOrderID,
    soh.OrderDate,
    SUM(sod.LineTotal) AS valor_total_pedido
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesOrderDetail sod
    ON soh.SalesOrderID = sod.SalesOrderID
WHERE YEAR(soh.OrderDate) >= 2013
GROUP BY
    soh.SalesOrderID,
    soh.OrderDate
ORDER BY soh.OrderDate;
```

## 2. Faturamento total por mês e ano

```sql
SELECT
    YEAR(soh.OrderDate)  AS ano,
    MONTH(soh.OrderDate) AS mes,
    SUM(sod.LineTotal)   AS faturamento_total
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesOrderDetail sod
    ON soh.SalesOrderID = sod.SalesOrderID
GROUP BY
    YEAR(soh.OrderDate),
    MONTH(soh.OrderDate)
ORDER BY ano, mes;
```

## 3. Quantidade de pedidos por status

```sql
SELECT
    Status,
    COUNT(*) AS quantidade_pedidos
FROM Sales.SalesOrderHeader
GROUP BY Status
ORDER BY quantidade_pedidos DESC;
```

## 4. Produtos com maior faturamento total

```sql
SELECT
    p.Name AS produto,
    SUM(sod.LineTotal) AS faturamento_total
FROM Sales.SalesOrderDetail sod
JOIN Production.Product p
    ON sod.ProductID = p.ProductID
GROUP BY p.Name
ORDER BY faturamento_total DESC;
```

## 5. Produtos com maior quantidade vendida

```sql
SELECT
    p.Name AS produto,
    SUM(sod.OrderQty) AS quantidade_total_vendida
FROM Sales.SalesOrderDetail sod
JOIN Production.Product p
    ON sod.ProductID = p.ProductID
GROUP BY p.Name
ORDER BY quantidade_total_vendida DESC;
```

## 6. Faturamento total por território

```sql
SELECT
    st.Name AS territorio,
    SUM(sod.LineTotal) AS faturamento_total
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesTerritory st
    ON soh.TerritoryID = st.TerritoryID
JOIN Sales.SalesOrderDetail sod
    ON soh.SalesOrderID = sod.SalesOrderID
GROUP BY st.Name
ORDER BY faturamento_total DESC;
```

## 7. Faturamento total por vendedor

```sql
SELECT
    sp.BusinessEntityID AS vendedor_id,
    SUM(sod.LineTotal) AS faturamento_total
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesPerson sp
    ON soh.SalesPersonID = sp.BusinessEntityID
JOIN Sales.SalesOrderDetail sod
    ON soh.SalesOrderID = sod.SalesOrderID
GROUP BY sp.BusinessEntityID
ORDER BY faturamento_total DESC;
```

## 8. Ticket médio dos pedidos por ano

```sql
SELECT
    YEAR(OrderDate) AS ano,
    AVG(pedido_total) AS ticket_medio
FROM (
    SELECT
        soh.SalesOrderID,
        soh.OrderDate,
        SUM(sod.LineTotal) AS pedido_total
    FROM Sales.SalesOrderHeader soh
    JOIN Sales.SalesOrderDetail sod
        ON soh.SalesOrderID = sod.SalesOrderID
    GROUP BY soh.SalesOrderID, soh.OrderDate
) t
GROUP BY YEAR(OrderDate)
ORDER BY ano;
```

## 9. Produtos com faturamento acima de um valor mínimo

```sql
SELECT
    p.Name AS produto,
    SUM(sod.LineTotal) AS faturamento_total
FROM Sales.SalesOrderDetail sod
JOIN Production.Product p
    ON sod.ProductID = p.ProductID
GROUP BY p.Name
HAVING SUM(sod.LineTotal) > 1000000
ORDER BY faturamento_total DESC;
```

## 10. Clientes com maior faturamento total

```sql
SELECT
    c.CustomerID,
    SUM(sod.LineTotal) AS faturamento_total
FROM Sales.SalesOrderHeader soh
JOIN Sales.Customer c
    ON soh.CustomerID = c.CustomerID
JOIN Sales.SalesOrderDetail sod
    ON soh.SalesOrderID = sod.SalesOrderID
GROUP BY c.CustomerID
ORDER BY faturamento_total DESC;
```

## 11. Ranking de clientes por faturamento

```sql
WITH faturamento_cliente AS (
    SELECT
        c.CustomerID,
        SUM(sod.LineTotal) AS faturamento_total
    FROM Sales.SalesOrderHeader soh
    JOIN Sales.Customer c
        ON soh.CustomerID = c.CustomerID
    JOIN Sales.SalesOrderDetail sod
        ON soh.SalesOrderID = sod.SalesOrderID
    GROUP BY c.CustomerID
)
SELECT
    CustomerID,
    faturamento_total,
    RANK() OVER (ORDER BY faturamento_total DESC) AS ranking_cliente
FROM faturamento_cliente;
```

## 12. Classificação de clientes por faixa de valor

```sql
WITH faturamento_cliente AS (
    SELECT
        c.CustomerID,
        SUM(sod.LineTotal) AS faturamento_total
    FROM Sales.SalesOrderHeader soh
    JOIN Sales.Customer c
        ON soh.CustomerID = c.CustomerID
    JOIN Sales.SalesOrderDetail sod
        ON soh.SalesOrderID = sod.SalesOrderID
    GROUP BY c.CustomerID
)
SELECT
    CustomerID,
    faturamento_total,
    CASE
        WHEN faturamento_total >= 500000 THEN 'Alto Valor'
        WHEN faturamento_total >= 100000 THEN 'Médio Valor'
        ELSE 'Baixo Valor'
    END AS classificacao_cliente
FROM faturamento_cliente;
```

## 13. Análise de sazonalidade (faturamento por mês)

```sql
SELECT
    MONTH(soh.OrderDate) AS mes,
    SUM(sod.LineTotal) AS faturamento_total
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesOrderDetail sod
    ON soh.SalesOrderID = sod.SalesOrderID
GROUP BY MONTH(soh.OrderDate)
ORDER BY mes;
```

## 14. Evolução do faturamento por território e ano

```sql
SELECT
    st.Name AS territorio,
    YEAR(soh.OrderDate) AS ano,
    SUM(sod.LineTotal) AS faturamento_total
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesTerritory st
    ON soh.TerritoryID = st.TerritoryID
JOIN Sales.SalesOrderDetail sod
    ON soh.SalesOrderID = sod.SalesOrderID
GROUP BY st.Name, YEAR(soh.OrderDate)
ORDER BY territorio, ano;
```

## 15. View consolidada de vendas

```sql
CREATE VIEW vw_vendas_consolidada AS
SELECT
    soh.SalesOrderID,
    soh.OrderDate,
    c.CustomerID,
    p.Name AS produto,
    sp.BusinessEntityID AS vendedor_id,
    st.Name AS territorio,
    sod.OrderQty,
    sod.LineTotal
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesOrderDetail sod
    ON soh.SalesOrderID = sod.SalesOrderID
JOIN Production.Product p
    ON sod.ProductID = p.ProductID
JOIN Sales.Customer c
    ON soh.CustomerID = c.CustomerID
LEFT JOIN Sales.SalesPerson sp
    ON soh.SalesPersonID = sp.BusinessEntityID
JOIN Sales.SalesTerritory st
    ON soh.TerritoryID = st.TerritoryID;
```

---

Documento de apoio para aulas e treinamentos práticos com SQL e Data Warehouse.
