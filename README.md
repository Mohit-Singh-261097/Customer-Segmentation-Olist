# Customer Segmentation Analysis — Olist E-Commerce

## Overview

Segmented **93,000+ customers** of a Brazilian e-commerce marketplace into four actionable groups using K-Means clustering, validated with Hierarchical Clustering and DBSCAN. The analysis identifies distinct customer personas based on purchasing behavior, delivery experience, geographic location, and satisfaction — enabling targeted marketing strategies and operational improvements.

**Dataset:** [Olist Brazilian E-Commerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (~100K orders, 2016–2018, 9 relational tables)

**Tools:** Python, Pandas, Scikit-learn, Seaborn, Matplotlib, SciPy

---

## Key Results

### Four Customer Segments

| Segment | Size | Avg Spend (R$) | Avg Review | Avg Delivery (days) | Defining Trait |
|---------|------|----------------|------------|---------------------|----------------|
| **Budget Casuals** | 44,209 (47%) | 67 | 4.65 | 9 | Happy one-time buyers, small items, fast delivery |
| **Dissatisfied At-Risk** | 27,081 (29%) | 125 | 1.62 | 19 | Poor experience, longest delivery, lowest satisfaction |
| **Premium Buyers** | 12,273 (13%) | 261 | 4.55 | 11 | High-value, heavy items, most installments |
| **Remote Regionals** | 9,773 (10%) | 168 | 4.04 | 20 | Northeast Brazil, far from logistics hubs |

### Segment Profiles (Radar Chart)

![Customer Segment Comparison](visuals/cluster_radar_chart.png)

### Geographic Distribution

![Geographic Distribution](visuals/geographic_scatter.png)

Remote Regionals (orange) cluster distinctly in Brazil's Northeast coast, while other segments overlap in the Southeast around São Paulo.

---

## Methodology

### 1. Data Cleaning & Merging
- Filtered to delivered orders only (~96K)
- Resolved duplicate reviews, cleaned erroneous geolocation coordinates
- Built unified order-item dataframe via 6-table merge chain using left joins



### 2. Customer-Level Aggregation
- Collapsed to **one row per unique customer** (not per order — critical distinction)
- Engineered 16 features across monetary, frequency, recency, product behavior, payment, satisfaction, delivery, and geography dimensions
- Used `customer_unique_id` (not `customer_id`) since Olist assigns new IDs per order

### 3. Feature Preparation
- Log-transformed skewed monetary and weight features
- Removed near-zero variance features (`order_count`, `total_items`, `unique_categories` — 97% of customers had value of 1)
- Dropped `avg_order_value` (0.92 correlation with `total_price`)
- StandardScaler applied to final 9 features

### 4. Clustering
- **K-Means (k=4):** Selected via elbow method + silhouette analysis. Silhouette scores 0.12–0.19, typical for real-world customer data
- **Hierarchical (Ward's):** 90%+ label agreement with K-Means — validates the 4-cluster structure as a genuine property of the data
- **DBSCAN (eps=1.2):** Placed 91.5% in one cluster, confirming customer behavior forms a continuous distribution rather than density-separated groups. K-Means provides more actionable segmentation

---

## Technical Challenges & Solutions

### 1. Resolving a Many-to-Many Data Architecture
**Challenge:**  
The Olist dataset contains **9 relational tables**, and a naive merge would create heavy duplication. For example, a single order can contain multiple items, each with its own shipping-related fields. This would inflate the dataset and distort customer behavior analysis.

**Solution:**  
I followed an **aggregation-first approach** before merging:
- Collapsed **item-level data** into **order-level metrics**
- Aggregated **order-level metrics** into **customer-level features**
- Used `customer_unique_id` as the final analytical grain

This ensured the clustering model learned from **unique customer behavior** rather than duplicated transaction rows.


### 2. Handling Extreme Data Skew
**Challenge:**  
Features such as **Total Price** and **Freight Value** showed strong right-skew / power-law behavior. This can bias K-Means because extreme values pull centroids toward outliers, weakening separation between realistic customer groups.

**Solution:**  
I applied a **log transformation** using `log(x + 1)` to normalize highly skewed monetary and weight-related variables.

This improved clustering by making the algorithm more sensitive to differences between:
- low spenders
- medium spenders
- high spenders

instead of compressing them into overly broad groups.


### 3. Reducing High-Dimensional Noise
**Challenge:**  
Initial feature engineering produced **16+ features**, but several were either:
- highly correlated
- near-constant across most customers
- redundant for clustering

Examples included overlapping spend variables such as `avg_order_value` and `total_price`.

**Solution:**  
I performed a **feature selection and noise-reduction step** by:
- removing features with **correlation > 0.90**
- dropping near-static features where **~97% of customers shared the same value**
- keeping only features that added meaningful behavioral signal

This reduced noise and helped clusters form around more interpretable customer patterns.


## ✅ Outcome
These steps made the dataset:
- analytically cleaner
- customer-centric in structure
- more suitable for unsupervised learning
- easier to interpret from a business perspective

As a result, the final segmentation was more stable, meaningful, and aligned with real customer behavior.

---

## Business Recommendations

| Segment | Strategy | Expected Impact |
|---------|----------|-----------------|
| **Budget Casuals** | Upsell via targeted product recommendations; free shipping on second order | Convert one-time buyers to repeat customers |
| **Dissatisfied At-Risk** | Root cause analysis on delivery failures; personalized recovery offers | Reduce churn in a 29% segment |
| **Premium Buyers** | VIP treatment — priority shipping, early sale access, loyalty tiers | Retain highest-revenue customers |
| **Remote Regionals** | Regional fulfillment centers; proactive delivery communication | Improve satisfaction in underserved Northeast |

---

## Key Insights

- **Delivery drives satisfaction:** Review scores and shipping days correlate at -0.33. Customers with 19+ day deliveries scored 1.6 vs 4.65 for those with 9-day deliveries
- **Geography matters:** Clustering naturally separated the Northeast (Remote Regionals) from the Southeast, reflecting real logistics infrastructure gaps
- **Most customers are one-time buyers:** ~97% placed a single order, meaning segmentation differentiates by *how* customers bought, not how *often*
- **Credit card dominates all segments** (~75%), so payment type doesn't differentiate personas

---
## 📈 Projected Business Impact

Based on the customer segmentation results, the analysis highlights several opportunities for measurable operational and financial improvement.

### 1. Revenue Recovery for the At-Risk Segment
**The Data:**  
Approximately **29% of customers** fall into the **Dissatisfied At-Risk** segment, representing over **$27,000 in customer value**. A major driver of dissatisfaction in this group is the long **average delivery time of 19 days**.

**Projected Impact:**  
By introducing **proactive delay notifications**, **service recovery messaging**, and **targeted recovery vouchers**, Olist could potentially retain **5–10% of customers** in this segment who might otherwise churn.

**Business Value:**  
This would help protect future **GMV**, improve retention, and reduce the revenue loss associated with poor delivery experiences.

### 2. Improved Marketing Efficiency for Budget Casuals
**The Data:**  
The **Budget Casuals** segment makes up approximately **47% of the customer base**, but shows relatively **low repeat purchase behavior**.

**Projected Impact:**  
Instead of relying on broad, untargeted email campaigns, Olist could improve performance through **personalized cross-sell campaigns** based on previous low-ticket or small-item purchases.

**Business Value:**  
This strategy could:
- improve **click-through rates (CTR)**
- increase second-purchase conversion
- reduce **customer acquisition cost (CAC)** for repeat buyers
- generate more value from the existing customer base without relying solely on new customer acquisition

### 3. Logistics Optimization for Remote Regionals
**The Data:**  
Around **10% of customers** are concentrated in the **Northeast region**, where they experience the **longest shipping times**, averaging **20 days**.

**Projected Impact:**  
This creates a strong business case for:
- **regional fulfillment centers**
- **localized inventory strategies**
- **specialized courier partnerships**

**Business Value:**  
Improving delivery speed in this region could increase customer satisfaction, reduce friction in the post-purchase experience, and potentially shift these customers into a **higher satisfaction / higher retention** segment.

## ✅ Strategic Takeaway
The segmentation does not just describe customer groups — it identifies where Olist can act with the highest business leverage:
- **retain revenue** from dissatisfied customers
- **improve marketing ROI** for casual buyers
- **optimize logistics** in underserved regions

This makes the project useful not only as a clustering exercise, but as a practical decision-support tool for **retention, marketing, and operations strategy**.

---

## Project Structure

```
Customer-Segmentation-Olist/
├── data/                        # Raw CSVs (not tracked — see .gitignore)
├── dashboard/
│   └──Customer_segmentation_dashboard.pdf                        
├── notebooks/
│   └── customer_segmentation.ipynb
├── visuals/                       # Exported charts
│   ├── cluster_radar_chart.png
│   ├── geographic_scatter.png
│   ├── correlation_matrix.png
│   └── cluster_distribution.png
├── customer_segments.csv          # Final output with cluster labels
├── requirements.txt
├── .gitignore
└── README.md
```

---

## How to Run

1. Clone the repository
2. Download the [Olist dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and place CSVs in `data/`
3. Install dependencies: `pip install -r requirements.txt`
4. Open and run `notebooks/customer_segmentation.ipynb`

---

## Requirements

```
pandas>=1.5.0
numpy>=1.23.0
matplotlib>=3.6.0
seaborn>=0.12.0
scikit-learn>=1.2.0
scipy>=1.10.0
```

---

## Dashboard

[Customer Segment Comparison Dashboard](dashboard/Customer_segmentation_dashboard.pdf)
