# SQL-Exploration

A Covid19 data set explored using SQL

Data exploration is the process of analyzing and understanding a dataset to uncover insights and patterns that can be used for analysis or decision-making. 

It involves reviewing the data from different aspects, describing its primary traits and attributes, and identifying any abnormality that may require additional research.


Gaining a better understanding of the data and providing trends, I think, is the major end goal of Data Exploration



select *
from covid19..coviddeaths

order by 3,4

select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as deathpercent
from covid19..coviddeaths
where location like '%states%'
order by 1,2


--looking at total cases of population
--shows what population got covid

select location, date, population, total_cases, (total_cases/population)*100 as casepercent
from covid19..coviddeaths
where location like '%states%'
order by 1,2


--loking at countries with highest infaection rate compared to population


select location, population,max(total_cases) as highestinfectioncount,max((total_cases/population))*100 as popinfected
from covid19..coviddeaths
--where location like '%states%'
group by location, population
order by popinfected desc

--showing countries with highest death counts per population


select location, max(total_deaths) as totaldeathcount
from Covid19..coviddeaths
--where location like '%states%'
group by location
order by totaldeathcount desc


--breaking things down by continent


select continent, max(cast(total_deaths as int)) as totaldeathcount
from Covid19..coviddeaths
--where location like '%states%'
where continent is not null
group by continent
order by totaldeathcount desc


--continents with highest death count per population

select continent, max(cast(total_deaths as int)) as totaldeathcount
from Covid19..coviddeaths
--where location like '%states%'
where continent is not null
group by continent
order by totaldeathcount desc


--global numbers

select sum(new_cases) as total_cases, sum(cast(new_deaths as int)) as total_deaths, 
sum(cast(new_deaths as int))/sum(new_cases)*100 as deathpercentage
from covid19..coviddeaths
where continent is not null
order by 1,2


--total population vs vaccinations

select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, sum(cast(vac.new_vaccinations as int)) over (partition by dea.location 
order by dea.location, dea.date) as rollingpeoplevaccinated
--(rollingpeoplevaccinated/population) * 100
from covid19..coviddeaths dea
join covid19..covidvaccinations vac
on dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
order by 2,3


use CTE

with popvsvac (continent, location, date, population, new_vaccinations, rollingpeoplevaccinated)
as
(select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, sum(cast(vac.new_vaccinations as int)) over (partition by dea.location 
order by dea.location, dea.date) as rollingpeoplevaccinated
--(rollingpeoplevaccinated/population) * 100
from covid19..coviddeaths dea
join covid19..covidvaccinations vac
on dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
--order by 2,3 
)
select *,( rollingpeoplevaccinated/population)*100
from popvsvac



--temp table

drop table if exists #percentpopulationvaccinated
create table #percentpopulationvaccinated
(
continent nvarchar(255),
location nvarchar(255),
date datetime,
population numeric,
new_vaccinations numeric,
rollingpeoplevaccinated numeric
)


insert into #percentpopulationvaccinated
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, sum(cast(vac.new_vaccinations as int)) over (partition by dea.location 
order by dea.location, dea.date) as rollingpeoplevaccinated
--(rollingpeoplevaccinated/population) * 100
from covid19..coviddeaths dea
join covid19..covidvaccinations vac
on dea.location = vac.location
and dea.date = vac.date
--where dea.continent is not null
--order by 2,3 

select *,( rollingpeoplevaccinated/population)*100
from #percentpopulationvaccinated
