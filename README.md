Members
DongHyeok Kim, DongJun Lee

Overview
Overview of NYC listing of Airbnb for analysis

Research Question
What distinct segments of Airbnb listings can be identified in New York City based on their 
pricing, booking, and review characteristics and which characteristics best explain each segment?

Data description
About one third of listings (9,903) have no price value, which will need handling before any price analysis.
last_review and reviews_per_month are missing for the same 8,309 listings — these are listings with zero reviews.
The median minimum stay is 30 nights, likely reflecting NYC's short-term rental regulations.
Prices are heavily right-skewed (median $175 vs max $ 31,211), so the median is a better summary than mean.
Name: Inside Airbnb – New York City Listings
Source: Inside Airbnb (http://insideairbnb.com/get-the-data/)













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

