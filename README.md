# Introduction
📊 Dive into the data job market! Focusing on data analyst roles this project explores 💰 top-paying jobs, 🔥 in-demand skills, and 📈 where high demand meets high salary in data analytics. 

🔍 SQL Queries? Check them out here: [SQL folder](/SQL/)
# Background
Driven by a quest to navigate the data analyst job market more effectively, this project was born from a desire to pinpoint top-paid and in-demand skills, streamlining others work to find optimal jobs. 

Data hails from my [SQL Course](https://lukebarousse.com/sql). It's packed with insights on job titles, salaries, locations, and essential skills

### The questions I wanted to answer thorugh my SQL queries were:

    1. What are the top-paying data analyst jobs?
    2. What skills are required for these top-paying jobs?
    3. What skills are most in demand for data analysts?
    4. Which skills are associated with higher salaries?
    5. What are the most optimal skills to learn?

# Tools I Used
For my deep dive into the data analyst job market, I harnessed the power of several key tools:

- **SQL**: The backbone of my analysis, allowing me to query the database and unearth critical insights.
- **PostgreSQL**: The choosen database management system, ideal for handling the job postings data.
- **Visual Studio Code**: My go-to for database management and executing SQL queries.
- **Git & GitHub**: Essential for version control and sharing my SQL scripts and analysis, ensuring collaboration and project tracking. 
# The Analysis
Each query for this project aimed at investigating specific aspects of the data analyst job market. Here's how I approached each question:

### 1. Top Paying Data Analyst Jobs
To identify the highest-paying roles, I filtered data analyst positions by average yearly salary and location, focusing on remote jobs. This query highlights the high paying opportunities in the field. 

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON company_dim.company_id = job_postings_fact.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location LIKE '%San Jose%' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

Here's the breakdown of the top data analyst jobs in 2023:
- **Wide Salary Range:** Top 10 paying data analyst roles span from $184,000 to $650,000, indicating significant salary potential in the field.
- **Diverse Employers:** Companies like SmarAsset, Meta, and AT&T are among those offering high salaries, showing a broad interest across different industries.
- **Job Title Variety:** There's a high diversity in job titles, from Data Analyst to Directos of Analytics, reflecting varied roles and specializations within data analytics. 

![Top Paying Roles](assets\top_paying_roles.png)
*Bar graph visualizing the salary for the top 10 salaries for data analysits; ChatGPT generated this graph from my SQL query results*

### 2. Skills for Top Paying Jobs

Employers value for high-compensation roles.

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON company_dim.company_id = job_postings_fact.company_id
    WHERE
        job_title_short = 'Data Analyst' AND
        job_location = 'Anywhere' AND
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
)

SELECT top_paying_jobs.*,
       skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_dim.skill_id = skills_job_dim.skill_id
ORDER BY
        salary_year_avg DESC
```

Here's the breakdown of the most demanded skills for the top 10 highest paying data analyst jobs in 2023:

- SQL is leading with a bold count of 398.
- Excel follows with a bold count of 256.
- Python remains a highly sought-after skill, with a bold count of 236.
- Tableau is also in demand, with a bold count of 230.Other skills like R, Snowflake, Pandas, and Excel show varying degrees of demand.

![Skills for Top Paying Jobs](assets\Skill Count for top 10 paying DA jobs.png)

### 3. Most Demanded Skills for Data Analysts

Retrieves the top 5 skills with the highest demand in the job market, 
providing insights into the most valuable skills for job seekers.

```sql
SELECT 
   skills,
   COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_dim.skill_id = skills_job_dim.skill_id
WHERE 
   job_postings_fact.job_title_short = 'Data Analyst' AND
   job_work_from_home = TRUE
GROUP BY
   skills
ORDER BY
   demand_count DESC
LIMIT 5
```
Here's the breakdown for most demanded skills for Data Analysts
- **SQL** and **Excel** remain fundamental, emphasizing the need for strong foundational skills in data processing and spreadsheet manipulation. 
- **Programming** and **Visualization Tools** like **Python**, **Tableau**, and **Power BI** are essential, pointing towards the increasing importance of technical skills in data storytelling and decision support.

| Skills   | Demand Count |
|----------|-------------|
| SQL      | 7291        |
| Excel    | 4611        |
| Python   | 4330        |
| Tableau  | 3745        |
| Power BI | 2609        |

*Table of the demand for the top 5 skills in data analyst job postings*

### 4. Skills Based on Salary

Exploring the average salaries associated with different skills revealed which skills are the highest paying

```sql
SELECT
    skills_dim.skills,
    ROUND(AVG(jp.salary_year_avg), 0) AS avg_salary
FROM
    job_postings_fact AS jp
        INNER JOIN skills_job_dim ON skills_job_dim.job_id = jp.job_id
        INNER JOIN skills_dim ON skills_dim.skill_id = skills_job_dim.skill_id
WHERE
    jp.salary_year_avg IS NOT NULL AND
    jp.job_title_short = 'Data Analyst'
    AND job_work_from_home = TRUE
