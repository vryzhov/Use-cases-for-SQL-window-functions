- [Case 3.](#case-3)
  - [Intro](#intro)
  - [Follow up](#follow-up)


#  Case 3.


## Intro 

How would you report the conversion time in a free-to-play game? [User conversion is the event of making a first purchase after the initial period of using the product for free. The time to conversion, i.e. time it takes for a user to become a customer, is an important metric for the evaluation of monetization strategy.] Or, how many sessions does it take a player to pass a level, or finish the tutorial, or reach any other milestone in the game? How about the number of chests players need to open or how much gold to spend to complete the game? Or, how many friends do players make before churning out?

Many people won’t hesitate answering these questions. For the conversion time, take all users who made at least one purchase, take the time each of them spent with the product before making the first purchase, then compute the average of these numbers. Let’s say, these calculations yield 3.14 days. Looks quite reasonable. Problem’s solved, right?

Not quite.

## Follow up


Why is it not the answer you should be satisfied with? There are two reasons. First, the average is meant to be something that people expect to be “expected.” That is, the average temperature is the temperature we anticipate to find if we go to a certain location, average salary is something we most likely to see for a randomly chosen professional working in a particular field, average age is the expected age of a random individual within a given population, etc. Averages are used everywhere to quickly convey what to expect within a specific context. The problem is that the average time to the first purchase of 3.14 days in a freemuim game is simply non-realistic. It implies that every player installing the game today will make her first purchase on average after 3.14 days of play. But it is a known fact that the percent of players who ever make a single purchase lies in a single digit range for most free-to-play games. In other words, a huge proportion of players prefer playing for free, never converting to paid customers. How can one expect a random player to convert in three days on average then?

The missing detail in the reasoning above is an often overlooked assumption: only paid users are considered for this calculation. The estimate 3.14 days of the average conversion time is valid for the paid players and not for any randomly selected one. The estimate of 3.14 days for the conversion time is too short, too optimistic as only the “best” ones have been chosen to make their contribution to this number. 

This clarification, however, does not make the number useless or non-informative. We still can make good decisions based on the value of 3.14 days assuming its meaning is well understood. Particularly, it must be clear that these 3.14 days do not carry any information on players’ propensity to make the first purchase; nobody is expected to make a first purchase on the fourth day. Also, on the conceptual level, the team needs to come to grips with a certain discomfort of “reversed causality.” The fact of becoming a paid user, i.e. the conversion event, is a consequence of making the first purchase, yet it serves as a necessary condition in estimating the first purchase time. And this is the second reason to doubt the naively calculated number.

The questions in the beginning of this post share exactly the same properties and are prone to the same error of ignoring cases when the event of interest did not occur. They all are concerned with the time to a certain event, which may or may not take place. Think of the time your car breaks down for the first time (or a failure of any other sophisticated machine), time to the birth of first child, time to the marriage (as well as the marriage duration), and ultimately time to death (in a video game and in real life). All these examples are concerned with the time-to-event metric; it can be found everywhere. 

The nature of event is determined by the question we have to answer, but the definition of time is not. Time can be defined as terms of the usual calendar time passed between the "beginning of time" and the event, or the time players spent in game, or the count of play sessions, count of acquired friends, count of completed missions, count of opened chests, etc. Any metric that increases with the calendar time can be used as time. The suitable choice of time unit also more or less free depending on the situation [A common question game designers like to ask is, how many sessions does it take to rank up? The time unit here is one play session and the event of interest is the ranking up.]

Going back to the discussion of conversion, the question then becomes: is it possible to account for those users who did not offer any information on the very metric we are interested in estimating, i.e. the users with the non-existent conversion time? It turns out that the meaningful answer is possible within the statistical discipline known as survival analysis that puts the matter on solid footing.

To develop some intuition, let us again consider free players who have not converted. Despite never payed, these people still carry some useful information. Namely, they inform us of the time during which they did not convert, i.e. the time they were using the product for free. And this is the key insight lying at the core of survival analysis. We can complement the data on conversion times available for paid customers with the second data set collected on users who have not converted, i.e. the usage data of free players. 

To illustrate the time-to-event computations developed within survival analysis I will use the data leukemia available in “survival” R package. It consists of survival data collected on 23 anonymous individuals and has three fields: time, status, and the third field that will not be used here. [The Wikipedia page provides much more details of the data and calculations sketched below. I chose to utilize this set so that the readers will have an excellent Wikipedia article to follow along and go beyond the material developed here.] For the purpose of this post, the time field will be interpreted as the time to conversion (the event) in a free-to-play game. I also added a fictitious player name to each record for convenience. The count of events (18) suggests a rather unrealistic conversion rate of 78% (=18/23) but this is not relevant for our discussion; any other type of event can do, for example the game completion or churn. I use conversion to maintain continuity with the discussion of previous paragraph and will use the terms ‘event’ and ‘conversion’ interchangeably. The time scale is the count of sessions with one session being the time unit.

A very important assumption that may not hold in general but assumed valid in what follows, is the assumption of statistical independence: the conversion of any player does not depend on conversion of others. The analysis becomes much more complicated (and the corresponding calculations are better done by specialized statistical software) when the independence assumption is not justifiable.

The leukemia data set augmented with the fictitious player name is generated by the following SQL query. 

with Surv as (
    select      'Liam' as player,   9 as time, 1 as status union
    select      'Noah' as player,  13 as time, 1 as status union
    select   'William' as player,  13 as time, 0 as status union
    select     'James' as player,  18 as time, 1 as status union
    select     'Logan' as player,  23 as time, 1 as status union
    select 'Elizabeth' as player,  28 as time, 0 as status union
    select     'Mason' as player,  31 as time, 1 as status union
    select    'Elijah' as player,  34 as time, 1 as status union
    select    'Oliver' as player,  45 as time, 0 as status union
    select     'Jacob' as player,  48 as time, 1 as status union
    select     'Lucas' as player, 161 as time, 0 as status union
    select   'Michael' as player,   5 as time, 1 as status union
    select 'Alexander' as player,   5 as time, 1 as status union
    select      'Emma' as player,   8 as time, 1 as status union
    select    'Olivia' as player,   8 as time, 1 as status union
    select       'Ava' as player,  12 as time, 1 as status union
    select  'Isabella' as player,  16 as time, 0 as status union
    select    'Sophia' as player,  23 as time, 1 as status union
    select       'Mia' as player,  27 as time, 1 as status union
    select 'Charlotte' as player,  30 as time, 1 as status union
    select    'Amelia' as player,  33 as time, 1 as status union
    select    'Evelyn' as player,  43 as time, 1 as status union
    select   'Abigail' as player,  45 as time, 1 as status 
)
select * from Surv order by time
|  player   | time | status |
|-----------|------|--------|
|  Michael  |  5   |   1    |
| Alexander |  5   |   1    |
|   Emma    |  8   |   1    |
|  Olivia   |  8   |   1    |
|   Liam    |  9   |   1    |
|    Ava    |  12  |   1    |
|   Noah    |  13  |   1    |
|  William  |  13  |   0    |
| Isabella  |  16  |   0    |
|   James   |  18  |   1    |
|   Logan   |  23  |   1    |
|  Sophia   |  23  |   1    |
|    Mia    |  27  |   1    |
| Elizabeth |  28  |   0    |
| Charlotte |  30  |   1    |
|   Mason   |  31  |   1    |
|  Amelia   |  33  |   1    |
|  Elijah   |  34  |   1    |
|  Evelyn   |  43  |   1    |
|  Oliver   |  45  |   0    |
|  Abigail  |  45  |   1    |
|   Jacob   |  48  |   1    |
|   Lucas   |  161 |   0    |


Each record represents one observation, the data collected on a single player. Records with status = 0 are called censored (right-censored, to be precise) meaning that all available information on these players is that they did not make any purchase during time passed since they installed the game. The conversion event may take place after this count of sessions but we do not have any data whether and when it happens. The value status = 1 is assigned to players who made their purchase, and for them the field time contains the count of sessions it took each of them to convert, i.e. the conversion time. Notice that the actual session and conversion timestamps do not play any role here - we only need the duration denoted by "time" and the censoring indicator status, regardless the calendar date when these events took place. We see that 18 players made a purchase and 5 continued playing for free up to the point when they were last seen. In the chart below, players who made a purchase are marked with dots placed on their timeline at the distance corresponding to the time of purchase. The five other players were last seen at the time marked with the cross. We know that they were still playing for free when they reached this count of sessions; they are censored observations. 

No alt text provided for this image
Starting from time t = 0 and moving to the right we see that at each value of t one of the two possible things takes place. A player can make her purchase (t = 5, 8, 9, 12, 13, 18, 23, 27, 30, 31, 33, 34, 43, 45, 48) or the last session is recorded if the player did not convert (t = 13, 16, 28, 45, 161). Intersection of these two sets (t = 13, 45) are times when both types of outcomes took place. Censored observations occur at t = 13, 16 , 28, 45, 161.

For each t > 0 in the table above, the survival analysis framework associates two non-negative integers n.risk(t) and n.event(t), the count of players who have neither converted up to the time t nor have been censored at t (they are prone to conversion at this time), and the count of players who had an event (i.e. converted) precisely at time t. Their ratio h(t) = n.event(t)/n.risk(t) is the “conversion risk”, i.e. the proportion of non-censored free players who made their first purchase at this very moment t. The function n.risk(t) declines as t grows and more and more players become paid users; the number n.event(t) takes values 0,1,2,3,… indicating the count of players converted precisely at the moment t. An important observation is that the players censored at time t = C do not contribute to n.event(C) and n.risk(C), but are accounted for in the calculations of n.risk(t) for all t < C. This subtle detail makes the survival analysis so powerful; we can use data on non-converted players (William, Isabella, Elizabeth, Oliver, and Lucas) collected prior to their censoring time and include it into the estimation of the conversion time for the whole population. 

The SQL query below illustrates calculations of n.risk(t), n.event(t), and h(t) using the data set Surv created above. Brief explanations of each step are found in the comments. 

with Surv as (
   ...
), G as (  -- summary for each value of 'time'
  select time 
       , sum(status) as status
       , count(*) as records
    from Surv 
  group by time
), S as (  -- replace zeros with nulls where status = 0
           -- This is a technical trick needed for 'ignore nulls' 
           -- statement in the query T below
  select time    
       , records
       , case when status > 0 then status 
              else null 
          end as status
       , sum(records) over () as total -- total observations
  from G
), T as (  -- for all records where status = null (i.e. censored)
           -- find the most recent time where status = 1
           -- Each censored record will be merged with the 
           -- immediately preceding event record (status > 0) 
  select S.*       
       , lag(time) ignore nulls over (order by time) 
                                              as last_event_time
  from S  
 ), D as (  -- replace time of censored records 
            -- with the most recent event time    
            -- found on the previous step
  select case when status is null 
              then last_event_time
              else time 
          end as time
       , total
       , sum(coalesce(status,0)) as n_event
       , sum(records)            as records
  from T 
  group by 1,2
), P as (  -- compute cumulative count over time of all events
           -- and censored observations combined
  select time, n_event, total, sum(records) over (order by time) 
                                                          as n_obs
    from D
), E as (  -- compute 'at risk' counts, use 'total' for 
           -- the first observation that has lag(...) is null
  select time
       , coalesce(total - lag(n_obs) over(order by time), total) 
                                                          as n_risk
       , n_event
    from P
), R as (
  select time, n_risk, n_event, n_event::float/n_risk::float as h
    from E
)
select * from R 
 order by time


| TIME | N_RISK | N_EVENT | H             |
|--- --|--------|---------|---------------|
| 5    | 23     | 2       | 0.08695652174 |
| 8    | 21     | 2       | 0.09523809524 |
| 9    | 19     | 1       | 0.05263157895 |
| 12   | 18     | 1       | 0.05555555556 |
| 13   | 17     | 1       | 0.05882352941 |
| 18   | 14     | 1       | 0.07142857143 |
| 23   | 13     | 2       | 0.1538461538  |
| 27   | 11     | 1       | 0.09090909091 |
| 30   | 9      | 1       | 0.1111111111  |
| 31   | 8      | 1       | 0.125         |
| 33   | 7      | 1       | 0.1428571429  |
| 34   | 6      | 1       | 0.1666666667  |
| 43   | 5      | 1       | 0.2           |
| 45   | 4      | 1       | 0.25          |
| 48   | 2      | 1       | 0.59          |

The field H is the ratio n_event/n_risk. Notice that all values of time where no conversions take place are dropped. To understand better what’s going on it is recommended to run each subquery G, S, T, D, etc. separately . For example,

with Surv as (
   ...
), G as (  
  ...
), S as ( 
  ...
)
select * from S order by time

|
| TIME | RECORDS | STATUS | TOTAL |
|------|---------|--------|-------|  
| 5    | 2       | 2      | 23    |
| 8    | 2       | 2      | 23    |
| 9    | 1       | 1      | 23    |
| 12   | 1       | 1      | 23    |
| 13   | 2       | 1      | 23    |
| 16   | 1       |        | 23    |
| 18   | 1       | 1      | 23    |
| 23   | 2       | 2      | 23    |
| 27   | 1       | 1      | 23    |
| 28   | 1       |        | 23    |
| 30   | 1       | 1      | 23    |
| 31   | 1       | 1      | 23    |
| 33   | 1       | 1      | 23    |
| 34   | 1       | 1      | 23    |
| 43   | 1       | 1      | 23    |
| 45   | 2       | 1      | 23    |
| 48   | 1       | 1      | 23    |
| 161  | 1       |        | 23    |

Let’s recap results of the previous section. For each value t of the time variable we constructed two integers, n.risk and n.event and their ratio h = n.event/n.risk interpreted as the probability of the event (i.e. conversion) happening at time t. In the literature, h(t) is called the instantaneous risk or hazard ratio of the event at time t. Values of t at which no events occur do not appear in the final results (because at these moments n.event is zero) and subsequently, are not associated with the “risk” of the event happening. Players who convert at time T are accounted for calculations of h(t) for all t ≤ T, and players who did not convert and are censored at time T contribute to calculations of h(t) for t < T. 

Now, let’s move on and try to answer the original question: for a random player and some given a priori count of sessions T, what is the probability of seeing her converted by time T? Also consider the question, what is the probability for this player to convert on or after time T? These two questions are complementary as the probabilities to convert by time T and probability to convert on or after time T sum up to one. It turns out that answering the latter is easier than the former.

Indeed, for a player to play free up to time T means that she failed to convert at each value of t where t < T. Conversion probabilities at these moments have been already computed and are equal to the hazard ratios h(t), t < T. Probabilities of not converting at these moments are 1 - h(t), t < T. Now recall the main assumption of independence: each player converts or not independently of conversions of others. This assumption allows us compute the probability of conversion on or after T as the product of probabilities of not converting at all preceding times t < T, i.e. the product of (1- h(t)) for all t such that t < T. This probability is a function of time T. It is called survival function and denoted by S(T).

The reasoning above helps justify the word “survival” and can clarify the matter: to get to the point T as a free player, she has to fail to “die” as a free player up to the time T, i.e. to “survive” through all the conversion hazards situated at the preceding “risky” points t, t < T. The true survival function is unknown, and the just described construction is the estimate of S(t) known as Kaplan-Meier estimate (of survival function), or K-M curve.

Let T = 3. There are no values of t smaller than T and the probability to survive up to T = 3 is S(3) = (1 - 0/23) = 1. 

Let T = 6. There is a single value of t smaller than T. It’s t = 5 with the hazard rate h(5) = 2/23 = 0.087. Therefore the survival probability S(6) = (1- 0/23)*(1–2/23) = 0.913

Let T = 10. Three values of t precede T = 10: t = 5, t = 8, and t = 9. Therefore, S(10) = (1 - 0/23)*(1 - 2/23)*(1 - 2/21)*(1- 1/19) = 0.7826

Let T = 15. Five values of t are smaller than T= 15, and S(15) = (1–0/23)*(1–2/23)*(1–2/21)*(1 - 1/19)*(1 - 1/18)*(1 - 1/17) = 0.6957

The survival function S(t) is a decreasing because the probability of not converting diminishes with time. It is a piecewise constant function (step function) equal to 1.0 at t = 0 with the minimum attained at the last non-censored observation, t = 48 in our data set. If the last observation (i.e. Lucas) was not censored, S(t) would reach zero.

The value of S(T) is the probability of event not happening at time T conditional on the individual not experiencing the event prior to time T. A player has to “survive” as a free player up to time T to be considered as a potential customer by that time. This interpretation is well aligned with the formula given above. For instance, S(8) = (1–2/23)*(1–2/21) is the product of (1–2/21), the probability of not converting at time t = 8 and the factor (1–2/23), the probability of not converting at time t = 5 (i.e. the probability of survival at t = 5). It’s the independence assumption that allows us to compute the conditional probability as the product (1–2/23)*(1–2/21). The first term is the probability of the condition (the event did not occur at t = 5) and the second term is the probability of the event not happening at the moment t = 8.

The logic above can be easily implemented in SQL. If we had a multiplicative analog of the window function sum() that can compute cumulative product over ordered partitions, the query for K-M estimate would look like this 

with Surv as (
   ...
)
   ...
), R as (
 ...
)
select time, n_risk, n_event
     , prod(1 - h) over (order by time) as survival 
from E
 order by time

where prod() is a hypothetical window function, the analog of sum(). Such a function does not exist (at least, in Snowflake) but we can reduce multiplication to summation by taking logarithms, sum them up using window sum() and then exponentiate. This leads to the following query, which completes our calculations (cf. Wikipedia).

with Surv as (
   ...
)
   ...
), R as (
 ...
), KM as (
   select time, n_risk, n_event
       , power(10,
            sum(log(10, 1.0 - h)) over (order by time) 
         ) as survival 
   from E
) 
select * from KM
order by time
| TIME | N_RISK | N_EVENT | SURVIVAL     |
|--- --|--------|---------|--------------|
| 5    | 23     | 2       | 0.9130434783 |
| 8    | 21     | 2       | 0.8260869565 |
| 9    | 19     | 1       | 0.7826086957 |
| 12   | 18     | 1       | 0.7391304348 |
| 13   | 17     | 1       | 0.6956521739 |
| 18   | 14     | 1       | 0.6459627329 |
| 23   | 13     | 2       | 0.5465838509 |
| 27   | 11     | 1       | 0.4968944099 |
| 30   | 9      | 1       | 0.4416839199 |
| 31   | 8      | 1       | 0.38647343   |
| 33   | 7      | 1       | 0.33126294   |
| 34   | 6      | 1       | 0.27605245   |
| 43   | 5      | 1       | 0.22084196   |
| 45   | 4      | 1       | 0.16563147   |
| 48   | 2      | 1       | 0.0828157350 |

Obtained K-M estimate makes it possible to estimate the median time to conversion defined as the smallest survival time for which S(t) is less than or equal to 0.5. As to the mean (average) survival time, it is not properly defined if S(t) does not go to zero, as happens in our case (because Lucas did not convert). In fact, practitioners commonly use the median to summarize survival data.

In SQL, the median is calculated by querying the data set KM. The result in our case is 27. This is the count of sessions after which we expect 50% of players make their first purchase. 

...
select min(time) as median_time 
from KM 
where survival <= 0.5

Notice that there are many situations when the median survival time would not exist. As mentioned above, the conversion rate in most free-to-play games rarely exceed 5%, and therefore to have 50% of the population converted to paid customers is unrealistic. Nevertheless, the definition of median is easily extended to the definition of percentile. For instance, to estimate the 99th percentile of conversion time (the time after which we expect 99% of players still playing for free, and 1% having made their first purchase), the number 0.5 in the query above needs to be replaced by 0.99. 

Considering practical applications, below is an example of survival data set pulled from a database. In this illustration time is the usual calendar time with one day as a time unit, and the event of interest is first purchase. 

Assume we have two tables holding players’ sessions player_sessions with the fields player_id, session_time_utc and iap_transactions table with fields player_id, transaction_time_utc. These tables are joined using left outer join to account for players from player_sessions who do not have any records in iap_transactions. The time of first transaction, install time, and the time of last session are obtained with group by clauses. The query returning all the necessary information would look like this:

with S as (  -- Install time and the time of last session
	select player_id
             , min(session_time_utc) as install_time
             , max(session_time_utc) as last_time
	 from player_sessions 
        group by player_id
), T as (   -- First purchase time
	select player_id
	     , min(transaction_time_utc) as conversion_time
	 from iap_transactions
	group by player_id
), D as (  -- Join both sets. Return time diff between install time 
           -- and either conversion or last session timestamp
           -- in 'time' field
    select S.player_id as session_player
         , T.player_id as iap_player
         , datediff(second
             , install_time
             , coalesce(conversion_time,last_time))/86400.0 as time
     from S left outer join T on (S.player_id = T.player_id)
), Surv as ( -- 'status' indicator of event/censored observations
    select session_player as player
         , time 
         , case when iap_player is null 
                then 0 -- free player, event not observed
                else 1 -- paid player, event observed
            end as status
     from D
)
select * from Surv

The first subquery S calculates timestamps of player’s first and last sessions. The second subquery T returns player’s conversion time (if exists) as timestamp of the first purchase. The outer join in D combines both results into a single data set making sure either conversion_time or last_time is mapped to time field. Finally, Surv computes the time to event - either player’s first purchase timestamp, or the time of player’s last session available in the system. The call to coalesce() returns conversion_time for players who made a purchase or session_time for all others. Additionally, Surv creates the binary flag status, an indicator of conversion. The statement returns a survival data set with three fields: player, time, and status, one row per player.

September 2019