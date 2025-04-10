# [PYTHON] E-Wallet Company Data Wrangling & Exploratory Analysis
Using Python to perform Data Wrangling and EDA on an E-Wallet company dataset.

## 1. Introduction
This project aims to analyze important datasets from an E-wallet company to uncover valuable business insights. The primary goal is to explore payment volumes, product information, and transaction patterns using the following datasets:
- payment_report.csv: Includes monthly payment volumes for different products, offering insights into the performance of each product over time
- product.csv: provides detailed product information, including IDs, categories, and team ownership, allowing for a thorough analysis of product segments
- transactions.csv: Contains transaction details, such as timestamps, amounts, and product IDs, enabling a detailed analysis of customer behavior and transaction trends

There are two main parts in this project:
- Part I: EDA\
Exploratory Data Analysis (EDA) includes merging and cleaning the payment and product datasets, detecting missing values, duplicates, incorrect data types, and ensuring overall data quality.
- Part II: Data Wrangling\
Data Wrangling focuses on extracting key insights, such as identifying top-performing products, evaluating team performance, and classifying various transaction types, including refunds and transfers. The goal is to derive actionable insights from the datasets to support better decision-making.

## 2. Dataset Overview
### payment_report
|report_month| payment_group| product_id| source_id| volume|
|------------|--------------|-----------|----------|-------|
|2023-01|	payment|	12|	45|	624110375|
|2023-01|	payment|	17|	45|	335715113|
|2023-01|	payment|	18|	45|	737784466|
|2023-01|	payment|	19|	45|	120963069|
|2023-01|	payment|	20|	45|	319653158|
|...|	...|	...|	...|	...|
|2023-04|	payment|	15067|	45|	1504000|
|2023-04|	refund|	1976|	37|	3542271587|
|2023-04|	refund|	1976|	38|	13831708189|
|2023-04|	refund|	1976|	39|	1905435543|
|2023-04|	refund|	1976|	39|	3679922071|

### product
|product_id|	category|	team_own|
|----------|----------|---------|
|17|	PXXXXXB|	ASD|
|18|	PXXXXXB|	ASD|
|20|	PXXXXXB|	ASD|
|287|	PXXXXXB|	ASD|
|372|	PXXXXXB|	ASD|
|...|	...|	...|
|321|	PXXXXXV|	ASD|
|322|	PXXXXXV|	ASD|
|341|	PXXXXXV|	ASD|
|342|	PXXXXXV|	ASD|
|367|	PXXXXXV|	ASD|

### transactions
|transaction_id|	merchant_id|	volume|	transType|	transStatus|	sender_id|	receiver_id|	extra_info|	timeStamp|
|--------------|-------------|--------|----------|-------------|-----------|-------------|------------|----------|
|3002692434|	5|	100000|	24|	1|	10199794.0|	199794.0|	NaN|	1682932054455|
|3002692437|	305|	20000|	2|	1|	14022211.0|	14022211.0|	NaN|	1682932054912|
|3001960110|	7255|	48605|	22|	1|	NaN|	10530940.0|	NaN|	1682932055000|
|3002680710|	2270|	1500000|	2|	1|	10059206.0|	59206.0|	NaN|	1682932055622|
|3002680713|	2275|	90000|	2|	1|	10004711.0|	4711.0|	NaN|	1682932056197|
|...|	...|	...|	...|	...|	...|	...|	...|	...|
|3003723030|	305|	20000|	2|	1|	24524311.0|	NaN|	NaN|	1683035672634|
|3003723033|	2270|	100000|	2|	1|	10277242.0|	277242.0|	NaN|	1683035672876|
|3003723036|	2270|	100000|	2|	1|	10144599.0|	144599.0|	NaN|	1683035672892|
|3003723039|	5|	400|	22|	1|	10028007.0|	21013762.0|	NaN|	1683035672896|
|3003602967|	2250|	1|	8	|1|	38559843.0|	24501638.0|	NaN|	1683035673053|

## 3. Part I: EDA
### Merge payment_report.csv with product.csv
- Python code
~~~python
payment_enriched = pd.merge(payment_report,product,how='left',on='product_id')
payment_enriched
~~~
- Results
  
