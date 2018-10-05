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

Seeing patterns in the data is made easier by generating charts. Scatterplots could show relationships between school budget and overall passing rates; school size and overall passing rates; school size and school budget; and school budget and budget per student.

```python
# scatterplot of budget vs overall passing rate
x = tot_sch_budget / 1000000
y = pct_sch_overall_passers
plt.scatter(x, y)
plt.xlabel("Budget (mln USD)")
plt.ylabel("Proportion of Overall Passers (%)")

# scatterplot of school size vs overall passing rate
x = no_students_per_sch
y = pct_sch_overall_passers
plt.scatter(x, y)
plt.xlabel("Number of Students")
plt.ylabel("Proportion of Overall Passers (%)")

# scatterplot of school size vs budget
x = no_students_per_sch
y = tot_sch_budget
plt.scatter(x, y)
plt.xlabel("Number of Students")
plt.ylabel("Budget (USD)")

# scatterplot of student budget vs total school budget
x = per_student_budget
y = tot_sch_budget
plt.scatter(x, y)
plt.xlabel("Budget per student (USD)")
plt.ylabel("School budget (USD)")
```

Insights from data visualisation can be used to enhance one's capacity to generate insights from the raw data. However, it is also important to take a deep dive into the data to understand the patterns more fully. Common characteristics of schools that performed similarly could provide additional information. In this case, the schools were ranked according to overall performance; the top five and the bottom five schools were put into separate tables.

```python
# sort the schools by overall passing rate
sch_passers_sorted = summary_school.sort_values("Proportion of Overall Passers (%)", ascending = False)

# display top five schools
sch_passers_sorted.head()

# display bottom five schools
sch_passers_sorted.tail()
```

### School Performance by Grade
It was interesting to see if standardised exam scores were different across the four grades because if some grades were performing particularly better or worse, the school district officers could implement district-level improvements for grades that were not faring well or to adopt best practices from schools which were performing better than the others. 

To get the school performance per grade, the data was first filtered by grade. 

```python
# filter the data by grade
grade_09 = school_data_complete.loc[school_data_complete["grade"] == "9th"]
grade_10 = school_data_complete.loc[school_data_complete["grade"] == "10th"]
grade_11 = school_data_complete.loc[school_data_complete["grade"] == "11th"]
grade_12 = school_data_complete.loc[school_data_complete["grade"] == "12th"]
```

Performance per school in the Math exam was then displayed by grade. A dataframe was generated to display the performance of each grade per school.

```python
# get average score for math for each grade by school
math_09_ave = round((grade_09.groupby("school_name")["math_score"].mean()),2)
math_10_ave = round((grade_10.groupby("school_name")["math_score"].mean()),2)
math_11_ave = round((grade_11.groupby("school_name")["math_score"].mean()),2)
math_12_ave = round((grade_12.groupby("school_name")["math_score"].mean()),2)

# merge series to dataframe
math_df = pd.DataFrame({"9th Grade Math Score (%)": math_09_ave, "10th Grade Math Score (%)": math_10_ave, "11th Grade Math Score (%)": math_11_ave, "12th Grade Math Score (%)": math_12_ave})                                 
math_df
```

A boxplot visualising the student's performance in the Math exam was then be generated. The graph provided insights about the distribution of scores in the district and to determine if there were outliers in the data (i.e., students who performed extraordinarily well or bad that could skew the data).

```python
# plot the Math average scores by grade
fig, ax = plt.subplots()
plt.boxplot([math_09_ave, math_10_ave, math_11_ave, math_12_ave])
plt.xlabel("Grade")
plt.ylabel("Average Math Score (%)")
ax.set_xticklabels(labels =[9,10,11,12])
```

The same process of summarising and visualising scores was implemented for the Reading test.

```python
# get average score for reading for each grade by school
rdg_09_ave = round((grade_09.groupby("school_name")["reading_score"].mean()),2)
rdg_10_ave = round((grade_10.groupby("school_name")["reading_score"].mean()),2)
rdg_11_ave = round((grade_11.groupby("school_name")["reading_score"].mean()),2)
rdg_12_ave = round((grade_12.groupby("school_name")["reading_score"].mean()),2)

# merge series to dataframe
rdg_df = pd.DataFrame({"9th Grade Reading Score (%)": rdg_09_ave, "10th Grade Reading Score (%)": rdg_10_ave, "11th Grade Reading Score (%)": rdg_11_ave, "12th Grade Reading Score (%)": rdg_12_ave})                                 
rdg_df

# plot the Reading average scores by grade
fig, ax = plt.subplots()
plt.boxplot([rdg_09_ave, rdg_10_ave, rdg_11_ave, rdg_12_ave])
plt.xlabel("Grade")
plt.ylabel("Average Reading Score (%)")
ax.set_xticklabels(labels =[9,10,11,12])
```

