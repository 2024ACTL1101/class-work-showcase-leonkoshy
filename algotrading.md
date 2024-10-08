---
title: "ACTL1101 Assignment Part A"
author: "Leon Koshy"
output:
  html_document:
    df_print: paged
  pdf_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Algorithmic Trading Strategy

## Introduction

In this assignment, you will develop an algorithmic trading strategy by incorporating financial metrics to evaluate its profitability. This exercise simulates a real-world scenario where you, as part of a financial technology team, need to present an improved version of a trading algorithm that not only executes trades but also calculates and reports on the financial performance of those trades.

## Background

Following a successful presentation to the Board of Directors, you have been tasked by the Trading Strategies Team to modify your trading algorithm. This modification should include tracking the costs and proceeds of trades to facilitate a deeper evaluation of the algorithm’s profitability, including calculating the Return on Investment (ROI).

After meeting with the Trading Strategies Team, you were asked to include costs, proceeds, and return on investments metrics to assess the profitability of your trading algorithm.

## Objectives

1. **Load and Prepare Data:** Open and run the starter code to create a DataFrame with stock closing data.

2. **Implement Trading Algorithm:** Create a simple trading algorithm based on daily price changes.

3. **Customize Trading Period:** Choose your entry and exit dates.

4. **Report Financial Performance:** Analyze and report the total profit or loss (P/L) and the ROI of the trading strategy.

5. **Implement a Trading Strategy:** Implement a trading strategy and analyze the total updated P/L and ROI. 

6. **Discussion:** Summarise your finding.


## Instructions

### Step 1: Data Loading

Start by running the provided code cells in the "Data Loading" section to generate a DataFrame containing AMD stock closing data. This will serve as the basis for your trading decisions. First, create a data frame named `amd_df` with the given closing prices and corresponding dates. 

```{r load-data}

# Load data from CSV file
amd_df <- read.csv("AMD.csv")

# Convert the date column to Date type and Adjusted Close as numeric
amd_df$date <- as.Date(amd_df$Date)
amd_df$close <- as.numeric(amd_df$Adj.Close)

amd_df <- amd_df[, c("date", "close")]
```


##Plotting the Data
Plot the closing prices over time to visualize the price movement.
```{r plot}
plot(amd_df$date, amd_df$close,'l')
```


## Step 2: Customize Trading Period
- Define a trading period you wanted in the past five years 
```{r period}
# Step 3: Customize Trading Period

# Define the start and end dates for the trading period
start_date <- as.Date("2023-04-03", format="%Y-%m-%d")
end_date <- as.Date("2024-04-01", format="%Y-%m-%d")

# Filter the dataframe to include only the specified date range
amd_df <- subset(amd_df, date >= start_date & date <= end_date)

# Display the filtered dataframe to check the implementation
head(amd_df)
tail(amd_df)


```

## Step 3: Trading Algorithm
Implement the trading algorithm as per the instructions. You should initialize necessary variables, and loop through the dataframe to execute trades based on the set conditions.

- Initialize Columns: Start by ensuring dataframe has columns 'trade_type', 'costs_proceeds' and 'accumulated_shares'.
- Change the algorithm by modifying the loop to include the cost and proceeds metrics for buys of 100 shares. Make sure that the algorithm checks the following conditions and executes the strategy for each one:
  - If the previous price = 0, set 'trade_type' to 'buy', and set the 'costs_proceeds' column to the current share price multiplied by a `share_size` value of 100. Make sure to take the negative value of the expression so that the cost reflects money leaving an account. Finally, make sure to add the bought shares to an `accumulated_shares` variable.
  - Otherwise, if the price of the current day is less than that of the previous day, set the 'trade_type' to 'buy'. Set the 'costs_proceeds' to the current share price multiplied by a `share_size` value of 100.
  - You will not modify the algorithm for instances where the current day’s price is greater than the previous day’s price or when it is equal to the previous day’s price.
  - If this is the last day of trading, set the 'trade_type' to 'sell'. In this case, also set the 'costs_proceeds' column to the total number in the `accumulated_shares` variable multiplied by the price of the last day.



