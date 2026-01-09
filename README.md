# 📈 Campaign Performance Optimization 

This project focuses on analyzing and optimizing the performance of digital marketing campaigns. I built an end-to-end data pipeline to process raw media data, performed KPI analysis using SQL, applied machine learning models to predict and improve campaign ROI, and created compelling visualizations to support strategic decision-making.

## 🗂 Dataset
- Source: [Marketing Campaign Dataset on Kaggle](https://www.kaggle.com/datasets/rahulchavan99/marketing-campaign-dataset)
- Description: Contains data on impressions, clicks, media spend, platforms, keywords, timestamps, and more across various marketing campaigns.

## 🛠 Tools & Technologies
- **AWS S3, Glue, Athena** – for building the data pipeline, storing and querying data.
- **SQL** – for calculating key performance indicators (KPIs) and aggregating campaign data.
- **Scikit-learn (Random Forest)** – for regression modeling to identify drivers of ROI.
- **Python (Pandas, Matplotlib)** – for data analysis, simulations, and visualization.
- **Tableau** – for interactive dashboards and insights visualization.

## 📊 KPIs Calculated
- Click-through Rate (CTR)
- Cost per Click (CPC)
- Cost per 1000 Impressions (CPM)
- Return on Investment (ROI)
- Channel-level and platform-level campaign performance over time

## 📈 Machine Learning Model
- **Model Type**: Random Forest Regressor
- **Target Variable**: ROI
- **Accuracy**: R² score of 0.98 (98% accuracy)
- **Key Findings**:
  - **Total Spend** and **CPC** are the strongest predictors of ROI.
  - **Low Spend + Mid CPC** combinations yielded the best returns.
  - **High Spend & High CPC** resulted in low efficiency.
- Check out the analysis in the Jupyter Notebook: [campaign.ipynb](campaign.ipynb)

## 📌 Key SQL Analysis

### Channel & Platform KPIs
This optimized SQL query calculates both channel-level and campaign-level performance metrics in a single pass using `window functions`. It provides key KPIs such as CTR, CPC, CPM, and ROI segmented by `campaign_id`, `advertising_platform`, and `channel_name`.
```sql
-- Combine metrics into a single aggregation step for efficiency
WITH campaign_metrics AS (
    SELECT
        campaign_item_id,
        ext_service_name AS advertising_platform,
        channel_name,
        
        -- Channel-Level Metrics
        SUM(impressions) AS impressions,
        SUM(clicks) AS clicks,
        ROUND((SUM(clicks) * 100.0 / NULLIF(SUM(impressions), 0)), 2) AS ctr,         -- Click-through rate (%)
        
        ROUND(SUM(media_cost_usd), 2) AS total_spent,
        ROUND((SUM(media_cost_usd) / NULLIF(SUM(clicks), 0)), 2) AS cpc,              -- Cost per click (USD)
        ROUND((SUM(media_cost_usd) / NULLIF(SUM(impressions), 0)) * 1000, 2) AS cpm,  -- Cost per 1000 impressions
        ROUND((SUM(clicks) / NULLIF(SUM(media_cost_usd), 0)), 2) AS roi,              -- Return on investment

        -- Campaign-Level Aggregated Metrics
        SUM(SUM(impressions)) OVER(PARTITION BY campaign_item_id) AS total_campaign_impressions,
        SUM(SUM(clicks)) OVER(PARTITION BY campaign_item_id) AS total_campaign_clicks,
        ROUND((SUM(SUM(clicks)) OVER(PARTITION BY campaign_item_id) * 100.0 / 
               NULLIF(SUM(SUM(impressions)) OVER(PARTITION BY campaign_item_id), 0)), 2) AS total_campaign_ctr,
        
        SUM(SUM(media_cost_usd)) OVER(PARTITION BY campaign_item_id) AS total_campaign_spent,
        ROUND((SUM(SUM(media_cost_usd)) OVER(PARTITION BY campaign_item_id) / 
               NULLIF(SUM(SUM(clicks)) OVER(PARTITION BY campaign_item_id), 0)), 2) AS total_campaign_cpc,
        ROUND((SUM(SUM(media_cost_usd)) OVER(PARTITION BY campaign_item_id) / 
               NULLIF(SUM(SUM(impressions)) OVER(PARTITION BY campaign_item_id), 0)) * 1000, 2) AS total_campaign_cpm,
        ROUND((SUM(SUM(clicks)) OVER(PARTITION BY campaign_item_id) / 
               NULLIF(SUM(SUM(media_cost_usd)) OVER(PARTITION BY campaign_item_id), 0)), 2) AS total_campaign_roi

    FROM raw_media
    GROUP BY campaign_item_id, ext_service_name, channel_name
)

SELECT 
    campaign_item_id,
    advertising_platform,
    channel_name,
    impressions,
    clicks,
    ctr,
    total_spent,
    cpc,
    cpm,
    roi,

    -- Campaign-Level KPIs
    total_campaign_impressions,
    total_campaign_clicks,
    total_campaign_ctr,
    total_campaign_spent,
    total_campaign_cpc,
    total_campaign_cpm,
    total_campaign_roi

FROM campaign_metrics
ORDER BY campaign_item_id, advertising_platform, channel_name
LIMIT 20;
```
### Time-Based Trends
```sql
-- Analyzing ROI by day of the week
SELECT DATE(time) AS campaign_date, weekday_cat,
       SUM(impressions), SUM(clicks), SUM(media_cost_usd),
       ROUND((SUM(clicks) * 100.0 / NULLIF(SUM(impressions), 0)), 2) AS ctr,
       ROUND((SUM(clicks) / NULLIF(SUM(media_cost_usd), 0)), 2) AS roi
FROM raw_media
GROUP BY DATE(time), weekday_cat
ORDER BY campaign_date
```

### 📊 Check out the charts here: [Tableau Visualizations](https://public.tableau.com/views/Book1_17262486487190/Sheet1?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)
