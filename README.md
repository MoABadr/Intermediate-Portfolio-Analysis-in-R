# Intermediate Portfolio Analysis in R / Python

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
# Create the portfolio specification
port_spec <- portfolio.spec(colnames(index_returns))

# Add a full investment constraint such that the weights sum to 1
port_spec <- add.constraint(portfolio = port_spec, type = "full_investment")

# Add a long only constraint such that the weight of an asset is between 0 and 1
port_spec <- add.constraint(portfolio =port_spec, type = "long_only")

# Add an objective to minimize portfolio standard deviation
port_spec <- add.objective(portfolio =port_spec, type = "risk", name = "StdDev")

# Solve the optimization problem
opt <- optimize.portfolio(index_returns, portfolio = port_spec, optimize_method = "ROI")

# Print the results of the optimization
print(opt)

# Extract the optimal weights
optimal_weights <- extractWeights(opt)

# Chart the optimal weights

chart.Weights(opt)
   In Python 
# Load required libraries
library(PortfolioAnalytics)
library(PerformanceAnalytics)
library(ROI)
library(ROI.plugin.quadprog)
library(quadprog)

# 1. GET COLUMN NAMES FROM RETURNS DATA
asset_names <- colnames(asset_returns)
cat("Asset names:\n")
print(asset_names)

# 2. CREATE PORTFOLIO SPECIFICATION
port_spec <- portfolio.spec(assets = asset_names)
cat("\nPortfolio specification created with", length(asset_names), "assets\n")

# 3. VIEW PORTFOLIO SPECIFICATION CLASS AND STRUCTURE
cat("\nClass of portfolio specification object:\n")
print(class(port_spec))

cat("\nPortfolio specification details:\n")
print(port_spec)

# 4. ADD CONSTRAINTS

# Add weight sum constraint (weights must sum to 1)
port_spec <- add.constraint(portfolio = port_spec, type = "weight_sum", min_sum = 1, max_sum = 1)

# Add box constraint (individual asset weight limits)
# IMPORTANT: Ensure the number of min values matches your number of assets!
# For 12 assets:
port_spec <- add.constraint(portfolio = port_spec, type = "box", 
                           min = c(0.1, 0.1, 0.1, 0.1, 0.1, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05), 
                           max = 0.4)

# Add group constraint (assets grouped and limited to 40-60% of portfolio)
port_spec <- add.constraint(portfolio = port_spec, type = "group", 
                           groups = list(c(1,5,7,9,10,11), c(2,3,4,6,8,12)), 
                           group_min = 0.4, group_max = 0.6)

# 5. ADD OBJECTIVES

# Add objective to maximize mean return
port_spec <- add.objective(portfolio = port_spec, type = "return", name = "mean")

# Add objective to minimize variance (with risk aversion parameter)
port_spec <- add.objective(portfolio = port_spec, type = "risk", name = "var", risk_aversion = 10)

# 6. VIEW FINAL PORTFOLIO SPECIFICATION
cat("\nFinal portfolio specification with all constraints and objectives:\n")
print(port_spec)

# 7. GENERATE RANDOM PORTFOLIOS (for random optimization method)
rp <- random_portfolios(portfolio = port_spec, permutations = 20000)

# 8. RUN SINGLE PERIOD OPTIMIZATION (using random portfolios)
opt <- optimize.portfolio(R = asset_returns, portfolio = port_spec, 
                         optimize_method = "random", rp = rp, trace = TRUE)

# 9. PRINT SINGLE PERIOD OPTIMIZATION RESULTS
cat("\n=== SINGLE PERIOD OPTIMIZATION RESULTS ===\n")
print(opt)

# 10. EXTRACT AND DISPLAY OPTIMAL WEIGHTS
optimal_weights <- extractWeights(opt)
cat("\nOptimal Portfolio Weights:\n")
print(round(optimal_weights, 4))

# 11. CHART THE OPTIMAL WEIGHTS
chart.Weights(opt, main = "Optimal Portfolio Allocation")

