-- Churn Data Analysis Using SQL --

SELECT * FROM datasets.`telco-churn`; # A quick glans at the data 

-- Data Cleaning 
# Total number of customers
select distinct count(customerID) as TotalCustomers from datasets.`telco-churn`;

# Check for the duplicate values
select customerID , count(customerID) as no_of_times from datasets.`telco-churn`
group by customerID having count(customerID)>1;

# check for null values
SELECT
    'SeniorCitizen' AS ColumnName,
    SUM(CASE WHEN SeniorCitizen IS NULL THEN 1 ELSE 0 END) AS nullcount
FROM datasets.`telco-churn`
UNION
SELECT
    'tenure' AS ColumnName,
    SUM(CASE WHEN tenure IS NULL THEN 1 ELSE 0 END) AS nullcount
FROM datasets.`telco-churn`
UNION
SELECT
    'TechSupport' AS ColumnName,
    SUM(CASE WHEN TechSupport IS NULL THEN 1 ELSE 0 END) AS nullcount
FROM datasets.`telco-churn`;

-- Data Exploration & Analysis

# Customer Demographics
SELECT gender, COUNT(*) AS customer_count
FROM datasets.`telco-churn`
GROUP BY gender;

# retention and churn distribution based on gender
SELECT gender, Churn, 
COUNT(*) AS count_by_gender,
ceil((COUNT(*) / SUM(COUNT(*)) OVER(PARTITION BY Gender)) * 100) AS retention_churn_rate_by_gender
FROM datasets.`telco-churn`
GROUP BY 1, 2; # using partition to get the total count for each gender and calculate the churn rate

# Note:  churn rate(%) =  number of churned customers / total customers * 100

SELECT Churn, COUNT(*) AS churn_count, 
ceil((COUNT(*) / (SELECT COUNT(*) FROM datasets.`telco-churn`)) * 100) AS retention_churn_rate
FROM datasets.`telco-churn`
GROUP BY Churn;

#  Income Revenue (total charges) made from customers based on their gender
select gender, count(*) as totalcustomer, 
round(sum(TotalCharges),2) as totalcharges 
from datasets.`telco-churn` group by gender order by totalcharges desc;

-- Note:  churn rate(%) =  number of churned customers / total customers * 100
# churn rate for Senior citizens
SELECT
CASE WHEN SeniorCitizen = 1 THEN 'Senior' ELSE 'Non-Senior' END AS SeniorCitizen,
COUNT(*) AS TotalCustomers,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
CEIL(SUM(TotalCharges)) AS TotalCharges,
ceil((SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100)) AS ChurnRate
FROM datasets.`telco-churn`
GROUP BY SeniorCitizen
ORDER BY ChurnRate DESC;

# churn rate for customers with and without partners, and what is the overall churn rate
Select Partner, count(*) as Totalcustomer, 
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
from datasets.`telco-churn` 
group by Partner order by churnrate desc;

