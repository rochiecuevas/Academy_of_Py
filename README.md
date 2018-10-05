# PyCity School District Performance Analysis
## Overview
Standardised tests are designed to evaluate a school's performance in two key indicators of student learning: Mathematics and Reading. In the PyCity School District, results of standardised tests for these two subjects were obtained for 39,170 students in 15 school. These students were from 9th to 12th Grades. The schools were classified as either "charter" or "district" schools. The object of this study was to uncover trends in student performance, which could generate insights that could then lead towards strategic district-level decisions.

## Getting Started
Python modules Panda, NumPy, SciPy, and MatPlotLib were used for data analyses and visualisation.

```
# Dependencies and Setup
import pandas as pd
import numpy as np
import scipy.stats as stats
import matplotlib.pyplot as plt
```

The data came from two files: `schools_complete.csv` and `students_complete.csv`.

```
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
To characterise the PySchool District, the data was first grouped by school.

```
# group the data by school
school_grped = school_data_complete.groupby("school_name")
```

Then, the basic information about the district was determined. 

```
# descriptive statistics
no_schools = len(school_grped) # number of schools in the district
no_students = school_grped["school_name"].count().sum() # number of students in the district
tot_dist_sch_budget = round(school_grped["budget"].mean().sum(),2) # total education budget of the district
dist_ave_math_score = round((school_grped["math_score"].sum().sum() / no_students),2) # district average math score
dist_ave_rdg_score = round((school_grped["reading_score"].sum().sum() / no_students),2) # district average reading score
```

To determine the proportion of students who passed per subject, the number of students whose grades were at least 70% for each subject were counted and then divided by the number of students in the district. The values were then converted to percentages. 

```
# proportion of students who passed?
pct_math_passers = round(((len(school_data_complete.loc[school_data_complete["math_score"] >= 70]) / no_students) * 100),2)
pct_rdg_passers = round(((len(school_data_complete.loc[school_data_complete["reading_score"] >= 70]) / no_students) * 100),2)
```

To get the overall passing rate, the number of students who passed both Math and Reading exams were counted and then divided by the number of students in the district.

```
# number of students who passed both math and reading tests (overall passing rate)
both_passers = len(school_data_complete.loc[(school_data_complete["reading_score"] >= 70) & (school_data_complete["math_score"] >= 70)])
pct_both_passers = round(((both_passers / no_students) * 100),2)
```

A dataframe was generated to put all these district-level statistics together.

```
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

### School Overview
### Scores by School Spending
### Scores by School Size
### Scores by School Type