### Scores by School Spending
An earlier scatterplot seemed to suggest that lower school budgets were related with better school performance. It is unwise to generate insights from this prematurely because it is important to develop a deeper understanding about the relationship of school budgets and school performance before making recommendations. Since school budget and the number of students were correlated and could confound subsequent analysis, each school's `budget per student` potentially was a better independent variable to use. 

Because the budget per student was calculated previously, it was possible to group the schools into categories based on arbitrarily set budget ranges. In this case, the schools were grouped into four classes. 

```python
# Slice the dataframe to include only average scores and passing rates by budget bracket
budget_rates = sch_passers_sorted.loc[:, ["Number of students",
                                          "Budget per Student (USD)", 
                                          "Average Math Score (%)", "Average Reading Score (%)", 
                                          "Proportion of Math Passers (%)", "Proportion of Reading Passers (%)",
                                          "Proportion of Overall Passers (%)"]]

# Sample bins.
spending_bins = [0, 585, 615, 645, 675]
group_names = ["<$585", "$585–615", "$615–645", "$645–675"]

# Bin the data by the school spending per student
budget_rates["Budget Bracket (USD)"] = pd.cut(budget_rates["Budget per Student (USD)"], bins = spending_bins, right = False, labels = group_names)

# Create a group based on the bins
budget_rate_grped = budget_rates.groupby("Budget Bracket (USD)")
```

To calculate the average scores per budget class, it is important to incorporate weights because the school budget classes had different population sizes. To get the weighted averages, the function `wavg` was used. The weight used was the `Number of students`.

```python
# weighted averages for reading and math scores
def wavg(df,ave_name,weight_name):
    d = df[ave_name]
    w = df[weight_name]
    return (d * w).sum() / w.sum()

grp_budget_means_mathscore = round(budget_rate_grped.apply(wavg, "Average Math Score (%)", "Number of students"), 2)
grp_budget_means_rdgscore = round(budget_rate_grped.apply(wavg, "Average Reading Score (%)", "Number of students"), 2)
```

For the same reason, it was also important to consider school budget class size in calculating the passing rates. The function `proportion` was used for the calculations.

```python
# proportion of students who passed the math and the reading exams, and both
def proportion(df,proportion_name,weight_name):
    d = df[proportion_name]
    w = df[weight_name]
    return (d * w).sum() / w.sum() # where (d * w).sum() is the number of students who passed per bracket
                                   # where w.sum() is the total number of students per bracket

grp_budget_means_mathrate = round(budget_rate_grped.apply(proportion, "Proportion of Math Passers (%)", "Number of students"), 2)
grp_budget_means_rdgrate = round(budget_rate_grped.apply(proportion, "Proportion of Reading Passers (%)", "Number of students"), 2)
grp_budget_means_overallrate = round(budget_rate_grped.apply(proportion, "Proportion of Overall Passers (%)", "Number of students"),2)
```

After generating these statistics, it was just a matter of putting them together in a dataframe.

```python
# create dataframe using weighted averages (accounting for different sizes of budget classes)
budget_comparison = pd.DataFrame({"Wtd Average Math Score (%)": grp_budget_means_mathscore,
                                  "Wtd Average Reading Score (%)": grp_budget_means_rdgscore,
                                  "Proportion of Math Passers (%)": grp_budget_means_mathrate,
                                  "Proportion of Reading Passers (%)": grp_budget_means_rdgrate,
                                  "Proportion of Overall Passers (%)": grp_budget_means_overallrate})
budget_comparison
```

The relationship between school budget per student and student performance in the standardised tests could be interesting, but this could not easily be seen in the dataframe. Hence a matrix of correlation coefficients was generated.

```python
# Calculate correlation coefficients
correlation = round((budget_rates.corr()),2) # correlation coefficient
correlation
```

The coefficients of determination (correlation coefficient squared) is a measure of how much an independent variable (e.g., budget per student) could explain the variance in a dependent variable (e.g., performance indicators). Additional insights could be generated from this statistic.

```python
# How much of the variation is explained by the different factors?
corr_squared = round(correlation**2,2) # coefficient of determination
corr_squared
```

### Scores by School Size
The schools were grouped into three classes based on size.

```python
# Slice the dataframe to include only average scores and passing rates by school size
sch_size = sch_passers_sorted.loc[:, ["Number of students", 
                                      "Average Math Score (%)", "Average Reading Score (%)", 
                                      "Proportion of Math Passers (%)", "Proportion of Reading Passers (%)",
                                      "Proportion of Overall Passers (%)"]]

# Sample bins
size_bins = [0, 1000, 2000, 5000]
group_names = ["Small (<1000)", "Medium (1000-2000)", "Large (2000-5000)"]

# Bin the data by the school size
sch_size["Size Class"] = pd.cut(sch_size["Number of students"], bins = size_bins, right = False, labels = group_names)

# Group the schools based on size
sch_size_grped = sch_size.groupby("Size Class")
```

