# Title: Crash level factors analysis 

---

### Team Outliers
Mustafa El Zayyat , 
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
This dataset has 1,276,673 rows and 29 columns. After standardizing the sentinel values like UNKNOWN and NONE, we found that there were serious deficiencies among the columns. The 10 columns that exceed 80% of the missing threshold have been deleted. 

Consistency issues include mixed case values in sex and person_type, and crash_date stored as an ISO timestamp string requiring datetime parsing. Validity concerns include the pervasive use of UNKNOWN and NONE as non null placeholder strings across categorical fields, which without standardization would inflate category counts in any groupby operation.

#### After Cleaning
After dropping the 10 high missingness columns, we have 19 columns. Column names were lowercased, crash_date converted to datetime, and the date filter applied as a safety check, no additional rows removed. The key outcome variable injury_classification has barely any missingness. We think this makes the target label is reliable.

#### Potential Drawbacks of Cleaning
We find igh missingness in driver_action, driver_vision, and physical_condition means these columns cannot be used as model features without imputation or subsetting, and the subset of records where they are populated likely corresponds to more severe or formally investigated crashes. So there might be selection bias. Dropping the pedestrian-specific columns (pedpedal_action, pedpedal_visibility, pedpedal_location) means pedestrian-involved crashes lose their specific behavioral context.

### Dataset 2: Traffic Crashes Dataset
#### Before Cleaning 
This dataset records one row per crash event, including road conditions, weather, time, location, and outcomes. After filtering 2021, it has 585,088 rows and 48 columns.    

The crash identifier and the key results column are in good condition. Crash_record_id has no missing values. And the injection related column has only 1,324 missing entries among 585,088 records. The geographic coordinates were lost by 5,546 lines, which is only a small part, but these crashes cannot be placed on the map.

A notable data quality issue emerged in the weather_condition column. 43,443 records were marked as unknown approximately 7.4% of the dataset. Technically speaking, this is not a true deficiency, but unknown does not provide actual information about the road conditions at the time of the accident. The speed limit ranges from 0 to 70 miles per hour, with both the average and median around 30 miles per hour, which is consistent with the urban driving dataset.

#### After Cleaning
The cleaning pipeline replaces sentinel strings like UNKNOWN with proper NA values, drops columns that are >80% empty, lowercases all column names, and filters to 2021+. Most columns in the Crashes dataset are well populated, so relatively few are dropped. After cleaning, the date, location, road condition, weather, and crash outcome fields are all retained.

#### Potential Drawbacks of Cleaning
The 43,443 UNKNOWN weather records become missing values after cleaning. If bad weather is underreported in cases where the reporting officer did not know or did not fill in the weather field, and if bad weather also correlates  with crash severity, then models trained on this data may underestimate the effect of weather on injury outcomes. The 5,546 rows missing coordinates are also non random crashes in certain neighborhoods or with less thorough incident reports may be more likely to lack location data, which could introduce a geographic bias if location based features are used.

### Dataset 3: Speed Camera Violations Dataset
#### Before Cleaning 
The Speed Camera dataset records one row per camera per day. It shows how many violations occurred at each camera location. After filtering to 2021 onward, it has 234,913 rows and 9 columns.

The dataset is generally clean. Columns lacking data are geographical. Approximately 3.23% of the records about 7,592 rows are missing coordinate columns. The violation count column is the core metric of this dataset and has no missing values. The distribution of violations. Minimum is 1, maximum is 1,888, average is 56.5, and median is 34. A large gap between the mean and the median means a right skewed distribution. Most days with cameras recorded a moderate number of violations, but a few busy traffic locations recorded a very high number of violations on certain days.

#### After Cleaning
The same pipeline applied. Replace sentinel strings with NA, drop >80% missing columns, lowercase column names, parse dates, and filter to 2021+. Because the camera dataset is already quite clean. Only coordinates have any notable missingness, and no column is >80% empty. The row and column counts change very little after cleaning. The main effect of cleaning is standardizing the date field and ensuring the data covers the same 2021 to present window as the other two datasets.

#### Potential Drawbacks and Integration Decision
While the Speed Camera dataset was profiled and cleaned following the same pipeline as the other two datasets, it cannot be directly merged into the crash level analysis. The reason is a structural mismatch. Crash records exist at the individual person level which is one row per person per crash, while camera records exist at the one row per camera per day level. There is no shared key, no crash ID, person ID, or unique identifier that links a specific camera record to a specific crash record.

