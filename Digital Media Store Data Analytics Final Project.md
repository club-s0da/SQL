```sql
--In this SQL, I'm querying a database called chinook with multiple tables in it, which represents a digital media store and includes tables for artists, albums, media tracks, invoices, and customers.

--Q1: What is the title and name of the employees to whom each employee reports?
SELECT e.employeeId,
       e.Title,
       e.firstName,
       e.lastName,
       r.employeeId AS ReportsToId,
       r.Title,
       r.firstName AS ReportsToFirstName,
       r.lastName AS ReportsToLastName
  FROM employees e
       LEFT JOIN
       employees r ON e.ReportsTo = r.employeeId;

--Q2: Provide a list of employees who have not reported to anyone.
SELECT Title,
       firstName,
       lastName
  FROM employees
 WHERE reportsTo IS NULL;

--Q3: What is the average number of tracks per playlist?
WITH playlistCTE AS (
    SELECT DISTINCT COUNT(playlistID) AS trackQuantity,
                    playlistID,
                    TrackID
      FROM playlist_track
     GROUP BY playlistID
)
SELECT avg(trackQuantity) AS avgTracksPerPlaylist
  FROM playlistCTE;

--Q4: What are the top three most purchased genres of music in 2009?
SELECT COUNT(invItems.trackID) AS quantityTracks,
           Genres.Name AS genreName
      FROM invoices
           JOIN invoice_items invItems ON invoices.InvoiceId = invItems.InvoiceId
           JOIN tracks ON invItems.trackId = tracks.TrackId
           JOIN genres ON tracks.GenreId = genres.GenreId
     WHERE invoices.InvoiceDate LIKE '2009%'
     GROUP BY genreName
     ORDER BY quantityTracks DESC LIMIT 3;

--Q5: Show Customers (their full names, customer ID, and country) who are not in the US. (Hint: != or <> can be used to say "is not equal to").
SELECT CustomerId,
       FirstName,
       LastName,
       Country
  FROM chinook.customers
 WHERE country != 'USA';

--Q6: Show only the Customers from Brazil.
SELECT *
  FROM chinook.customers
 WHERE country = 'Brazil';

--Q7: Find the Invoices of customers who are from Brazil.
SELECT cust.FirstName,
       cust.LastName,
       inv.InvoiceId,
       inv.InvoiceDate,
       inv.BillingCountry
  FROM customers cust
       LEFT JOIN invoices inv ON cust.CustomerId = inv.CustomerId
 WHERE cust.country = 'Brazil';

--Q8: Show the Employees who are Sales Agents.
SELECT EmployeeId,
       FirstName,
       LastName,
       Title
  FROM employees
 WHERE Title LIKE 'Sales%Agent';

--Q9: Find a unique/distinct list of billing countries from the Invoice table.
SELECT DISTINCT BillingCountry
  FROM invoices
 ORDER BY BillingCountry;

--Q10: Provide a query that shows the invoices associated with each sales agent. Include the Sales Agent's full name.
SELECT inv.InvoiceId,
       emp.LastName,
       emp.FirstName,
       emp.Title
  FROM employees emp
       LEFT JOIN customers cust ON emp.EmployeeId = cust.SupportRepId
       INNER JOIN invoices inv ON cust.CustomerId = inv.CustomerId
 ORDER BY emp.LastName;

--Q11: Show the Invoice Total, Customer name, Country, and Sales Agent name for all invoices and customers.
SELECT cust.FirstName AS CustFirstName,
       cust.LastName AS CustLastName,
       inv.Total AS InvTotal,
       cust.Country,
       emp.FirstName AS SalesAgentFirstName,
       emp.LastName AS SalesAgentLastName
  FROM employees emp
       LEFT JOIN customers cust ON emp.EmployeeId = cust.SupportRepId
       LEFT JOIN invoices inv ON cust.CustomerId = inv.CustomerId
 WHERE cust.Country NOT NULL;

--Q12: How many Invoices were there in 2009?
SELECT COUNT(invoiceID) 
  FROM invoices
 WHERE InvoiceDate LIKE '2009%';

--Q13: What are the total sales for 2009?
SELECT sum(Total) AS totalSales,
       InvoiceDate
  FROM invoices
 WHERE InvoiceDate BETWEEN '2009-01-01' AND '2009-12-31';

--Q14: Write a query that includes the purchased track name with each invoice line ID.
SELECT invItems.InvoiceLineId,
       Tracks.Name
  FROM invoice_items InvItems
       LEFT JOIN Tracks ON invItems.TrackId = tracks.TrackId
 ORDER BY InvoiceLineId;
 
--Q15: Write a query that includes the purchased track name AND artist name with each invoice line ID.
SELECT invItems.InvoiceLineId,
       tracks.Name AS songName,
       artists.Name AS artistName-- invoiceline to tracks to albums to artists
  FROM invoice_items InvItems
       LEFT JOIN Tracks ON invItems.TrackId = tracks.TrackId
       INNER JOIN albums ON albums.AlbumId = tracks.AlbumId
       INNER JOIN artists ON albums.ArtistId = artists.ArtistId
 ORDER BY InvoiceLineId;

--Q16: Provide a query that shows all the Tracks, and include the Album name, Media type, and Genre.
SELECT tracks.Name AS trackName,
       albums.Title AS albumTitle,
       genres.Name AS genre,
       med.Name AS mediaType
  FROM tracks
       LEFT JOIN albums ON albums.AlbumId = tracks.AlbumId
       INNER JOIN genres ON tracks.GenreId = genres.GenreId
       INNER JOIN media_types med ON tracks.MediaTypeId = med.MediaTypeId
 ORDER BY trackName;

--Q17: Show the total sales made by each sales agent.
SELECT DISTINCT emp.LastName AS salesRepLastName,
                emp.FirstName AS salesRepFirstName,
                ROUND(SUM(inv.Total),2) AS totalSales
  FROM employees emp
       LEFT JOIN customers cust ON emp.EmployeeId = cust.SupportRepId
       JOIN invoices inv ON inv.CustomerId = cust.CustomerId
 WHERE emp.Title LIKE 'Sales%Agent'
 GROUP BY emp.LastName,
          emp.FirstName;

--Q18: Which sales agent made the most dollars in sales in 2009?
SELECT emp.FirstName,
       emp.LastName,
       ROUND(SUM(Inv.Total), 2) AS [Total Sales]
  FROM chinook.Employees emp
       JOIN chinook.Customers cust ON cust.SupportRepId = emp.EmployeeId
       JOIN chinook.Invoices Inv ON Inv.CustomerId = cust.CustomerId
 WHERE emp.Title = 'Sales Support Agent' AND 
       Inv.InvoiceDate LIKE '2009%'
 GROUP BY emp.FirstName
 ORDER BY 'Total Sales' DESC LIMIT 1;

--Q19: What is the name of the track that grossed the most in 2010? (count the most purchased track multiplied by the unitprice and quantity)
WITH tracks_CTE AS (
    SELECT invItems.trackId AS track_ID,
           COUNT(*) AS trackIdCount,
           tracks.Name AS track,
           invItems.UnitPrice
      FROM tracks
           JOIN invoice_items invItems ON tracks.TrackId = invItems.TrackId
           JOIN invoices inv ON invItems.invoiceId = inv.invoiceId
     WHERE inv.InvoiceDate LIKE '2010%'
     GROUP BY track_Id
)
SELECT (trackIdCount * UnitPrice) AS HighestGrossTrack,
       track
  FROM tracks_CTE
 ORDER BY highestGRosstrack DESC LIMIT 1;

```
