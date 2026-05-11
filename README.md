# Drug Overdose Mortality Analysis

This project analyzes drug overdose mortality data using SQL and Excel.

The goal of the project was to practice working with structured public health data, write SQL queries for data exploration, and create visualizations that summarize key patterns in overdose deaths. The analysis focuses on toxic substances, opioid involvement, drug combinations, demographics, cities, and monthly death distribution.

- SQL queries: [sql_files folder](/sql_files/)
- Excel file: [excel_file folder](/excel_file/)
- Visualizations: [assets folder](/assets/)

## Project Structure

```text
Drug-Overdose-Mortality-Analysis/
├── assets/
│   ├── 1.png
│   ├── 2.png
│   ├── 3.png
│   ├── 4.png
│   ├── 5.png
│   ├── 6.png
│   ├── 7.png
│   ├── 8.png
│   ├── 9.png
│   └── 10.png
├── csv_files/
│   ├── Demographics_and_Location.csv
│   └── Substances_Detected.csv
├── excel_file/
│   └── Overdose_Data_Project.xlsx
├── sql_files/
│   ├── 10_monthly_death_distribution.sql
│   ├── 1_top_toxic_substances.sql
│   ├── 2_common_drug_combinations.sql
│   ├── 3_opioid_percentage.sql
│   ├── 4_opioid_deaths_by_age_group.sql
│   ├── 5_fentanyl_deaths_by_race.sql
│   ├── 6_deaths_by_gender.sql
│   ├── 7_deaths_by_city.sql
│   ├── 8_top_substance_by_city.sql
│   └── 9_age_distribution.sql
├── .gitignore
└── README.md
```

## Questions Answered

The SQL queries in this project answer the following questions:

1. Which toxic substances are responsible for the highest number of overdose deaths?
2. What drug combinations are most common among overdose cases?
3. What percentage of overdose deaths involve opioids?
4. How are opioid-related deaths distributed across age groups?
5. What is the racial distribution of fentanyl-related deaths?
6. What is the gender distribution in drug overdose deaths?
7. How do drug-related deaths vary by city?
8. Which drug is the leading substance in each city?
9. What is the overall age distribution of overdose deaths?
10. Which months record the highest number of overdose deaths?

## Tools Used

- **SQL** — for querying, filtering, grouping, and aggregating structured data
- **PostgreSQL** — as the relational database system
- **Excel** — for charts and visualizations
- **Visual Studio Code** — for writing and organizing SQL scripts
- **Git & GitHub** — for version control and project documentation

## SQL Concepts Practiced

This project helped me practice:

- `SELECT`, `WHERE`, `GROUP BY`, `ORDER BY`
- aggregate functions such as `COUNT`, `SUM`, and percentage calculations
- `CASE` expressions for categorization
- Common Table Expressions using `WITH`
- `JOIN` operations between related tables
- `UNION ALL` for combining substance-level results
- window functions such as `ROW_NUMBER()`
- filtering null values
- organizing multiple SQL scripts by analytical question

## Analysis and Results

### 1. Which toxic substances are responsible for the highest number of overdose deaths?

To identify the most frequently detected substances in overdose deaths, I queried the toxicology data and counted how often each substance appeared in fatal cases. I used a `UNION ALL` approach to combine counts across multiple substance columns and then ordered the results by death count.

![Top Toxic Substances](assets/1.png)

#### Key Findings

- Opioids were the most common substance category, with more than 8,800 cases.
- Fentanyl appeared in around 8,000 deaths.
- Cocaine and heroin were also major contributors.
- Other substances such as ethanol, benzodiazepines, and xylazine appeared frequently as well.

### 2. What drug combinations are most common among overdose cases?

To analyze multi-substance cases, I created a query that checked which substances were detected in each case and built readable drug-combination labels. Only combinations involving two or more substances were included.

![Top 10 Common Drug Combinations](assets/2.png)

#### Key Findings

- Fentanyl + any opioid was the most common combination.
- Cocaine + fentanyl + any opioid was also highly frequent.
- Other common combinations included fentanyl with ethanol, benzodiazepines, or xylazine.
- Several high-frequency combinations involved three or more substances.

### 3. What percentage of overdose deaths involve opioids?

To calculate opioid involvement, I used CTEs to count opioid-related deaths and total deaths, then calculated the percentage.

```sql
WITH opioid_deaths AS
(
    SELECT 
        COUNT(*) AS opioid_count
    FROM toxicology_results
    WHERE
        any_opioid = TRUE
), 
total_deaths AS 
(
    SELECT
        COUNT(*) AS total_count
    FROM toxicology_results
)
SELECT 
    ROUND(100.0 * opioid_deaths.opioid_count / total_deaths.total_count, 2) AS opioid_percent
FROM opioid_deaths, total_deaths;
```

![Opioid Involvement in Overdose Deaths](assets/3.png)

#### Key Findings

- Approximately 73.68% of overdose deaths involved at least one opioid.
- Non-opioid cases accounted for approximately 26% of the dataset.
- Opioids were the dominant substance category in the data.

### 4. How are opioid-related deaths distributed across age groups?

To analyze opioid-related deaths by age, I grouped individuals into age categories and calculated the share of opioid-related cases within each group.

![Share of Opioid-Related Deaths by Age Group](assets/4.png)

