---
title: "Streamer Hachet Technical Test"
author: "Alfonso Noguer"
date: "26/9/2019"
output:
 html_document:
   keep_md: TRUE
   theme: cerulean
   toc: true
   toc_depth: 3
   toc_float:
     collapsed: true
     smooth_scroll: true

---



****


## 1. You are given the following SQL tables:

<br>

a) streamers: it contains time series data, at a 1-min granularity, of all the channels that broadcast on
Twitch. The columns of the table are:


 * username: Channel username
 * timestamp: Epoch timestamp, in seconds, corresponding to the moment the data was captured
 * game: Name of the game that the user was playing at that time
 * viewers: Number of concurrent viewers that the user had at that time
 * followers: Number of total followers that the channel had at that time


b) games_metadata: it contains information of all the games that have ever been broadcasted on Twitch.
The columns of the table are:


* game: Name of the game
* release_date: Timestamp, in seconds, corresponding to the date when the game was released
* publisher: Publisher of the game
* genre: Genre of the game


First we need to create our tables.

<br>


```sql
CREATE TABLE `streamers` (
  `username` varchar(64) NOT NULL,
  `timestamp` datetime NOT NULL,
  `game` varchar(32) NOT NULL,
  `viewers` integer NOT NULL,
  `followers` integer NOT NULL
);
```


```sql
CREATE TABLE `games_metadata`(
    `game` varchar(64) NOT NULL,
    `release_date` datetime NOT NULL, 
    `publisher` varchar(64) NOT NULL, 
    `genre` varchar(64) 
);
```

<br>

## Write an SQL query to:

<br>

### 1.1 Obtain, for each month of 2018, how many streamers broadcasted on Twitch and how many hours of content were broadcasted.


#### The output should contain **month**, **unique_streamers** and **hours_broadcast**.

<br>


```sql
SELECT  strftime('%m',`timestamp`) as months ,COUNT(DISTINCT username) as `unique_streamers`, COUNT( strftime('%M',`timestamp`))/(60*1.0) as hours_broadcast FROM streamers where strftime('%Y',`timestamp`) = '2018' GROUP BY months 
```

<br>

So, first we select the month with the strftime function for the month display (and to later aggregate the data by month), since we only want the months of 2018, we specify the timestamp year for 2018 in the **FROM** statement. <br>
We use **COUNT (DISTINCT username)** in order to obtain the total number of different streamers that will be aggregated by the months column we created beforehand.<br>     
The data is captured on a per minute basis, duplicated timestamps are valid since you'll most likely have multiple streams at the same time.<br>
My approach was to count all rows (duplicates included, the datetime format doesn't matter since this is a time series with 1 minute granularity), and divide them by 60 to get the number of hours.       

<br>


### 1.2 Obtain the Top 10 streamers that have percentually gained more followers during January 2019, and that primarily stream FPS games. 

#### The output should contain the **username** and **follower_growth**.

<br>



```sql
SELECT username, ((MAX(followers)*1.0-MIN(followers)*1.0)/MIN(followers)*1.0) AS follower_growth FROM (SELECT username,followers, genre, "timestamp" FROM (SELECT * FROM streamers AS A INNER JOIN games_metadata AS B ON A.game = B.game) WHERE strftime('%Y',"timestamp") = '2019' and strftime('%m',"timestamp") = '01' and genre = 'FPS') GROUP BY usernameOrder by follower_growth DESC LIMIT 10 
```

<br>

The first thing we need to do is a inner join table to dintiguish FPS from non FPS games.


**SELECT FROM streamers AS A INNER JOIN games_metadata AS B ON A.game = B.game**


Now we do a subquery on the the table we just "created", where we select what we need: **username** to later display and also to group by **followers** to calculate the growth, **genre** to use as a condition for FPS games, **timestamp** to filter only Jan of 2019.

With this "newly created table" (it's not a table it's only a query, but we can think of it as a table because we are gonna query from a query), and we use:


**WHERE strftime('%Y',"timestamp") = '2019' and strftime('%m',"timestamp") = '01' and genre = 'FPS'**


To filter Jan 2019 and FPS games 


**SELECT username, ((MAX(followers)*1.0-MIN(followers)*1.0)/MIN(followers)*1.0) AS follower_growth**


To calculate growth we used the formula above (multiplication by 1.0 to typecast to decimal) 

**GROUP BY username Order by follower_growth DESC LIMIT 10**

And of course we need to aggregate and order the data as requested.

<br>

### 1.3 Obtain the Top 10 publishers that have been watched the most during the first quarter of 2019.

#### The output should contain publisher and hours_watched.
 
#### Note: Hours watched can be defined as the total amount of hours watched by all the viewers combined. Ie: 10 viewers watching for 2 hours will generate 20 Hours Watched.

<br>


```sql
SELECT publisher, (cast(strftime('%m', "timestamp") as integer) + 2) / 3 as quarter, COUNT((strftime('%M',`timestamp`)/(60*1.0)) * viewers) as total_hours_watch
FROM streamers AS A INNER JOIN games_metadata AS B ON A.game = B.game 
WHERE quarter = 1
GROUP BY publisher 
ORDER BY total_hours_watch DESC
LIMIT 10 ;
```

<br>


## 2. The tub
<br>

#### "A 4-year-old is trying to build a tub for his goldfish out of Lego. 

#### Every Lego piece is stuck to the piece to its left and its right (except for the first and last one). All the pieces have a width of 1 unit.

#### Write a program, using the programming language of your choice, that given the heights (in units) of the lego pieces from left to right, outputs the total amount of water held over the pieces that the kid built."

To solve this problem i chose to find the minimum values in the vector, if the minimum is in one of the extremes we remove them, and if the minimum is equal to the maximum value of the vector we end the loop, if those conditions aren't met we "fill" the minimum values of the vector and count the number of positions filled and repeat the process. 

Lets show an example, we start with:
the vector (1,3,1,2,1,3,1,3)
 
 o#ooo#o#<br>
 o#o#o#o#<br>
 ########<br>

then we find the position of the minimum values here are
1 3 5 7
because we have a minimum in the first position we drop the first element of the vector so the positions we still have are
2 4 6
and we fill them so the vector is (3,2,2,2,3,2,3) and we had a volume of 3 units of water

 #ooo#o#<br>
 #w#w#w#<br>
 #######<br>
 
Then we find the minimum positions again 
2 3 4 6

They aren't in the extremes so we fill them and the vector is (3,3,3,3,3,3,3) and we have 7 units of water

 #www#w#<br>
 #w#w#w#<br>
 #######
 
And because the maximum value and the minimum value of the vector are the same we finish the process 



```r
altura <- c(2,2,5,2,6,1,4,3,4,2)

water_height <- function(alturas){
  wa <- alturas
  maxheight <- max(alturas)
  water <-  0
  # we repeat the loop while there is the posibility to have water so
  # at least 3 elemnts or a minimum in the function "hole"
  while (length(wa)>2 | min(wa) < maxheight ) {
    # if the minimum its in the extremes we remove them because they       # don't hold water (badum tss!)
    while (length(wa) > 1 & head(wa[1] == min(wa),n = 1)) {
      wa <- wa[-1]
    }
    while (length(wa) > 1 & tail(wa == min(wa),n = 1)) {
      wa <- wa[-length(wa)]
    }
    # we fill 
    if(length(wa) > 2) {
      pos_to_fill <- wa == min(wa)
      wa[pos_to_fill] <- wa[pos_to_fill] + 1
      water <- water + sum(pos_to_fill)
    }
  }
  return(water)
}
 
water_height(altura)
```

```
## [1] 7
```

