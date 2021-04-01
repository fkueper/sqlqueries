# SQL querying

This file contains queries solving question raised in a database challenge on codeacademy.

Basic Requirements

Which tracks appeared in the most playlists? how many playlist did they appear in?

	  SELECT t.name track, COUNT(pt.trackid) trackfrequency, COUNT(pt.playlistid) playlists
	  FROM tracks t
	  JOIN playlist_track pt
	   ON t.trackid=pt.trackid
	  GROUP BY pt.trackid
	  ORDER BY playlists DESC;	

Which track generated the most revenue? which album? which genre?
	
    SELECT t.name track, SUM(ii.unitprice*ii.quantity) revenue
	  FROM tracks t
	  JOIN invoice_items ii
	   ON t.trackid=ii.trackid
	  GROUP BY t.trackid
    ORDER BY revenue DESC;

	  SELECT a.title album, SUM(ii.unitprice*ii.quantity) revenue
	  FROM tracks t
	  JOIN invoice_items ii
	   ON t.trackid=ii.trackid
	  JOIN albums a 
	   ON a.albumid=t.albumid
	  GROUP BY a.albumid
	  ORDER BY revenue DESC;

	  SELECT g.name genre, SUM(ii.unitprice*ii.quantity) revenue
	  FROM tracks t
	  JOIN invoice_items ii
	   ON t.trackid=ii.trackid
	  JOIN genres g 
	   ON g.genreid=t.genreid
	  GROUP BY g.genreid
	  ORDER BY revenue DESC;


Which countries have the highest sales revenue? What percent of total revenue does each country make up?

    SELECT i.billingcountry country, ROUND(SUM(ii.unitprice*ii.quantity),2) revenue
	  FROM invoices i
    JOIN invoice_items ii
	   ON i.invoiceid=ii.invoiceid
	  GROUP BY country
	  ORDER BY revenue DESC;
	
	  WITH total_revenue(totalrev) AS (
		  SELECT SUM(ii.unitprice*ii.quantity) revenue
    	  	FROM invoice_items ii 
		  )
	  SELECT i.billingcountry country, ROUND(SUM(ii.unitprice*ii.quantity),2) revenue, 
		  ROUND(SUM(ii.unitprice*ii.quantity)/totalrev,2) revenue_share
	  FROM invoices i, total_revenue
	  JOIN invoice_items ii
     ON i.invoiceid=ii.invoiceid
	  GROUP BY  country
	  ORDER BY  revenue DESC;

How many customers did each employee support, what is the average revenue for each sale, and what is their total sale?
Additional Challenges

	  SELECT e.firstname,
         	e.lastname,
         	COUNT(DISTINCT c.customerid) customers,
         	ROUND(SUM(i.total),2) total_sales,
         	ROUND(AVG(i.total),2) average_sales
	  FROM invoices i
	  JOIN customers c
     ON i.customerid = c.customerid
	  JOIN employees e
	  ON e.employeeid = c.supportrepid
	  GROUP BY  e.employeeid;
	

Intermediate Challenge

Do longer or shorter length albums tend to generate more revenue?
Hint: We can use the WITH clause to create a temporary table that determines the number of tracks in each album, then group by the length of the album to compare the average revenue generated for each.

     WITH full_query AS(
      WITH track_value AS (
        SELECT TrackId, SUM(UnitPrice) AS "SumPrice" 
        FROM invoice_items
        GROUP BY invoice_items.TrackId
        ), 
      album_length AS (
        SELECT AlbumId, COUNT(*)
        FROM tracks
        GROUP BY AlbumId
        ) 
      SELECT tracks.AlbumId AS AlbumNo, SUM(track_value.SumPrice) AS "AlbumValue", COUNT(*) AS AlbumLength FROM tracks
      LEFT JOIN track_value ON tracks.TrackId = track_value.TrackId
      GROUP BY tracks.AlbumId)
      SELECT AlbumNo, AVG(AlbumValue), AlbumLength FROM full_query
      GROUP BY AlbumLength
      ORDER BY AVG(AlbumValue) DESC;

Is the number of times a track appear in any playlist a good indicator of sales?
Hint: We can use the WITH clause to create a temporary table that determines the number of times each track appears in a playlist, then group by the number of times to compare the average revenue generated for each.

      WITH playlist_numbers AS (
        SELECT TrackId, COUNT(*) AS TrackCount FROM playlist_track
        GROUP BY TrackId
        )
      SELECT playlist_numbers.TrackCount AS "PlaylistAppearances", AVG(invoice_items.UnitPrice) AS "Sales" FROM invoice_items
      INNER JOIN playlist_numbers ON invoice_items.TrackId = playlist_numbers.TrackId
      GROUP BY playlist_numbers.TrackCount
      ORDER BY Sales DESC;


Advanced Challenge

How much revenue is generated each year, and what is its percent change 51 from the previous year?
Hint: The InvoiceDate field is formatted as ‘yyyy-mm-dd hh:mm:ss’. Try taking a look at using the strftime() function to help extract just the year. Then, we can use a subquery in the SELECT statement to query the total revenue from the previous year. Remember that strftime() returns the date as a string, so we would need to CAST it to an integer type for this part. Finally, since we cannot refer to a column alias in the SELECT statement, it may be useful to use the WITH clause to query the previous year total in a temporary table, and then calculate the percent change in the final SELECT statement.

    WITH current_year AS (
      SELECT strftime('%Y',invoices.InvoiceDate) AS CurrentYear, SUM(invoice_items.UnitPrice) AS YearSales FROM invoices
      INNER JOIN invoice_items ON invoices.InvoiceId = invoice_items.InvoiceId
      GROUP BY CurrentYear),
      previous_year AS (SELECT strftime('%Y',invoices.InvoiceDate) + 1 AS PreviousYear, SUM(invoice_items.UnitPrice) AS YearSales FROM invoices
      INNER JOIN invoice_items ON invoices.InvoiceId = invoice_items.InvoiceId
      GROUP BY PreviousYear)
    SELECT current_year.CurrentYear, current_year.YearSales, ((current_year.YearSales-previous_year.YearSales)/previous_year.YearSales * 100) AS PreviousSales 
    FROM current_year
    LEFT JOIN previous_year ON CAST(current_year.CurrentYear AS INTEGER) = CAST(previous_year.PreviousYear AS INTEGER);
