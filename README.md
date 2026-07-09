# Intermediate Portfolio Analysis in R

## Overview

This project demonstrates basic portfolio analysis using the `PortfolioAnalytics` package in R.

## Technologies Used

* R
* PortfolioAnalytics

## Code Example

```r
# Load the package
library(PortfolioAnalytics)
# Load the data
data(indexes)

# Subset the data
index_returns <- indexes[, 1:4]

# Print the first rows
head(index_returns)
```

## Purpose

The objective of this project is to explore financial index return data and prepare it for portfolio analysis and optimization.

## Author

Mo A. Badr
