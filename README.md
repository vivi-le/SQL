# SQL

## 1


WITH
TotalOrders AS
(
SELECT
OL.OrderID AS OrderID,
SUM(OL.Quantity * OL.UnitPrice) AS TotalOder
FROM WideWorldImporters.Sales.OrderLines AS OL
GROUP BY OL.OrderID
),

TotalInvoices AS
(
SELECT
IL.InvoiceID AS InvoiceID,
SUM(IL.Quantity * IL.UnitPrice) AS TotalInvoice
FROM WideWorldImporters.Sales.InvoiceLines AS IL
GROUP BY IL.InvoiceID
)

SELECT
O.CustomerID,
C.CustomerName,
COUNT(O.OrderID) AS TotalNBOrders,
COUNT(I.OrderID) AS TotalNBInvoices,
SUM(TotalOder) AS OrdersTotalValue,
SUM(TotalInvoice) AS InvoicesTotalValue,
ABS(SUM(TotalOder) - SUM(TotalInvoice)) AS AbsoluteValueDifference

FROM
WideWorldImporters.Sales.Orders AS O,
TotalOrders AS TO1,
WideWorldImporters.Sales.Invoices AS I,
TotalInvoices AS TI2,
WideWorldImporters.Sales.Customers AS C

WHERE
O.CustomerID = C.CustomerID
AND O.OrderID = I.OrderID
AND O.OrderID = TO1.OrderID
AND I.InvoiceID = TI2.InvoiceID

GROUP BY O.CustomerID, C.CustomerName
ORDER BY AbsoluteValueDifference DESC, TotalNBOrders, C.CustomerName
