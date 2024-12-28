```sql
# For this project, I downloaded Spotify data from Kaggle.
# Then I created a table to insert Spotify data into.
# Finally, I performed analytics on the data using SQL. 

#Creating the table: 
CREATE TABLE BIT_DB.Spotifydata (
id integer PRIMARY KEY,
artist_name varchar NOT NULL,
track_name varchar NOT NULL,
track_id varchar NOT NULL,
popularity integer NOT NULL,
danceability decimal(4,3) NOT NULL,
energy decimal(4,3) NOT NULL,
key integer NOT NULL,
loudness decimal(5,3) NOT NULL,
mode integer NOT NULL,
speechiness decimal(5,4) NOT NULL,
acousticness decimal(6,5) NOT NULL,
instrumentalness text NOT NULL,
liveness decimal(5,4) NOT NULL,
valence decimal(4,3) NOT NULL,
tempo decimal(6,3) NOT NULL,
duration_ms integer NOT NULL,
time_signature integer NOT NULL 
)

#Then I inserted the Spotify Data .csv into the table.
#Next, I explored the data using the following SQL. 

# Q1: What is the average danceability and energy by artist?
SELECT artist_name, ROUND(avg(danceability),3) AS avg_danceability, ROUND(avg(energy),3) AS avg_energy
FROM BIT_DB.Spotifydata
GROUP BY artist_name
ORDER BY avg_danceability DESC, avg_energy;

# Practice using CTE's:
# Q2: Calculate the average popularity for the artists in the Spotify data table. 
# Q2a: Then, for every artist with an average popularity of 90 or above, show their name, their average popularity, and label them as a “Top Star”.

# My original attempt:
WITH avg_popularity_CTE AS (
    SELECT avg(s.popularity) AS avg_popularity,
           s.artist_name
      FROM Spotifydata s
     GROUP BY s.artist_name
)
SELECT artist_name,
       CASE WHEN avg_popularity >= 90 THEN 'Top Star' END AS star_quality,
       avg_popularity
  FROM avg_popularity_CTE
 WHERE star_quality = 'Top Star';-- or: WHERE avg_popularity >= 90;

# A slightly improved way to do this:
WITH popularity_average_CTE AS (
    SELECT s.artist_name,
           AVG(s.popularity) AS average_popularity
      FROM SpotifyData s
     GROUP BY s.artist_name
)
SELECT artist_name,
       average_popularity,
       'Top Star' AS tag
  FROM popularity_average_CTE
 WHERE average_popularity >= 90;

```
