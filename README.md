# ABAP_Project 1
## 1. Database Table for Products
_Objective_: Create a table to store product details.
### Requirements:
- Include at least 5 fields + client number.
Carefully choose data types and lengths to optimize memory.
Example fields: ProductID, ProductName, Category, Price, StockQuantity, ClientNumber.
Use data elements and domains to maintain consistency.
## 2. Database Table for Invoices
_Objective_: Create a table to store invoice details.
### Requirements:
- Include fields: InvoiceNumber, InvoiceDate, CustomerName, City, PostalCode, ProductID, Quantity, Amount.
Ensure the ProductID references the product table (foreign key).
Use appropriate data types and domains.
## 3. Input Screen Setup
a. Display All Products 
b. Display Invoices for a Specific Product
c.Recalculate and Update Invoice Amounts
d.Invoice Summary by Product
