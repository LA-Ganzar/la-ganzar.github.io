---
title: "Sample A/B Experiment for Strava"
author: "Leigh Ann Ganzar, DrPH"
date: '2022-04-07'
collection: portfolio
---

```{r setup, echo = FALSE, include=FALSE, warning=FALSE}
library(tidyverse)
library(powerMediation)
# Set up colors 
col1 <- "#fc5200"
col2 <- "#ade6d6"
# Read hypothetical dataset
full_df <- readr:: read_csv("/Users/leighannganzar/Desktop/Job Search 2021/Strava/Product Analyst/Strava-AB-Test/sample_AB_data.csv")
```

### Context
The Recommended Routes feature of Strava provides users with pre-defined route options based on user input of distance, elevation, and activity type. However, especially with cycling routes, there may be construction, detours, increased traffic, or other factors that impact the current usability of that route. The goal of this experiment is to assess whether adding a feature that provide the user with a date of the last time the majority of the route segments were used will increase the probability of the user saving the route. 

### Variables
Research Question: Does adding a "Last time route used: [date]" feature to the recommended route options increase the probability of saving the route?

### Assumptions
I.	Test Version A is the control group which depicts the existing features of recommended routes.
II.	Test Version B is the experimental group to experiment the new version of recommended routes with the date feature to see if it increases saves of the route (conversions).
III.	Converted – Based on the given hypothetical dataset, there are two categories defined by binary variable:
  (a)	Converted = 1 when user saves a route 
  (b) Converted = 0 when user visits the recommended route page but does not save a route

### A/B Test Hypothesis
#### Null Hypothesis
Both version A and B have the same probability of driving user conversion. 

#### Alternative Hypothesis
Versions A and B have different probabilities of driving user conversion. There is a difference between version A and B. Version B is better than A in driving user route saves. PExp_B != Pcont_A  

### Sample Size Calculations
To determine sample size for the experiment, the following inputs are used:

* Statistical test - logistic regression with binary predictor
* Baseline value - value for control condition, assumed in this hypothetical to be that 10% of users who visit the recommended routes page save a route
* Desired value - value for test condition, assumed in this hypothetical to be 15% of users who visit the recommended routes page and save a route after seeing the "Last time route used: [date]" feature
* Proportion of data from test condition - ideally 0.5
* Significance - 0.05
* Power - probability of correctly rejecting null hypothesis, generally 0.80

```{r}
sample_size <- SSizeLogisticBin(p1 = 0.10,
                                p2 = 0.15,
                                B = 0.5,
                                alpha = 0.05,
                                power = 0.80)
sample_size
```
Results show that a sample of at least 1,372 users is needed to detect a difference in conversion proportions between the control and experimental groups of 0.5. 

### Results
```{r echo = FALSE, warning = FALSE}
prop <- full_df %>%
  mutate(condition = factor(condition,
                            labels = c("Control", "Experimental"))) %>%
  group_by(condition) %>%
  summarize(prop = round(mean(conversion), digits = 2))

ggplot(prop, aes(x = condition, y = prop, fill = condition, label = prop)) +
  geom_bar(stat = "identity") +
  geom_text(aes(label = prop), hjust = 1.25) +
  theme_minimal() + 
  coord_flip() +
  scale_fill_manual(values = c(col1, col2)) +
  labs(title = "Number of Saved Routes for Control and Experimental Groups",
       y = "Proportion of Saved Routes when Visiting Recommended Routes Page",
       x = "") +
  theme(legend.position = "none")

prop_cont <- prop %>%
  filter(condition == "Control")

prop_exp <- prop %>%
  filter(condition == "Experimental")
```

### Calculate relative uplift
```{r echo = FALSE}
# Calculate relative uplift
uplift <- (prop_exp$prop - prop_cont$prop)/ prop_cont$prop * 100
uplift
```


### Hypothesis Testing
```{r}
# Create and view contingency table
strava_test <- table(full_df$condition, full_df$conversion)
strava_test

# Confirm expected counts assumption
chisq.test(strava_test)$expected

# Run chi-squared test of independence
chisq.test(strava_test, correct=FALSE)
```

### Conclusions Drawn from Data
1.	There were 75 saved routes (conversions) for test version A (control) and 125 saved routes for test version B (experimental).
2.	The relative uplift was 63.64%, based on a conversion proportion for A = 0.11, and the conversion rate for B = 0.18.
3.	P-value computed for this analysis was 0.0001, indicating that **the proportion of saved routes from the experimental condition was significantly higher than the proportion of saved routes from the control condition.** 

### Future Research
Given the significant increase in routes saved using the experimental feature, I recommend the following:

1.    Repeat experiment to confirm findings over time
2.    Segment by activity type (run, ride, walk etc) to assess whether the association between the added feature and saving routes varies by activity type

