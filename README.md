# Internship-
import pandas as pd
import numpy as np

# Load the dataset
file_path = 'path/to/your/trade_data.csv'  # Update this path
data = pd.read_csv(file_path)
# Data Exploration
print(data.head())
print(data.info())
print(data.describe())
# Data Cleaning
# Handle missing values
data.dropna(inplace=True)
# Feature Engineering
# Convert timestamps to datetime
data['timestamp'] = pd.to_datetime(data['timestamp'])
# Define positions based on 'side' and 'positionSide'
data['position'] = np.where(data['side'] == 'BUY', 'long_open', 'long_close')
data.loc[data['side'] == 'SELL', 'position'] = 'short_open'  # Assuming you have a similar condition
# Calculate realized profit
data['realizedProfit'] = np.where(data['side'] == 'SELL', data['quantity'] * data['price'], 0)  # Adjust as needed
# Calculate metrics
def calculate_metrics(group):
    total_positions = len(group)
    win_positions = len(group[group['realizedProfit'] > 0])
    total_investment = group['quantity'].sum()  # Adjust based on how you define investment
    total_profit = group['realizedProfit'].sum()
    roi = (total_profit / total_investment) * 100 if total_investment != 0 else 0
    sharpe_ratio = (total_profit / total_positions)  # Simplified Sharpe Ratio
    mdd = (group['realizedProfit'].min() if total_positions > 0 else 0)  # Maximum Drawdown calculation
    win_rate = (win_positions / total_positions) * 100 if total_positions != 0 else 0

    return pd.Series({
        'Total Positions': total_positions,
        'Win Positions': win_positions,
        'ROI': roi,
        'PnL': total_profit,
        'Sharpe Ratio': sharpe_ratio,
        'MDD': mdd,
        'Win Rate': win_rate
    })
# Group by Port_ID and calculate metrics
metrics = data.groupby('Port_IDs').apply(calculate_metrics)
# Rank accounts based on metrics
metrics['Rank'] = metrics['ROI'].rank(ascending=False)  # You can adjust the ranking criteria
top_20_accounts = metrics.nsmallest(20, 'Rank')
# Save the metrics to CSV
metrics.to_csv('calculated_metrics.csv')

# Save the top 20 accounts
top_20_accounts.to_csv('top_20_accounts.csv')
# Documentation: Output report detailing approach and findings
report = f"""
Analysis Report:
=================
- Total Accounts Analyzed: {len(metrics)}
- Top 20 Accounts based on ROI:
{top_20_accounts}

Metrics calculated include:
- Total Positions
- Win Positions
- ROI
- PnL
- Sharpe Ratio
- MDD
- Win Rate
"""
with open('analysis_report.txt', 'w') as report_file:
    report_file.write(report)
print("Analysis completed. Results saved.")
