# Análises Avançadas com SQL – AdventureWorksDW2022
## Window Functions, Séries Temporais e Análises Analíticas

Este documento contém **questões avançadas de negócio** e suas **respostas em SQL**, utilizando o modelo
**AdventureWorksDW2022** (Data Warehouse em estrela).

Banco: **SQL Server / SSMS**  
Fato principal: `dbo.FactInternetSales`  
Dimensões: `DimDate`, `DimProduct`, `DimSalesTerritory`

---

# QUESTÕES DE NEGÓCIO

## 1. Crescimento mês a mês do faturamento (MoM)
A diretoria deseja acompanhar a evolução do faturamento ao longo do tempo, comparando cada mês com o mês imediatamente anterior, incluindo variação absoluta e percentual.

---

## 2. Média móvel de 3 meses do faturamento
Para reduzir oscilações pontuais e identificar tendência, é necessário calcular a média móvel de 3 meses do faturamento total.

---

## 3. Ranking anual de produtos por faturamento
A área de portfólio deseja identificar os produtos mais relevantes dentro de cada ano, criando um ranking anual de faturamento.

---

## 4. Participação percentual dos territórios no faturamento anual
A diretoria solicita entender quanto cada território representa do faturamento total da empresa em cada ano.

---


## Análise 5 – Pareto do Faturamento por Produto

---

# RESPOSTAS EM SQL

## 1. Crescimento mês a mês do faturamento (MoM)

```sql
WITH faturamento_mensal AS (
    SELECT
        d.CalendarYear AS ano,
        d.MonthNumberOfYear AS mes,
        SUM(f.SalesAmount) AS faturamento
    FROM dbo.FactInternetSales f
    JOIN dbo.DimDate d
        ON d.DateKey = f.OrderDateKey
    GROUP BY d.CalendarYear, d.MonthNumberOfYear
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

---

## 2. Média móvel de 3 meses do faturamento

```sql
WITH faturamento_mensal AS (
    SELECT
        d.CalendarYear AS ano,
        d.MonthNumberOfYear AS mes,
        SUM(f.SalesAmount) AS faturamento
    FROM dbo.FactInternetSales f
    JOIN dbo.DimDate d
        ON d.DateKey = f.OrderDateKey
    GROUP BY d.CalendarYear, d.MonthNumberOfYear
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

---

## 3. Ranking anual de produtos por faturamento

```sql
WITH faturamento_produto_ano AS (
    SELECT
        d.CalendarYear AS ano,
        p.EnglishProductName AS produto,
        SUM(f.SalesAmount) AS faturamento_total
    FROM dbo.FactInternetSales f
    JOIN dbo.DimDate d
        ON d.DateKey = f.OrderDateKey
    JOIN dbo.DimProduct p
        ON p.ProductKey = f.ProductKey
    GROUP BY d.CalendarYear, p.EnglishProductName
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

---

## 4. Participação percentual dos territórios no faturamento anual

```sql
WITH faturamento_territorio AS (
    SELECT
        d.CalendarYear AS ano,
        st.SalesTerritoryCountry AS pais,
        st.SalesTerritoryRegion AS regiao,
        SUM(f.SalesAmount) AS faturamento
    FROM dbo.FactInternetSales f
    JOIN dbo.DimDate d
        ON d.DateKey = f.OrderDateKey
    JOIN dbo.DimSalesTerritory st
        ON st.SalesTerritoryKey = f.SalesTerritoryKey
    GROUP BY d.CalendarYear, st.SalesTerritoryCountry, st.SalesTerritoryRegion
)
SELECT
    ano,
    pais,
    regiao,
    faturamento,
    faturamento
        / NULLIF(SUM(faturamento) OVER (PARTITION BY ano), 0) * 100 AS participacao_percentual
FROM faturamento_territorio
ORDER BY ano, participacao_percentual DESC;
```

---

# Análise 5 – Pareto do Faturamento por Produto
---

## Query – Pareto do Faturamento por Pedido (Ano/Mês)

```sql
WITH produto_valor AS (
    SELECT
        f.ProductKey,
        SUM(f.SalesAmount) AS faturamento
    FROM dbo.FactInternetSales f
    GROUP BY
        f.ProductKey
),
produto_ordenado AS (
    SELECT
        ProductKey,
        faturamento,
        SUM(faturamento) OVER () AS faturamento_total,
        SUM(faturamento) OVER (
            ORDER BY faturamento DESC, ProductKey
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS faturamento_acumulado
    FROM produto_valor
)
SELECT
    p.EnglishProductName AS produto,
    o.ProductKey,
    o.faturamento,
    CAST(
        100.0 * o.faturamento_acumulado / NULLIF(o.faturamento_total, 0)
        AS decimal(10,2)
    ) AS perc_acumulado
FROM produto_ordenado o
JOIN dbo.DimProduct p
    ON p.ProductKey = o.ProductKey
ORDER BY
    o.faturamento DESC,
    o.ProductKey;


---

Documento de apoio para aulas avançadas de SQL analítico utilizando Data Warehouse.
