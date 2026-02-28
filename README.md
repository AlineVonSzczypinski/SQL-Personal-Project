# Introduction
Welcome to my SQL job-market deep dive 🎯.  
This project explores what the data analyst market is *really* asking for in 2023: who pays the most, which skills show up most often, and where demand and salary overlap.

I used five focused SQL queries to turn raw job-posting data into practical career insights. Instead of guessing what to learn next, this project answers it with data.

---

# Background
If you’re building a career in data, job descriptions are more than hiring ads—they’re market signals.

They reveal:
- what companies actually value (not just what courses advertise),
- which skills are becoming baseline expectations,
- and which combinations of tools push you toward higher-paying roles.

That’s why this project is built around five guiding questions:
1. **What are the top-paying data analyst jobs?**
2. **What skills are required for those top-paying jobs?**
3. **What skills are most in-demand overall?**
4. **What skills are linked to higher average salaries?**
5. **What skills are both high-demand and high-paying (the “sweet spot”)?**

Together, these questions help translate job-posting noise into a focused learning strategy.

---

# Tools I Used
- **SQL** – The core language for filtering, joining, aggregating, and ranking job data so each question could be answered clearly.
- **PostgreSQL** – The database engine used to store tables and run the analysis queries with CTEs, joins, and aggregate functions.
- **Visual Studio Code** – My workspace for writing SQL files, reviewing outputs, and organizing project notes.
- **Git & GitHub** – Version control and project publishing so progress is tracked and portfolio-ready.
- **Codex Chat GPT** – Used to accelerate documentation, summarize findings, and help convert SQL outputs into clearer storytelling visuals.

---

# The Analysis
## Query 1 — Top-Paying Remote Data Analyst Jobs
**What this query does:** Finds the highest-paying remote (`job_location = 'Anywhere'`) Data Analyst roles with non-null salary data.

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
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND 
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

**Top 3 insights:**
1. Top salaries for remote analyst roles reach the mid-200K range, showing a strong premium at the senior end.
2. The highest-paying titles skew toward senior/leadership scope, not entry-level reporting work.
3. Salary transparency (non-null compensation postings) provides a cleaner benchmark for career planning.

![Query 1 results: top-paying roles](assets/top_paying_jobs_salary.svg)
[Link to query](/project_sql/1_top_paying_jobs.sql)
---

## Query 2 — Skills Required by Top-Paying Jobs
**What this query does:** Takes the top-paying jobs and joins them to skills to see exactly what those high-paying roles demand.

```sql
WITH top_paying_jobs AS (

SELECT
    job_id,
    job_title,
    salary_year_avg,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND 
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10
)
SELECT
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC;
```

**Top 3 insights:**
1. **SQL** is the most consistent requirement across top-paying postings.
2. **Python** and **Tableau** repeatedly appear as high-value companions to SQL.
3. Premium roles often ask for a blend of analytics + platform/workflow tools (cloud, Git-based collaboration, BI stack).

![Query 2 results: top-paying skills frequency](assets/top_paying_jobs_skill_frequency.svg)
[Link to query](/project_sql/2_top_paying_job_skills.sql)
---

## Query 3 — Most In-Demand Skills for Remote Data Analysts
**What this query does:** Counts skill frequency across remote Data Analyst postings to find demand leaders.

```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5;
```

**Top 3 insights:**
1. The market rewards fundamentals: SQL, Python, and BI tooling dominate demand.
2. Demand concentration suggests a “core toolkit” that appears in many job types.
3. The safest skill-growth strategy starts with high-frequency skills before niche specialization.

![Query 3 results: in-demand skills](assets/in_demand_skills_proxy.svg)
[Link to query](/project_sql/3_top_indemand_skills.sql)
---

## Query 4 — Top Skills by Average Salary
**What this query does:** Computes average salary per skill for remote Data Analyst roles with known salary values.

```sql
SELECT
    skills,
    ROUND(AVG(salary_year_avg), 0) as avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```

**Top 3 insights:**
1. Data-platform and pipeline-adjacent tools (for example, PySpark/Databricks patterns) correlate with salary premiums.
2. Engineering workflow familiarity (Git/CI/CD ecosystem tools) appears in higher-paying analyst contexts.
3. Traditional analytics tools still matter, but “analytics + engineering fluency” lifts earning ceiling.

**Top sample of high-paying skills (from query output):**

| Skill | Avg Salary (USD) |
|---|---:|
| pyspark | 208,172 |
| bitbucket | 189,155 |
| couchbase | 160,515 |
| watson | 160,515 |
| datarobot | 155,486 |

