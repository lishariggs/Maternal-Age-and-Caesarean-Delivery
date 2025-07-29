# Maternal-Age-and-Caesarean-Delivery
This project explores whether maternal age significantly influences the likelihood of a Caesarean delivery, using the 2015–2016 NHANES Reproductive Health and Demographic data.

# Maternal Age and Caesarean Delivery: An NHANES Data Analysis
This project explores whether maternal age significantly influences the likelihood of a Caesarean delivery, using the 2015–2016 NHANES Reproductive Health and Demographic data.

**About** 
The National Center for Health Statistics (NCHS) conducts the National Health and Nutrition Examination Surveys (NHANES), which have been ongoing since 1999 after starting periodically in the early 1960s. Each year, NHANES surveys about 5,000 individuals across the U.S., using mobile examination centers to collect high-quality, standardized health data. The surveys target the noninstitutionalized civilian population and periodically oversample specific subgroups to improve the reliability of health estimates for these groups. The reproductive health section (RHQ) contains questions for females only, covering menstrual and pregnancy history, hormone replacement therapy use, and related reproductive conditions. Females aged 12 and older are eligible. Some pregnancy and hysterectomy-related variables are excluded for privacy reasons for those aged 12-19 and women over 44. For the purpose of this study, NHANES demographic and reproductive health data from 2015-2016 was used.

📊 **Data Source**
- The data comes from the National Health and Nutrition Examination Survey (NHANES) conducted by the National Center for Health Statistics (NCHS). The survey includes health and nutritional data from a representative sample of about 5,000 U.S. residents annually.