#### Key Findings

- Teens had a high proportion of opioid-related deaths.
- Adults also showed a high rate of opioid involvement.
- Opioids appeared across all age groups, not only among one demographic segment.

### 5. What is the racial distribution of fentanyl-related deaths?

To analyze fentanyl involvement by race, I joined the demographic table with the toxicology table and calculated fentanyl-related death percentages by racial group.

![Fentanyl-Related Death Rate by Race](assets/5.png)

#### Key Findings

- Some racial groups showed very high fentanyl-involvement rates.
- Smaller groups may show extreme percentages because of low sample size.
- Fentanyl appeared as a major substance across multiple demographic groups.

### 6. What is the gender distribution in drug overdose deaths?

To examine overdose deaths by gender, I grouped records by sex and calculated each group’s share of total deaths.

```sql
WITH deaths_by_sex_cte AS
(
    SELECT
        sex,
        COUNT(*) AS deaths
    FROM victim_profiles
    GROUP BY
        sex
    HAVING
        COUNT(*) > 10
)
SELECT
    dbs.sex,
    dbs.deaths,
    ROUND(dbs.deaths * 100.0 / 
    (
        SELECT 
            COUNT(*) AS total_deaths
        FROM victim_profiles
    ), 2) AS deaths_percentage
FROM deaths_by_sex_cte AS dbs;
```

![Share of Overdose Deaths By Gender](assets/6.png)

#### Key Findings

- Around 74% of overdose deaths were male.
- Around 26% of overdose deaths were female.
- Male deaths were nearly three times higher than female deaths in this dataset.

### 7. How do drug-related deaths vary by city?

To identify cities with the highest overdose death counts, I grouped records by `death_city`, excluded null values, and sorted the results by total deaths.

```sql
SELECT
    death_city,
    COUNT(*) AS deaths
FROM victim_profiles
WHERE
    death_city IS NOT NULL
GROUP BY
    death_city
ORDER BY
    deaths DESC;
```

![Top 10 Cities](assets/7.png)

#### Key Findings

- Hartford, New Haven, and Waterbury had the highest overdose death counts.
- Bridgeport, New Britain, and Bristol also appeared among the cities with high death counts.
- Death counts varied significantly by location.

### 8. Which drug is the leading substance in each city?

To find the leading substance by city, I counted deaths per substance per city using multiple `UNION ALL` queries. Then I used `ROW_NUMBER()` to rank substances inside each city and return the top substance.

![Top Drugs by Number of Cities](assets/8.png)

#### Key Findings

- Any opioid was the leading substance in the largest number of cities.
- Fentanyl was the second most common leading substance by city.
- Other substances such as heroin, ethanol, and benzodiazepines appeared as leading substances in fewer cities.
- The result shows variation in substance patterns across locations.

### 9. What is the overall age distribution of overdose deaths?

To analyze the age distribution, I grouped individuals into age categories using a `CASE` expression and calculated both raw counts and percentages.

```sql
WITH age_groups_cte AS
(
    SELECT
        CASE 
            WHEN age BETWEEN 10 AND 20 THEN 'teen'
            WHEN age BETWEEN 20 AND 50 THEN 'adult'
            ELSE 'elderly'
        END AS age_group,
        COUNT(*) AS total
    FROM victim_profiles
    GROUP BY
        CASE 
            WHEN age BETWEEN 10 AND 20 THEN 'teen'
            WHEN age BETWEEN 20 AND 50 THEN 'adult'
            ELSE 'elderly'
        END
)
SELECT
    age_group,
    total,
    ROUND(total * 100.0 / (SELECT COUNT(*) FROM victim_profiles), 2) AS deaths_percentage
FROM age_groups_cte;
```

![Overdose Deaths by Age Group](assets/9.png)

#### Key Findings

- Adults made up the largest share of overdose deaths.
- The elderly group also represented a meaningful share.
- Teens had the lowest representation in the dataset.

### 10. Which months record the highest number of overdose deaths?

To analyze monthly distribution, I extracted the month from each death record and counted the number of deaths by month.

```sql
SELECT
    EXTRACT(MONTH FROM date) AS month,
    COUNT(*) AS deaths
FROM victim_profiles
GROUP BY
    EXTRACT(MONTH FROM date)
ORDER BY
    deaths DESC;
```

![Monthly Overdose Deaths](assets/10.png)

#### Key Findings

- July had the highest number of overdose deaths.
- May, June, and November also showed elevated death counts.
- January and February had the lowest death counts in the dataset.

## What I Learned

Through this project, I improved both my SQL and data visualization skills.

The main technical skills practiced were:

- writing structured SQL queries
- using CTEs to organize query logic
- combining results with `UNION ALL`
- joining demographic and toxicology tables
- using window functions for ranking
- grouping data by category, city, age group, and month
- calculating percentages from aggregated results
- creating Excel visualizations from SQL query outputs
- documenting a data project clearly on GitHub

## Conclusion

This project demonstrates practical SQL skills using a public overdose mortality dataset.

The project combines PostgreSQL queries with Excel visualizations to explore overdose deaths by substance, drug combination, opioid involvement, demographics, geography, and month.

While the project is analytical in nature, the technical skills used here are relevant for software engineering and backend development, especially working with relational databases, writing structured queries, transforming data, and organizing project files.
