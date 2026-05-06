# Title: Crash level factors analysis 

---

### Team Outliers
Mustafa El Zayyat
Chenxi Zhang

---

## Summary: [500-600 words] Description of your project, motivation, research question(s), and any findings
Originally planned on doing an airplane ticket analysis based on jet fuel price and demand. However, after facing integration issues due to a lack of common variables and a lack of descriptive variables that can be utilized for analysis, we decided to adopt a different project plan. After receiving feedback we chose to analyze the crashes and people datasets from the city of Chicago. Since we began our analysis of the new datasets our analytical focus shifted from originally focusing on linking speed cameras, crashes, and injuries to establishing a strong crash-level injury prediction framework first and building a structured dataset that identifies key drivers of injury severity (speed, lighting, weather). This can improve the project because we now have a baseline model of injury risk and speed camera analysis can be layered on top more rigorously. There was also some methodological improvement, instead of only descriptive analysis, we also added Logistic regression modeling and Feature engineering (grouped weather, lighting, speed bins) to improve analysis. The plan was updated to complete crash-level integration from crash and people datasets first then integrate enforcement data from speed camera dataset. Additionally, As discussed after the lab on Thursday, due to scheduling and time coordination issues, we have decided to move forward as a two person team.
Research question: What factors are most strongly associated with traffic crash occurrences in Chicago?

---

## Data profile: [max 2000 words] For each dataset used, describe its structure, content, and characteristics. Specify the location of the dataset files in your project 

---

## Data quality: [500-1000 words] Summary of the quality assessment.

### Dataset 1: Traffic Crashes – People
#### Before Cleaning 
The People dataset, loaded via the Socrata API filtered to records from 2021-01-01, has 1,276,673 rows and 29 columns. After standardizing sentinel values ("UNKNOWN", "NONE", and empty strings to pd.NA). We find severe missingness across columns. Ten columns exceed the 80% missingness threshold and were dropped: cell_phone_use (100.0%), bac_result_value (99.9%), ejection (99.7%), ems_run_no (98.6%), pedpedal_visibility, pedpedal_location, and pedpedal_action (all ~97.9%), ems_agency (92.3%), hospital (87.3%), and seat_no (80.2%). These columns carry negligible signal across the dataset. 

Consistency issues include mixed-case values in sex (M/F/X) and person_type, and crash_date stored as an ISO timestamp string requiring datetime parsing. Validity concerns include the pervasive use of "UNKNOWN" and "NONE" as non-null placeholder strings across categorical fields, which without standardization would inflate category counts in any groupby operation.

#### After Cleaning
After dropping the 10 high-missingness columns, 19 columns remain. Column names were lowercased, crash_date converted to datetime, and the date filter applied as a safety check (no additional rows removed since the API already filtered to 2021+). Residual missingness in retained columns: driver_action (69.5%), driver_vision (63.6%), drivers_license_class (54.2%), physical_condition (49.2%), drivers_license_state (41.8%), zipcode (33.0%), age (29.5%), city (28.5%), state (26.8%), bac_result (20.0%). The key outcome variable injury_classification has near-zero missingness (0.02%), confirming the target label is reliable.

#### Potential Drawbacks
We find igh missingness in driver_action, driver_vision, and physical_condition means these columns cannot be used as model features without imputation or subsetting, and the subset of records where they are populated likely corresponds to more severe or formally investigated crashes. So there might be selection bias. Dropping the pedestrian-specific columns (pedpedal_action, pedpedal_visibility, pedpedal_location) means pedestrian-involved crashes lose their specific behavioral context.
  
---


## Data cleaning: [max 1000 words] Summarize the data cleaning operations you performed and explain how each operation addressed specific data quality issues in your datasets.

---

## Findings: [~500 words] Description of any findings including numeric results and/or visualizations.

---

## Future work: [~500-1000 words] Brief discussion of any lessons learned and potential future work.

---

## Challenges: [~500 words] Discuss the main challenges you encountered while working on the project.

---

## Reproducing: Sequence of steps required for someone else to reproduce your results.

---

References: Formatted citations for any papers, datasets, or software used in your project.
City of Chicago. (2026). Traffic Crashes – People. Chicago Data Portal. https://data.cityofchicago.org/Transportation/Traffic-Crashes-People/u6pd-qa9d/about_data
City of Chicago. (2026). Traffic Crashes – Crashes. Chicago Data Portal. https://data.cityofchicago.org/Transportation/Traffic-Crashes-Crashes/85ca-t3if/about_data