```{r trading}
# Initialize columns for trade type, cost/proceeds, and accumulated shares in amd_df
amd_df$trade_type <- NA
amd_df$costs_proceeds <- NA
amd_df$accumulated_shares <- 0

# Initialize variables for trading logic
previous_price <- 0
share_size <- 100
accumulated_shares <- 0

# Loop through the dataframe to execute trades based on the set conditions
for (i in 1:nrow(amd_df)) {
  current_price <- amd_df$close[i]
  
  # If the previous price is 0 (first day of trading), buy 100 shares
  if (previous_price == 0) {
    amd_df$trade_type[i] <- 'buy'
    amd_df$costs_proceeds[i] <- -current_price * share_size
    accumulated_shares <- accumulated_shares + share_size
  } else if (current_price < previous_price) { # Buy 100 shares if today's price is less than yesterday's price
    amd_df$trade_type[i] <- 'buy'
    amd_df$costs_proceeds[i] <- -current_price * share_size
    accumulated_shares <- accumulated_shares + share_size
  } else if (i == nrow(amd_df)) { # Sell all accumulated shares on the last day
    amd_df$trade_type[i] <- 'sell'
    amd_df$costs_proceeds[i] <- accumulated_shares * current_price
    accumulated_shares <- 0
  }
  
  # Update the accumulated shares column
  amd_df$accumulated_shares[i] <- accumulated_shares
  
  # Update the previous price for the next iteration
  previous_price <- current_price
}

# Display the resulting dataframe to check the implementation
head(amd_df)
tail(amd_df)
```



## Step 4: Run Your Algorithm and Analyze Results
After running your algorithm, check if the trades were executed as expected. Calculate the total profit or loss and ROI from the trades.

- Total Profit/Loss Calculation: Calculate the total profit or loss from your trades. This should be the sum of all entries in the 'costs_proceeds' column of your dataframe. This column records the financial impact of each trade, reflecting money spent on buys as negative values and money gained from sells as positive values.
- Invested Capital: Calculate the total capital invested. This is equal to the sum of the 'costs_proceeds' values for all 'buy' transactions. Since these entries are negative (representing money spent), you should take the negative sum of these values to reflect the total amount invested.
- ROI Formula: $$\text{ROI} = \left( \frac{\text{Total Profit or Loss}}{\text{Total Capital Invested}} \right) \times 100$$

```{r}
# Step 4: Run Your Algorithm and Analyze Results

# Calculate the total profit or loss
total_profit_loss <- sum(amd_df$costs_proceeds, na.rm = TRUE)

# Calculate the total capital invested
total_invested <- -sum(amd_df$costs_proceeds[amd_df$trade_type == 'buy'], na.rm = TRUE)

# Calculate ROI
ROI <- (total_profit_loss / total_invested) * 100

# Print the results
print(paste("Total Profit or Loss: ",total_profit_loss))
print(paste("Total Capital Invested:",total_invested))
print(paste("ROI:",ROI,"%"))


```

## Step 5: Profit-Taking Strategy 
- Option 1: Implement a profit-taking strategy that you sell half of your holdings if the price has increased by a certain percentage (e.g., 20%) from the average purchase price.
- Option 2: Implement a stop-loss mechanism in the trading strategy that you sell half of your holdings if the stock falls by a certain percentage (e.g., 20%) from the average purchase price. You don't need to buy 100 stocks on the days that the stop-loss mechanism is triggered.