To illustrate why a naive join would be problematic: on a sample date (2026-02-25), there are 700 crash person records and 208 camera records for that day. Joining them purely on date would produce 700 × 208 = 145,600 rows. Each crash person paired with every camera, regardless of location or relevance. This kind of cartesian product join would introduce massive artificial duplication and would not represent any real relationship between the two data sources.         

For these reasons, the Speed Camera dataset was excluded from model integration. It is profiled here to document its characteristics and evaluation. Future work could potentially aggregate camera violations by geographic area and join at that coarser level, but this is outside the scope of the current analysis.

---

## Data cleaning: [max 1000 words] Summarize the data cleaning operations you performed and explain how each operation addressed specific data quality issues in your datasets.

### Shared Cleaning Pipeline
We applied the same base cleaning steps to all three datasets. We think this can help keep our process consistent and makes it easier to reproduce.
1.	Replacing placeholder strings. All empty entries were replaced with missing values. Without this step, these strings would appear as real categories and inflate group counts in any analysis. We did this because we found fields like weather_condition and driver_action used UNKNOWN as a placeholder, which would otherwise appear as a real category.
2.	Lowercasing column names. All column names were converted to lowercase. We did this because we think this can help prevents case mismatch errors when referencing the same column across datasets.
3.	Parsing date columns. Date fields were converted from text strings to datetime format. We think it's important before any date based filtering or comparison.
4.	We kept only records from January 1, 2021. This gives us a consistent time window across all three datasets.

### Dataset-Specific Steps
For the People set, we added one extra step, like dropping columns with more than 80% missing values. We chose 80% as the threshold because columns beyond that point contain too little data to be useful for modeling. Ten columns were removed. We also uppercased values in sex and person_type to fix mixed case inconsistencies found during profiling.
For the Crashes set, the shared pipeline was applied. The >80% threshold step was not run on this dataset because profiling showed the columns were generally well populated. 

### Merging People and Crashes
We joined the People and Crashes datasets on crash_record_id using a left join. We chose a left join to keep every person record. Crash level attributes like weather, road conditions, time, and location were added to each row.
We verified that crash_record_id had zero missing values after the join, confirming no records were lost.
At this stage the data is still person level. Multiple people in the same crash each have their own row, all sharing the same crash attributes.
We then constructed two binary outcome variables on dfm:

- any_injury: 1 if anyone in the crash had non incapacitating, incapacitating, or fatal injury
- severe_injury: 1 if anyone had an incapacitating or fatal injury

### Aggregating to Crash Level
We then aggregated the merged data from person level to crash level by grouping on crash_record_id. We did this because our goal is to predict whether a crash results in injury not to model outcomes at the individual person level. And we think this kind of converting is more efficient when doing actual analysis.
Each group was collapsed into one row:

- any_injury: maximum across all people - person_id: count, renamed to num_people 
- All other attributes: first value in the group
The resulting crash level dataset was used as model input, with any_injury as the target variable. 

### Feature Engineering
We simplified four variables to reduce the number of categories going into the model. Fewer, broader categories reduce sparsity and make results easier to interpret. 
For example, weather_condition originally had many categories, several of which became missing after placeholder replacement.

- weather_group: clear and cloudy to CLEAR; rain, snow, fog, sleet, and freezing conditions to BAD
- lighting_group: daylight to DAY; darkness with or without street lights to NIGHT; dusk and dawn to LOW_LIGHT 
- speed_bin: four bands, LOW (0–25 mph), MEDIUM (25–35 mph), HIGH (35–45 mph), VERY_HIGH (45+ mph)
- is_night: True if the crash occurred between 8 PM and 5 AM 

### Speed Camera Dataset
We applied the same shared pipeline to the Speed Camera dataset. The placeholder string is replaced with NaN, column names are in lowercase, violation_date is parsed to datetime, and records are filtered beyond 2021. No pillars fell. We checked the missing rate of each column, and none exceeded the threshold of 80%.
For transparency and reproducibility, we recorded and cleaned this dataset, although it was not used in the model. The reason why we exclude it is structural, as we explianed in data quality section. There is no shared row level key between camera records and crash records.

---

## Findings: Description of any findings including numeric results and/or visualizations.

## What Was Done

### Feature Engineering & Grouping
- Created categorical groupings for:
  - Weather conditions (`weather_group`)
  - Lighting conditions (`lighting_group`)
  - Speed levels (`speed_bin`)
