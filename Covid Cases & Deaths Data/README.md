
# COVID-19 Death & Infection Rate Queries 

## Overview
This documentation provides a summary of the SQL queries used to analyze COVID-19 data from the [CovidDeaths file](CovidDeaths.xlsx) and [CovidVaccinations file](CovidVaccinations.xlsx) datasets for a Tableau dashboard. The queries extracts total cases, deaths, vaccination trends, and infection percentages. 

Here is a picture of the dashboard

![picture of the dashboard](https://github.com/user-attachments/assets/922f7c60-f288-45f8-8c61-0aaa3720ef1f) 

and a [link to the dashboard on Tableau public](https://public.tableau.com/app/profile/lateefah8519/viz/CovidRecentDataDashboard/CovidRecentDataDashboard)

## SQL Query Descriptions

### 1. Rolling Vaccination Count Per Country
This query retrieves the total cumulative vaccinations per location on a given date. It joins the `CovidDeaths` and `CovidVaccinations` tables to find the highest recorded vaccination count (`MAX(total_vaccinations)`) while ensuring that only country-level data is included by filtering out world-level data. This query helps track the progress of vaccination campaigns globally.

```sql
Select dea.continent, dea.location, dea.date, dea.population,
       MAX(vac.total_vaccinations) as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
    On dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null 
group by dea.continent, dea.location, dea.date, dea.population
order by 1,2,3
```

### 2. Total Cases, Total Deaths, and Death Percentage (**Used in Tableau**)
This query calculates the total number of COVID-19 cases and deaths worldwide. Additionally, it computes the death percentage by dividing total deaths by total cases. It excludes global, European Union, and international data to focus only on country-level analysis. The result is essential for understanding the overall impact of COVID-19.

```sql
Select SUM(new_cases) as total_cases, 
       SUM(cast(new_deaths as int)) as total_deaths, 
       SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
where continent is not null 
order by 1,2
```

### 3. Global Data Validation Check
This commented-out query is a validation check for global statistics. It calculates total cases and deaths specifically for cases where `location = 'World'` to cross-check global aggregate numbers against other analyses. It serves as an internal consistency check.

```sql
-- Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, 
--        SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
-- From PortfolioProject..CovidDeaths
-- where location = 'World'
-- order by 1,2
```

### 4. Death Count Per Continent (**Used in Tableau**)
This query summarizes the total death count for each continent, excluding world-level data. It ensures that only continents are included in the analysis and ranks them in descending order of total deaths. This data helps in understanding which continents were most affected by COVID-19.

```sql
Select location, SUM(cast(new_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
Where continent is null 
and location not in ('World', 'European Union', 'International')
Group by location
order by TotalDeathCount desc
```

### 5. Highest Infection Count and Population Infection Percentage
This query finds the highest number of recorded COVID-19 cases for each country and calculates the percentage of the population that was infected. The results are sorted by the highest infection rates, making it useful for identifying the countries most impacted by the pandemic.

```sql
Select Location, Population, MAX(total_cases) as HighestInfectionCount,  
       Max((total_cases/population))*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
Group by Location, Population
order by PercentPopulationInfected desc
```

### 6. Death Count Per Continent with Additional Details (**Used in Tableau**)
This query extends the previous death count analysis by including population data along with total cases and deaths. It provides a detailed view of each location by date, making it valuable for trend analysis and visualization in Tableau.

```sql
Select Location, date, population, total_cases, total_deaths
From PortfolioProject..CovidDeaths
where continent is not null 
order by 1,2
```

### 7. Rolling Vaccination Percentage Per Country
Using a window function (`SUM() OVER()`), this query calculates rolling cumulative vaccinations per country. Additionally, it computes the percentage of the population that has been vaccinated over time. This can be used for monitoring vaccination efforts and trends at the country level.

```sql
With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
    On dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null 
)
Select *, (RollingPeopleVaccinated/Population)*100 as PercentPeopleVaccinated
From PopvsVac
```

### 8. Percent Population Infected Per Country (**Used in Tableau**)
This query tracks the peak infection counts for each country over time. It calculates the percentage of the population infected and includes date-level granularity, making it useful for analyzing the spread of COVID-19 within individual countries.

```sql
Select Location, Population, date, MAX(total_cases) as HighestInfectionCount,  
       Max((total_cases/population))*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
Group by Location, Population, date
order by PercentPopulationInfected desc
```

---

## **Queries Result Used in Tableau Dashboard**

| **Query Number** | **Sections in Tableau** |
|-----------------|---------------------------|
| **2** | Total Cases & Total Deaths |
| **4** | Death Count Per Continent |
| **6** | Death Count Per Continent (with extra details) |
| **8** | Percent of the Population Infected Per Country |

---
