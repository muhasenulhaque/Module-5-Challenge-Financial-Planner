# Module-5-Challenge-Financial-Planner
Financial Planner for Retirement with Investment in Crypto and Shares 
## Initial imports
import os
import json
import requests
import pandas as pd
from dotenv import load_dotenv
import alpaca_trade_api as tradeapi
from MCForecastTools import MCSimulation


%matplotlib inline

## Load .env enviroment variables
load_dotenv()

## Set current amount of crypto assets
my_btc = 1.2
my_eth = 5.3

## Crypto API URLs for Bitcoin
bitcoin_price_url = "https://api.alternative.me/v2/ticker/Bitcoin/?convert=USD"

## Fetch current BTC price
response_data_btc = requests.get(bitcoin_price_url)
response_content = response_data_btc.content
data_btc = response_data_btc.json()
data_btc

## Creating Indent to read the data more organized way
print(json.dumps(data_btc, indent=4))

## getting the price data of BitCoin
price_btc = data_btc['data']['1']['quotes']['USD']['price']
price_btc

## Crypto API URLs for Ethereum
etherium_price_url = "https://api.alternative.me/v2/ticker/Ethereum/?convert=USD"

## Fetch current ETH price
response_data_ETH = requests.get(etherium_price_url)
print(response_data_ETH)

## Creating Indent to read the data more organized way
data_ETH = response_data_ETH.json()
print(json.dumps(data_ETH, indent=4))

## getting the price data of Ethereum
price_ETH = data_ETH['data']['1027']['quotes']['USD']['price']
price_ETH

## Compute current value of my crpto
my_btc_value = my_btc * price_btc
my_eth_value=my_eth * price_ETH
## Print current crypto wallet balance
print(f"The current value of your {my_btc} BTC is ${my_btc_value:0.2f}")
print(f"The current value of your {my_eth} ETH is ${my_eth_value:0.2f}")


## Set current amount of shares
my_agg = 200
my_spy = 50

## Set Alpaca API key and secret

key_id = os.getenv("alpaca_api_key")
secret_key = os.getenv("alpaca_secret_key")

## Verify that Alpaca key and secret were correctly loaded
print(f"Alpaca Key type: {type(alpaca_api_key)}")
print(f"Alpaca Secret Key type: {type(secret_key)}")
## Create the Alpaca API object
alpaca = tradeapi.REST(
    key_id,
    secret_key,
    api_version="v2")
alpaca

## Format current date as ISO format
today = pd.Timestamp("2020-08-07", tz="America/New_York").isoformat()
## Set the tickers
tickers = ["AGG", "SPY"]

## Set timeframe to "1Day" for Alpaca API
timeframe = "1Day"
start_date ="2020-08-07"
end_date = "2020-08-07"
## Get current closing prices for SPY and AGG
df_portfolio = alpaca.get_bars(
    tickers,
    timeframe,
    start = today,
    end = today
).df
## Reorganize the DataFrame
df_portfolio

## Separate ticker data
AGG = df_portfolio[df_portfolio['symbol']=='AGG'].drop('symbol', axis=1)
SPY = df_portfolio[df_portfolio['symbol']=='SPY'].drop('symbol', axis=1)

## Concatenate the ticker DataFrames
df_portfolio = pd.concat([AGG,SPY],axis =1, keys = ['AGG','SPY'])
## Preview DataFrame
df_portfolio


## Pick AGG and SPY close prices
df_closing_prices = pd.DataFrame()

## Fetch the closing prices 
df_closing_prices["AGG"] = df_portfolio["AGG"]["close"]
df_closing_prices["SPY"] = df_portfolio["SPY"]["close"]

df_closing_prices.index = df_closing_prices.index.date

df_closing_prices

## Fetch the current closing prices from the DataFrame
agg_close_price = float(df_closing_prices['AGG'])
spy_close_price =float(df_closing_prices['SPY'])

