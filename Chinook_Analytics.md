# Chinook Music Analytics

## Project Steps

### 1. Get Data in MySQL

```sql
mysql -p
source /home/akshat/Desktop/Data Ingestion/chinook_MySql_AutoIncrementPKs.sql
```

### 2. Ingest All Tables into Hive from MySQL

```bash
sqoop import-all-tables \
  --connect jdbc:mysql://localhost:3306/Chinook \
  --username akshat \
  --password root \
  --hive-import \
  --hive-database chinook \
  --create-hive-table \
  --fields-terminated-by ',' \
  --lines-terminated-by '\n' \
  --relaxed-isolation
```

### 3. Hive Queries

#### Top Selling Genres

```sql
-- INSERT query for top_selling_genres
INSERT OVERWRITE DIRECTORY '/user/hive/output/top_selling_genres'
SELECT
    g.Name AS Genre,
    SUM(il.UnitPrice * il.Quantity) AS TotalRevenue
FROM Genre g
JOIN Track t ON g.GenreId = t.GenreId
JOIN InvoiceLine il ON t.TrackId = il.TrackId
GROUP BY g.Name
ORDER BY TotalRevenue DESC;
```

#### Employee Sales Performance

```sql
-- INSERT query for employee_sales_performance
INSERT OVERWRITE DIRECTORY '/user/hive/output/employee_sales_performance'
SELECT
    e.EmployeeId,
    e.FirstName || ' ' || e.LastName AS EmployeeName,
    SUM(il.UnitPrice * il.Quantity) AS TotalSales
FROM Employee e
LEFT JOIN Customer c ON e.EmployeeId = c.SupportRepId
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId
LEFT JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
GROUP BY e.EmployeeId, e.FirstName, e.LastName
ORDER BY TotalSales DESC;
```

#### Customer Analysis

```sql
-- INSERT query for customer_analysis
INSERT OVERWRITE DIRECTORY '/user/hive/output/customer_analysis'
SELECT
    c.CustomerId,
    c.FirstName || ' ' || c.LastName AS CustomerName,
    COUNT(i.InvoiceId) AS NumberOfPurchases
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId
GROUP BY c.CustomerId, c.FirstName, c.LastName
ORDER BY NumberOfPurchases DESC
LIMIT 1;
```

#### Playlist Analysis

```sql
-- INSERT query for playlist_analysis
INSERT OVERWRITE DIRECTORY '/user/hive/output/playlist_analysis'
SELECT
    p.PlaylistId,
    p.Name AS PlaylistName,
    COUNT(pt.TrackId) AS NumberOfTracks
FROM Playlist p
JOIN PlaylistTrack pt ON p.PlaylistId = pt.PlaylistId
GROUP BY p.PlaylistId, p.Name
ORDER BY NumberOfTracks DESC
LIMIT 1;
```

#### Invoice Analysis

```sql
-- INSERT query for invoice_analysis
INSERT OVERWRITE DIRECTORY '/user/hive/output/invoice_analysis'
SELECT
    i.InvoiceId,
    i.CustomerId,
    c.FirstName || ' ' || c.LastName AS CustomerName,
    i.InvoiceDate,
    SUM(il.UnitPrice * il.Quantity) AS TotalSales
FROM Invoice i
JOIN Customer c ON i.CustomerId = c.CustomerId
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
GROUP BY i.InvoiceId, i.CustomerId, c.FirstName, c.LastName, i.InvoiceDate
ORDER BY TotalSales DESC;
```

#### Top Billing Countries

```sql
-- INSERT query for top_billing_countries
INSERT OVERWRITE DIRECTORY '/user/hive/output/top_billing_countries'
SELECT
    i.BillingCountry,
    SUM(il.UnitPrice * il.Quantity) AS TotalSales
FROM Invoice i
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
GROUP BY i.BillingCountry
ORDER BY TotalSales DESC
LIMIT 10;
```

#### Sales by Genre

```sql
-- INSERT query for sales_by_genre
INSERT OVERWRITE DIRECTORY '/user/hive/output/sales_by_genre'
SELECT
    g.Name AS Genre,
    SUM(il.UnitPrice * il.Quantity) AS TotalSales
FROM Genre g
JOIN Track t ON g.GenreId = t.GenreId
JOIN InvoiceLine il ON t.TrackId = il.TrackId
GROUP BY g.Name
ORDER BY TotalSales DESC
LIMIT 10;
```

#### Customer Lifetime Value

```sql
-- INSERT query for customer_lifetime_value
INSERT OVERWRITE DIRECTORY '/user/hive/output/customer_lifetime_value'
SELECT
    c.CustomerId,
    c.FirstName || ' ' || c.LastName AS CustomerName,
    SUM(il.UnitPrice * il.Quantity) AS TotalSales
FROM Customer c
JOIN Invoice

 i ON c.CustomerId = i.CustomerId
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
GROUP BY c.CustomerId, c.FirstName, c.LastName
ORDER BY TotalSales DESC
LIMIT 10;
```

#### Popular Artists

```sql
-- INSERT query for popular_artists
INSERT OVERWRITE DIRECTORY '/user/hive/output/popular_artists'
SELECT
    a.Name AS Artist,
    SUM(il.UnitPrice * il.Quantity) AS TotalSales
FROM Artist a
JOIN Album al ON a.ArtistId = al.ArtistId
JOIN Track t ON al.AlbumId = t.AlbumId
JOIN InvoiceLine il ON t.TrackId = il.TrackId
GROUP BY a.Name
ORDER BY TotalSales DESC
LIMIT 10;
```

#### Track Downloads by Country

```sql
-- INSERT query for track_downloads_by_country
INSERT OVERWRITE DIRECTORY '/user/hive/output/track_downloads_by_country'
SELECT
    i.BillingCountry,
    t.Name AS TrackName,
    COUNT(il.TrackId) AS Downloads
FROM Invoice i
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
JOIN Track t ON il.TrackId = t.TrackId
GROUP BY i.BillingCountry, t.Name
ORDER BY Downloads DESC
LIMIT 10;
```

### 4. Results Storage

Data from each query is stored in separate HDFS directories (e.g., `/user/hive/output/top_selling_genres`).

## Instructions for Use

1. Run MySQL commands to set up the database.
2. Use Sqoop to import MySQL tables into Hive.
3. Execute each Hive query to perform analytics.
4. View results in the corresponding HDFS directories.