- Demographic Data Codebook: [DEMO_I](https://wwwn.cdc.gov/Nchs/Data/Nhanes/Public/2015/DataFiles/DEMO_I.htm)

- Reproductive Health Data Codebook: [RHQ_I](https://wwwn.cdc.gov/Nchs/Data/Nhanes/Public/2015/DataFiles/RHQ_I_R.htm)

# ❓ Research Question
- **Does maternal age significantly influence the likelihood of a Caesarean delivery?**

# 🧾 Methodology

📦 Packages Used

`
library(haven)
library(tidyverse)
library(dplyr)
library(psych)
library(table1)
library(MASS)
`

📥 Data Import

<pre lang="markdown"> 
```r 
# Load demographic and reproductive health data
demographic <- read_xpt("DEMO_I.XPT") 
reproductive_health <- read_xpt("RHQ_I.XPT") 

# Combine by SEQN 
combine_data <- left_join(demographic, reproductive_health, by = "SEQN") 
``` </pre>

🧹 Data Cleaning

<pre lang="markdown"> 
```r 
# Rename variables for clarity
combine_data <- combine_data %>%
  rename(
    age = RIDAGEYR,
    marital = DMDMARTL,
    cesarean = RHQ169,
    education = DMDEDUC2
  )

# Convert to factors
combine_data$education <- as.factor(combine_data$education)
combine_data$marital <- as.factor(combine_data$marital)

# Subset relevant data
cesarean_data <- combine_data %>%
  select(age, cesarean, marital, education)

# Create binary variable
cesarean_data$cesareanbinary <- ifelse(cesarean_data$cesarean > 0 & cesarean_data$cesarean <= 7, 1, 0)
``` </pre>

📈 Descriptive Statistics


<pre lang="markdown"> 
```r 
# Recode and label factors
cesarean_data$cesareanbinary <- factor(cesarean_data$cesareanbinary, levels = c(1, 0), labels = c("Yes", "No"))

cesarean_data$marital <- factor(cesarean_data$marital,
  levels = c(1, 2, 3, 4, 5, 6),
  labels = c("Married", "Widowed", "Divorced", "Separated", "Never married", "Living with partner")
)

cesarean_data$education <- factor(cesarean_data$education,
  levels = c(1, 2, 3, 4, 5),
  labels = c("Less than 9th grade", "9–11th grade", "High school/GED", "Some college/AA", "College graduate or above")
)

# Label for table1
label(cesarean_data$cesareanbinary) <- "Cesarean Delivery"
label(cesarean_data$marital) <- "Marital Status"
label(cesarean_data$education) <- "Education"
units(cesarean_data$age) <- "Years"

# Descriptive summary
describe(cesarean_data)

# Descriptive table
table1(~ cesareanbinary + age + marital + education, data = cesarean_data, caption = "Basic Stats")
``` </pre>

<img width="562" height="192" alt="image" src="https://github.com/user-attachments/assets/cf55d97f-da88-4f4b-af88-6e39fb4d9c51" />

# Data Visualization
Next, after obtaining the descriptive statistics, I proceeded to visualize the distribution of ages in my dataset using a histogram. This helps me understand the spread and shape of the age data. Lastly,I box plot to compare the distribution of age between mothers who had Caesarean deliveries and those who did not. This can help identify any differences or similarities in the age distributions between the two groups. 

The ggplot(data= cesarean_data, aes(x = factor(cesareanbinary), y = age, fill = factor(cesareanbinary))) line specifies the data frame (cesarean_data) and the aesthetic mappings (aes) for the plot. Next, factor(cesareanbinary) is mapped to the x-axis (to create separate box plots for each category), age is mapped to the y-axis, and ‘fill’ is used to differentiate between mothers who had Caesarean deliveries and those who did not. 

Next, geom_boxplot(alpha = 0.7, outlier.alpha = 0) adds a layer of box plots to the plot. The alpha argument controls the transparency of the box plots, and outlier.alpha controls the transparency of any outlier points. The labs() function is used to customize the plot labels, title, and legend title (fill). You can specify the x-axis label (x), y-axis label (y), and the plot title (title) within this function. The scale_fill_manual(values = c(“red”, “#purple”)) line customizes the fill colors of the box plots for the two categories (0 = No Caesarean, 1 = Caesarean). Lastly, the theme_minimal() function applies a minimal theme to the plot, removing unnecessary elements to improve clarity.

 <pre lang="markdown"> 
```r 
  hist(cesarean_data$age)
  ``` </pre>
 
  <img width="694" height="447" alt="image" src="https://github.com/user-attachments/assets/6240eb36-2d4a-4aa0-afbe-6f4d78e44145" />

 <pre lang="markdown"> 
```r 
ggplot(data = cesarean_data, aes(x = factor(cesareanbinary), y = age, fill = factor(cesareanbinary))) +
geom_boxplot(alpha = 0.7, outlier.alpha = 0) +
labs(x = "Caesarean Delivery", y = "Age of Mother",
title = "Box Plot of Age by Caesarean Delivery Status",
fill = "Caesarean Delivery") +
scale_fill_manual(values = c("red", "purple")) +
theme_minimal()  
``` </pre>

<img width="685" height="466" alt="image" src="https://github.com/user-attachments/assets/941bc7aa-e2b9-47be-bb6a-5652aa44a9c7" />


# 🧠 Logistic Regression
In this study, the primary research question is:
- Does maternal age significantly influence the likelihood of a Caesarean delivery?

To answer this, we needed to model the relationship between a set of predictor variables (like age, marital status, and education level) and an outcome variable that is binary—in this case, whether a mother did or did not have a Caesarean delivery.

Logistic regression is used when the dependent variable (outcome) is binary (i.e., it can take on only two possible outcomes). In our case:

Outcome (dependent variable): cesareanbinary

- 1 = Yes, the mother had a Caesarean

- 0 = No, the mother did not have a Caesarean

The goal of logistic regression is to estimate the probability that a particular outcome occurs (e.g., a Caesarean) given the values of one or more independent variables (e.g., age, education, marital status).

First, I will load the “MASS” package in order to conduct a to perform logistic regression with my cesarean data. I will use the “cesareanbinary” as the outcome variable and “age” as the predictor variable in the logistic regression model. I will perform logistic regression using the glm() function, specifying family = binomial to fit a logistic regression model. Lastly, I will use the summary() function to print the summary of the logistic regression model to examine the coefficients, standard errors, and significance levels.

<pre lang="markdown"> 
```r 
# Logistic regression model
logistic_model <- glm(cesareanbinary ~ age + marital + education, data = cesarean_data, family = binomial)

# Model summary
summary(logistic_model)
``` </pre>

# 📋 Results
- Age: For each additional year of age, the odds of having a Caesarean slightly decrease.

- Marital Status: Being widowed, never married, or living with a partner reduces the odds compared to being married.

- Education: Not statistically significant in this model.


# ✅ Conclusion
The analysis highlights maternal age and marital status as significant predictors of Caesarean delivery. Education level does not show a significant impact. These insights can guide healthcare providers in prenatal care planning.

