CREATE DATABASE transportation_project;
USE transportation_project;

SELECT * FROM trips;
SELECT * FROM trips_details;
SELECT * FROM payment;
SELECT * FROM loc;
SELECT * FROM duration;

--Q1. TOTAL TRIPS
SELECT COUNT(DISTINCT tripid) FROM trips_details;

--Q2. TOTAL DRIVERS
SELECT COUNT(DISTINCT driverid) AS Total_Drivers FROM trips;

--Q3. TOTAL EARNINGS
SELECT SUM(fare) AS Total_Earning FROM trips;

--Q4. TOTAL COMPLETED TRIPS
SELECT COUNT(DISTINCT tripid) AS Completed_Trips FROM trips;

--Q5. TOTAL SEARCHES
SELECT SUM(searches) AS Total_Searches FROM trips_details;

--Q6. TOTAL SEARCHES WHICH GOT ESTIMATE
SELECT SUM(searches_got_estimate) AS Estimate_Search FROM trips_details;

--Q7. TOTAL SEARCHES FOR QUOTES
SELECT SUM(searches_for_quotes) AS Quotes_For_Search FROM trips_details;

--Q8. TOTAL SEARCHES GOT QUOTES
SELECT SUM(searches_got_quotes) AS Quotes_Got_Search FROM trips_details;

--Q9. TOTAL TRIPS CANCELLED BY DRIVERS
SELECT COUNT(*)-SUM(driver_not_cancelled) AS Trips_Cancelled_By_Drivers FROM trips_details;

--Q10. TOTAL OTP ENTERED
SELECT SUM(otp_entered) AS Otp_Entered FROM trips_details;

--Q11. TOTAL OTP ENTERED
SELECT SUM(end_ride) AS Otp_Entered FROM trips_details;

--Q12. AVERAGE DISTANCE PER TRIP
SELECT SUM(distance)/COUNT(tripid) AS Avg_Dist_Per_Trip FROM trips;
--OR
SELECT AVG (distance) AS Avg_Dist_Per_Trip FROM trips;

--Q13. AVERAGE FARE PER TRIP
SELECT SUM(fare)/COUNT(tripid) AS Avg_Fare_Per_Trip FROM trips;
--OR
SELECT AVG (fare) AS Avg_Fare_Per_Trip FROM trips;

--Q14. TOTAL DISTANCE TRAVELLED
SELECT SUM(distance) AS Total_Distance_Travelled FROM trips;

--Q15. MOST USED PAYMENT METHOD
SELECT  P.method, T.faremethod
FROM TRIPS T
INNER JOIN PAYMENT AS P
ON T.faremethod = P.id
WHERE P.id = (SELECT MAX(faremethod) AS MOST_USED_PAYMENT_METHOD FROM trips)
GROUP BY P.method, T.faremethod;
--OR
SELECT method FROM payment
WHERE id = (SELECT MAX(faremethod) AS MOST_USED_PAYMENT_METHOD FROM trips);
--OR
SELECT P.method, Q.faremethod, Q.CNT 
FROM payment AS P
INNER JOIN (
SELECT TOP 1 faremethod, COUNT(tripid) AS CNT FROM trips
GROUP BY faremethod
ORDER BY CNT DESC
) AS Q
ON Q.faremethod = P.id

--Q16. THE HIGHEST PAYMENT WAS MADE THROUGH WHICH INSTRUMENT
SELECT TOP 1 fare,faremethod FROM trips
ORDER BY fare DESC;
--OR
SELECT TOP 1 Q.fare, Q.faremethod, P.method FROM payment AS P
INNER JOIN trips AS Q
ON P.id = Q.faremethod
WHERE P.id = (SELECT TOP 1 faremethod FROM trips
ORDER BY fare DESC)
GROUP BY Q.fare,Q.faremethod,P.method
ORDER BY Q.fare DESC;

--Q17. WHICH TWO LOCATIONS HAD THE MOST TRIPS
SELECT * 
FROM
(SELECT *, DENSE_RANK() OVER(ORDER BY Most_Trip DESC) AS Rank_no
FROM
(SELECT loc_from, loc_to, COUNT(tripid) AS Most_Trip FROM trips
GROUP BY loc_from, loc_to
)AS A
)AS B
WHERE Rank_no = 1; 

--Q18. TOP 5 EARNING DRIVER
SELECT TOP 5 driverid, SUM(fare) AS earning FROM trips
GROUP BY driverid
ORDER BY earning DESC;
--OR
SELECT * 
FROM
(SELECT *, DENSE_RANK() OVER(ORDER BY Earning DESC) AS Rank_no
FROM
(SELECT driverid, SUM(fare) AS Earning FROM trips
GROUP BY driverid
)AS A
)AS B
WHERE Rank_no < 6;