## Print AGG and SPY close prices
print(f"Current AGG closing price: ${agg_close_price}")
print(f"Current SPY closing price: ${spy_close_price}")

## Compute the current value of shares
my_agg_value = my_agg * agg_close_price
my_spy_value = my_spy * spy_close_price
## Print current value of shares
print(f"The current value of your {my_spy} SPY shares is ${my_spy_value:0.2f}")
print(f"The current value of your {my_agg} AGG shares is ${my_agg_value:0.2f}")

## Set monthly household income
monthly_income = 70000

## Consolidate financial assets data
shares = my_agg_value+my_spy_value
crypto = my_btc_value +my_eth_value
total_savings = shares + crypto
display(crypto)

## Create savings DataFrame
savings = [shares,crypto]
df_savings = pd.DataFrame(data = savings,index =['shares','crypto'],columns = ['Amount'])
## Display savings DataFrame
display(df_savings)

## Plot savings pie chart
df_savings.plot(kind='pie',y='Amount')

## Set ideal emergency fund
emergency_fund = monthly_income * 3

## Calculate total amount of savings
savings = shares + crypto

## Validate saving health
if savings>emergency_fund:
    print(f"Congratulations!You have Enough Fund to support emergency"),
else:
    print(f"Not Enough Fund for Emergency. Need to save additional {emergency_fund -savings}")

## Set start and end dates of five years back from today.
### Sample results may vary from the solution based on the time frame chosen
start_date = pd.Timestamp('2016-05-01', tz='America/New_York').isoformat()
end_date = pd.Timestamp('2021-05-01', tz='America/New_York').isoformat()

## Get 5 years' worth of historical data for SPY and AGG

## Set timeframe to "1Day" for Alpaca API
timeframe = "1Day"
## Get current closing prices for SPY and AGG
df_portfolio_5Y = alpaca.get_bars(
    tickers,
    timeframe,
    start = start_date,
    end = end_date
).df
## Reorganize the DataFrame
df_portfolio_5Y

## Reorganize the DataFrame
## Separate ticker data
AGG = df_portfolio_5Y[df_portfolio_5Y['symbol']=='AGG'].drop('symbol', axis=1)
SPY = df_portfolio_5Y[df_portfolio_5Y['symbol']=='SPY'].drop('symbol', axis=1)


## Concatenate the ticker DataFrames
df_stock_data = pd.concat([AGG,SPY],axis =1, keys = ['AGG','SPY'])

## Display sample data
df_stock_data.head()

## Configuring a Monte Carlo simulation to forecast 30 years cumulative returns
num_sims = 100

MC_AGG = MCSimulation(
    portfolio_data = df_stock_data,
    num_simulation = num_sims,
    num_trading_days = 252*30
)
MC_AGG

## Printing the simulation input data
MC_AGG.portfolio_data.head()

## Running a Monte Carlo simulation to forecast 30 years cumulative returns
MC_AGG.calc_cumulative_return()

## definging a veriable for the return
simulated_returns = MC_AGG.calc_cumulative_return()

## Plot simulation outcomes
MC_AGG.plot_simulation()

## Plot distribution of outcomes
MC_AGG.plot_distribution()

# Set initial investment
initial_investment = 20000

## Compute summary statistics from the simulated daily returns
## Multiply an initial investment by the daily returns of simulative stock prices to return the progression of daily returns in terms of money
cumulative_pnl = initial_investment * simulated_return
cumulative_pnl.head()

## Summary of simulated returns 
summary = MC_AGG.summarize_cumulative_return()

## Use the lower and upper `95%` confidence intervals to calculate the range of the possible outcomes of our $10,000 investments in TSLA stocks
ci_lower = round(summary[8]*initial_investment,2)
ci_upper = round(summary[9]*initial_investment,2)

## Print results
print(f"There is a 95% chance that an initial investment of ${initial_investment} in the portfolio"
      f" over the next 30 year will end within in the range of"
      f" ${ci_lower} and ${ci_upper}.")
