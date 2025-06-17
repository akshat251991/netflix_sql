# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);

select * from netflix n


select count(*) from netflix

select distinct country from netflix
select distinct type from netflix
select distinct release_year from netflix
select distinct rating from netflix
select distinct duration from netflix

--1. Count the Number of Movies vs TV Shows

select type,count(*)
from netflix
group by type

--2. Find the Most Common Rating for Movies and TV Shows (imp)
select type,rating from
(
select type,
rating,
count(*),
rank() over(partition by type order by count(*) desc) as ranking
from netflix
group by 1,2
order by type,count(*) desc
)
where ranking=1

--3. List All Movies Released in a Specific Year (e.g., 2020)
select * from netflix
where type='Movie'
and release_year='2020'


--4. Find the Top 5 Countries with the Most Content on Netflix

select
unnest(string_to_array(country,',')) as new_country,
count(*) as count_of_contents
from netflix
group by 1
order by count(*) desc

--5. Identify the Longest Movie
select * from netflix
where type='Movie'
	and 
		duration=(select max(duration) from netflix) 


--6. Find Content Added in the Last 5 Years
select * 
from netflix
where to_date(date_added,'Month DD,YYYY') >= current_date-interval '5 year'


--7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'
select * from
(
SELECT title,
unnest(string_to_array(director,',')) as director_name
from netflix
group by 1,2
)
where director_name='Rajiv Chilaka'
group by 1,2

--Alternate way
-- can also use ilike to deal with case sensitive characters like for eg if a new record has name as 'rajiv chilaka'.
--Meanwhile like can also be used
select * from netflix
where director ilike '%Rajiv Chilaka%'


--8. List All TV Shows with More Than 5 Seasons
select *
from netflix
where type='TV Show'
AND split_part(duration,' ',1)::NUMERIC > 5

--9. Count the Number of Content Items in Each Genre desc

SELECT 
unnest(string_to_array(listed_in,',')) as genre,
count(*) as number_of_contents
FROM NETFLIX
group by 1
order by 2 desc

--10.Find each year and the average numbers of content release in India on netflix. (imp.)
--return top 5 year with highest avg content release!
select
extract(year from to_date(date_added,'Month dd, yyyy')) as year,
count(*) as yearly_contents,
round(count(*)::numeric/(select count(*) from netflix where country='India'):: numeric * 100,2) as avg_content_per_year
from netflix
where country='India'
group by 1

--11. List All Movies that are Documentaries
select * from
(
select title,
unnest(string_to_array(listed_in,',')) as genre
from netflix
where type='Movie'
group by 1,2
)
where genre='Documentaries'

--alternate way
SELECT * 
FROM netflix
WHERE listed_in iLIKE '%Documentaries%'

--12. Find All Content Without a Director
select * from netflix
where director is null

--13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years
select *
from netflix
where casts ilike '%Salman Khan%'
and release_year > extract(year from current_date)-10

--14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India
select 
unnest(string_to_array(casts,',')) as new_casts,
count(*)
from netflix
where country ilike '%India%'
group by 1
order by 2 desc
limit 10

--15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
--Objective: Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise.
--Count the number of items in each category. (imp)
With Category_col as
(
select *,case when description ilike '%kill%' or 
				description ilike '%violence%' then 'Bad'
				else 'Good' end Category
				from netflix
)

select category,count(*) as cnt from category_col group by 1
## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.