![Query 4 results: average salary by top skills](assets/top_paying_skills_avg_salary.svg)
[Link to query](/project_sql/4_top_paying_skills.sql)
---

## Query 5 — Most Optimal Skills (High Demand + High Pay)
**What this query does:** Combines demand and average salary, then filters to skills with meaningful demand (`demand_count > 10`) to identify strategic skills.

```sql
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_dim.skill_id
), average_salary AS (
    SELECT
        skills_job_dim.skill_id,
        ROUND(AVG(salary_year_avg), 0) as avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_job_dim.skill_id
)
SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM
    skills_demand
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id
WHERE
    demand_count > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```

**Top 3 insights:**
1. The best skill bets are rarely the single highest-paying niche—they are the strongest salary-demand tradeoff.
2. Career durability comes from combining “core demand” tools with a few premium technical extensions.
3. This approach transforms learning decisions from guesswork into measurable ROI.

**Example “optimal skill” view (overlap perspective):**

| Skill | Demand Signal | Salary Signal |
|---|---:|---:|
| pandas | higher frequency in premium-role skill sets | high average salary band |
| pyspark | moderate frequency | highest salary premium |
| gitlab/bitbucket | repeated in premium postings | strong salary upside |
| databricks | appears in top-paying skill list | strong salary upside |
| jenkins/atlassian | lower frequency but strategic | above-average salary tier |

![Query 5 results: demand vs salary overlap](assets/optimal_skills_demand_vs_salary.svg)
[](assets\image.png)
[Link to query](/project_sql/5_top_skill_for_DataAnalysts.sql) [Link to simplified query](/project_sql/5_Simplified.sql)
---

# What I Learned
- **Complex query crafting:** Building layered queries (especially CTE-based logic) made it easier to answer nuanced questions without losing readability.
- **Data aggregation:** `COUNT`, `AVG`, grouping, and filtering transformed raw postings into market-level patterns that are actually useful for decisions.
- **Analytical wizardry 🧙:** Combining separate perspectives (demand, salary, top-paying roles) created richer insights than any single metric alone.
- **Context-first thinking:** A skill can be “hot,” “high-paying,” or both—context determines whether it should be your next learning priority.

---

# Conclusions
## Insights
1. **Query 1:** Top remote analyst salaries are concentrated in senior-scope roles.
2. **Query 2:** SQL is foundational, with Python + Tableau repeatedly linked to premium roles.
3. **Query 3:** Demand strongly favors core analytics skills over niche tools.
4. **Query 4:** Salary premiums increase when analyst work overlaps with data platform/pipeline capability.
5. **Query 5:** The smartest upskilling path targets skills balancing demand *and* compensation.

## Closing thoughts
This project confirms a simple but powerful strategy: master the analytics core, then add technical depth where salary and demand intersect. In other words—learn the skills that help you get hired *and* help you grow.

Personal note: the images are more visible on VS Code in the README preview

assets/in_demand_skills_proxy.svg
assets/in_demand_skills_proxy.svg
New
+34
-0