Determining the relationship between school performance (average scores and passing rates) and school size required weighted averages again because it is important to consider the differences in school population per size category. The weight used here was `number of students`.

```python 
# weighted averages for reading, math, and overall
def wavg(df,ave_name,weight_name):
    d = df[ave_name]
    w = df[weight_name]
    return (d * w).sum() / w.sum()

grp_size_means_mathscore = round(sch_size_grped.apply(wavg, "Average Math Score (%)", "Number of students"), 2)
grp_size_means_rdgscore = round(sch_size_grped.apply(wavg, "Average Reading Score (%)", "Number of students"), 2)

# proportion of students who passed the math and the reading exams, and both
def proportion(df,proportion_name,weight_name):
    d = df[proportion_name]
    w = df[weight_name]
    return (d * w).sum() / w.sum() # where (d * w).sum() is the number of students who passed per bracket
                                   # where w.sum() is the total number of students per bracket

grp_size_means_mathrate = round(sch_size_grped.apply(proportion, "Proportion of Math Passers (%)", "Number of students"), 2)
grp_size_means_rdgrate = round(sch_size_grped.apply(proportion, "Proportion of Reading Passers (%)", "Number of students"), 2)
grp_size_means_overallrate = round(sch_size_grped.apply(proportion, "Proportion of Overall Passers (%)", "Number of students"),2)
```

The statistics generated were then placed in a dataframe.

```python
# create dataframe using weighted averages (accounting for different sizes of schools)
sch_size_comparison = pd.DataFrame({"Wtd Average Math Score (%)": grp_size_means_mathscore,
                                  "Wtd Average Reading Score (%)": grp_size_means_rdgscore,
                                  "Proportion of Math Passers (%)": grp_size_means_mathrate,
                                  "Proportion of Reading Passers (%)": grp_size_means_rdgrate,
                                  "Proportion of Overall Passers (%)": grp_size_means_overallrate})

sch_size_comparison
```

### Scores by School Type
There are two school types in this district. To compare the two types, the schools were first grouped into either "charter" or "district".

```python
# Group the data by school type
sch_type = sch_passers_sorted.groupby('type')
```

An initial look at the dataset indicated that charter schools tended to have smaller sizes than district schools. Hence, it was important to consider weighting the average scores and the passing rates per school type.

```python
# weighted averages for reading, math, and overall
def wavg(df,ave_name,weight_name):
    d = df[ave_name]
    w = df[weight_name]
    return (d * w).sum() / w.sum()

grp_type_means_mathscore = round(sch_type.apply(wavg, "Average Math Score (%)", "Number of students"), 2)
grp_type_means_rdgscore = round(sch_type.apply(wavg, "Average Reading Score (%)", "Number of students"), 2)

# proportion of students who passed the math and the reading exams, and both
def proportion(df,proportion_name,weight_name):
    d = df[proportion_name]
    w = df[weight_name]
    return (d * w).sum() / w.sum() # where (d * w).sum() is the number of students who passed per bracket
                                   # where w.sum() is the total number of students per bracket

grp_type_means_mathrate = round(sch_type.apply(proportion, "Proportion of Math Passers (%)", "Number of students"), 2)
grp_type_means_rdgrate = round(sch_type.apply(proportion, "Proportion of Reading Passers (%)", "Number of students"), 2)
grp_type_means_overallrate = round(sch_type.apply(proportion, "Proportion of Overall Passers (%)", "Number of students"),2)
```

To summarise results, the statistics were put in a dataframe.

```python
# create dataframe using weighted averages (accounting for different sizes of school types)
sch_type_comparison = pd.DataFrame({"Wtd Average Math Score (%)": grp_type_means_mathscore,
                                  "Wtd Average Reading Score (%)": grp_type_means_rdgscore,
                                  "Wtd Ave Proportion of Math Passers (%)": grp_type_means_mathrate,
                                  "Wtd Ave Proportion of Reading Passers (%)": grp_type_means_rdgrate,
                                  "Wtd Ave Proportion of Overall Passers (%)": grp_type_means_overallrate})

sch_type_comparison
```

## Resources
The code for the function of weighted averages and weighted proportions was adapted from the one found [here](http://pbpython.com/weighted-average.html).

The definition of overall passing (i.e., pass both exams) was adopted from the standards of the [California High School Exit Examination (CAHSEE)](http://www.ppic.org/content/pubs/report/R_612JBR.pdf).
