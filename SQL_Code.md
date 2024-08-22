~~~ SQL

-- SQL CODE

-- Creating the tables and adding data 
	
CREATE TABLE country_list (
	   country_code  varchar primary key,
       country  varchar);


  
--Importing data from csv
COPY public.country_list
FROM 'C:\Users\Usuario\Desktop\Country_List.csv'
WITH CSV HEADER


---Creating the second table
CREATE TABLE energy_production (
  country_code varchar,
  year integer,
  geothermal_energy numeric,
  electricity_generation numeric,
  biofuel_electricity numeric,
  coal_electricity numeric,
  fossil_electricity numeric,
  gas_electricity numeric,
  hydro_electricity numeric,
  nuclear_electricity numeric,
  oil_electricity numeric,
  other_renewable_electricity numeric,
  other_renewable_exc_biofuel_electricity numeric,
  renewables_electricity numeric,
  solar_electricity numeric,
  wind_electricity numeric,
  gas_production numeric,
  oil_production numeric,
  oil_prod_per_capita numeric );


--importing data from csv
COPY public.energy_production
FROM 'C:\Users\Usuario\Desktop\Energy_Production.csv'
WITH CSV HEADER




-- Inspecting the data

Select *
From public.country_list;

Select *
From public.energy_production;




-- Checking for unique values

SELECT Count(distinct country_code) 
FROM public.energy_production;




SELECT Count(distinct country) 
FROM public.country_list;
-- 223 country_codes and country names in the energy_production table




SELECT Count(distinct country_code) 
FROM public.country_list;
-- 223 country_code in coutry_list table



--Finding the min and max years


SELECT min(year)
FROM public.energy_production
--1900  is the min year 


SELECT max(year)
FROM public.energy_production
--2020  is the max year 





--Analysis
-- 1. How much energy was produced in Central America by energy type, from year 2000 to 2020?

--- Creating a CTE to get the total production of energy, organized by country.
WITH cte_total_energy AS( 
     SELECT 
	 country_code,
	 SUM(wind_electricity+
	 solar_electricity+
	 biofuel_electricity+ 
	 hydro_electricity+
	 geothermal_energy+ 
	 other_renewable_exc_biofuel_electricity+
     oil_electricity+
     gas_electricity+
     nuclear_electricity+
     coal_electricity) 
		AS total_sum
	 FROM 
	  	public.energy_production
    	 GROUP BY 
	  	country_code)
--Finding the percentage of energy produced in Central America by source 		
SELECT 
	country,
ROUND(	
	SUM(wind_electricity) 
	/
	total_sum *100,2) AS "% wind_electricity_terawatt-hours",
ROUND( 
	SUM(solar_electricity) 
	/
	total_sum *100,2) AS "% solar_electricity_terawatt-hours",
ROUND( 	
	SUM(biofuel_electricity) 
	/
	total_sum *100,2) AS "% biofuel_electricity_terawatt-hours",
ROUND( 
	SUM(hydro_electricity) 
	/
	total_sum *100,2) AS "% hydro_electricity_terawatt-hours",
ROUND( 
	SUM(geothermal_energy) 
	/
	total_sum *100,2) AS "% geothermal_energy_terawatt-hours",
ROUND( 
	SUM(other_renewable_exc_biofuel_electricity) 
	/
	total_sum *100,2) AS "% other_renewable_exc_biofuel_electricity_terawatt-hours",
ROUND( 
	SUM(oil_electricity) 
	/
	total_sum *100,2) AS "% oil_electricity_terawatt-hours",
ROUND( 
	SUM(gas_electricity) 
	/
	total_sum *100,2) AS "% gas_electricity_terawatt-hours",
ROUND( 
	SUM(nuclear_electricity) 
	/
	total_sum *100,2) AS "% nuclear_electricity_terawatt-hours",
ROUND( 
	SUM(coal_electricity) 
	/
	total_sum *100,2) AS "% coal_electricity_terawatt-hours"	
FROM 
	cte_total_energy total
INNER JOIN 
	public.energy_production prod
ON total.country_code = prod.country_code
INNER JOIN
	public.country_list list 
ON prod.country_code = list.country_code
WHERE
	country in ('Costa Rica','Nicaragua','Panama','Belize', 'Guatemala', 'El Salvador', 'Honduras') 
AND
	year BETWEEN 2000 and 2020
GROUP BY 
	total_sum,
	list.country_code;



-- 2. What is the energy production trend in Central America since 2000 to 2020 by source?
SELECT 
	country,
	Year,
	wind_electricity, 
	solar_electricity,
	biofuel_electricity,
	hydro_electricity,
	geothermal_energy,
	other_renewable_exc_biofuel_electricity,
	oil_electricity,
	gas_electricity,
	nuclear_electricity,
	coal_electricity
FROM
	public.energy_production prod
inner join
	public.country_list list  On prod.country_code = list.country_code
WHERE 
	country in ('Costa Rica','Nicaragua','Panama','Belize', 'Guatemala', 'El Salvador', 'Honduras')
AND
    year BETWEEN 2000 and 2020;




--3. Finding the among of renewable and non renewable energy that is produced in Central America, from 2000 to 2020.
SELECT
	country,
	SUM(wind_electricity+solar_electricity+biofuel_electricity+hydro_electricity+geothermal_energy+other_renewable_exc_biofuel_electricity)
		AS Renewable_Energy,
 	SUM(oil_electricity+gas_electricity+coal_electricity) 
		AS Fossil_energy
FROM
	public.energy_production prod
inner join
	public.country_list list 
On prod.country_code = list.country_code
WHERE 
	country in ('Costa Rica','Nicaragua','Panama','Belize', 'Guatemala', 'El Salvador', 'Honduras')
AND
	year BETWEEN 2000 and 2020
GROUP BY country;




--4. What is the percentage of energy produced form renewable sources and fossil sources in countries of Central America (2000-2020)
SELECT 
	country, 
	ROUND(
		(SUM(geothermal_energy+wind_electricity+solar_electricity+biofuel_electricity+hydro_electricity+other_renewable_exc_biofuel_electricity))
		/
		(SUM(geothermal_energy+wind_electricity+solar_electricity+biofuel_electricity+hydro_electricity+other_renewable_exc_biofuel_electricity+oil_electricity+gas_electricity+coal_electricity))
		*100,2) 
			AS porcentage_renewable_energy_production,
	ROUND(
		(SUM(oil_electricity+gas_electricity+coal_electricity))
		/
		(SUM(geothermal_energy+wind_electricity+solar_electricity+biofuel_electricity+hydro_electricity+other_renewable_exc_biofuel_electricity+oil_electricity+gas_electricity+coal_electricity))
		*100,2) 
			AS porcetage_fossil_energy_production	
FROM
	public.energy_production prod
inner join
	public.country_list list 
On prod.country_code = list.country_code
WHERE 
	country in ('Costa Rica','Nicaragua','Panama','Belize', 'Guatemala', 'El Salvador', 'Honduras')
AND
	year BETWEEN 2000 and 2020
GROUP BY 
	country
ORDER BY 
 porcentage_renewable_energy_production DESC;