```{r option}
# Step 5: Implement a Profit-Taking Strategy

# Initialize columns for trade type, cost/proceeds, and accumulated shares in amd_df
amd_df$trade_type <- NA
amd_df$costs_proceeds <- NA
amd_df$accumulated_shares <- 0

# Initialize variables for trading logic
previous_price <- 0
share_size <- 100
accumulated_shares <- 0
total_cost <- 0

# Define the profit-taking percentage
profit_taking_percentage <- 0.20

# Loop through the dataframe to execute trades based on the set conditions
for (i in 1:nrow(amd_df)) {
  current_price <- amd_df$close[i]
  
  # If the previous price is 0 (first day of trading), buy 100 shares
  if (previous_price == 0) {
    amd_df$trade_type[i] <- 'buy'
    amd_df$costs_proceeds[i] <- -current_price * share_size
    accumulated_shares <- accumulated_shares + share_size
    total_cost <- total_cost + current_price * share_size
  } else if (current_price < previous_price) { # Buy 100 shares if today's price is less than yesterday's price
    amd_df$trade_type[i] <- 'buy'
    amd_df$costs_proceeds[i] <- -current_price * share_size
    accumulated_shares <- accumulated_shares + share_size
    total_cost <- total_cost + current_price * share_size
  } else if (accumulated_shares > 0 && current_price >= (total_cost / accumulated_shares) * (1 + profit_taking_percentage)) {
    # Sell half of the holdings if the price has increased by the profit-taking percentage
    shares_to_sell <- accumulated_shares / 2
    amd_df$trade_type[i] <- 'sell'
    amd_df$costs_proceeds[i] <- shares_to_sell * current_price
    accumulated_shares <- accumulated_shares - shares_to_sell
  } else if (i == nrow(amd_df) && accumulated_shares > 0) { # Sell all accumulated shares on the last day
    amd_df$trade_type[i] <- 'sell'
    amd_df$costs_proceeds[i] <- accumulated_shares * current_price
    accumulated_shares <- 0
  }
  
  # Update the accumulated shares column
  amd_df$accumulated_shares[i] <- accumulated_shares
  
  # Update the previous price for the next iteration
  previous_price <- current_price
}

# Display the resulting dataframe to check the implementation
head(amd_df)
tail(amd_df)
```


## Step 6: Summarize Your Findings
- Did your P/L and ROI improve over your chosen period?
- Relate your results to a relevant market event and explain why these outcomes may have occurred.


```{r}
# Calculate the total profit or loss
total_profit_loss <- sum(amd_df$costs_proceeds, na.rm = TRUE)

# Calculate the total capital invested
total_invested <- -sum(amd_df$costs_proceeds[amd_df$trade_type == 'buy'], na.rm = TRUE)

# Calculate ROI
ROI <- (total_profit_loss / total_invested) * 100

# Print the results
print(paste("Total Profit/Loss:", total_profit_loss))
print(paste("Total Capital Invested:", total_invested))
print(paste("Return on Investment (ROI):", ROI, "%"))
# Discussion
discussion <- "
### Discussion

Did your P/L and ROI improve over your chosen period?
The implementation of the profit-taking strategy provided insights into the effectiveness of the algorithm. By selling half of the holdings when the price increased by 20%, the strategy aimed to lock in profits and minimize risk. 

Relate your results to a relevant market event and explain why these outcomes may have occurred.
For instance, on Wednesday, December 6, 2023, AMD CEO Lisa Su discussed a new graphics processor designed for AI servers, with Microsoft and Meta as committed users. The rise in AMD shares on the following Thursday suggests that investors believe in the chipmaker’s upward potential and market expectations. 

Analyzing the results during this period, we observed a significant rise in AMD's stock price, leading to an execution of the profit-taking strategy. The strategy earned a substantial profit on this day, contributing positively to the overall ROI. Specifically, the increase in share price by more than 20% triggered the sale of half of the holdings, which resulted in a realized profit and reduced exposure to potential future price drops.

My first strategy, without the profit-taking mechanism, would have held onto the shares longer, potentially missing the opportunity to lock in gains. Therefore, the profit-taking strategy provided a better ROI during this period, demonstrating its effectiveness in capitalizing on positive market movements and securing profits.

"


```