GROUP BY
    skills_dim.skills
ORDER BY
    avg_salary DESC
LIMIT 25
```

Here's a breakdown of the results for top paying skills based on salary
- High Demand for Big Data & ML Skills: Top salaries are commanded by analysts skilled in big data tech.
- Software Development & Deployment Proficiency: Knowledge in development and deployment tools
- Cloud Computing Expertise: Familiarity with cloud and data engineering tools

### 5. Most Optimal Skills to Learn

Combining insights from demand and salary data, this query aimed to pinpoint skills that are both in high demand and have high salaries, offering a strategic focus for skill development.

```sql
SELECT
    skills_dim.skill_id,
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count,
    ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = TRUE
GROUP BY
    skills_dim.skill_id
HAVING
    COUNT(skills_job_dim.job_id) > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```

| Skill ID | Skills     | Demand Count | Avg Salary ($) |
|----------|-----------|--------------|---------------:|
| 8        | Go        | 27           | 115320        |
| 234      | Confluence| 11           | 114210        |
| 97       | Hadoop    | 22           | 113193        |
| 80       | Snowflake | 37           | 112948        |
| 74       | Azure     | 34           | 111225        |
| 77       | BigQuery  | 13           | 109654        |
| 76       | AWS       | 32           | 108317        |
| 4        | Java      | 17           | 106906        |
| 194      | SSIS      | 12           | 106683        |
| 233      | Jira      | 20           | 104918        |
| 79       | Oracle    | 37           | 104534        |
| 185      | Looker    | 49           | 103795        |
| 2        | NoSQL     | 13           | 101414        |
| 1        | Python    | 236          | 101397        |
| 5        | R         | 148          | 100499        |
| 78       | Redshift  | 16           | 99936         |
| 187      | Qlik      | 13           | 99631         |
| 182      | Tableau   | 230          | 99288         |
| 197      | SSRS      | 14           | 99171         |
| 92       | Spark     | 13           | 99077         |
| 13       | C++       | 11           | 98958         |
| 186      | SAS       | 63           | 98902         |
| 7        | SAS       | 63           | 98902         |
| 61       | SQL Server| 35           | 97786         |
| 9        | JavaScript| 20           | 97587         |

*Table of the most optimal skills for data analyst sorted by salary*

Here's a breakdown of the most optimal skills for Data Analysts in 2023:
- **High-Demand Programming Languages:** Python and R stand out for their high deman, with demand counts of 236 and 148 respectively. Despite their high demand, their average salaries are around $101,397 for Python and $100,499 for R, indicating that proficiency in these languages is highly valued but also widely available. 
- **Cloud Tools and Technologies:** Skills in speclialized technologies such as Snowflake, Azure, AWS, and BigQuery show significant demand with relatively high average salaries, pointing towards the growing importance of cloud platforms and big data technologies in data analysis.
-- **Business Intelligence and Visualization Tools:** Tableau and Looker with demand counts of 230 and 49 respectively, and average salaries around $99,288 and $103,795, highlight the critical role of data visualization and business intelligence in deriving actionable insights from data.
-- **Database Technologies:** The demand for skills in traditional and NoSQL databases (Oracle, SQL Server, NoSQL) with average salaries ranging from $97,786 to $104,534, reflects the enduring need for data storage, retrieval, and management expertise.
# What I Learned

Throughout this adventure, I've turbocharged my SQL toolkit with some serious firepower:

- **🧩 Complex Query Crafting:** Mastered the art of advanced SQL, merging tables like a pro and wielding WITH clauses for ninja-level temp table maneuvers.
- **📊 Data Aggregation:** Got cozy with GROUP BY and turned aggregate functions like COUNT() and AVG() into my data_summarizing sidekics.
- **💡 Analytical Wizardry:** Leveled up my real-world puzzle-solving skills, turning questions into actionable, insightful SQL queries.

# Conclusion

### Insights
From the analysis, several general insights emerged:

1. **Top-Paying Data Analyst Jobs**: The highest-paying jobs for data analysts that allow remote work offer a wide range of salaries, the highest at $650,000!
2. **Skills for Top-Paying Jobs**: High-paying data analyst jobs require advanced proficiency in SQL, suggesting it's a critical skill for earning a top salary. 
3. **Most In-Demand Skills**: SQL is also the most demanded sill in the data analyst job market, thus making it essential for job seekers.
4. **Skills with Higher Salaries**: Specialized skills, such as SVN and Solidity, are associated with the highest average salaries, indicating a premium on niche expertise. 
5. **Optimal Skills for Job Market Value**: SQL leads in demand and offers for a high average salary, positioning it as one of the most optimal skills for data analysts to learn to maximize their market value. 

### Closing Thoughts
This project enhanced my SQL skillls and provided valuable insights into the data analyst job market. The findings from the analysis serve as a guide to priortizing skill development and job search efforts. Aspiring data analysts can better position themselves in a competitive job market by focusing on high-demand, high-salary skills. This exploration highlights the importance of continuous learning and adaptation to emerging trends in the field of data analytics.