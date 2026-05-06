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
1.	Replacing placeholder strings. All empty entries were replaced with missing values. Without this step, these strings would appear as real categories and inflate group counts in any analysis.
2.	Lowercasing column names. All column names were converted to lowercase. We did this because we think this can help prevents case mismatch errors when referencing the same column across datasets.
3.	Parsing date columns. Date fields were converted from text strings to datetime format. We think it's important before any date based filtering or comparison.
4.	We kept only records from January 1, 2021. This gives us a consistent time window across all three datasets.

### Dataset-Specific Steps

### Merging People and Crashes

### Aggregating to Crash Level

### Feature Engineering

### Speed Camera Dataset

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