# 12. RUN PORTFOLIO OPTIMIZATION WITH REBALANCING
opt_rebal <- optimize.portfolio.rebalancing(R = asset_returns, 
                                           portfolio = port_spec, 
                                           optimize_method = c("DEoptim", "random", "ROI"), 
                                           search_size = 20000, 
                                           trace = TRUE,
                                           rebalance_on = "quarters", 
                                           training_period = 60, 
                                           rolling_window = 60)

# 13. PRINT REBALANCING OPTIMIZATION RESULTS
cat("\n=== REBALANCING OPTIMIZATION RESULTS ===\n")
print(opt_rebal)

# 14. VIEW REBALANCING WEIGHTS OVER TIME
chart.Weights(opt_rebal, main = "Rebalancing Portfolio Weights Over Time")


# Import required libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from pypfopt import EfficientFrontier, risk_models, expected_returns
from pypfopt import objective_functions
from pypfopt import plotting
from pypfopt.discrete_allocation import DiscreteAllocation
import warnings
warnings.filterwarnings('ignore')

# Assume asset_returns is a pandas DataFrame with returns data
# If you need to create sample data, uncomment this:
# np.random.seed(42)
# dates = pd.date_range('2020-01-01', periods=252, freq='D')
# assets = ['AAPL', 'GOOGL', 'MSFT', 'AMZN', 'TSLA', 'META', 
#           'NFLX', 'NVDA', 'JPM', 'V', 'WMT', 'PG']
# asset_returns = pd.DataFrame(np.random.normal(0.001, 0.02, (252, 12)), 
#                             index=dates, columns=assets)

# 1. GET COLUMN NAMES FROM RETURNS DATA
asset_names = asset_returns.columns.tolist()
print("Asset names:")
print(asset_names)

# 2. CALCULATE EXPECTED RETURNS AND COVARIANCE MATRIX
mu = expected_returns.mean_historical_return(asset_returns)
S = risk_models.sample_cov(asset_returns)

print(f"\nPortfolio created with {len(asset_names)} assets")

# 3. CREATE EFFICIENT FRONTIER OBJECT
ef = EfficientFrontier(mu, S)

# 4. ADD CONSTRAINTS

# Weight sum constraint (weights sum to 1) and long-only are default
# But we can explicitly set them:
ef.add_constraint(lambda w: w >= 0)  # Long-only

# Box constraint (individual asset weight limits)
# For 12 assets with different min/max values
min_weights = [0.1, 0.1, 0.1, 0.1, 0.1, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05]
max_weights = [0.4] * 12  # Max 40% for each asset

for i in range(len(asset_names)):
    ef.add_constraint(lambda w, i=i: w[i] >= min_weights[i])
    ef.add_constraint(lambda w, i=i: w[i] <= max_weights[i])

# Group constraint (assets grouped and limited to 40-60% of portfolio)
# Group 1: assets at positions 1,5,7,9,10,11 (0-indexed: 0,4,6,8,9,10)
group1_indices = [0, 4, 6, 8, 9, 10]  # 1,5,7,9,10,11 (1-indexed)
group2_indices = [1, 2, 3, 5, 7, 11]  # 2,3,4,6,8,12 (1-indexed)

ef.add_constraint(lambda w: sum(w[i] for i in group1_indices) >= 0.4)
ef.add_constraint(lambda w: sum(w[i] for i in group1_indices) <= 0.6)
ef.add_constraint(lambda w: sum(w[i] for i in group2_indices) >= 0.4)
ef.add_constraint(lambda w: sum(w[i] for i in group2_indices) <= 0.6)

# 5. ADD OBJECTIVES

# Minimize variance (risk) with risk aversion parameter
# In PyPortfolioOpt, we maximize return with a risk penalty
# Equivalent to minimize variance with risk aversion

# 6. SINGLE PERIOD OPTIMIZATION

# Method 1: Maximize Sharpe Ratio (equivalent to mean-variance optimization)
ef = EfficientFrontier(mu, S)
ef.add_constraint(lambda w: w >= 0)

