```sql
/** 
This analysis aims to provide an overview by calculate revenue by month and year over year analyses by month.
It also organizes the data by hotel type and distribution channel to uncover greatest revenue driver.
This is hotel data July 2018 to August 2020, a two-year span, sourced from Kaggle.
Analyses performed in PostgreSQL
**/

--create the 5 tables: 
--3 tables for hotel revenue per each year, 1 table for market segment discount data and 1 table for meal pricing data

1. create the 3 tables for 3 years of the hotel data: 2018, 2019, 2020 (2019&2020 not shown)
create table hotel18 (
	 hotel varchar(255)
	,is_cancelled smallint
	,lead_time numeric
	,arrival_date_year integer
	,arrival_date_month varchar(255)
	,arrival_date_week_number numeric
	,arrival_date_day_of_month numeric
	,stays_in_weekend_nights numeric
	,stays_in_week_nights numeric
	,adults numeric
	,children numeric
	,babies numeric
	,meal varchar(255)
	,country varchar(255)
	,market_segment varchar(255)
	,distribution_channel varchar(255)
	,is_repeated_guest smallint
	,previous_cancellations integer
	,previous_boookings_not_cancelled integer
	,reserved_room_type char(1)
	,assigned_room_type char(1)
	,booking_changes integer
	,deposit_type varchar(255)
	,agent integer
	,company bigint
	,days_in_waiting_list integer
	,customer_type varchar(255)
	,adr numeric
	,required_car_parking_spaces numeric
	,total_of_special_requests integer
	,reservation_status varchar(255)
	,reservation_status_date date
);

2. create table for market segment discounts:
create table market_seg (
	 discount numeric
	,market_segment varchar(255)
);

3. create table for meal pricing:
create table meal_cost (
	 meal_price
	,meal_type
);

4. Analyses:

--Intention 1: Revenue for completed stays
with hotels as (
	select * from hotel18
union all
	select * from hotel19
union all
	select * from hotel20
)
select
	 h.arrival_date_year
	,h.hotel
	,h.distribution_channel
	,round(sum((h.stays_in_weekend_nights + h.stays_in_week_nights)*(h.adr*(1-ms.discount))
	+(h.stays_in_weekend_nights + h.stays_in_week_nights)*mc.meal_price)) as revenue
from
	hotels h
left join
	market_seg ms on h.market_segment = ms.market_segment
left join 
	meal_cost mc on mc.meal_type = h.meal
group by
	 h.arrival_date_year
	,h.hotel
	,h.distribution_channel
	--,h.is_cancelled = false
	,h.reservation_status ilike lower('%check-out%')
having
	h.reservation_status ilike lower('%check-out%')
	--h.is_cancelled =  false
order by
	arrival_date_year


--Intention 2: Lost potential revenue when deposit was not required or was refundable:
with hotels as (
	select * from hotel18
union all
	select * from hotel19
union all
	select * from hotel20
)
select
	 --to_date(h.arrival_date_year||'-'||h.arrival_date_month||'-'||h.arrival_date_day_of_month, 'YYYY-month-dd') as arrival_date
	 to_date(h.arrival_date_year||' '||h.arrival_date_month, 'YYYY month') as arrival_month
	,h.reservation_status
	,h.deposit_type
	,h.hotel
	,h.distribution_channel
	,round(sum((h.stays_in_weekend_nights + h.stays_in_week_nights)*(h.adr*(1-ms.discount))
	+(h.stays_in_weekend_nights + h.stays_in_week_nights)*mc.meal_price)) as revenue
from
	hotels h
left join
	market_seg ms on h.market_segment = ms.market_segment
left join 
	meal_cost mc on mc.meal_type = h.meal
group by
	 arrival_month
	,h.hotel
	,h.distribution_channel
	,h.reservation_status
	,deposit_type
having
	(h.reservation_status ilike lower('%no-show%') 
	or h.reservation_status ilike lower('%canceled%')) 
	and (deposit_type ilike lower('%no deposit%') or deposit_type ilike lower('%refundable%'))
order by
	 arrival_month asc

	
--Intention 3: Year Over Year Analysis in Months using CTE's:
-- Step 1: Union the three years of data
with 
all_stays as (
    select
		 cast(concat(arrival_date_month,'/',arrival_date_day_of_month,'/',arrival_date_year) as date) as arrival_date
		,to_date(arrival_date_year || ' ' || arrival_date_month, 'YYYY month') as arrival_year_month
		,stays_in_week_nights + stays_in_weekend_nights as total_stays
		,adr
		,market_segment
		,meal
		,'2018' as the_year
    from
		hotel18
    where 
		reservation_status ilike lower('check-out')
union all
    select 
		 cast(concat(arrival_date_month,'/',arrival_date_day_of_month,'/',arrival_date_year) as date) as arrival_date
		,to_date(arrival_date_year || ' ' || arrival_date_month, 'YYYY month') as arrival_year_month
		,stays_in_week_nights + stays_in_weekend_nights as total_stays
		,adr
		,market_segment
		,meal
		,'2019' as the_year
    from
		hotel19
    where 
		reservation_status ilike lower('check-out')
union all
    select 
		 cast(concat(arrival_date_month,'/',arrival_date_day_of_month,'/',arrival_date_year) as date) as arrival_date
		,to_date(arrival_date_year || ' ' || arrival_date_month, 'YYYY month') as arrival_year_month
		,stays_in_week_nights + stays_in_weekend_nights as total_stays
		,adr
		,market_segment
		,meal
		,'2020' as the_year
    from
		hotel20
    where 
		reservation_status ilike lower('check-out')
),

-- Step 2: Join with the discounts and meal plan data
revenue_data as (
    select 
		 a.*
		,ms.discount
		,mc.meal_price
		--,coalesce(ms.discount, 0) as discount_per_night --this has to be multiplied by adr (1-adr)
		--,coalesce(mc.meal_price, 0) as meal_price_per_night 
    from all_stays a
    	left join market_seg ms on a.market_segment = ms.market_segment
    	left join meal_cost mc on a.meal = mc.meal_type
),

-- Step 3: Calculate total revenue per stay
revenue_per_stay as (
    select
		 arrival_date
		,arrival_year_month
		,(total_stays * adr *(1-discount) + total_stays * meal_price) as total_revenue
    from
		revenue_data
)
	
-- Step 4: Year-over-Year Revenue Analysis by Month, YoY calculation, using window function:
select
	 arrival_year_month
	,round(sum(total_revenue),2) as monthly_revenue
    -- Use LAG function to get the previous year's revenue for the same month
    ,round(lag (sum(total_revenue)) over (partition by extract(month from arrival_year_month) order by extract(year from arrival_year_month)),2) as prev_year_revenue
    -- Calculate the Year-over-Year Difference
    ,round(sum(total_revenue) - lag(sum(total_revenue)) over (partition by extract(month from arrival_year_month) order by extract(year from arrival_year_month)),2) as yoy_difference
	--Calculate the YoY percent Change: ((New Value - Old value) / Old value) * 100 
	,round((((sum(total_revenue) - lag(sum(total_revenue)) over (partition by extract(month from arrival_year_month) order by extract(year from arrival_year_month))) / (lag(sum(total_revenue)) over (partition by extract(month from arrival_year_month) order by extract(year from arrival_year_month))))*100),2) as yoy_percent_change
from 
	revenue_per_stay
group by 
	arrival_year_month
order by
	 extract(year from arrival_year_month)
	,extract(month from arrival_year_month)
;
```