- Calculated average injury rates (`any_injury`) within each group to identify patterns

### Model Building
- Built a logistic regression model to predict probability of injury
- Features used:
  - Posted speed limit
  - Weather group
  - Lighting group
  - Night indicator (`is_night`)
- Applied one-hot encoding (`pd.get_dummies`) for categorical variables

### Model Outputs
- Generated:
  - Predicted probabilities (`predict_proba`)
  - Binary predictions (`predict`)
- Evaluated model using:
  - Accuracy
  - ROC-AUC


## Key Findings

### Weather Impact on Injuries
- Injury rates are similar across weather conditions (~11–11.5%)
- “Bad” and “Severe crosswind” conditions are only slightly higher than clear weather  

**Insight:**  
Weather alone is not a strong differentiator for injury likelihood


### Lighting Conditions
- Night: ~13.4% (highest)
- Low light: ~12.0%
- Day: ~9.8% (lowest)

**Insight:**  
Injury likelihood increases significantly as visibility decreases  
Lighting is a stronger predictor than weather


### Speed (Strongest Driver)
- Low speed: ~6.6%
- Medium: ~11.2%
- High: ~14.7% (peak)
- Very high: ~14.0%

**Insight:**  
Injury risk more than doubles from low to high speeds  
Speed is the most impactful variable in the dataset


## Model Performance

- Accuracy: ~89.6%
- ROC-AUC: ~0.587

**Interpretation:**
- High accuracy is misleading (likely due to class imbalance)
- Low AUC (~0.59) indicates weak predictive power
- Current features do not strongly separate injury vs. non-injury cases


## Overall Takeaways

- Speed and lighting conditions are the primary drivers of injury risk
- Weather has minimal impact relative to other factors
- The model captures some signal but lacks strong predictive ability


## Future work: [~500-1000 words] Brief discussion of any lessons learned and potential future work.

---

## Challenges: [~500 words] Discuss the main challenges you encountered while working on the project.

### 1. Mismatched Data Granularity

**Challenge:**  
The datasets were structured at different levels:
- `Traffic Crashes – People` dataset: person-level (multiple records per crash)
- `Traffic Crashes – Crashes` dataset: crash-level (one record per crash)

This created inconsistency when trying to merge and analyze the data because one crash could have multiple associated people records.

**Solution:**  
- Aggregated the people-level dataset up to the crash level using `groupby` on `crash_record_id`
- Created aggregated metrics such as:
  - Whether any injury occurred in the crash (`any_injury`)
  - Counts or indicators derived from person-level attributes
- This ensured a **one-to-one match** between datasets and allowed for consistent merging and analysis

### 2. Unstructured and Inconsistent Categorical Data

**Challenge:**  
Categorical variables (e.g., weather, lighting) were:
- Highly granular and inconsistent
- Difficult to interpret directly
- Contained missing or non-standardized values across datasets

**Solution:**  
- Grouped raw categories into more meaningful and interpretable buckets:
  - `weather_group`: based on severity (ex: clear, moderate, severe)
  - `lighting_group`: based on visibility levels (ex: daylight, low light, night)
- Standardized missing values across datasets to ensure consistency
- Cleaned and aligned categorical labels to avoid duplication (ex: variations in naming)

**Impact:**  
Improved interpretability of data and made variables more useful for both analysis and modeling


### 3. Large Dataset Size and Computational Constraints

**Challenge:**  
- Two datasets exceeded **1 million rows** which created performance and memory challenges
- Slower processing times during merging and cleaning

**Solution:**  
- Filtered data to the most recent 5 years (from **01-01-2021 onward**) to:
  - Maintain relevance
  - Reduce dataset size
- Used efficient operations in `pandas`:
  - Vectorized transformations
  - `groupby` for aggregation
- Reduced data to the crash level, significantly decreasing total row count

### 4. Weak Initial Model Performance

**Challenge:**  
- Initial logistic regression model used only **speed-related variables**
- Resulted in weak predictive power and limited insight
- Failed to capture the complexity of injury outcomes

**Solution:**  
- Expanded the feature set to include:
  - Weather conditions
  - Lighting conditions
  - Time of day (night indicator)
- Incorporated engineered categorical variables for better signal extraction
- Re-ran the model with a broader set of predictors

## Reproducing: Sequence of steps required for someone else to reproduce your results.

---

## References: Formatted citations for any papers, datasets, or software used in your project.


