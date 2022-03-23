# ⚕️ Used SQL Server to explore global COVID-19 data :earth_americas:

# :pushpin: Solutions

1.  **Shows likehood of dying if you contract COVID-19 in your country**

```sql
SELECT Location, date, total_cases, total_deaths, (Total_deaths/total_cases)*100 AS DeathPercentage
FROM PortfolioProject1..[CovidDeaths]
WHERE location LIKE '%states%'
ORDER BY 1,2;
```

***

2.  **Looking at Total Cases vs Population**

```sql
SELECT Location, date, total_cases, Population, (total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject1..[CovidDeaths]
WHERE location LIKE '%states%'
ORDER BY 1,2;
```

***

3.  **Looking at countries with highest infection rate compared to population**

```sql
SELECT Location, Population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases/population))*100 AS PercentPopulationInfected
FROM PortfolioProject1..[CovidDeaths]
GROUP BY Location, Population
ORDER BY PercentPopulationInfected desc;
```

***

4.  **Showing Countries with Highest Death Count per Population**

```sql
SELECT Location, MAX(cast(Total_deaths as int)) AS TotalDeathCount --Cast was needed to convert totaldeaths from nvarchar to interger
FROM PortfolioProject1..[CovidDeaths]
WHERE continent is not null -- This was used since N.America, S.America, Whole World locations appeared in results
GROUP BY Location
ORDER BY TotalDeathCount desc;
```

***

5.  **Showing continents with the highest death count per population**

```sql
SELECT continent, MAX(cast(Total_deaths as int)) AS TotalDeathCount 
FROM PortfolioProject1..[CovidDeaths]
WHERE continent is not null 
GROUP BY continent
ORDER BY TotalDeathCount desc;
```

***

6.  **Looking at Total Population Vs Vaccinations**

```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(convert(bigint,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) AS RollingPeopleVaccinated
--,(RollingPeopleVaccinated/population
FROM PortfolioProject1..CovidVaccinations vac
JOIN PortfolioProject1..CovidDeaths dea
ON dea.location = vac.location
and dea.date = vac.date
WHERE dea.continent is not null
ORDER BY 2,3;
```

***

7.  **Looking at Total Population Vs Vaccinations with Rolling Count of People Vaccinated**

```sql
With PopvsVac(Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as 
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(convert(bigint,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) AS RollingPeopleVaccinated
--,(RollingPeopleVaccinated/population
FROM PortfolioProject1..CovidVaccinations vac
JOIN PortfolioProject1..CovidDeaths dea
ON dea.location = vac.location
and dea.date = vac.date
WHERE dea.continent is not null
--ORDER BY 2,3
)
Select *,(RollingPeopleVaccinated/Population)*100 AS PercentVaccinated
FROM PopvsVac;
```

***

8.  **Looking at Total Population Vs Vaccinations with Rolling Count of People Vaccinated (Done with Temp Table)**

```sql
DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vacciantions numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(convert(bigint,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) AS RollingPeopleVaccinated
--,(RollingPeopleVaccinated/population
FROM PortfolioProject1..CovidVaccinations vac
JOIN PortfolioProject1..CovidDeaths dea
ON dea.location = vac.location
and dea.date = vac.date
--WHERE dea.continent is not null
--ORDER BY 2,3

Select *,(RollingPeopleVaccinated/Population)*100
FROM #PercentPopulationVaccinated;
```

***