--Q19. WHICH DURATION HAD MORE TRIPS
SELECT * FROM
(SELECT *, DENSE_RANK() OVER(ORDER BY Most_Duration DESC) AS Rank_No FROM
(SELECT duration, COUNT( DISTINCT tripid) AS Most_Duration FROM trips
GROUP BY duration
)AS A) AS B
WHERE Rank_No = 1;

--Q20. WHICH DRIVER:CUSTOMER PAIR HAD MOST ORDERS
SELECT * FROM
(SELECT *, DENSE_RANK() OVER(ORDER BY Common DESC) AS R_No FROM
(SELECT driverid, custid, COUNT(DISTINCT tripid) AS Common FROM trips
GROUP BY driverid, custid)AS A)AS B
WHERE R_No = 1;
--OR
SELECT driverid, custid, Common
FROM (SELECT driverid, custid, COUNT(*) AS Common, DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS R_No FROM trips
GROUP BY driverid, custid)AS A
WHERE R_No = 1;
--OR USING CTE
WITH DriverCustomerTrips AS
(SELECT driverid, custid, COUNT(*) AS Common, DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS R_No FROM trips
GROUP BY driverid, custid)
SELECT driverid, custid, Common
FROM DriverCustomerTrips
WHERE R_No = 1;
--OR USING HAVING
SELECT driverid, custid, MAX(trip_count) AS max_trip_count
FROM (SELECT driverid, custid, COUNT(*) AS trip_count
FROM trips
GROUP BY driverid, custid) AS Subquery
GROUP BY driverid, custid
HAVING MAX(trip_count) = (SELECT MAX(trip_count) FROM (SELECT driverid, custid, COUNT(*) AS trip_count
FROM trips
GROUP BY driverid, custid) AS Subquery);
--OR USING WHERE
SELECT driverid, custid, trip_count
FROM (SELECT driverid, custid, COUNT(*) AS trip_count 
FROM trips
GROUP BY driverid, custid) AS Subquery
WHERE trip_count = (SELECT MAX(trip_count) FROM (SELECT driverid, custid, COUNT(*) AS trip_count 
FROM trips GROUP BY driverid, custid) AS MaxSubquery);

--Q21. SEARCH TO ESTIMATE RATES
SELECT SUM(searches_got_estimate)*1.0/SUM(searches) AS ESTIMATED_RATES FROM trips_details;

--Q22. ESTIMATE TO SEARCH FOR QUOTE RATES
SELECT SUM(searches_for_quotes)*1.0/SUM(searches) AS SEARCH_QUOTES_RATES FROM trips_details;

--Q23. QUOTE ACCEPTANCE RATE
SELECT SUM(searches_got_quotes)*1.0/SUM(searches) AS QUOTES_ACCEPTANCE_RATES FROM trips_details;

--Q24. QUOTE TO BOOKING RATE
SELECT SUM(customer_not_cancelled)*1.0/SUM(searches) AS BOOKING_RATES FROM trips_details;

--Q25. BOOKING CANCELLATION RATE
SELECT (COUNT(*)-SUM(customer_not_cancelled))*1.0/SUM(searches) AS BOOKING_RATES FROM trips_details;

--Q26. WHICH AREA GOT HIGHEST TRIPS IN WHICH DURATION
SELECT TOP 1 COUNT(DISTINCT tripid) AS MAX_TRIPS, loc_from, duration FROM trips
GROUP BY duration, loc_from
ORDER BY MAX_TRIPS DESC;

--Q27. WHICH AREA GOT THE HIGHEST FARES
SELECT loc_from, SUM(fare) AS Highest_Fare FROM trips
GROUP BY loc_from
ORDER BY Highest_Fare DESC;

--Q28. WHICH AREA GOT THE HIGHEST CANCELLATIONS
SELECT loc_from, COUNT(*) - SUM(otp_entered) AS Highest_Cancellations FROM trips_details
GROUP BY loc_from
ORDER BY Highest_Cancellations DESC;

--Q29. WHICH AREA GOT THE HIGHEST TRIPS
SELECT loc_from, COUNT(DISTINCT tripid) Trip_Count FROM trips
GROUP BY loc_from
ORDER BY Trip_Count DESC;

--Q30. WHICH DURATION GOT THE HIGHEST TRIPS
SELECT duration, COUNT(DISTINCT tripid) Trip_Count FROM trips
GROUP BY duration
ORDER BY Trip_Count DESC;

--Q31. WHICH DURATION GOT THE HIGHEST FARES 
SELECT duration, SUM(fare) AS Highest_Fare FROM trips
GROUP BY duration
ORDER BY Highest_Fare DESC;