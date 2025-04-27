---
title: "README"
output:
 github_document:
 pandoc_args: ["--wrap=none"]
always_allow_html: true
---
# Exploratory-Data-Analysis-with-R

## Business Understandings

Goal: a wine producer has hired me to better understand what non-obvious
factors affect wine quality

After some background research I have found that: - Wine quality depends
on chemical properties (like acidity, sugar, pH, alcohol content) as
well as human taste. - Higher alcohol is typically associated with
better quality wines. - Too much residual sugar or acidity can
negatively impact taste. - Balance among components is important as
extreme values might hurt quality.

Therefore, I will explore how each chemical property relates to wine
quality.

## Data Understandings

I will firstly load and inspect the datasets to get a better
understanding of them. 
```{r results='asis'}
# Load libraries
library(tidyverse)
library(knitr) # for nice tables later

# Read the data
red_wine <- read.csv("/Users/nickmckenzie/Downloads/wine+quality/winequality-red.csv", sep = ";")
white_wine <- read.csv("/Users/nickmckenzie/Downloads/wine+quality/winequality-white.csv", sep = ";")

# Quick look
glimpse(red_wine)
glimpse(white_wine)

# Dimensions
cat("Red Wine: ", dim(red_wine)[1], "rows,", dim(red_wine)[2], "columns\n")
cat("White Wine: ", dim(white_wine)[1], "rows,", dim(white_wine)[2], "columns\n")

# ----- NEW: Data Preparation -----

# Add a 'type' column to each dataset
red_wine$type <- "red"
white_wine$type <- "white"

# Combine into one dataset
wine_data <- bind_rows(red_wine, white_wine)

# Quick check of combined data
glimpse(wine_data)

# ----- NEW: Create Feature Table -----

# Feature names
features <- colnames(wine_data)

# Define feature types manually
feature_types <- c(
  "Ratio",   # fixed acidity
  "Ratio",   # volatile acidity
  "Ratio",   # citric acid
  "Ratio",   # residual sugar
  "Ratio",   # chlorides
  "Ratio",   # free sulfur dioxide
  "Ratio",   # total sulfur dioxide
  "Ratio",   # density
  "Ratio",   # pH
  "Ratio",   # sulphates
  "Ratio",   # alcohol
  "Ordinal", # quality
  "Nominal"  # type
)

# Create a table showing feature names and types
feature_table <- data.frame(
  Feature = features,
  Data_Type = feature_types
)

# Print the feature table
kable(feature_table, caption = "Feature Names and Their Data Types (Wine Quality Dataset)", format = "markdown")

```

From the datasets we can find a lot of information. We can see the data
types of the features and I have listed the key findings below: 
##### Dataset info: 
Red Wine: 1599 samples, 12 features

White Wine: 4898 samples, 12 features 
Target Variable: quality (scored from 0-10)

## Data Preparation
First we will check for any missing values.
```{r}
# Check for missing values
colSums(is.na(red_wine))
colSums(is.na(white_wine))
```
As there are no missing values, we can move on to better preparing the data. I will add a "type" column to combine red and white wine into one dataset.
```{r}
# Add a 'type' column
red_wine$type <- "red"
white_wine$type <- "white"
# Combine
wine_data <- bind_rows(red_wine, white_wine)
# Check
glimpse(wine_data)
```
Now we have a full dataset with both wine types labeled.

## Modeling & Evaluating
I will now perform an Exploratory Data Analysis (EDA). First I will check general distributions and explore some key variables.

```{r}
# Generate and save the quality plot
wine_data %>%
  ggplot(aes(x = factor(quality), fill = type)) +
  geom_bar(position = "dodge") +
  labs(title = "Wine Quality Distribution", x = "Quality Score", y = "Count")
ggsave("images/quality_plot.png", width = 6, height = 4)

# Generate and save the alcohol plot
wine_data %>%
  ggplot(aes(x = quality, y = alcohol, fill = type)) +
  geom_boxplot() +
  labs(title = "Alcohol Content by Wine Quality", x = "Quality", y = "Alcohol (%)")
ggsave("images/alcohol_plot.png", width = 6, height = 4)
```

```
## Quality Distribution
![Wine Quality Distribution](images/quality_plot.png)

## Alcohol vs Quality  
![Alcohol Content by Quality](images/alcohol_plot.png)
```
From the Wine Quality Distribution we see that the distribution follows a near-normal curve peaking at quality score 6 (most common rating). White wines (blue bars) are more prevalent than reds at all quality levels and very few wines score at extremes (almost none at 3 or 9). 

In the Alcohol Content by Wine Quality visualization we see a clear positive correlation: higher alcohol content associates with better quality. White wines generally have lower alcohol content than reds at the same quality level and outliers appear more frequently in lower quality wines. 