# Add box constraints
for i in range(len(asset_names)):
    ef.add_constraint(lambda w, i=i: w[i] >= min_weights[i])
    ef.add_constraint(lambda w, i=i: w[i] <= max_weights[i])

# Add group constraints
ef.add_constraint(lambda w: sum(w[i] for i in group1_indices) >= 0.4)
ef.add_constraint(lambda w: sum(w[i] for i in group1_indices) <= 0.6)
ef.add_constraint(lambda w: sum(w[i] for i in group2_indices) >= 0.4)
ef.add_constraint(lambda w: sum(w[i] for i in group2_indices) <= 0.6)

# Optimize for maximum Sharpe ratio (which handles risk-return tradeoff)
ef.max_sharpe()

# Get optimal weights
optimal_weights = ef.clean_weights()
print("\n=== SINGLE PERIOD OPTIMIZATION RESULTS ===")
print(f"Expected Return: {ef.portfolio_performance()[0]:.4f}")
print(f"Volatility: {ef.portfolio_performance()[1]:.4f}")
print(f"Sharpe Ratio: {ef.portfolio_performance()[2]:.4f}")

print("\nOptimal Portfolio Weights:")
for asset, weight in optimal_weights.items():
    if weight > 0.001:  # Only show non-zero weights
        print(f"{asset}: {weight:.4f} ({weight*100:.2f}%)")

# 7. CHART THE OPTIMAL WEIGHTS
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 6))

# Bar chart
weights_df = pd.DataFrame(list(optimal_weights.items()), columns=['Asset', 'Weight'])
weights_df = weights_df[weights_df['Weight'] > 0.001]
ax1.bar(weights_df['Asset'], weights_df['Weight'], color='steelblue')
ax1.set_title('Optimal Portfolio Allocation')
ax1.set_xlabel('Assets')
ax1.set_ylabel('Weight')
ax1.set_ylim(0, max(weights_df['Weight']) * 1.2)
ax1.axhline(y=0, color='black', linestyle='-', linewidth=0.5)
plt.setp(ax1.xaxis.get_majorticklabels(), rotation=45)

# Pie chart
ax2.pie(weights_df['Weight'], labels=weights_df['Asset'], autopct='%1.1f%%')
ax2.set_title('Portfolio Allocation (Pie Chart)')

plt.tight_layout()
plt.show()

# 8. REBALANCING PORTFOLIO (Rolling Window Optimization)

def rolling_portfolio_optimization(returns, window=60, rebalance_freq=1):
    """
    Perform rolling window portfolio optimization
    
    Parameters:
    returns: DataFrame of asset returns
    window: Training period (60 days)
    rebalance_freq: How often to rebalance (1 = every day)
    """
    n_assets = returns.shape[1]
    n_periods = returns.shape[0]
    
    # Store results
    weights_history = []
    dates = returns.index
    
    # Rolling optimization
    for i in range(window, n_periods - window, rebalance_freq * window):
        # Get training data
        train_returns = returns.iloc[i-window:i]
        test_date = dates[i]
        
        try:
            # Calculate expected returns and covariance
            mu_roll = expected_returns.mean_historical_return(train_returns)
            S_roll = risk_models.sample_cov(train_returns)
            
            # Optimize
            ef_roll = EfficientFrontier(mu_roll, S_roll)
            
            # Add constraints
            ef_roll.add_constraint(lambda w: w >= 0)
            
            # Add group constraints
            ef_roll.add_constraint(lambda w: sum(w[i] for i in group1_indices if i < n_assets) >= 0.4)
            ef_roll.add_constraint(lambda w: sum(w[i] for i in group1_indices if i < n_assets) <= 0.6)
            ef_roll.add_constraint(lambda w: sum(w[i] for i in group2_indices if i < n_assets) >= 0.4)
            ef_roll.add_constraint(lambda w: sum(w[i] for i in group2_indices if i < n_assets) <= 0.6)
            
            # Max Sharpe ratio
            ef_roll.max_sharpe()
            weights_roll = ef_roll.clean_weights()
            
            weights_history.append({
                'date': test_date,
                'weights': weights_roll,
                'return': ef_roll.portfolio_performance()[0],
                'volatility': ef_roll.portfolio_performance()[1]
            })
            
        except Exception as e:
            print(f"Error at date {test_date}: {e}")
            continue
    
    return pd.DataFrame(weights_history)

