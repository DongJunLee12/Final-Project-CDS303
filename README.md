# Explainable Cluster Analysis of NYC Airbnb Listings

Members
DongHyeok Kim, DongJun Lee

## Overview
We analyze listing-level Airbnb data from New York City to discover natural market segments among listings and to explain, in interpretable terms, what makes each segment distinct.

This matters to real decision makers:
- a new host deciding how to position and price a listing
- city regulators who want to understand what kinds of short-term rental activity dominate the market

Research Question
What distinct segments of Airbnb listings can be identified in New York City based on their pricing, booking, and review characteristics, and which characteristics best explain each segment?

Data description
Name: Inside Airbnb – New York City Listings (September 14, 2026 snapshot)
Source: Inside Airbnb (http://insideairbnb.com/get-the-data/)
Size: 29,918 listings and 19 variables
Unit of observation: one row = one Airbnb listing

Key findings from our initial exploration:
- About 35% of listings (10,512 out of 29,918) have no price value, which will need to be handled before any price analysis.
- last_review and reviews_per_month are missing for the same 8,114 listings; these are listings with zero reviews.
- The median minimum stay is 30 nights, which likely reflects NYC's short-term rental regulations.
- Prices are heavily right-skewed (median $182 vs. max $31,538), so the median is a better summary than the mean.




## Paper summary

The paper introduces a four-step framework for performing explainable cluster analysis. 

The main purpose is to find groups of similar observations in large datasets to understand which characteristics make each group unique.
Clustering means that the data does not have predefined group labels, it does not automatically explain why observations belong to certain groups.
The four step methodology in the paper is data preprocessing, dimensionality reduction, clustering, and classification.

Data Preprocessing
-Data preprocessing consists of handling missing values, detecting and removing outliers, preparing numerical and categorical variables, and scaling numerical variables.

Dimensionality Reduction
-Dimensionality reduction means to create smaller sets of variables to represent the data through the methods of SPCA and MCA each for numerical and categorical 
values. This method helps in understanding which variables contribute to the resulting components.

Clustering
-Clustering is to find patterns and identify groups with similar characteristics and it is emphasized that different clustering algorithms will produce different results
making it important to compare clustering.

Classification
-Classification is to find which variables are useful in predicting the cluster labels through methods like feature selection, hyperparameter optimization, cross-validation,
and explainability methods to find which variables are important. The steps are important to distinguish one cluster from another.

Why is it important?
-Classification is important for explaining clusters because clustering does not use any predefined labels and classification uses those clusters by clustering to learn in 
their model. The paper uses SHAP to measure importance to show which variables are most important for distinguishing clusters and contribute to predicting the cluster of a 
particular observation. This is important because classification lets us fully explain which variables distinguish those groups by clustering. 
Importantly, the cluster labels are used only as the target of the classification model and are never included as input features. Since the classifier largely reuses the same variables as the clustering stage, it characterizes and describes the clusters rather than independently validating them.







## Variable-Selection Table

| Variable | Description and unit | Data type | Proposed role | Rationale / preparation needed |
|---|---|---|---|---|
| id | Unique listing identifier | Numerical (ID) | Exclude | Identifier only; no analytical meaning. Kept aside as a join key for supplementary data. |
| name | Listing title text | Categorical (text) | Exclude | Free text; not usable for distance-based clustering without NLP. |
| host_id | Unique host identifier | Numerical (ID) | Exclude | Identifier; host scale is already captured by calculated_host_listings_count. |
| host_profile_id | Host profile identifier | Numerical (ID) | Exclude | Identifier; duplicates the role of host_id. |
| host_name | Host first name | Categorical | Exclude | Identifier-like; no analytical value. |
| neighbourhood_group | Borough (Manhattan, Brooklyn, …) | Categorical (5 levels) | Classification input / contextual | Strong candidate for explaining clusters geographically. Needs one-hot encoding. Kept out of clustering inputs so clusters form on listing behavior, then explained by location. |
| neighbourhood | Neighbourhood name (200+ levels) | Categorical | Contextual | Too high-cardinality for direct encoding; useful for labeling/interpreting clusters. |
| latitude / longitude | Geographic coordinates (degrees) | Numerical | Contextual (candidate clustering input) | Allow mapping of clusters and spatial joins with supplementary data. Raw coordinates in clustering can force purely geographic clusters, so we start with them excluded and revisit. |
| room_type | Entire home / private / shared / hotel | Categorical (4 levels) | Classification input | Core structural difference between listings; one-hot encoding required. |
| price | Nightly price (USD) | Numerical | Clustering input | Central to market segmentation. About one third of values missing (drop or impute); heavy right skew → log-transform + standardization. |
| minimum_nights | Minimum stay (nights) | Numerical | Clustering input | Separates short-stay vs. 30-night (regulation-driven) rental models. Extreme outliers (max 1,124) need capping. |
| number_of_reviews | Total review count | Numerical | Clustering input | Proxy for cumulative demand/activity. Right-skewed → scaling needed. |
| last_review | Date of most recent review | Date | Exclude (derive feature) | Raw dates unusable directly; may derive "days since last review" (recency) later. Missing = never reviewed. |
| reviews_per_month | Avg. monthly reviews | Numerical | Clustering input | Demand intensity normalized by time. Missing for zero-review listings → impute 0. Partially redundant with number_of_reviews; keep the less redundant one after correlation check. |
| calculated_host_listings_count | Number of listings the host operates | Numerical | Clustering input | Distinguishes individual hosts from commercial multi-listing operators — directly relevant to segmentation and regulation. Right-skewed → scaling. |
| availability_365 | Days available per year (0–365) | Numerical | Clustering input | Separates full-time rentals from occasionally shared homes. Already bounded; standardize. |
| number_of_reviews_ltm | Reviews in last 12 months | Numerical | Candidate / redundancy check | Overlaps with reviews_per_month; keep only if correlation analysis shows added information. |
| license | NYC registration/license number | Categorical | Exclude (possible derived flag) | About 83% missing; raw values unusable. A binary "has license" flag may serve as a contextual variable for regulation-related interpretation. |



- **Explain how the selected clustering variables relate to your research question.**

The selected clustering inputs (price, minimum_nights, reviews_per_month, number_of_reviews, calculated_host_listings_count, availability_365) each describe a different behavioral axis of a listing. 
How much it charges, how it is allowed to be booked, how much demand it receives, who operates it, and how often it is available. All six are continuous variables, which keeps them compatible with PCA-based dimensionality reduction in the next stage. Segments defined along these axes correspond directly to the "market segments" in our research question (e.g., budget long-stay rooms vs. premium full-time entire homes vs. commercial multi-listing operations).

- **Identify candidate predictors for explaining the resulting clusters through classification.**

The clustering inputs themselves plus room_type and neighbourhood_group serve as candidate predictors. Because the classifier reuses the clustering variables, it characterizes the clusters rather than independently validating them; room_type and borough additionally let us describe clusters in terms a stakeholder recognizes ("private rooms in Brooklyn").

- **Identify variables that should be excluded, such as identifiers or redundant variables.**

We exclude identifiers (id, host_id, host_profile_id, host_name, name), high-cardinality or mostly-missing fields (neighbourhood as input, license), and raw dates (last_review). Cluster labels generated in Step 3 will be used only as the classification target, never as input features.


- **Note potential preparation needs, including missing values, scaling, and categorical encoding.**

(1) Missing values: drop or impute `price` (about 35% missing; we will compare both strategies), and impute `reviews_per_month` with 0 for never-reviewed listings.
(2) Outliers: cap extreme values of `minimum_nights` and `price` before clustering.
(3) Scaling: log-transform heavily skewed variables, then standardize all clustering inputs. Scaling is mandatory before distance-based clustering and PCA.
(4) Encoding: one-hot encode `room_type` and `neighbourhood_group` for the classification stage.
(5) Correlation check: before dimensionality reduction, build a correlation matrix of the clustering inputs. PCA only helps when variables are linearly correlated, and this check will also settle the redundancy among the three review-count variables.






## Supplementary Dataset

https://data.cityofnewyork.us/City-Government/Demographic-Social-Economic-and-Housing-Profiles-b/kvuc-fg9b/about_data

Demographic, Social, Economic, and Housing Profiles by Community District/PUMA, Data Provided By Department of City Planning (DCP) Population, household income, employment, housing characteristics are the variables we want to focus on for now possibly changing depending on future findings.
We will be able to use this data to find which neighborhood one will be located to find characteristics by the variables given in this supplementary dataset
The original dataset provides information about individual listings, including price, room type, availability, and location. The supplementary dataset adds 
neighborhood-level information. By connecting our original data to their corresponding community districts or PUMAs using their geographic locations, 
we can compare Airbnb clusters with neighborhood characteristics. This will help explain differences between groups of Airbnb listings.



## Project Updates
- Session 1 (Nov 8, 2026): Repository created; paper framework summarized; variable-selection table completed; supplementary datasets identified.
- Session 2: (to be updated)
- Session 3: (to be updated)
- Final report & presentation: (to be updated)

*Generative AI (Claude & Chat Gpt) was used to assist in structuring and drafting this page; all analytical decisions were reviewed by the team.*
