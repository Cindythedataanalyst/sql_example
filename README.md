# sql_example
A collection of my sample SQL files.  
--Target question: How do annual members and casual riders use Cyclistic bikes differently?

--Import the last 12 months(202302-202401) bike share history.

--Preview data.

Select *
From dbo.[202401-divvy-tripdata];

Select *
From dbo.[202302-divvy-tripdata];

--Combine all the data into one table by using CTE and union all function.

WITH joined_data AS (
SELECT *, '2303' AS source_table FROM Casestudy.dbo.[202303-divvy-tripdata]
UNION ALL
SELECT *, '2304' AS source_table FROM Casestudy.dbo.[202304-divvy-tripdata]
UNION ALL
SELECT *, '2305' AS source_table FROM Casestudy.dbo.[202305-divvy-tripdata]
UNION ALL
SELECT *, '2306' AS source_table FROM Casestudy.dbo.[202306-divvy-tripdata]
UNION ALL
SELECT *, '2307' AS source_table FROM Casestudy.dbo.[202307-divvy-tripdata]
UNION ALL
SELECT *, '2308' AS source_table FROM Casestudy.dbo.[202308-divvy-tripdata]
UNION ALL
SELECT *, '2310' AS source_table FROM Casestudy.dbo.[202310-divvy-tripdata]
UNION ALL
SELECT *, '2311' AS source_table FROM Casestudy.dbo.[202311-divvy-tripdata]
UNION ALL
SELECT *, '2312' AS source_table FROM Casestudy.dbo.[202312-divvy-tripdata]
UNION ALL
SELECT *, '2302' AS source_table FROM Casestudy.dbo.[202302-divvy-tripdata]
UNION ALL
SELECT *, '2309' AS source_table FROM Casestudy.dbo.[202309-divvy-tripdata]
UNION ALL
SELECT *, '2401' AS source_table FROM Casestudy.dbo.[202401-divvy-tripdata]
)
SELECT * INTO divvy_tripdata FROM joined_data;

--Preview divvy_tripdata.

Select *
From divvy_tripdata;

--Start analyzing the data.

--Preview data type before calculation.

EXEC sp_help 'dbo.divvy_tripdata';

--Convert data type before calculation.
--Convert day_of_week int, ride length minutes decimal.

Alter table dbo.divvy_tripdata
Alter Column day_of_week int;

Select CAST(ride_length_minutes AS decimal(10,2)) As ride_length
From divvy_tripdata;

Alter table dbo.divvy_tripdata
Add ride_length decimal(10,2);

Update dbo.divvy_tripdata
Set ride_length = CAST(ride_length_minutes AS decimal(10,2));

--Select data we are going to be using.

--Looking at the frequency of rides for each group,
--to see if one group tends to use the bikes more frequently than the other.

SELECT 
    member_casual,
    COUNT(*) AS ride_count
FROM 
    [Casestudy].[dbo].[divvy_tripdata]
GROUP BY 
    member_casual;

--Analyzing difference in average ride length between members and casual riders.

SELECT 
    member_casual,
    COUNT(*) AS total_rides,
    AVG(ride_length)
	AS average_ride_length_minutes
FROM dbo.divvy_tripdata
GROUP BY 
    member_casual;

--Analyzing the percentage of members and casual riders.

Select 
	member_casual,
	COUNT (member_casual) AS member_casual_count
From dbo.divvy_tripdata
Group by
	member_casual;

SELECT 
    member_casual,
    COUNT(*) AS member_casual_count,
    (COUNT(*) * 100.0 / SUM(COUNT(*)) OVER ()) AS percentage
FROM dbo.divvy_tripdata
GROUP BY member_casual;

--Analyzing the average ride length on different days of the week for both member and casual riders.

Select 
	member_casual,
	COUNT (member_casual) AS member_casual_count,
	day_of_week
From [Casestudy].[dbo].[divvy_tripdata]
Group by
	member_casual,
	day_of_week
Order by day_of_week;

--Analyzing the longest ride length, and the shortest ride length.

Select 
	member_casual,
	max (ride_length) AS longest_ride_length,
	min (ride_length) as shortest_ride_length
From [Casestudy].[dbo].[divvy_tripdata]
Group by
	member_casual;

--Analyzing the difference of rideable type between members and casual riders.

Select 
	member_casual,
	rideable_type,
	Count (*) As ride_count
From [Casestudy].[dbo].[divvy_tripdata]
Group by
	member_casual,
	rideable_type;

--Trying to get rid of repeated ride type rows.

