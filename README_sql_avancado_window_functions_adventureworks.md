# Solicitações Avançadas de Análise – AdventureWorks
## Window Functions, Séries Temporais e Análises Analíticas

Este documento contém solicitações avançadas de negócio seguidas de suas respectivas respostas em SQL,
utilizando recursos analíticos como LAG, médias móveis e window functions.
As queries estão escritas para SQL Server (SSMS).

---

# QUESTÕES DE NEGÓCIO

## 1. Crescimento mês a mês do faturamento (MoM)
A diretoria deseja acompanhar a evolução do faturamento ao longo do tempo, comparando cada mês com o mês imediatamente anterior, incluindo a variação absoluta e percentual.

Tabelas:
Sales.SalesOrderHeader, Sales.SalesOrderDetail

---

## 2. Média móvel de 3 meses do faturamento
Para análise de tendência e redução de oscilações pontuais, precisamos calcular a média móvel de 3 meses do faturamento total.

Tabelas:
Sales.SalesOrderHeader, Sales.SalesOrderDetail

---

## 3. Ranking anual de produtos por faturamento
A área de portfólio deseja identificar os produtos mais relevantes dentro de cada ano, criando um ranking de faturamento anual.

Tabelas:
Sales.SalesOrderHeader, Sales.SalesOrderDetail, Production.Product

---

## 4. Participação percentual dos territórios no faturamento anual
A diretoria solicita uma análise comparativa para entender quanto cada território representa do faturamento total da empresa em cada ano.

Tabelas:
Sales.SalesOrderHeader, Sales.SalesOrderDetail, Sales.SalesTerritory

---

## 5. Identificação de pedidos fora do padrão (outliers)
A área financeira quer identificar pedidos com valores anormalmente altos em relação ao padrão do mês, para fins de auditoria.

Tabelas:
Sales.SalesOrderHeader, Sales.SalesOrderDetail

---

# RESPOSTAS EM SQL

## 1. Crescimento mês a mês do faturamento (MoM)

```sql
WITH faturamento_mensal AS (
    SELECT
        YEAR(soh.OrderDate) AS ano,
        MONTH(soh.OrderDate) AS mes,
        SUM(sod.LineTotal) AS faturamento
    FROM Sales.SalesOrderHeader soh
    JOIN Sales.SalesOrderDetail sod
        ON soh.SalesOrderID = sod.SalesOrderID
    GROUP BY YEAR(soh.OrderDate), MONTH(soh.OrderDate)
)
SELECT
    ano,
    mes,
    faturamento,
    LAG(faturamento) OVER (ORDER BY ano, mes) AS faturamento_mes_anterior,
    faturamento - LAG(faturamento) OVER (ORDER BY ano, mes) AS variacao_absoluta,
    (faturamento - LAG(faturamento) OVER (ORDER BY ano, mes))
        / NULLIF(LAG(faturamento) OVER (ORDER BY ano, mes), 0) * 100 AS variacao_percentual
FROM faturamento_mensal
ORDER BY ano, mes;
```

## 2. Média móvel de 3 meses do faturamento

```sql
WITH faturamento_mensal AS (
    SELECT
        YEAR(soh.OrderDate) AS ano,
        MONTH(soh.OrderDate) AS mes,
        SUM(sod.LineTotal) AS faturamento
    FROM Sales.SalesOrderHeader soh
    JOIN Sales.SalesOrderDetail sod
        ON soh.SalesOrderID = sod.SalesOrderID
    GROUP BY YEAR(soh.OrderDate), MONTH(soh.OrderDate)
)
SELECT
    ano,
    mes,
    faturamento,
    AVG(faturamento) OVER (
        ORDER BY ano, mes
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS media_movel_3_meses
FROM faturamento_mensal
ORDER BY ano, mes;
```

## 3. Ranking anual de produtos por faturamento

```sql
WITH faturamento_produto_ano AS (
    SELECT
        YEAR(soh.OrderDate) AS ano,
        p.Name AS produto,
        SUM(sod.LineTotal) AS faturamento_total
    FROM Sales.SalesOrderHeader soh
    JOIN Sales.SalesOrderDetail sod
        ON soh.SalesOrderID = sod.SalesOrderID
    JOIN Production.Product p
        ON sod.ProductID = p.ProductID
    GROUP BY YEAR(soh.OrderDate), p.Name
)
SELECT
    ano,
    produto,
    faturamento_total,
    RANK() OVER (
        PARTITION BY ano
        ORDER BY faturamento_total DESC
    ) AS ranking_anual
FROM faturamento_produto_ano
ORDER BY ano, ranking_anual;
```

## 4. Participação percentual dos territórios no faturamento anual

```sql
WITH faturamento_territorio AS (
    SELECT
        YEAR(soh.OrderDate) AS ano,
        st.Name AS territorio,
        SUM(sod.LineTotal) AS faturamento
    FROM Sales.SalesOrderHeader soh
    JOIN Sales.SalesTerritory st
        ON soh.TerritoryID = st.TerritoryID
    JOIN Sales.SalesOrderDetail sod
        ON soh.SalesOrderID = sod.SalesOrderID
    GROUP BY YEAR(soh.OrderDate), st.Name
)
SELECT
    ano,
    territorio,
    faturamento,
    faturamento
        / SUM(faturamento) OVER (PARTITION BY ano) * 100
        AS participacao_percentual
FROM faturamento_territorio
ORDER BY ano, participacao_percentual DESC;
```

## 5. Identificação de pedidos fora do padrão (outliers)

```sql
WITH pedido_valor AS (
    SELECT
        soh.SalesOrderID,
        YEAR(soh.OrderDate) AS ano,
        MONTH(soh.OrderDate) AS mes,
        SUM(sod.LineTotal) AS valor_pedido
    FROM Sales.SalesOrderHeader soh
    JOIN Sales.SalesOrderDetail sod
        ON soh.SalesOrderID = sod.SalesOrderID
    GROUP BY
        soh.SalesOrderID,
        YEAR(soh.OrderDate),
        MONTH(soh.OrderDate)
),
estatisticas_mensais AS (
    SELECT
        ano,
        mes,
        AVG(valor_pedido) AS media_mensal,
        STDEV(valor_pedido) AS desvio_padrao
    FROM pedido_valor
    GROUP BY ano, mes
)
SELECT
    p.SalesOrderID,
    p.ano,
    p.mes,
    p.valor_pedido,
    e.media_mensal,
    e.desvio_padrao
FROM pedido_valor p
JOIN estatisticas_mensais e
    ON p.ano = e.ano
   AND p.mes = e.mes
WHERE p.valor_pedido > e.media_mensal + 2 * e.desvio_padrao
ORDER BY p.valor_pedido DESC;
```

---

Documento de apoio para aulas avançadas de SQL analítico e BI.
