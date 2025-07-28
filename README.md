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

# 🧠 Logistic Regression
In this study, the primary research question is:
- Does maternal age significantly influence the likelihood of a Caesarean delivery?

To answer this, we needed to model the relationship between a set of predictor variables (like age, marital status, and education level) and an outcome variable that is binary—in this case, whether a mother did or did not have a Caesarean delivery.

Logistic regression is used when the dependent variable (outcome) is binary (i.e., it can take on only two possible outcomes). In our case:

Outcome (dependent variable): cesareanbinary

- 1 = Yes, the mother had a Caesarean

- 0 = No, the mother did not have a Caesarean

The goal of logistic regression is to estimate the probability that a particular outcome occurs (e.g., a Caesarean) given the values of one or more independent variables (e.g., age, education, marital status).

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