From these visualizations and our background understandings we can generate some hypotheses to test:
##### Hypothesis 1: Higher alcohol content leads to better wine quality
##### Hypothesis 2: Lower volatile acidity leads to better wine quality (especially for red wine)
##### Hypothesis 3: Wines with moderate residual sugar have better quality (nonlinear relationship)

Now I will get into some statistical testing to test these hypotheses. We will first test Hypothesis 1: Higher alcohol content leads to better wine quality. We can do this by checking the correlation first and then performing a linear regression:

```{r}
cor(wine_data$alcohol, wine_data$quality)
lm1 <- lm(quality ~ alcohol, data = wine_data)
summary(lm1)
```
As we can see from the generated statistics, the statistical tests fail to reject this hypothesis. The correlation coefficient (0.444) shows a moderate positive relationship between alcohol and quality. From the regression results we can see that Highly significant p-value (<2e-16) confirms alcohol is a meaningful predictor. The coefficient (0.325) means each 1% increase in alcohol corresponds to ~0.33 point increase in quality score. However, the R-squared (0.197) indicates alcohol alone explains only about 20% of quality variation, suggesting other factors are important.

Next we will test Hypothesis 2: Lower volatile acidity leads to better wine quality (especially for red wine). We will also check the correlations and then run a linear regression:

```{r}
# Filter red wine
red_only <- wine_data %>% filter(type == "red")

# Correlation
cor(red_only$volatile.acidity, red_only$quality)

# Linear regression
lm2 <- lm(quality ~ volatile.acidity, data = red_only)
summary(lm2)
```
From the correlation results we find that their is a Strong negative correlation (-0.391). This also fails to reject this Hypothesis that lower volatile acidity associates with better quality in red wines. The relationship is statistically significant (p < 2.2e-16) and directionally meaningful. When looking at the regression, the coefficient the intercept is 6.566, which is the expected quality score when volatile acidity is zero. The slope is -1.761, meaning each 1 g/L increase in volatile acidity corresponds to a ~1.76 point decrease in quality score. The R-squared of 0.152 means volatile acidity explains about 15% of quality variation in red wines. While significant, this suggests other factors play larger roles in determining quality. This analysis strongly supports Hypothesis 2, revealing volatile acidity as a key negative quality predictor in red wines. The effect is both statistically and practically significant, though other factors clearly contribute to quality perceptions.

Finally, we can look into Hypothesis 3: Wines with moderate residual sugar have better quality (nonlinear relationship). We should first create a visualization to get a better understanding of the data. 

```{r}
wine_data %>%
  ggplot(aes(x = residual.sugar, y = quality, color = type)) +
  geom_point(alpha = 0.5) +
  geom_smooth(method = "loess") +
  labs(title = "Residual Sugar vs Wine Quality", x = "Residual Sugar", y = "Quality")
```

From this visualization we see the curves for both red and white wines appear to show a nonlinear relationship between residual sugar and quality. Quality seems to peak at moderate residual sugar levels (likely between ~5–20 g/L, though exact values depend on the data) and declines at very low or very high sugar levels. This supports Hypothesis 3. Now we can categorize sugar into bins to check differences to check if there is a significant difference in sugar levels in regards to quality. We can do this by splitting them into "low", "medium", "high" sugar levels and test.

```{r}
wine_data <- wine_data %>%
  mutate(sugar_level = case_when(
    residual.sugar < 5 ~ "low",
    residual.sugar < 15 ~ "medium",
    TRUE ~ "high"
  ))

# Boxplot
wine_data %>%
  ggplot(aes(x = sugar_level, y = quality, fill = type)) +
  geom_boxplot() +
  labs(title = "Sugar Levels vs Quality", x = "Sugar Level", y = "Quality")

# ANOVA
anova_sugar <- aov(quality ~ sugar_level, data = wine_data)
summary(anova_sugar)
```
```{r}
#              Df Sum Sq Mean Sq F value   Pr(>F)    
#sugar_level    2     25  12.650   16.67 6.02e-08 ***
#Residuals   6494   4928   0.759                     
#---
#Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1 
```

The significant p-value (6.02e-08) confirms that at least one sugar level group (low/medium/high) has a statistically different mean quality. The small sum of squares (Sum Sq = 25 vs. Residuals = 4928) suggests sugar level explains a modest portion of quality variation. Other factors (e.g., alcohol, acidity) likely play larger roles. Ultimately we also fail to reject this hypothesis. 



## Conclusion
Through exploratory data analysis of the UCI Wine Quality Dataset, I found that alcohol content and volatile acidity are significant predictors of wine quality, with alcohol showing a positive relationship and volatile acidity a negative one (especially in red wines). Additionally, residual sugar appears to have a nonlinear relationship with wine quality, with moderate levels associated with higher scores. These insights can help wine producers optimize chemical balances to enhance product quality. While these factors are important, wine quality is subjective and influenced by additional elements such as sensory perception and production methods that are not captured in this dataset.
