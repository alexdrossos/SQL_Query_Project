# Project 1: Query Project

- As part of last quarter's retrospective, increasing ridership was identified as the main priority for our Bay Wheels division over the next few months. The business strategy team has already determined financially that offering certain deals through the mobile application would be the most successful tactic in doing this. The questions posed to our data science team were:

    * What are the 5 most popular trips that you would call "commuter trips"? 
  
    * What are your recommendations for offers (justify based on your findings)?
  
--- 

## Part 1: Initial Exploration

- To answer these business-driven questions, we will be running data analysis through Google Cloud Platform using public datasets. To begin to understand the data we're working with, some initial data exploration was conducted with the following outputs.

    * Question 1: What's the size of this dataset? (i.e., how many trips)
    ```
    SELECT COUNT (DISTINCT trip_id) AS Num_Unique_Trips  
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`;
    ```
    | Column                | Value       |
    | --------------------- | ----------- |
    | Num_Unique_Trips      | 983648      |

    From the output of this query, the size of the dataset in terms of the number of unique trips, based on id, is 983648.
    
    * Question 2: What is the earliest start date and time and latest end date and time for a trip?
    ```
    SELECT MIN(start_date) AS earliest_start, MAX(end_date) AS latest_end  
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`;
    ```
    | Column                | Value                            |
    | --------------------- | ---------------------------------|
    | earliest_start        | 2013-08-29 09:08:00 UTC          |
    | latest_end            | 2016-08-31 23:48:00 UTC          |

    From the output of the above query, we can see that the earliest start date/time is August 29th, 2013 at 9:08AM and the latest end date/time for a trip is August 31st, 2016 11:48PM
    
    * Question 3: How many bikes are there?
    ```
    SELECT COUNT (DISTINCT bike_number) AS Num_Bikes  
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`;
    ```
    | Column                | Value       |
    | --------------------- | ----------- |
    | Num_Bikes             | 700         |

    From the output of the above query, we can see that the total number of bikes used, through getting the distinct count of 'bike_number', is 700 bikes.
    
- After getting a good understanding of the public bikeshare_trips dataset, we did some additional exploration to start understanding typical rider behavior.

    * Which stations have the most trips > 30 minutes taken from them (top 5)?
    ```
    SELECT COUNT (DISTINCT trip_id) AS count_trips, start_station_name  
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    WHERE duration_sec > 2000  
    GROUP BY start_station_name 
    ORDER BY count_trips DESC
    LIMIT 5;
    ```
    | count_trips           | start_station_name                   |
    | --------------------- | -------------------------------------|
    | 4666                  | Harry Bridges Plaza (Ferry Building) |
    | 3529                  | Embarcadero at Sansome               |
    | 1900                  | Market at 4th                        |
    | 1665                  | Powell Street BART                   |
    | 1652                  | Embarcadero at Vallejo               |
    
    We were intersted in this query because we're assuming that trips longer than 30 minutes are leisure trips rather than commuter trips. This would be important because these customers are more likely to be one to three day pass holders rather than owners of a monthly or annual membership. However, after seeing the results of this query, some of these stations could actually be commuter locations, such as the BART station and those located in the financial district. These results disproved our initial thought that commuter trips were likely less than 30 minutes. We cannot make that assumption when conducting the remainder of our analysis. 
    
    * What subscriber type generates the most amount of trips?
    ```
    SELECT COUNT (DISTINCT trip_id) AS count_trips, subscriber_type  
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`  
    GROUP BY subscriber_type;
    ```
    | count_trips           | subscriber_type    |
    | --------------------- | -------------------|
    | 136809                | Customer           |
    | 846839                | Subscriber         |
    
    In order to determine which subcriber type we should focus our promotions on, we needed to determine which subscriber type was making up the biggest portion of the market. In the bikeshare_trips dataset, a 'Customer' value in the subscriber_type column refers to a rider who has either purchased a one day or three day pass. Whereas, a 'Subscriber' refers to a rider who has purchased a monthly or annual membership. However, we have no data on whether they are using a corporate or university discount with their membership. We do know that both subscriber types descibed here have registered in the mobile app and are eligible to recieve the promotions that we decide on. The results of the query show that from the total of 983,648 trips, 846,839 were taken by riders with memberships, a large majority. This tells that the customer sector of the business, selling one-day and three-day passes, could use a substantial rise in ridership.
    
    * What's the count of trips that started and ended in the same station?
    ```
    SELECT COUNT (DISTINCT trip_id) AS count_trips, subscriber_type  
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`  
    WHERE start_station_name = end_station_name
    GROUP BY subscriber_type;
    ```
    | subscriber_type       | count_trips   |
    | --------------------- | --------------|
    | Customer              | 22153         |
    | Subscriber            | 9894          |
    
    This query again was an attempt to quantify the number of pure leisure trips that are taken. To do this, we got the count of trips that started and ended at the same location. While of course this specification does not encompass *all* leisure trips, it's definitely a portion of them. This query did show us that these types of trips only make up around 3% of the total, 32,047. Most of these, unsuprisingly, are taken by pass holders rather than members, but it also confirms that the leisure trip space needs attention and promotion. 
    
--- 

## Part 2: Further Exploration and Recommendation Development

- Now that we have bearings on the data at our disposal and some initial insights on customer behaviors, we can continue our exploration, but this time through the CLI. For practice, first we will re-run the previous three queries from Part 1 in the CLI.
    * What's the size of this dataset? (i.e., how many trips)
    ```
    bq query --use_legacy_sql=false '
    SELECT COUNT (DISTINCT trip_id) AS Num_Unique_Trips  
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`;'
    ```
    
    | Num_Unique_Trips |
    | -----------------|
    | 983648           |

    * What is the earliest start date and time and latest end date and time for a trip?
    ```
    bq query --use_legacy_sql=false '
    SELECT MIN(start_date) AS earliest_start, MAX(end_date) AS latest_end  
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`;'
    ```
   
    | earliest_start         | latest_end                  |
    | -----------------------| ----------------------------|
    | 2013-08-29 09:08:00    | 2016-08-31 23:48:00         |
    
    * How many bikes are there?
    ```
    bq query --use_legacy_sql=false '
    SELECT COUNT (DISTINCT bike_number) AS Num_Bikes  
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`;'
    ```
    | Num_Bikes |
    | ----------|
    | 700       |
    
    * New Query: How many trips are in the morning vs in the afternoon?
    ```
    bq query --use_legacy_sql=false '
    CREATE VIEW Project_1.trip_times AS
    SELECT trip_id, EXTRACT(TIME FROM start_date AT TIME ZONE "UTC") AS trip_time, subscriber_type, zip_code, duration_sec, start_station_id 
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`;'
    ```
    ```
    bq query --use_legacy_sql=false '
    WITH Input AS (
    SELECT trip_id, CASE WHEN trip_time BETWEEN "05:00:00" AND "11:59:59" THEN "Morning"
    WHEN trip_time BETWEEN "12:00:00" AND "16:00:00" THEN "Afternoon"
    ELSE "Evening"
    END AS time_of_day
    FROM `alien-house-324100.Project_1.trip_times`
    )
    SELECT time_of_day, COUNT (DISTINCT trip_id) AS num_trips
    FROM Input
    WHERE time_of_day = "Morning" OR time_of_day = "Afternoon"
    GROUP BY time_of_day;'
    ```
    | time_of_day         | num_trips |
    | --------------------| ----------|
    | Morning             | 404919    |
    | Afternoon           | 177293    |
    
    To answer this question, first I created a view with the trip id values and the trip times extracted from the TIMESTAMP start_date column. After creating the view, I queried from it by creating a case statement column that delineated the time of day from the extracted trip_time column. From the   question it seems they only care about trips in morning and afternoon, not evening. Given this, I made the  assumption that morning trips were considered to be between 5AM and Noon, and afternoon trips to be between Noon and 4PM. After that I took the count of unique trip ids grouped by these times of day, leaving out any trips in the evening time. The results I got were that the total number of morning trips was 404,919 and the total number of afternoon trips was 177293.
    
- What last questions to we need answers to in order to develop our final findings and recommendations? After strategizing collectively, we deemed these the most important unknowns and ran analysis to get answers. To answer the first of the two key questions, "what are the 5 most popular trips that you would call commuter trips?", we searched the following:

    * What's the most popular route for trips between 7:45 AM to 8:45 AM starting at Caltrain stations with memberships? 
    ```
    bq query --use_legacy_sql=false '
    WITH Input AS (
    SELECT trip_id, CONCAT(start_station_id, '-', end_station_id) AS route_id,
    CONCAT(start_station_name, '-', end_station_name) AS route_name,
    EXTRACT(TIME FROM start_date AT TIME ZONE "UTC") AS time
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    WHERE start_station_name LIKE '%Caltrain%' 
    AND subscriber_type = 'Subscriber'
    )
    SELECT COUNT(DISTINCT trip_id) AS route_count, route_name
    FROM Input
    WHERE time BETWEEN '07:45:00' AND '08:45:00'
    GROUP BY route_id, route_name
    ORDER BY route_count DESC
    LIMIT 1;'
    ```
    
    | route_count | route_name                                                                    |
    | ------------| ------------------------------------------------------------------------------|
    | 1795        | San Francisco Caltrain (Townsend at 4th)-Harry Bridges Plaza (Ferry Building) |
    
    Caltrain stations would be ideal morning destinations for commuters coming in from different parts of the city. To fitler the data further, we're assuming that commuters would have some kind of membership and that they would occur between 7:45AM and 8:45AM. From the results of the query, the most common corporate morning commute trip is from the Townsend Caltrain station to the Ferry Building station. 
    
    * What's the most popular route for trips between 4:30 PM to 6:00 PM ending at Caltrain stations with memberships? 
    ```
    bq query --use_legacy_sql=false '
    WITH Input AS (
    SELECT trip_id, CONCAT(start_station_id, '-', end_station_id) AS route_id,
    CONCAT(start_station_name, '-', end_station_name) AS route_name,
    EXTRACT(TIME FROM start_date AT TIME ZONE "UTC") AS time
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    WHERE end_station_name LIKE '%Caltrain%' 
    AND subscriber_type = 'Subscriber'
    )
    SELECT COUNT(DISTINCT trip_id) AS route_count, route_name 
    FROM Input
    WHERE time BETWEEN '16:30:00' AND '18:00:00'
    GROUP BY route_id, route_name
    ORDER BY route_count DESC
    LIMIT 1;'
    ```  
    
    | route_count | route_name                                                                    |
    | ------------| ------------------------------------------------------------------------------|
    | 2739        | Embarcadero at Folsom-San Francisco Caltrain (Townsend at 4th)                |
    
    From the output of this query, the most popular evening commute trip is from Embarcadero at Folsom to the Townsend Caltrain station, likely taken by commuters that live in more distanced parts of the city.

    * What's the most popular route for trips between 7:45 AM to 9:00 AM ending in the financial district with memberships?
    ```
    bq query --use_legacy_sql=false '
    WITH Input AS (
    SELECT trip_id, CONCAT(start_station_id, '-', end_station_id) AS route_id,
    CONCAT(start_station_name, '-', end_station_name) AS route_name,
    EXTRACT(TIME FROM end_date AT TIME ZONE "UTC") AS end_time
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    WHERE end_station_id IN (41, 42, 45, 46, 48, 82)
    AND subscriber_type = 'Subscriber'
    )
    SELECT COUNT(DISTINCT trip_id) AS route_count, route_name 
    FROM Input
    WHERE end_time BETWEEN '08:30:00' AND '09:15:00'
    GROUP BY route_id, route_name
    ORDER BY route_count DESC
    LIMIT 1;'
    ```
    
    | route_count | route_name                                                                    |
    | ------------| ------------------------------------------------------------------------------|
    | 2739        | Beale at Market-Commercial at Montgomery                                      |
    
    Another type of commuter trip that we considered was trips that end in the financial district, because that particular area of the city has a lot of office buildings. Using the station map online, we determined the IDs of the 6 main stations in that area and filtered the data on those values. From the output of the query, the most popular route for a morning commute trip into the financial district is from Beale at Market to Commercial at Montgomery.
    
    * What's the most popular route for trips between 8:00 AM to 9:30 AM ending at university stations with memberships?  
    ```
    bq query --use_legacy_sql=false '
    WITH Input AS (
    SELECT trip_id, CONCAT(start_station_id, '-', end_station_id) AS route_id,
    CONCAT(start_station_name, '-', end_station_name) AS route_name,
    EXTRACT(TIME FROM start_date AT TIME ZONE "UTC") AS start_time
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    WHERE end_station_id IN (12, 16, 25, 35)
    AND subscriber_type = 'Subscriber'
    )
    SELECT COUNT(DISTINCT trip_id) AS route_count, route_name 
    FROM Input
    WHERE start_time BETWEEN '08:00:00' AND '09:30:00'
    GROUP BY route_id, route_name
    ORDER BY route_count DESC
    LIMIT 1;'
    ```  
    | route_count | route_name                                                                    |
    | ------------| ------------------------------------------------------------------------------|
    | 222         | Santa Clara at Almaden-SJSU 4th at San Carlos                                 |  
    
    University students, in this context, should also be considered commuters. To determine what trips this demographic is taking, we looked up the top 5 busiest stations near universities and found the most popular route for students getting to morning classes. From the output of the query, it appears the most popular trip is students coming from Santa Clara to SJSU.  
    
    * What's the most popular route for trips between 3:00 PM to 5:00PM starting at university station with memberships?
    ```
    bq query --use_legacy_sql=false '
    WITH Input AS (
    SELECT trip_id, CONCAT(start_station_id, '-', end_station_id) AS route_id,
    CONCAT(start_station_name, '-', end_station_name) AS route_name,
    EXTRACT(TIME FROM start_date AT TIME ZONE "UTC") AS start_time
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    WHERE start_station_id IN (12, 16, 25, 35)
    AND subscriber_type = 'Subscriber'
    )
    SELECT COUNT(DISTINCT trip_id) AS route_count, route_name 
    FROM Input
    WHERE start_time BETWEEN '15:00:00' AND '17:00:00'
    GROUP BY route_id, route_name
    ORDER BY route_count DESC
    LIMIT 1;'
    ```
    
    | route_count | route_name                                                                    |
    | ------------| ------------------------------------------------------------------------------|
    | 303         | Stanford in Redwood City-Redwood City Caltrain Station                        |
    
    The most popular afternoon student commuter trip is from the Stanford Campus to the Caltrain Station, likely those trying to get back into the city after class. 
    
- To begin drafting our recommendation for which promotions to offer we performed the following data analysis by answering these questions: 
    1.  Are monthly or annual memberships being taken advantage of by commuters?
        a. Create view that shows the top 10 end station destinations during morning trips (starting 730-9) and < 30 min which are likely commuter trips.
        ```
        bq query --use_legacy_sql=false '
        CREATE VIEW Project_1.commuter_trip_info AS
        SELECT trip_id, end_station_id, end_station_name, subscriber_type, EXTRACT(TIME FROM start_date AT  TIME ZONE "UTC") AS start_time, duration_sec 
        FROM bigquery-public-data.san_francisco.bikeshare_trips;'
        ```
        ```
        bq query --use_legacy_sql=false '
        CREATE VIEW Project_1.top_10_commuter_stations AS
        SELECT end_station_id, end_station_name, COUNT (DISTINCT trip_id) AS num_trips
        FROM alien-house-324100.Project_1.commuter_trip_info
        WHERE start_time BETWEEN "07:30:00" AND "09:00:00"
        AND duration_sec < 1500
        GROUP BY end_station_id, end_station_name
        ORDER BY num_trips DESC
        LIMIT 10;'
        ```
        ```
        bq query --use_legacy_sql=false '
        CREATE VIEW Project_1.commuter_trip_counts AS
        SELECT end_station_id, COUNT (DISTINCT trip_id) as num_trips, subscriber_type
        FROM bigquery-public-data.san_francisco.bikeshare_trips
        WHERE time(CAST(start_date AS datetime)) BETWEEN "07:30:00" AND "09:00:00"
        AND duration_sec < 1500
        GROUP BY end_station_id, subscriber_type
        ORDER BY end_station_id;'
        ```
        ```
        bq query --use_legacy_sql=false '
        SELECT stations.end_station_id, 
        counts.num_trips, counts.subscriber_type, 
        FROM `alien-house-324100.Project_1.top_10_commuter_stations` AS stations
        INNER JOIN `alien-house-324100.Project_1.commuter_trip_counts` AS counts
        ON stations.end_station_id = counts.end_station_id;'
        ```
        
        | end_station_id | num_trips | subscriber_type |
        |----------------|-----------|-----------------|
        |             70 |     13354 | Subscriber      |
        |             70 |       305 | Customer        |
        |             61 |     10127 | Subscriber      |
        |             61 |       170 | Customer        |
        |             65 |      9902 | Subscriber      |
        |             65 |       225 | Customer        |
        |             77 |      8050 | Subscriber      |
        |             77 |       189 | Customer        |
        |             60 |      7324 | Subscriber      |        
        |             60 |       467 | Customer        |
        |             74 |      6606 | Subscriber      |
        |             74 |       231 | Customer        |
        |             51 |      6674 | Subscriber      |
        |             51 |        75 | Customer        |
        |             69 |      6493 | Subscriber      |
        |             69 |       175 | Customer        |
        |             63 |      6145 | Subscriber      |
        |             63 |       110 | Customer        |
        |             50 |      5923 | Subscriber      |
        |             50 |       320 | Customer        |
        
        From this output, we can see that the primary morning commuter stations are all primarily utilized by subscribers rather than customers. What this means is that those we are assuming to be corporate commuters do have memberships. Therefore, we do not need to funnel more money into promoting this option. However, we cannot tell from our data if they are signed up for corporate membership. If we had this data, we could determine if corporate memberships were being utilized and if they weren't we could advise members to look into whether their company offers that subsidy. 
        
    2.  Which stations are most busy at 9AM and 5PM (these are most likely common commuter stations)?
        a. Create view
        ```
        bq query --use_legacy_sql=false '
        CREATE VIEW Project_1.station_avail AS
        WITH Input AS (
        SELECT stations.dockcount, stations.station_id, stations.name, status.docks_available, status.time
        FROM `bigquery-public-data.san_francisco.bikeshare_stations` AS stations
        INNER JOIN `bigquery-public-data.san_francisco.bikeshare_status` AS status
        ON stations.station_id = status.station_id
        )
        SELECT station_id, name, docks_available/dockcount AS percent_avail, time
        FROM Input;'
        ```
        b. Get morning busy stations
         ```
        bq query --use_legacy_sql=false '
        WITH Input AS (
        Select station_id, name, ROUND(percent_avail, 2) AS percent_avail,
        EXTRACT(HOUR FROM time) AS hour
        FROM `alien-house-324100.Project_1.station_avail`
        )
        SELECT station_id, ROUND(AVG(percent_avail), 2) AS percent_avail
        FROM Input
        WHERE hour IN (8,9)
        GROUP BY station_id
        ORDER BY percent_avail ASC
        LIMIT 5;'
        ```
        
        | station_id | percent_avail |
        |------------|---------------|
        |         36 |          0.43 |
        |         73 |          0.45 |
        |          3 |          0.46 |
        |         35 |          0.46 |
        |          7 |          0.47 |
        
        c. Get evening busy stations
        ```
        bq query --use_legacy_sql=false '
        WITH Input AS (
        Select station_id, name, ROUND(percent_avail, 2) AS percent_avail,
        EXTRACT(HOUR FROM time) AS hour
        FROM `alien-house-324100.Project_1.station_avail`
        )
        SELECT station_id, ROUND(AVG(percent_avail), 2) AS percent_avail
        FROM Input
        WHERE hour IN (16,17)
        GROUP BY station_id
        ORDER BY percent_avail ASC
        LIMIT 5;'
        ```
        | station_id | percent_avail |
        |------------|---------------|
        |         70 |          0.43 |
        |         36 |          0.43 |
        |         35 |          0.46 |
        |          3 |          0.46 |
        |         50 |          0.47 |
        
        After determining from question 1 that we should possibly be advertising the corporate membership subsidy to increase ridership, we need to know which riders to target. The above queries produce the busiest stations during the times of 8AM-9AM and 4PM-5PM by bike availability. Here, we're making the assumption that this is due to commuter trips. Using this information, it would likely be helpful to advertise the corporate subsidy to riders checking into these stations at the specified times. 
        
    3.  Are pass customers typically riding for more than 30 min?
        ```
        bq query --use_legacy_sql=false '
        WITH Input AS (
        SELECT duration_sec FROM bigquery-public-data.san_francisco.bikeshare_trips
        WHERE subscriber_type = "Customer"
        )
        SELECT COUNT(*) AS num_trips
        FROM Input
        WHERE duration_sec > 1800;'
        ```
        | num_trips |
        |-----------|
        |     42013 |
        
        We know that there are 136,809 customer ride trips total. 42,013 of them, 30% of the total, are greater than 30 minutes. The reason that this is important is because after pass rides are longer than 30 minutes, they're charged 3 more dollars off the bat, meaning they are paying half the cost of a monthly unlimited membership. Riders that do this should be prompted with an offer promoting the monthly membership and an explanation of the cost breakdown. 
        
    4.  Are most of the trips around universities taken by pass or membership holders?  
        ```
        bq query --use_legacy_sql=false '
        SELECT COUNT (DISTINCT trip_id) AS num_trips, subscriber_type, zip_code
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`
        WHERE time(CAST(start_date AS datetime)) BETWEEN "09:00:00" AND "16:00:00"
        AND duration_sec < 1500
        AND zip_code IN ("94117", "94132", "94305", "94720",
        "94112", "94613", "94618", "95053", "94105", "94704", "90240",
        "94133", "94542", "95192", "95128")
        GROUP BY subscriber_type, zip_code
        ORDER BY zip_code;'
        ```

        | num_trips | subscriber_type | zip_code |
        |-----------|-----------------|----------|
        |     21696 | Subscriber      | 94105    |
        |       617 | Customer        | 94105    |
        |       101 | Customer        | 94112    |
        |      2133 | Subscriber      | 94112    |
        |      5471 | Subscriber      | 94117    |
        |       377 | Customer        | 94117    |
        |      1088 | Subscriber      | 94132    |
        |        55 | Customer        | 94132    |
        |     13875 | Subscriber      | 94133    |
        |       452 | Customer        | 94133    |
        |       217 | Subscriber      | 94305    |
        |        27 | Customer        | 94305    |
        |        62 | Subscriber      | 94542    |
        |         6 | Customer        | 94542    |
        |      1450 | Subscriber      | 94618    |
        |        60 | Customer        | 94618    |
        |        71 | Customer        | 94704    |
        |       959 | Subscriber      | 94704    |
        |        35 | Customer        | 95128    |
        |        58 | Subscriber      | 95128    |

        The output of this query shows that for all of these zip codes, a large majority of the rides are coming from subcribers over one time customers, so memberships are probably being utlitized by these university students. However, I can't tell if they have used the monthly membership or an annual membership so we may want to get additional customer data to see if the annual relationship is being utilized. 
        
    5.  Are members of communities in the lowest income areas taking advantage of our bikes and bike share for all offer?
        a. Find average number of trips taken per zip code registered
        ```
        bq query --use_legacy_sql=false '
        WITH Input AS (SELECT COUNT (DISTINCT trip_id) AS num_trips, zip_code
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`
        WHERE  zip_code BETWEEN "90000" AND "99999"
        GROUP BY zip_code
        ORDER BY num_trips DESC)
        SELECT AVG(num_trips) AS avg_num_trips FROM Input;'
        ```

        |   avg_num_trips   |
        |-------------------|
        | 433.0581506196383 |
 
        b. What are number of trips by zip code and how does that compare to the lowest income neighborhoods?
        ```bq query --use_legacy_sql=false '
        CREATE VIEW Project_1.trips_by_zip AS
        SELECT COUNT (DISTINCT trip_id) AS num_trips, zip_code
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`
        WHERE  zip_code BETWEEN "90000" AND "99999"
        GROUP BY zip_code
        ORDER BY num_trips DESC;'
        ```
        ```bq query --use_legacy_sql=false '
        SELECT income.median_income, income.zip_code,zip.num_trips
        FROM `alien-house-324100.Project_1.CA_Income` AS income
        INNER JOIN `alien-house-324100.Project_1.trips_by_zip` AS zip
        ON TRIM(income.zip_code) = TRIM(zip.zip_code)
        ORDER BY median_income
        LIMIT 15;'
        ```
        
        | median_income | zip_code | num_trips |
        |---------------|----------|-----------|
        |         10625 | 90089    |        16 |
        |         11922 | 93721    |         2 |
        |         12748 | 90021    |         7 |
        |         15988 | 90058    |         3 |
        |         16649 | 92401    |         3 |
        |         17134 | 90014    |        17 |
        |         19922 | 95225    |         2 |
        |         20896 | 93258    |         1 |
        |         21009 | 90017    |        46 |
        |         21120 | 90013    |        18 |
        |         21333 | 94950    |         1 |
        |         22646 | 90007    |        65 |
        |         24127 | 94102    |     30222 |
        |         25662 | 96047    |        15 |
        |         25934 | 95422    |         1 |

        To answer this question, first I had to find the average number of trips taken per zip code registered as a benchmark to compare against. Next, I pulled in the median household income by california zip code data and stored it in a table. Then I took the number of trips by zip code and compared them against the lowest income niehgborhoods. Because we see that the average number of trips per zip code is 433 trips, the final outputs shows that the fifteen lowest income zipcodes all have a number of rides far below the average, so on the mobile app we should promote bike access for all to people in these zip codes.
        
    5. What are the top 5 zip codes where customer trips are taken?
        ```bq query --use_legacy_sql=false '
        SELECT COUNT (DISTINCT trip_id) AS num_trips, zip_code
        FROM `alien-house-324100.Project_1.trip_times`
        WHERE subscriber_type = "Customer"
        AND zip_code BETWEEN "90000" AND "99999"
        GROUP BY zip_code
        ORDER BY num_trips DESC
        LIMIT 5;'
        ```

        | num_trips | zip_code |
        |-----------|----------|
        |      2876 | 94107    |
        |      1650 | 94105    |
        |      1555 | 94103    |
        |      1505 | 94102    |
        |      1347 | 94109    |

        These are the zip codes of registered riders that we should promote monthly memberships to, because they are the locations with the largest number of customer pass trips. 