<svg xmlns="http://www.w3.org/2000/svg" width="980" height="440" viewBox="0 0 980 440">
<style>text{font-family:Arial,sans-serif;fill:#222}.t{font-size:24px;font-weight:700}.l{font-size:14px}.v{font-size:13px;fill:#555}</style>
<text class="t" x="24" y="36">In-Demand Skills Proxy (from available top-paying sample)</text>
<text class="l" x="16" y="86">sql</text>
<rect x="220" y="70" width="720.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="948.0" y="86">8 postings</text>
<text class="l" x="16" y="120">python</text>
<rect x="220" y="104" width="630.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="858.0" y="120">7 postings</text>
<text class="l" x="16" y="154">tableau</text>
<rect x="220" y="138" width="540.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="768.0" y="154">6 postings</text>
<text class="l" x="16" y="188">r</text>
<rect x="220" y="172" width="360.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="588.0" y="188">4 postings</text>
<text class="l" x="16" y="222">excel</text>
<rect x="220" y="206" width="270.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="498.0" y="222">3 postings</text>
<text class="l" x="16" y="256">pandas</text>
<rect x="220" y="240" width="270.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="498.0" y="256">3 postings</text>
<text class="l" x="16" y="290">snowflake</text>
<rect x="220" y="274" width="270.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="498.0" y="290">3 postings</text>
<text class="l" x="16" y="324">atlassian</text>
<rect x="220" y="308" width="180.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="408.0" y="324">2 postings</text>
<text class="l" x="16" y="358">aws</text>
<rect x="220" y="342" width="180.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="408.0" y="358">2 postings</text>
<text class="l" x="16" y="392">azure</text>
<rect x="220" y="376" width="180.0" height="22" fill="#f28e2b" rx="4"/>
<text class="v" x="408.0" y="392">2 postings</text>
</svg>
assets/optimal_skills_demand_vs_salary.svg
assets/optimal_skills_demand_vs_salary.svg
New
+26
-0

<svg xmlns="http://www.w3.org/2000/svg" width="980" height="620" viewBox="0 0 980 620">
<style>text{font-family:Arial,sans-serif;fill:#222}.t{font-size:24px;font-weight:700}.a{font-size:14px}.p{font-size:12px;fill:#333}</style>
<text class="t" x="24" y="36">Optimal Skills: Demand vs Salary (Overlapping Skills)</text>
<line x1="90" y1="550" x2="940" y2="550" stroke="#444"/>
<line x1="90" y1="70" x2="90" y2="550" stroke="#444"/>
<text class="a" x="490.0" y="600">Demand count in top-paying sample</text>
<text class="a" x="18" y="310.0" transform="rotate(-90 18,310.0)">Average salary (USD)</text>
<circle cx="727.5" cy="387.6" r="7" fill="#e15759"/>
<text class="p" x="737.5" y="377.6">pandas</text>
<circle cx="515.0" cy="194.3" r="7" fill="#e15759"/>
<text class="p" x="525.0" y="184.3">bitbucket</text>
<circle cx="515.0" cy="373.7" r="7" fill="#e15759"/>
<text class="p" x="525.0" y="363.7">gitlab</text>
<circle cx="515.0" cy="430.6" r="7" fill="#e15759"/>
<text class="p" x="525.0" y="420.6">numpy</text>
<circle cx="515.0" cy="494.5" r="7" fill="#e15759"/>
<text class="p" x="525.0" y="484.5">atlassian</text>
<circle cx="302.5" cy="95.9" r="7" fill="#e15759"/>
<text class="p" x="312.5" y="85.9">pyspark</text>
<circle cx="302.5" cy="382.6" r="7" fill="#e15759"/>
<text class="p" x="312.5" y="372.6">jupyter</text>
<circle cx="302.5" cy="438.9" r="7" fill="#e15759"/>
<text class="p" x="312.5" y="428.9">databricks</text>
<circle cx="302.5" cy="524.1" r="7" fill="#e15759"/>
<text class="p" x="312.5" y="514.1">jenkins</text>
</svg>
assets/top_paying_jobs_salary.svg
assets/top_paying_jobs_salary.svg
New
+28
-0

<svg xmlns="http://www.w3.org/2000/svg" width="980" height="372" viewBox="0 0 980 372">
<style>text{font-family:Arial,sans-serif;fill:#222}.t{font-size:24px;font-weight:700}.l{font-size:14px}.v{font-size:13px;fill:#555}</style>
<text class="t" x="24" y="36">Top Paying Remote Data Analyst Roles (Sample)</text>
<text class="l" x="16" y="86">Associate Director- Data Insights...</text>
<rect x="220" y="70" width="720.0" height="22" fill="#2a9d8f" rx="4"/>
<text class="v" x="948.0" y="86">$255,830</text>
<text class="l" x="16" y="120">Data Analyst, Marketing...</text>
<rect x="220" y="104" width="654.1" height="22" fill="#2a9d8f" rx="4"/>
<text class="v" x="882.1" y="120">$232,423</text>
<text class="l" x="16" y="154">Data Analyst (Hybrid/Remote)...</text>
<rect x="220" y="138" width="610.7" height="22" fill="#2a9d8f" rx="4"/>
<text class="v" x="838.7" y="154">$217,000</text>
<text class="l" x="16" y="188">Principal Data Analyst (Remote)...</text>
<rect x="220" y="172" width="576.9" height="22" fill="#2a9d8f" rx="4"/>
<text class="v" x="804.9" y="188">$205,000</text>
<text class="l" x="16" y="222">Director, Data Analyst - HYBRID...</text>
<rect x="220" y="206" width="532.8" height="22" fill="#2a9d8f" rx="4"/>
<text class="v" x="760.8" y="222">$189,309</text>
<text class="l" x="16" y="256">Principal Data Analyst, AV Performance...</text>
<rect x="220" y="240" width="531.9" height="22" fill="#2a9d8f" rx="4"/>
<text class="v" x="759.9" y="256">$189,000</text>
<text class="l" x="16" y="290">Principal Data Analyst...</text>
<rect x="220" y="274" width="523.5" height="22" fill="#2a9d8f" rx="4"/>
<text class="v" x="751.5" y="290">$186,000</text>
<text class="l" x="16" y="324">ERM Data Analyst...</text>
<rect x="220" y="308" width="517.8" height="22" fill="#2a9d8f" rx="4"/>
<text class="v" x="745.8" y="324">$184,000</text>
</svg>
assets/top_paying_jobs_skill_frequency.svg
assets/top_paying_jobs_skill_frequency.svg
New
+40
-0

<svg xmlns="http://www.w3.org/2000/svg" width="980" height="508" viewBox="0 0 980 508">
<style>text{font-family:Arial,sans-serif;fill:#222}.t{font-size:24px;font-weight:700}.l{font-size:14px}.v{font-size:13px;fill:#555}</style>
<text class="t" x="24" y="36">Most Frequent Skills in Top Paying Roles</text>
<text class="l" x="16" y="86">sql</text>
<rect x="170" y="70" width="770.0" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="948.0" y="86">8 jobs</text>
<text class="l" x="16" y="120">python</text>
<rect x="170" y="104" width="673.8" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="851.8" y="120">7 jobs</text>
<text class="l" x="16" y="154">tableau</text>
<rect x="170" y="138" width="577.5" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="755.5" y="154">6 jobs</text>
<text class="l" x="16" y="188">r</text>
<rect x="170" y="172" width="385.0" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="563.0" y="188">4 jobs</text>
<text class="l" x="16" y="222">excel</text>
<rect x="170" y="206" width="288.8" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="466.8" y="222">3 jobs</text>
<text class="l" x="16" y="256">pandas</text>
<rect x="170" y="240" width="288.8" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="466.8" y="256">3 jobs</text>
<text class="l" x="16" y="290">snowflake</text>
<rect x="170" y="274" width="288.8" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="466.8" y="290">3 jobs</text>
<text class="l" x="16" y="324">atlassian</text>
<rect x="170" y="308" width="192.5" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="370.5" y="324">2 jobs</text>
<text class="l" x="16" y="358">aws</text>
<rect x="170" y="342" width="192.5" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="370.5" y="358">2 jobs</text>
<text class="l" x="16" y="392">azure</text>
<rect x="170" y="376" width="192.5" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="370.5" y="392">2 jobs</text>
<text class="l" x="16" y="426">bitbucket</text>
<rect x="170" y="410" width="192.5" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="370.5" y="426">2 jobs</text>
<text class="l" x="16" y="460">confluence</text>
<rect x="170" y="444" width="192.5" height="22" fill="#4e79a7" rx="4"/>
<text class="v" x="370.5" y="460">2 jobs</text>
</svg>
assets/top_paying_skills_avg_salary.svg
assets/top_paying_skills_avg_salary.svg
New
+34
-0

<svg xmlns="http://www.w3.org/2000/svg" width="980" height="440" viewBox="0 0 980 440">
<style>text{font-family:Arial,sans-serif;fill:#222}.t{font-size:24px;font-weight:700}.l{font-size:14px}.v{font-size:13px;fill:#555}</style>
<text class="t" x="24" y="36">Top Skills by Average Salary (Query 4 Results)</text>
<text class="l" x="16" y="86">pyspark</text>
<rect x="170" y="70" width="770.0" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="948.0" y="86">$208,172</text>
<text class="l" x="16" y="120">bitbucket</text>
<rect x="170" y="104" width="699.7" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="877.7" y="120">$189,155</text>
<text class="l" x="16" y="154">couchbase</text>
<rect x="170" y="138" width="593.7" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="771.7" y="154">$160,515</text>
<text class="l" x="16" y="188">watson</text>
<rect x="170" y="172" width="593.7" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="771.7" y="188">$160,515</text>
<text class="l" x="16" y="222">datarobot</text>
<rect x="170" y="206" width="575.1" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="753.1" y="222">$155,486</text>
<text class="l" x="16" y="256">gitlab</text>
<rect x="170" y="240" width="571.5" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="749.5" y="256">$154,500</text>
<text class="l" x="16" y="290">swift</text>
<rect x="170" y="274" width="568.7" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="746.7" y="290">$153,750</text>
<text class="l" x="16" y="324">jupyter</text>
<rect x="170" y="308" width="565.1" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="743.1" y="324">$152,777</text>
<text class="l" x="16" y="358">pandas</text>
<rect x="170" y="342" width="561.6" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="739.6" y="358">$151,821</text>
<text class="l" x="16" y="392">elasticsearch</text>
<rect x="170" y="376" width="536.3" height="22" fill="#59a14f" rx="4"/>
<text class="v" x="714.3" y="392">$145,000</text>
</svg>