| |report_month|	payment_group|	product_id|	source_id|	volume|	category|	team_own|
|-|------------|---------------|------------|----------|--------|---------|---------|
|0|	2023-01|	payment|	12|	45|	624110375|	PXXXXXT|	ASD|
|1|	2023-01|	payment|	17|	45|	335715113|	PXXXXXB|	ASD|
|2|	2023-01|	payment|	18|	45|	737784466|	PXXXXXB|	ASD|
|3|	2023-01|	payment|	19|	45|	120963069|	PXXXXXM2|	ASD|
|4|	2023-01|	payment|	20|	45|	319653158|	PXXXXXB|	ASD|
|...|	...|	...|	...|	...|	...|	...|	...|
|914|	2023-04|	payment|	15067|	45|	1504000|	PXXXXXR|	ASL|
|915|	2023-04|	refund|	1976|	37|	3542271587|	NaN|	NaN|
|916|	2023-04|	refund|	1976|	38|	13831708189|	NaN|	NaN|
|917|	2023-04|	refund|	1976|	39|	1905435543|	NaN|	NaN|
|918|	2023-04|	refund|	1976|	39|	3679922071|	NaN|	NaN|
### Missing data
- Python code
~~~python
payment_enriched.isnull().sum()
transactions.isnull().sum()
~~~
- Results

| transaction_id | merchant_id | volume | transType | transStatus | sender_id | receiver_id | extra_info | timeStamp |
|----------------|-------------|--------|-----------|-------------|-----------|-------------|------------|-----------|
| 0              | 0           | 0      | 0         | 0           | 49059     | 164795      | 1317907    | 0         |

**Note**:
- No null from payment_enriched
- 49059 NaN from transactions[sender_id]
- 164795 NaN from transactions[receiver_id]
- 1317907 NaN from transactions[extra_info]

**Solutions**:
- Ignore NaN from extra_info
- Delete if both sender_id and receiver_id are missing
~~~python
# Delete if both sender_id and receiver_id are missing
null_both = transactions[transactions['sender_id'].isnull() & transactions['receiver_id'].isnull()]
transactions_cleaned = transactions.dropna(subset=['sender_id','receiver_id'], how='all')

transactions_cleaned['sender_id'].fillna(-1, inplace=True)
transactions_cleaned['receiver_id'].fillna(-1, inplace=True)
transactions_cleaned.isnull().sum()
~~~
### Duplicates
~~~python
transactions_final = transactions_cleaned.drop_duplicates('transaction_id')
~~~

## 4. Part II: Data Wrangling
### Using payment_report.csv & product.csv
**4.1: Top 3 product_ids with the highest volume**
- Python code
~~~python
volume_by_product = payment_enriched.groupby('product_id',as_index=False).agg({'volume':'sum'})
top_3_product = volume_by_product.nlargest(3,'volume')
top_3_product
~~~~
- Results

|product_id|	volume|
|----------|--------|
|1976|	61797583647|
|429|	14667676567|
|372|	13713658515|

**4.2: Given that 1 product_id is only owed by 1 team, are there any abnormal products against this rule?**
- Python code
~~~python
unique_team_by_product = payment_enriched.groupby('product_id',as_index=False).agg({'team_own':'nunique'})
unique_team_by_product[unique_team_by_product['team_own'] !=1]
~~~
- Results

|product_id|	team_own|
|----------|----------|
|3|	0|
|1976|	0|
|10033| 0|	

**4.3: Find the team has had the lowest performance (lowest volume) since Q2.2023. Find the category that contributes the least to that team.**
- Python code
~~~python
# Edit date_time value
payment_enriched['report_month'] = pd.to_datetime(payment_enriched['report_month'])
payment_enriched['year'] = payment_enriched['report_month'].dt.year
payment_enriched['quarter'] = payment_enriched['report_month'].dt.quarter

# Filter data since Q2.2023
from_q2_2023 = payment_enriched[(payment_enriched['year']>=2023) & (payment_enriched['quarter']>=2)]

# Total volume by team
team_performance = from_q2_2023.groupby('team_own').agg({'volume':'sum'})

# Find the team with the lowest total volume
lowest_performance_team = team_performance['volume'].idxmin()
print(f"The lowest performing team since Q2.20023 is:{lowest_performance_team}")

# Find the category that contributes the least to that team
## Filter from the dataset the lowest performace team
data_team = from_q2_2023[from_q2_2023['team_own'] == lowest_performance_team]
## Group volume by category
category_contribute = data_team.groupby('category').agg({'volume':'sum'})
## The least contribute
lowest_category_contribute = category_contribute['volume'].idxmin()
print(f"Category that contributes the least to {lowest_performance_team} is:{lowest_category_contribute}")
~~~
- Results\
The lowest performing team since Q2.20023 is:APS\
Category that contributes the least to APS is:PXXXXXE

**4.4: Find the contribution of source_ids of refund transactions (payment_group = ‘refund’), what is the source_id with the highest contribution?**
- Python code
~~~python
# Filter payment_group = refund
refund_payment = payment_enriched[payment_enriched['payment_group'] == 'refund']

# Total volume group by source_id
soure_id_contribute = refund_payment.groupby('source_id')['volume'].sum()