# Run rolling optimization
print("\n=== REBALANCING OPTIMIZATION RESULTS ===")
rebalancing_results = rolling_portfolio_optimization(asset_returns, window=60, rebalance_freq=1)

if len(rebalancing_results) > 0:
    print(f"Number of rebalancing periods: {len(rebalancing_results)}")
    print("\nRebalancing Results:")
    print(rebalancing_results[['date', 'return', 'volatility']].head())
    
    # Plot weights over time
    weights_df_roll = pd.DataFrame({
        rebalancing_results.iloc[i]['date']: rebalancing_results.iloc[i]['weights']
        for i in range(len(rebalancing_results))
    }).T
    
    # Plot
    fig, ax = plt.subplots(figsize=(14, 6))
    weights_df_roll.plot(kind='area', stacked=True, ax=ax, alpha=0.7)
    ax.set_title('Rebalancing Portfolio Weights Over Time')
    ax.set_xlabel('Date')
    ax.set_ylabel('Portfolio Weight')
    ax.legend(loc='center left', bbox_to_anchor=(1, 0.5))
    ax.axhline(y=0, color='black', linestyle='-', linewidth=0.5)
    plt.tight_layout()
    plt.show()
else:
    print("No rebalancing results available")

# 9. ALTERNATIVE: DIRECT MINIMUM VARIANCE OPTIMIZATION (Risk Aversion)
print("\n=== MINIMUM VARIANCE OPTIMIZATION (Equivalent to risk_aversion=10) ===")
ef_risk = EfficientFrontier(mu, S)
ef_risk.add_constraint(lambda w: w >= 0)

# Add box constraints
for i in range(len(asset_names)):
    ef_risk.add_constraint(lambda w, i=i: w[i] >= min_weights[i])
    ef_risk.add_constraint(lambda w, i=i: w[i] <= max_weights[i])

# Add group constraints
ef_risk.add_constraint(lambda w: sum(w[i] for i in group1_indices) >= 0.4)
ef_risk.add_constraint(lambda w: sum(w[i] for i in group1_indices) <= 0.6)
ef_risk.add_constraint(lambda w: sum(w[i] for i in group2_indices) >= 0.4)
ef_risk.add_constraint(lambda w: sum(w[i] for i in group2_indices) <= 0.6)

# Minimize volatility (variance)
ef_risk.min_volatility()
weights_risk = ef_risk.clean_weights()

print("Minimum Variance Portfolio Weights:")
for asset, weight in weights_risk.items():
    if weight > 0.001:
        print(f"{asset}: {weight:.4f} ({weight*100:.2f}%)")

# Display efficient frontier
print("\n=== EFFICIENT FRONTIER ===")
ef_plot = EfficientFrontier(mu, S)
fig, ax = plt.subplots(figsize=(10, 6))
plotting.plot_efficient_frontier(ef_plot, ax=ax, show_assets=True)
ax.set_title('Efficient Frontier')
plt.show()

# 10. SUMMARY STATISTICS
print("\n=== SUMMARY STATISTICS ===")
print(f"Number of assets: {len(asset_names)}")
print(f"Date range: {asset_returns.index[0]} to {asset_returns.index[-1]}")
print(f"Number of observations: {len(asset_returns)}")
print(f"Annual returns (mean): {mu.mean():.4f}")
print(f"Annual volatility (mean): {np.sqrt(np.diag(S)).mean():.4f}")
```

## Purpose

The objective of this project is to explore financial index return data and prepare it for portfolio analysis and optimization.