SELECT 
    member_casual,
    SUM(CASE WHEN rideable_type = 'classic_bike' THEN 1 ELSE 0 END) AS bike_count,
    SUM(CASE WHEN rideable_type = 'electric_bike' THEN 1 ELSE 0 END) AS ebike_count,
    SUM(CASE WHEN rideable_type = 'docked_bike' THEN 1 ELSE 0 END) AS docked_bike_count
FROM 
    [Casestudy].[dbo].[divvy_tripdata]
GROUP BY 
    member_casual,
	rideable_type;

SELECT 
    member_casual,
    SUM(CASE WHEN rideable_type = 'classic_bike' THEN 1 ELSE 0 END) AS bike_count,
    SUM(CASE WHEN rideable_type = 'electric_bike' THEN 1 ELSE 0 END) AS ebike_count,
    SUM(CASE WHEN rideable_type = 'docked_bike' THEN 1 ELSE 0 END) AS docked_bike_count
FROM 
    [Casestudy].[dbo].[divvy_tripdata]
GROUP BY 
    member_casual
HAVING 
    SUM(CASE WHEN rideable_type = 'classic_bike' THEN 1 ELSE 0 END) > 0
    OR SUM(CASE WHEN rideable_type = 'electric_bike' THEN 1 ELSE 0 END) > 0
    OR SUM(CASE WHEN rideable_type = 'docked_bike' THEN 1 ELSE 0 END) > 0;

--Add ride year month to calculate the difference in months.

Alter table [Casestudy].[dbo].[divvy_tripdata]
Add ride_year varchar (4);

UPDATE [Casestudy].[dbo].[divvy_tripdata]
SET ride_year = SUBSTRING(CONVERT(VARCHAR, ride_date, 120), 1, 4);

Select top 100 *
From [Casestudy].[dbo].[divvy_tripdata];

--Delete unused columns.

Alter table [Casestudy].[dbo].[divvy_tripdata]
Drop Column source_table,
Drop Column ride_date;

--Analyzing the longest and shortest ride lengths of the month.

Select 
	member_casual,
	ride_year,
	max (ride_length) AS longest_ride_length,
	min (ride_length) as shortest_ride_length
From [Casestudy].[dbo].[divvy_tripdata]
Group by
	member_casual,
	ride_year
Order by
	ride_year;

--Analyzing the total ride length by month for both member and casual riders.

Select 
	member_casual,
	ride_year,
	SUM (ride_length) AS total_ride_length
From [Casestudy].[dbo].[divvy_tripdata]
Group by
	member_casual,
	ride_year
Order by
	ride_year;

--Analyzing the difference in average ride length between members and casual riders across different months.

Select 
	member_casual,
	ride_year,
	avg (ride_length) AS average_ride_length
From [Casestudy].[dbo].[divvy_tripdata]
Group by
	member_casual,
	ride_year
Order by
	ride_year;

--Displaying the top 3 months with the longest average ride length for both members and casual riders.

ALTER TABLE Casestudy.dbo.divvy_tripdata
ADD ride_year_month varchar (4);

UPDATE Casestudy.dbo.divvy_tripdata
SET ride_year_month = ride_year;

Select top 100 *
From [Casestudy].[dbo].[divvy_tripdata];

ALTER TABLE Casestudy.dbo.divvy_tripdata
DROP COLUMN ride_year;

SELECT 
--TOP 3
    member_casual,
    ride_year_month,
    AVG(ride_length) AS average_ride_length
FROM [Casestudy].[dbo].[divvy_tripdata]
GROUP BY
    member_casual,
    ride_year_month
ORDER BY
    average_ride_length DESC;

--Calculating the average ride length in different days of the week between members and casual riders.

Select 
	day_of_week,
	member_casual,
	avg(ride_length) as average_ride_length
FROM 
	[Casestudy].[dbo].[divvy_tripdata]
Group By
	day_of_week,
	member_casual
Order by 
	day_of_week;

--Analyzing ride patterns during weekdays vs. weekends by segmentation by time periods.

SELECT
    CASE
        WHEN day_of_week = 1 THEN 'Sunday'
        WHEN day_of_week = 2 THEN 'Monday'
        WHEN day_of_week = 3 THEN 'Tuesday'
        WHEN day_of_week = 4 THEN 'Wednesday'
        WHEN day_of_week = 5 THEN 'Thursday'
        WHEN day_of_week = 6 THEN 'Friday'
        WHEN day_of_week = 7 THEN 'Saturday'
    END AS day_of_week_name,
    COUNT(*) AS total_rides
FROM
    [Casestudy].[dbo].[divvy_tripdata]
GROUP BY
    CASE
        WHEN day_of_week IN (1, 7) THEN 'Weekend'
        ELSE 'Weekday'
    END,
    day_of_week; 