# Highest contribution source_id
highest_source_id_contribute = soure_id_contribute.idxmax()
print(f"Source ID with the highest contribution to refund trasactions is: {highest_source_id_contribute}")
~~~
- Results\
Source ID with the highest contribution to refund trasactions is: 38

### Using transactions.csv
**4.5: Define type of transactions (‘transaction_type’) for each row, given:**\
transType = 2 & merchant_id = 1205: Bank Transfer Transaction\
transType = 2 & merchant_id = 2260: Withdraw Money Transaction\
transType = 2 & merchant_id = 2270: Top Up Money Transaction\
transType = 2 & others merchant_id: Payment Transaction\
transType = 8 & merchant_id = 2250: Transfer Money Transaction\
transType = 8 & others merchant_id: Split Bill Transaction\
Remained cases are invalid transactions
- Python code
~~~python
def define_transaction_type(row):
  if row['transType'] == 2 and row['merchant_id'] == 1205:
    return 'Bank Transfer Transaction'
  elif row['transType'] == 2 and row['merchant_id'] == 2260:
    return 'Withdraw Money Transaction'
  elif row['transType'] == 2 and row['merchant_id'] == 2270:
    return 'Top Up Money Transaction'
  elif row['transType'] == 2:
    return 'Payment Transaction'
  elif row['transType'] == 8 and row['merchant_id'] == 2250:
    return 'Transfer Money Transaction'
  elif row['transType'] == 8:
    return 'Split Bill Transaction'
  else:
    return 'Invalid Transactions'

transactions['transaction_types'] = transactions.apply(define_transaction_type, axis =1)
transactions
~~~
- Results

| |transaction_id|	merchant_id|	volume|	transType|	transStatus|	sender_id|	receiver_id|	extra_info|	timeStamp|	transaction_types|
|-|--------------|-------------|--------|----------|-------------|-----------|-------------|------------|----------|-------------------|
|0|	3002692434|	5|	100000|	24|	1|	10199794.0|	199794.0|	NaN|	1682932054455|	Invalid Transactions|
|1|	3002692437|	305|	20000|	2|	1|	14022211.0|	14022211.0|	NaN|	1682932054912|	Payment Transaction|
|2|	3001960110|	7255|	48605|	22|	1|	NaN|	10530940.0|	NaN|	1682932055000|	Invalid Transactions|
|3|	3002680710|	2270|	1500000|	2|	1|	10059206.0|	59206.0|	NaN|	1682932055622| Top Up Money Transaction|
|4|	3002680713|	2275|	90000|	2|	1|	10004711.0|	4711.0|	NaN|	1682932056197|	Payment Transaction|
|...|	...|	...|	...|	...|	...|	...|	...|	...| ...|	...|
|1323997|	3003723030|	305|	20000|	2|	1|	24524311.0|	NaN|	NaN|	1683035672634|	Payment Transaction|
|1323998|	3003723033|	2270|	100000|	2|	1|	10277242.0|	277242.0|	NaN|	1683035672876|	Top Up Money Transaction|
|1323999|	3003723036|	2270|	100000|	2|	1|	10144599.0|	144599.0|	NaN|	1683035672892|	Top Up Money Transaction|
|1324000|	3003723039|	5|	400|	22|	1|	10028007.0|	21013762.0|	NaN|	1683035672896|	Invalid Transactions|
|1324001|	3003602967|	2250|	1|	8|	1|	38559843.0|	24501638.0|	NaN|	1683035673053|	Transfer Money Transaction|

**4.6: Of each transaction type (excluding invalid transactions): find the number of transactions, volume, senders and receivers.**
- Python code
~~~python
# Excluding invalid transactions
filter_transactions = transactions[transactions['transaction_types'] != 'Invalid Transactions']

# Number of transactions, volume, senders and receivers group by transaction type
transaction_type_summary = filter_transactions.groupby('transaction_types').agg(
    num_transactions =('transaction_id','count'),
    total_volume = ('volume','sum'),
    num_senders = ('sender_id','nunique'),
    num_receivers =('receiver_id','nunique')).reset_index()
transaction_type_summary
~~~
- Results

|transaction_types|	num_transactions|	total_volume|	num_senders|	num_receivers|
|-----------------|-----------------|-------------|------------|---------------|
|Bank Transfer Transaction|	37879|	50605806190|	23156|	9271|
|Payment Transaction|	398677|	71851515181|	139583|	113298|
|Split Bill Transaction|	1376|	4901464|	1323|	572|
|Top Up Money Transaction|	290502|	108606478829|	110409|	110409|
|Transfer Money Transaction|	341177|	37033171492|	39021|	34585|
|Withdraw Money Transaction|	33725|	23418181420|	24814|	24814|
