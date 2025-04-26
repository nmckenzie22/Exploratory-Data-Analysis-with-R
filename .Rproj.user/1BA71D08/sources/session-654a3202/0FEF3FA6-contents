README
================

# Exploratory-Data-Analysis-with-R

### Business Understandings

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

### Data Understandings

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

White Wine: 4898 samples, 12 features Target Variable: quality (scored
form 0-10)

### Data Preparation

### Modeling & Evaluating

### Deployment
