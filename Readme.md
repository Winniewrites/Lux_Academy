### **RFM Analysis**
RFM analysis is a powerful technique used by companies to better understand customer behaviour and optimize engagement strategies. It revolves around three key dimensions: recency, frequency, and monetary value. These dimensions capture essential aspects of customer transactions, providing valuable information for segmentation and personalized marketing campaigns.

The given dataset is provided by an e-commerce platform containing customer transaction data including customer ID, purchase date, transaction amount, product information, ID command and location. The platform aims to leverage RFM (recency, frequency, monetary value) analysis to segment customers and optimize customer engagement strategies.

Your task is to perform RFM analysis and develop customer segments based on their RFM scores. The analysis should provide insights into customer behaviour and identification of high-value customers, at-risk customers, and potential opportunities for personalized marketing campaigns.

### **The Code on Jupyter Notebook**

import pandas as pd
import datetime as dt

# Load the dataset
data = pd.read_csv('C:/Users/Admin/Desktop/python/Lux academy/week2 ass/rfm_data.csv')

# Check the data
print(data.head())

# Convert 'PurchaseDate' column to datetime
data['PurchaseDate'] = pd.to_datetime(data['PurchaseDate'])

# Calculate the recency, frequency, and monetary values
snapshot_date = max(data['PurchaseDate']) + dt.timedelta(days=1)
rfm = data.groupby('CustomerID').agg({
    'PurchaseDate': lambda x: (snapshot_date - x.max()).days,
    'OrderID': 'count',
    'TransactionAmount': 'sum'
})

# Rename columns for clarity
rfm.rename(columns={'PurchaseDate': 'Recency', 'OrderID': 'Frequency', 'TransactionAmount': 'Monetary'}, inplace=True)

# Check the RFM dataframe
print(rfm.head())

# Define quartiles for Recency, Frequency, and Monetary
quartiles = rfm.quantile(q=[0.25, 0.5, 0.75])

# Function to assign R, F, M scores
def r_score(x, p, d):
    if x <= d[p][0.25]:
        return 4
    elif x <= d[p][0.50]:
        return 3
    elif x <= d[p][0.75]:
        return 2
    else:
        return 1

def fm_score(x, p, d):
    if x <= d[p][0.25]:
        return 1
    elif x <= d[p][0.50]:
        return 2
    elif x <= d[p][0.75]:
        return 3
    else:
        return 4

# Calculate R, F, M scores
rfm['R_Score'] = rfm['Recency'].apply(r_score, args=('Recency', quartiles))
rfm['F_Score'] = rfm['Frequency'].apply(fm_score, args=('Frequency', quartiles))
rfm['M_Score'] = rfm['Monetary'].apply(fm_score, args=('Monetary', quartiles))

# Calculate RFM Group and Score
rfm['RFM_Group'] = rfm['R_Score'].map(str) + rfm['F_Score'].map(str) + rfm['M_Score'].map(str)
rfm['RFM_Score'] = rfm['R_Score'] + rfm['F_Score'] + rfm['M_Score']

# Check the RFM dataframe with scores
print(rfm.head())

# Define RFM segments
def rfm_segment(row):
    if row['RFM_Score'] >= 9:
        return 'High Value'
    elif (row['RFM_Score'] >= 5) and (row['RFM_Score'] < 9):
        return 'Medium Value'
    else:
        return 'Low Value'

rfm['RFM_Segment'] = rfm.apply(rfm_segment, axis=1)

# Check the RFM dataframe with segments
print(rfm.head())

