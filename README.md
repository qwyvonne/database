# Data Manipulation Using PostgreSQL, R, and Python With Examples 

**DATASET** <br/>
covid_19_nyt.csv - Live case update from New York Times's Github respository: https://github.com/nytimes/covid-19-data<br/> 
population.csv - Estimate of population from United States Census Bureau https://www.census.gov/programs-surveys/popest.html
<br/> 
<br/>
**METADATA** <br/>
1) **covid_19_nyt.csv <br/>**
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

# Examples with Solutions (PostgreSQL, R, Python) 
Scenario 1: Find state that have the highest percentage of infection rate in July 2020, round up to two decimals. (Infection rate is defined as case number/population, and here we do not consider people who got infected many times and only considered people who tested positive as infected) <br/> 

**PostgreSql**
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
