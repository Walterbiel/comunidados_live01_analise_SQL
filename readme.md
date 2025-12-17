# Solicitações de Análise de Negócio – AdventureWorks

Este documento reúne solicitações reais de análise feitas pelas áreas de negócio da empresa, com base no Data Warehouse AdventureWorks.  
As perguntas estão organizadas em **ordem crescente de dificuldade**, iniciando em análises básicas e evoluindo para análises mais estruturadas e reutilizáveis.

O objetivo é apoiar decisões das áreas Comercial, Financeira, CRM e Operações.

---

## 1. Análise de pedidos recentes
Precisamos analisar o desempenho recente da empresa considerando apenas os pedidos realizados a partir de um determinado ano do histórico.  
Quais pedidos foram realizados nesse período e qual foi o valor total de cada pedido?

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.SalesOrderDetail`

---

## 2. Evolução do faturamento ao longo do tempo
A diretoria solicita uma visão consolidada da evolução do faturamento da empresa.  
Qual foi o faturamento total por mês e por ano durante todo o histórico disponível?

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.SalesOrderDetail`

---

## 3. Distribuição de pedidos por status
Queremos entender o comportamento operacional da empresa.  
Quantos pedidos foram realizados por status (por exemplo: concluído, cancelado, em processamento)?

**Tabelas:**  
`Sales.SalesOrderHeader`

---

## 4. Produtos com maior faturamento
Para apoiar decisões de portfólio, precisamos identificar quais produtos geraram maior receita ao longo do tempo.  
Quais são os produtos com maior faturamento total no histórico de vendas?

**Tabelas:**  
`Sales.SalesOrderDetail`  
`Production.Product`

---

## 5. Produtos com maior volume vendido
Além de receita, queremos analisar volume de vendas.  
Quais produtos apresentaram a maior quantidade total vendida ao longo do tempo?

**Tabelas:**  
`Sales.SalesOrderDetail`  
`Production.Product`

---

## 6. Performance de vendas por território
O time comercial deseja entender o desempenho regional das vendas.  
Qual foi o faturamento total por território de vendas considerando todo o histórico?

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.SalesTerritory`  
`Sales.SalesOrderDetail`

---

## 7. Performance dos vendedores
Precisamos avaliar a contribuição individual dos vendedores.  
Qual foi o faturamento total gerado por cada vendedor ao longo do histórico?

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.SalesPerson`  
`Sales.SalesOrderDetail`

---

## 8. Ticket médio dos pedidos
A área financeira solicitou uma análise do ticket médio.  
Qual é o valor médio dos pedidos por ano?

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.SalesOrderDetail`

---

## 9. Produtos mais relevantes financeiramente
Para focar esforços nos produtos mais estratégicos, precisamos identificar aqueles que realmente impactam o resultado.  
Quais produtos apresentaram faturamento total acima de um valor mínimo definido pela área de negócio?

**Tabelas:**  
`Sales.SalesOrderDetail`  
`Production.Product`

---

## 10. Clientes com maior faturamento
A área de CRM deseja entender o valor gerado pelos clientes ao longo do tempo.  
Quais clientes foram responsáveis pelo maior faturamento total no histórico de vendas?

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.Customer`  
`Sales.SalesOrderDetail`

---

## 11. Ranking de clientes
Com base no faturamento acumulado, precisamos de um ranking dos clientes mais valiosos para o negócio, do maior para o menor valor gerado.

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.Customer`  
`Sales.SalesOrderDetail`

---

## 12. Classificação de clientes por faixa de valor
Para apoiar estratégias comerciais e de relacionamento, é necessário classificar os clientes em faixas de valor (alto, médio e baixo), considerando o faturamento total histórico de cada cliente.

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.Customer`  
`Sales.SalesOrderDetail`

---

## 13. Análise de sazonalidade
O time de operações deseja entender o comportamento sazonal das vendas.  
Existem meses ou períodos do ano com maior concentração de faturamento?

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.SalesOrderDetail`

---

## 14. Evolução do faturamento por território
A diretoria solicitou uma análise comparativa entre territórios ao longo do tempo.  
Como o faturamento de cada território evoluiu ano a ano?

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.SalesTerritory`  
`Sales.SalesOrderDetail`

---

## 15. Visão consolidada para consumo das áreas
Para facilitar análises recorrentes por diferentes áreas, é solicitada a criação de uma visão consolidada que reúna informações de vendas, produtos, clientes, vendedores e territórios, permitindo consultas padronizadas e reutilizáveis.

**Tabelas:**  
`Sales.SalesOrderHeader`  
`Sales.SalesOrderDetail`  
`Production.Product`  
`Sales.Customer`  
`Sales.SalesPerson`  
`Sales.SalesTerritory`

---

## Observação Final
Essas solicitações representam demandas típicas de negócio e servem como base para desenvolvimento de análises, relatórios e ativos de dados reutilizáveis dentro do Data Warehouse.
