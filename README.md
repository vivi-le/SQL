# SQL

Question 1:

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


Question 2: 
-- 1. Identify the first InvoiceLine of the customerID 1060

USE WideWorldImporters

SELECT I.CustomerID, 
			 I.InvoiceID,
			 Il.InvoiceLineID, 
			 Il.Quantity, 
			 Il.UnitPrice

FROM Sales.Invoices as I,
		 Sales.InvoiceLines as Il

WHERE I.InvoiceID = Il.InvoiceID AND I.CustomerID = 1060

ORDER BY I.InvoiceID, Il.InvoiceLineID

-- 2. Modify the UnitPrice of this InvoiceLine by adding 20

UPDATE Sales.InvoiceLines
SET UnitPrice = 260
WHERE InvoiceLineID = 225394

-- 3. Verify the new UnitPrice of the target InvoiceLine by re-running step 1 

-- 4. Then re-run the query in order to have the resultset 



Question 3: 

WITH AllLoss AS 
(

		SELECT Cat.CustomerCategoryName,
		Cu.CustomerID,
		Cu.CustomerName,
		SUM (OL.UnitPrice * OL.Quantity) AS 'MaxLoss'
		
		FROM Sales.Orders AS O,
		Sales.Customers AS Cu,
		Sales.CustomerCategories AS Cat,
		Sales.OrderLines AS OL
		
		WHERE NOT EXISTS
		(
			SELECT *
			FROM Sales.Invoices AS I
			WHERE I.OrderID = O.OrderID
		)
		
		AND O.CustomerID = Cu.CustomerID
		AND Cu.CustomerCategoryID = Cat.CustomerCategoryID
		AND O.OrderID = OL.OrderID
		
		GROUP BY Cat.CustomerCategoryName, Cu.CustomerName, Cu.CustomerID
)

SELECT L1.CustomerCategoryName,
L1.MaxLoss,
L1.CustomerName,
L1.CustomerID

FROM AllLoss as L1

JOIN (
	SELECT
	L.CustomerCategoryName,
	MAX(L.MaxLoss) AS TotalLoss
	FROM AllLoss AS L
	GROUP BY L.CustomerCategoryName
) AS L2
ON L1.CustomerCategoryName = L2.CustomerCategoryName AND L1.MaxLoss = L2.TotalLoss


ORDER BY L1.MaxLoss DESC




Question 4: 

USE SQLPlayground
SELECT *
FROM dbo.Customer as Cu

-- Select all the customers' data who have purchased all the products
WHERE (SELECT COUNT(DISTINCT ProductId)
       FROM dbo.Purchase as Pu
       WHERE Pu.CustomerId = Cu.CustomerId) = (SELECT COUNT(*) FROM dbo.Product)

-- Select all the customers who have bought more than 50 products in total
AND (SELECT SUM(Qty)
     FROM dbo.Purchase as Pu
     WHERE Pu.CustomerId = Cu.CustomerId) > 50