# churn rate for customers at different levels of tenure
Select distinct tenure from datasets.`telco-churn` order by tenure desc ;
SELECT 
	CASE 
		WHEN tenure <=12 THEN '0–12'
		WHEN tenure <=24 THEN '12–24'
		WHEN tenure <=36 THEN '24–36'
		WHEN tenure <=48 THEN '36–48'
		WHEN tenure <=60 THEN '48–60'
		ELSE '60+' 
	END AS tenurerange, 
    (SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate,
	SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
	COUNT(*) AS Totalcustomer
	FROM datasets.`telco-churn`
	GROUP BY tenurerange
	ORDER BY ChurnRate DESC;

# churn rate for customers with Online Security
Select OnlineSecurity, COUNT(*) AS Totalcustomer,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
group by 1 
order by 4 desc;

# churn rate for customers with Online Backup
Select OnlineBackup, COUNT(*) AS Totalcustomer,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
group by 1 order by 4 desc;

# churn rate for customers with Tech Support
Select TechSupport, COUNT(*) AS Totalcustomer,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
group by 1 order by 4 desc;

# Does churn rate vary based on the contract type, and if so, how does it vary?
Select Contract, COUNT(*) AS Totalcustomer,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
group by 1 order by 4 desc;

#Does Streaming TV, and Streaming Movies impact the churn rate
Select StreamingTV, COUNT(*) AS Totalcustomer,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
group by 1 order by 4 desc;

Select StreamingMovies, COUNT(*) AS Totalcustomer,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
group by 1 order by 4 desc;

-- Does streaming services influence by subscription contract
# we can derive that customer churn rate in month-to-month streaming services is significantly higher, which can be because 
# customers tend to subscribe to these services only for the duration of the extra 1-month free trial leading to high attrition rate 
SELECT Contract,
SUM(CASE WHEN StreamingTV IN ('yes') THEN 1 ELSE 0 END) AS TotalStreamingTvCustomers,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedStreamingCustomers,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
WHERE StreamingTV IN ('yes')
GROUP BY Contract ORDER BY ChurnRate DESC;

SELECT Contract,
SUM(CASE WHEN StreamingMovies IN ('yes') THEN 1 ELSE 0 END) AS TotalStreamingTvCustomers,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedStreamingMoviesCustomers,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
WHERE StreamingTV IN ('yes')
GROUP BY Contract ORDER BY ChurnRate DESC;

-- Feature Analysis 

# Does the Paperless Billing feature effectively attract customers?
# Paperless billing shows a negative trend, indicating potential payment issues and customers do not particularly appreciate this feature.
Select PaperlessBilling, COUNT(*) AS Totalcustomer,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
cast(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100 as decimal (10,2)) AS ChurnRate
FROM datasets.`telco-churn`
group by 1 order by 4 desc;

# commonly used payment methods
Select PaymentMethod, COUNT(*) AS Totalcustomer,
SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) AS ChurnedCustomers,
cast(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100 as decimal (10,2)) AS ChurnRate
FROM datasets.`telco-churn`
group by 1 order by 4 desc;

# Where is the most revenue generated via payment method
select PaymentMethod, count(*) as totalcustomer, 
round(sum(TotalCharges),2) as totalRevenuecharges 
from datasets.`telco-churn` group by PaymentMethod order by totalRevenuecharges desc;

-- More Analysis

# Segment customers into usage
SELECT 
CASE 
WHEN MonthlyCharges > 70 THEN 'High-Usage'
WHEN MonthlyCharges <= 70 AND MonthlyCharges >= 50 THEN 'Medium-Usage'
ELSE 'Low-Usage'
END AS usage_segment, 
COUNT(*) AS churn_count,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
WHERE MonthlyCharges IS NOT NULL
GROUP BY usage_segment;

# calculate the CLV for customers using Tenure and avg monthly charges
SELECT 
    CASE 
        WHEN tenure < 12 THEN '0-12 months'
        WHEN tenure >= 12 AND tenure < 24 THEN '12-24 months'
        WHEN tenure >= 24 AND tenure < 36 THEN '24-36 months'
        ELSE 'Over 36 months'
    END AS tenure_range,
    AVG(tenure) AS avg_tenure,
    round(AVG(MonthlyCharges),2) AS avg_monthly_charges,
    round(SUM(TotalCharges),2) AS total_charges,
    round(SUM(TotalCharges) / COUNT(*),2) AS clv_per_customer
FROM datasets.`telco-churn`
GROUP BY tenure_range
order by tenure_range;

-- Analyze Customer Satisfaction Analysis with services OnlineSecurity, TechSupport, and StreamingTV
SELECT 
OnlineSecurity,
TechSupport,
StreamingTV,
COUNT(*) AS churn_count,
(SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
FROM datasets.`telco-churn`
WHERE OnlineSecurity IS NOT NULL
AND TechSupport IS NOT NULL
AND StreamingTV IS NOT NULL
GROUP BY OnlineSecurity, TechSupport, StreamingTV;

-- Family Analysis
# Analyze how the presence of partners and dependents affects churn.
SELECT 
    CASE 
        WHEN Partner = 'Yes' AND Dependents = 'Yes' THEN 'Family'
        WHEN Partner = 'Yes' AND Dependents = 'No' THEN 'Partner (No Dependents)'
        WHEN Partner = 'No' AND Dependents = 'Yes' THEN 'Dependents (No Partner)'
        ELSE 'Single'
    END AS family_status,
    COUNT(*) AS churn_count,
    (SUM(CASE WHEN Churn = 'Yes' THEN 1 ELSE 0 END) / COUNT(*) * 100) AS ChurnRate
    FROM datasets.`telco-churn`
	WHERE Partner IS NOT NULL AND Dependents IS NOT NULL
	GROUP BY family_status
    order by ChurnRate desc;