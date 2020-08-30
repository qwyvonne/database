# Data Manipulation Using PostgreSQL, R, and Python With Examples 

**DATASET** <br/>
covid_19_nyt.csv - Live case update from New York Times's Github respository: https://github.com/nytimes/covid-19-data<br/> 
population.csv - Estimate of population from United States Census Bureau https://www.census.gov/programs-surveys/popest.html
<br/> 
<br/>
**METADATA** <br/>
1) **covid_19_nyt.csv**<br/>
**date** - date of the COVID-19 case number and deaths being reported <br/>
**state** - states in the U.S.<br/>
**fips** - The Federal Information Processing Standard Publication 6-4 was a five-digit Federal Information Processing Standards code which uniquely identified counties and county equivalents in the United States, certain U.S. possessions, and certain freely associated states<br/>
**county** - a political and administrative division of a state, providing certain local governmental services<br/>
**cases** - The total number of cases of Covid-19, including both confirmed and probable<br/>
**deaths** - The total number of deaths from Covid-19, including both confirmed and probable<br/>


2) **population.csv**<br/>
**county** - a political and administrative division of a state, providing certain local governmental services<br/>
**state** - states in the U.S.<br/>
**population** - U.S. population as of 2010 <br/>

3) **gov_order.csv**<br/>
**state** - 50 U.S. states and Washington D.C.<br/>
**closure_school** - date that closure of schools order was issued by state government<br/>
**closure_workplaces** - date that closure of workplaces order was issued by state government<br/>
**cancellation_public_events** - date that cancellation of public events order was issued by state government<br/>

# Examples with Solutions (PostgreSQL, R, Python) 
Scenario 1: Find cummulative case number in each state in descending order <br/>

**PostgreSql**
```
select state, sum(cases) as cases from covid_19_nyt
group by state 
order by sum(cases) desc 
```

**R**
```
covid_19_nyt %>% group_by(state) %>% 
	summarize(cases = sum(cases)) %>% 
	arrange(desc(cases))
```

Scenario 2: Find the states that have fatality rate that is above the national fatality rate  <br/>
**PostgreSql**
```
select a.state from 
	(select state, sum(deaths)/sum(cases) as state_fatality_rate 
	from covid_19_nyt
	group by state)a
inner join 
	(select sum(deaths)/sum(cases) as national_fatality_rate
	from covid_19_nyt)b
on a.state_fatality_rate >= b.national_fatality_rate 
```

**R: distinct()**
```
covid_19_nyt %>% mutate(national_fatality_rate = sum(deaths)/sum(cases)) %>% 
  group_by(state) %>% mutate(state_fatality_rate = sum(deaths)/sum(cases)) %>% 
  filter(state_fatality_rate >= national_fatality_rate) %>% distinct(state)
```

Scenario 3: Find state that have the highest percentage of infection rate in July 2020, round up to two decimals. (Infection rate is defined as case number/population, and here we do not consider people who got infected many times and only considered people who tested positive as infected) <br/> 

**PostgreSql: sum(), round(), left join, order by, limit**
```
select b.state, round(c.state_case/b.state_population,2) as infection_rate from 
  (select state, sum(population) as state_population from population 
	group by state)b
  left join 
  (select a.state, sum(a.county_case) as state_case from 
	    (select state, county, sum(cases) as county_case 
      from covid_19_nyt	
	    where to_char(date, 'YYYY-MM') = '2020-07'
	    group by state, county)a
  group by a.state)c
  on b.state = c.state 
order by 2 desc 
limit 1 
```

**R: dyplr, filter(), summarize(), group_by(), drop_na(), inner_join(), transmute()**
```
a = covid_19_nyt %>% 
	filter(month(as.Date(covid_19_nyt$date, "%m/%d/%Y")) == 7) %>%
  	group_by(state) %>% 
	summarize(total_case = sum(cases)) %>% 
 	drop_na(state)

b = population %>% 
	group_by(state) %>% 
	summarize(total_population = sum(population))

inner_join(a,b) %>% 
	mutate(infection_rate = total_case/total_population) %>% 
 	filter(infection_rate == max(infection_rate)) %>% 
  	transmute(state,round(infection_rate,2))
```
Scenario 4: Calculate average number of cases within 15 days of timeframe per state before and after the first time that cancellation of public events is being placed in that state if there is any. If the average case number after the lauch of the policy is smaller than the that of before, we will return "effective", otherwise "not effective" and name this column as effectiveness; Return state and effectiveness. <br/>

**R**

**PostgreSql: date_part(), case when ... then, else ...**
```
select c.state, 
case 
	when c.case_after > d.case_before then 'effective'
	else 'not effective'
end as effectiveness from 
	(select c.state, round(avg(c.state_case),0) as case_after from 
		(select a.state, a.date, a.state_case, b.first_order from 
			(select state, date, sum(cases) as state_case from covid_19_nyt 
			group by 1, 2)a
			inner join 
			(select state, min(cancellation_public_events) as first_order from gov_order
			group by 1)b
			on a.state = b.state and date_part('day', a.date - b.first_order) = 15)c
		group by 1)c
left join
	(select c.state, round(avg(c.state_case),0) as case_before from 
	       (select a.state, a.date, a.state_case, b.first_order from 
			(select state, date, sum(cases) as state_case from covid_19_nyt 
			group by 1, 2)a
			inner join 
			(select state, min(cancellation_public_events) as first_order from gov_order
			group by 1)b
			on a.state = b.state and date_part('day', b.first_order - a.date) = 15)c
		group by 1)d 
on c.state = d.state
order by 2
```
