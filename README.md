# PyCity School District Performance Analysis
## Overview
Standardised tests are designed to evaluate a school's performance in two key indicators of student learning: Mathematics and Reading. In the PyCity School District, results of standardised tests for these two subjects were obtained for 39,170 students in 15 school. These students were from 9th to 12th Grades. The schools were classified as either "charter" or "district" schools. The object of this study was to uncover trends in student performance, which could generate insights that could then lead towards strategic district-level decisions.

## Getting Started
Python modules Panda, NumPy, SciPy, and MatPlotLib were used for data analyses and visualisation.

```python
# Dependencies and Setup
import pandas as pd
import numpy as np
import scipy.stats as stats
import matplotlib.pyplot as plt
```

The data came from two files: `schools_complete.csv` and `students_complete.csv`.

```python
# Data sources
school_data_to_load = "schools_complete.csv"
student_data_to_load = "students_complete.csv"

# Read School and Student Data File and store into Pandas Data Frames
school_data = pd.read_csv(school_data_to_load)
student_data = pd.read_csv(student_data_to_load)

# Combine the data into a single dataset
school_data_complete = pd.merge(student_data, school_data, how="left", on=["school_name", "school_name"])
```

## Data Analyses
### District Overview
To characterise the PySchool District, the data was first grouped by `school name`.

```python
# group the data by school
school_grped = school_data_complete.groupby("school_name")
```

Then, the basic information about the district was determined, including the number of schools, the student population size, the district education budget, and the average scores for the two tests. 

```python
# descriptive statistics
no_schools = len(school_grped) # number of schools in the district
no_students = school_grped["school_name"].count().sum() # number of students in the district
tot_dist_sch_budget = round(school_grped["budget"].mean().sum(),2) # total education budget of the district
dist_ave_math_score = round((school_grped["math_score"].sum().sum() / no_students),2) # district average math score
dist_ave_rdg_score = round((school_grped["reading_score"].sum().sum() / no_students),2) # district average reading score
```

To determine the proportion of students who passed per subject, the number of students whose grades were at least 70% for each subject were counted and then divided by the number of students in the district. The values were then converted to percentages. 

```python
# proportion of students who passed?
pct_math_passers = round(((len(school_data_complete.loc[school_data_complete["math_score"] >= 70]) / no_students) * 100),2)
pct_rdg_passers = round(((len(school_data_complete.loc[school_data_complete["reading_score"] >= 70]) / no_students) * 100),2)
```

To get the overall passing rate, the number of students who passed both Math and Reading exams were counted and then divided by the number of students in the district.

```python
# number of students who passed both math and reading tests (overall passing rate)
both_passers = len(school_data_complete.loc[(school_data_complete["reading_score"] >= 70) & (school_data_complete["math_score"] >= 70)])
pct_both_passers = round(((both_passers / no_students) * 100),2)
```

A dataframe was generated to put all these district-level statistics together.

```python
# summary for the district
summary_district = pd.DataFrame({"Number of schools": [no_schools],
                                 "Number of students": "{:,}".format(no_students),
                                 "District education budget (USD)": "{:,.2f}".format(tot_dist_sch_budget),
                                 "Average Math Score (%)": [dist_ave_math_score],
                                 "Average Reading Score (%)": [dist_ave_rdg_score],
                                 "Proportion of Math Passers (%)": [pct_math_passers],
                                 "Proportion of Reading Passers (%)": [pct_rdg_passers],
                                 "Overall Passing Rate (%)": [pct_both_passers]
                                })
summary_district
```

To check if there was a difference in performance of students between Math and Reading tests, a Welch t-test was used.

```python
# Comparing results using Welch test
score_math_v_rdg = stats.ttest_ind(a = school_data_complete["math_score"], 
                                   b = school_data_complete["reading_score"], 
                                   equal_var=False)

rate_math_v_rdg = stats.ttest_ind(a = school_data_complete["math_score"] >= 70, 
                                  b = school_data_complete["reading_score"] >= 70, 
                                   equal_var=False)

print("ave scores: " + str(score_math_v_rdg), "\nproportion of passers: " + str(rate_math_v_rdg))
```


### School Overview
After getting the district overview, the next step was to generate information about each school. To do so, the schools were grouped by `school name` and by `type`.

```python
# group the data by school name and type
school_grped2 = school_data_complete.groupby(["school_name","type"]) # group schools by name
```

The basic information per school were then generated, including the number of students, the budget, the budget per student, and the average scores for each subject. Getting the average scores per subject involved dividing the sum of scores by the number of students per school. 

```python
# descriptive statistics
no_students_per_sch = school_grped2["school_name"].count() # number of students per school
tot_dist_sch_budget = school_grped2["budget"].mean() # total education budget per school
per_student_budget = tot_dist_sch_budget / no_students_per_sch
sch_ave_math_score = round((school_grped2["math_score"].sum() / no_students_per_sch),2) # school average math score
sch_ave_rdg_score = round((school_grped2["reading_score"].sum() / no_students_per_sch),2) # school average reading score
```

To determine the passing rate for each school per exam, the number of students who passed each exam was divided by the number of students per school. The values were then converted to percentages.

```python
sch_math_passers = school_grped2["math_score"].apply(lambda x: x[x >=70].count()) # number of math passers per school
pct_sch_math_passers = round(((sch_math_passers * 100) / no_students_per_sch),2) # proportion of math passers per school

sch_rdg_passers = school_grped2["reading_score"].apply(lambda x: x[x >=70].count()) # number of reading passers per school
pct_sch_rdg_passers = round(((sch_rdg_passers * 100) / no_students_per_sch),2) # proportion of reading passers per school
```

To then determine the overall passing rate per school, the number of students who passed both exams were divided by the number of students per school. The value was converted to percentage.

```python
# filter raw data for students who passed both math and reading tests
sch_passers = school_data_complete.loc[(school_data_complete["math_score"] >= 70) & (school_data_complete["reading_score"] >= 70)]
sch_passers_grped = sch_passers.groupby(["school_name"])
pct_sch_overall_passers = round(((sch_passers_grped["student_name"].count() / no_students_per_sch) * 100),2)
```

A summary of results were generated per school by putting all the statistics into a dataframe.

```python
# summary per school
summary_school = pd.DataFrame({"Number of students": no_students_per_sch,
                               "Budget (USD)": tot_dist_sch_budget.map("{:,.2f}".format),
                               "Budget per Student (USD)": per_student_budget,
                               "Average Math Score (%)": sch_ave_math_score,
                               "Average Reading Score (%)": sch_ave_rdg_score,
                               "Proportion of Math Passers (%)": pct_sch_math_passers,
                               "Proportion of Reading Passers (%)": pct_sch_rdg_passers,
                               "Proportion of Overall Passers (%)": pct_sch_overall_passers
                                })
summary_school
```
### Scores by School Spending
### Scores by School Size
### Scores by School Type
