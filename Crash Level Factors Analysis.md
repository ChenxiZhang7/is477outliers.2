# Crash Level Factors Analysis 

---

### Team Outliers
- Mustafa El Zayyat
- Chenxi Zhang

---

## Summary:
- Description: Our group originally planned on doing an airplane ticket analysis based on jet fuel price and demand. However, after facing integration issues due to a lack of common variables and a lack of descriptive variables that can be utilized for analysis, we decided to adopt a different project plan. After receiving feedback we chose to analyze the crashes and people datasets from the city of Chicago. Since we began our analysis of the new datasets, our analytical focus shifted from originally focusing on linking speed cameras, crashes, and injuries to establishing a strong crash level injury prediction framework first and building a structured dataset that identifies key drivers of injury severity (speed, lighting, weather). This can improve the project because we now have a baseline model of injury risk and speed camera analysis can be layered on top more rigorously for analysis. There was also some methodological improvement, instead of only descriptive analysis, we also added logistic regression modeling and feature engineering (grouped weather, lighting, speed bins) to improve analysis. The plan was updated to complete crash level integration from crash and people datasets first then integrate enforcement data from the speed camera dataset.

- Motivation: The motivation behind this analysis is the increasing reliance on data-driven decision making in urban planning and public safety. Cities heavily invest within their infrastructure and traffic enforcement, but these efforts are most effective when they are directed by evident insights into the risk factors that influence car crashes. By examining crash data, this project can potentially contribute to that understanding by examining the relative impact of variables such as speed, lighting conditions, and weather. Additionally, the project explores whether these variables can be used not just in a descriptive capacity, but also in a predictive capacity, through the development of a statistical model.

- Additionally, as discussed with Professor Carboni and course staff, due to scheduling and time coordination issues, we have decided to move forward as a two person team with only Mustafa and Chenxi.

- Research question: What factors are most strongly associated with traffic crash occurrences in Chicago? Another question we sought to answer was how well can these variables predict injury outcomes when used in a logistic regression model?

- In order to address the research questions we began by cleaning and standardizing the dataset and then merged them at the crash level. The main difference was that the People dataset was at the individual level and the crashes dataset was at the crash level, so we aggregated the data using the `crash_record_id` to create a combined dataset. We also did some feature engineering to improve the dataset’s interpretability, including grouping weather conditions by severity, lighting conditions by visibility, and speed into discrete categories. We also restricted the dataset only to the recent five years to improve its relevance to analysis and for storage capacity.

---

## Data profile:

### Dataset 1: Speed Camera Violations

#### Source: 
City of Chicago Data Portal 

#### Link: 
https://data.cityofchicago.org/Transportation/Speed-Camera-Violations/hhkd-xvj4/about_data                                                                             
#### File location: 
The raw data is accessed directly from the Chicago Data Portal API. No manual download is required, the notebook fetches it automatically at runtime. The cleaned dataset is included in the project GitHub repository as speed_camera_violations.csv.

#### Structure: 
This is the smallest of the three datasets, with only 9 columns. We filtered a total of 234,913 lines after 2021. We found that this dataset appears once a day for each camera. It can be understood as the daily log recorded by each camera. The contents listed are divided into three categories: camera information `camera_id`, `address`, `violations` of the day such as `violation_date`, `violations`, And the geographical locations of the camera: `latitude`, `longitude`, `x_coordinate`, `y_coordinate`, `location`. The entire dataset has a very regular structure and no complex nested relationships. `violations` is the only numerical measurement column, and the rest of the columns are either identifiers or geographic information.

#### Content:
This dataset tracks daily speed violations recorded by cameras in Children's Safety Zones across Chicago. Each camera detects vehicles that exceed the speed limit. Violations are reviewed by two separate contractors before being recorded. The full dataset covers July 2014 to the present, minus the most recent 14 days. We filtered to 2021 onward to match the time window of the other two datasets. 

#### Variable Examples: 
ADDRESS, CAMERA_ID, VIOLATION_DATE, VIOLATIONS, X_COORDINATE, Y_COORDINATE, LATITUDE, LONGITUDE, LOCATION

#### Characteristics:
The violations column is the core measure. Its distribution is right skewed. The minimum is 1 violation per camera day, the maximum is 1,888, the mean is 56.5, and the median is 34. Most camera days record moderate counts, but a small number of high traffic locations produce very high numbers on certain days. Geographic coordinates are missing for 3.23% of records.

We loaded and cleaned this dataset for completeness. However, it was not integrated into the crash model. There is no shared row level key linking camera records to crash records. This is explained in the Data Quality section.  

### Dataset 2: Traffic Crashes – People

#### Source: 
City of Chicago Data Portal

#### Link: 
https://data.cityofchicago.org/Transportation/Traffic-Crashes-People/u6pd-qa9d/about_data

#### File location: 
The raw data is accessed directly from the Chicago Data Portal API. No manual download is required, the notebook fetches it automatically at runtime. The cleaned dataset is included in the project GitHub repository as people_dataset.csv.

#### Structure:
This is the dataset with the largest volume of data that we have used. After filtering up to 2021, there were a total of 1,276,673 rows and 29 columns. We retained 19 columns after cleaning. There is a key point in understanding this dataset: an incident can generate multiple rows of data. Each person involved has their own line of record. For example, in an accident involving three people, there will be three lines here, all sharing the same `crash_record_id`. It is this `crash_record_id` that enables us to concatenate this dataset with the Crashes dataset. The remaining 19 The column contents cover the basic information of personnel such as `person_id`, `person_type`, `sex`, `age`, the protection status at that time `safety_equipment`, `airbag_deployed`, injury result `injury_classification`, and driving behavior `driver_a ction`, `driver_vision`.

`injury_classification` is the column we care about the most. It records the severity of injury for each person, ranging from no obvious injury to death in five grades.

#### Content:
This dataset records everyone involved in a Chicago traffic crash. It captures who was in the crash, their demographics, how they were protected, and how severely they were injured. Each person is linked to a crash event through` crash_record_id`, which we used to join this dataset with the Crashes dataset. This linkage is what allows us to combine person-level injury outcomes with crash level environmental conditions.

#### Variable Examples: 
PERSON_ID, PERSON_TYPE, CRASH_RECORD_ID, VEHICLE_ID, CRASH_DATE, SEAT_NO, CITY, STATE, ZIPCODE, SEX, AGE, DRIVERS_LICENSE_STATE, DRIVERS_LICENSE_CLASS, SAFETY_EQUIPMENT, AIRBAG_DEPLOYED, EJECTION, INJURY_CLASSIFICATION, HOSPITAL, EMS_AGENCY, EMS_RUN_NO, DRIVER_ACTION, DRIVER_VISION, PHYSICAL_CONDITION, PEDPEDAL_ACTION, PEDPEDAL_VISIBILITY, PEDPEDAL_LOCATION, BAC_RESULT, BAC_RESULT_VALUE, CELL_PHONE_USE

#### Characteristics:
The outcome variable injury_classification is missing for only 0.02% of records. So we think it is reliable as a modeling target. Fatal injuries are rare, 354 for drivers and 210 for pedestrians. Several behavioral columns have high missingness after placeholder replacement, driver_action is 69.5% missing, driver_vision is 63.6% missing. These fields are likely only filled when a formal investigation occurs, which may introduce selection bias toward more severe crashes.                                       

### Dataset 3 : Traffic Crashes – Crashes

#### Source: 
City of Chicago Data Portal

#### Link: 
https://data.cityofchicago.org/Transportation/Traffic-Crashes-Crashes/85ca-t3if/about_data

#### File location: 
The raw data is accessed directly from the Chicago Data Portal API. No manual download is required, the notebook fetches it automatically at runtime. The cleaned dataset is included in the project GitHub repository as crashes_dataset.csv. The merged dataset combining People and Crashes is also included as merged_dataset.csv.

#### Structure:
This dataset has one line corresponding to one incident. After filtering to 2021, there are a total of 585,088 lines. This dataset is characterized by a large number of columns, totaling 48 columns, which record a large amount of background information at the time of each accident. In terms of time, there are `crash_hour`, `crash_day_of_week` and `crash_month`. For road conditions and weather, there are condition, `lightingcondition`, `roadway_surface_cond`, `posted_speed_limit`, and `traffic_control_device`. In terms of geographical location, there are latitude and longitude. The casualty results are split into five severity columns, ranging from no obvious injury `injuries_no_indication` to death `injuries_fatal`, with each column recording the number of people corresponding to the level of the accident. Each row is connected to the corresponding personnel record in the People dataset through `crash_record_id`, which is also the basis for us to merge the two datasets.

#### Content: 
This dataset records the circumstances of each crash location, road and weather conditions, time, and injury outcomes. Data comes from CPD's electronic crash reporting system and is updated as reports are finalized. It covers both police reported and self reported crashes within CPD jurisdiction. Crashes on interstate highways and outside CPD jurisdiction are excluded. Citywide coverage begins in September 2017. We filtered to 2021 to match the other datasets.

#### Variable Examples: 
CRASH_RECORD_ID, CRASH_DATE_EST_I, CRASH_DATE, POSTED_SPEED_LIMIT, TRAFFIC_CONTROL_DEVICE, DEVICE_CONDITION, WEATHER_CONDITION, LIGHTING_CONDITION, FIRST_CRASH_TYPE, TRAFFICWAY_TYPE, LANE_CNT, ALIGNMENT, ROADWAY_SURFACE_COND, ROAD_DEFECT, REPORT_TYPE, CRASH_TYPE

#### Characteristics:
Posted speed limits range from 0 to 70 mph, with both the mean and median around 30 mph, which is consistent with urban street driving in Chicago. The average crash involves just over 2 vehicles. Crash hour peaks in the afternoon, with a mean of 13:15. Injury columns are missing for 1,324 records. Geographic coordinates are missing for 5,546 rows. One notable issue is that 43,443 records have `weather_condition` recorded as UNKNOWN, which we converted to missing during cleaning. Some environmental fields such as weather and road conditions are based on the reporting officer's best judgment at the time, which may introduce inconsistencies.

#### Ethical and Legal Constraints
All three datasets are published by the City of Chicago on Chicago Data Portal under the City of Chicago Data Portal Terms of Use. It permits unrestricted access, use and redistribution for non commercial and research purposes. No special permissions or licenses are required.  
                                                                                                                                                                      
The datasets do not contain personally identifiable information. The People dataset records demographic attributes and injury classification at the incident level but does not include names, addresses, or any direct  identifier. Crash records are linked by a system generated crash_record_id that cannot be traced back to individuals. 


---

## Data quality:


### Dataset 1: Traffic Crashes – People

#### Before Cleaning 
This dataset has 1,276,673 rows and 29 columns. After standardizing the sentinel values like `UNKNOWN` and NONE, we found that there were serious deficiencies among the columns. The 10 columns that exceed 80% of the missing threshold have been deleted. 

Consistency issues include mixed case values in sex and person_type, and crash_date stored as an ISO timestamp string requiring datetime parsing. Validity concerns include the pervasive use of `UNKNOWN` and `NONE` as non null placeholder strings across categorical fields, which without standardization would inflate category counts in any groupby operation.

#### After Cleaning
After dropping the 10 high missingness columns, we have 19 columns. Column names were lowercased, crash_date converted to datetime, and the date filter applied as a safety check, no additional rows removed. The key outcome variable `injury_classification` has barely any missingness. We think this makes the target label is reliable.

#### Potential Drawbacks of Cleaning
We find high missingness in driver_action, driver_vision, and physical_condition means these columns cannot be used as model features without imputation or subsetting, and the subset of records where they are populated likely corresponds to more severe or formally investigated crashes. So there might be selection bias. Dropping the pedestrian-specific columns (`pedpedal_action`, `pedpedal_visibility`, `pedpedal_location`) means `pedestrian-involved` crashes lose their specific behavioral context.

### Dataset 2: Traffic Crashes Dataset

#### Before Cleaning 
This dataset records one row per crash event, including road conditions, weather, time, location, and outcomes. After filtering 2021, it has 585,088 rows and 48 columns.    

The crash identifier and the key results column are in good condition. `Crash_record_id` has no missing values. And the injection related column has only 1,324 missing entries among 585,088 records. The geographic coordinates were lost by 5,546 lines, which is only a small part, but these crashes cannot be placed on the map.

A notable data quality issue emerged in the weather_condition column. 43,443 records were marked as unknown approximately 7.4% of the dataset. Technically speaking, this is not a true deficiency, but unknown does not provide actual information about the road conditions at the time of the accident. The speed limit ranges from 0 to 70 miles per hour, with both the average and median around 30 miles per hour, which is consistent with the urban driving dataset.

#### After Cleaning
The cleaning pipeline replaces sentinel strings like `UNKNOWN` with proper `NA` values, drops columns that are >80% empty, lowercases all column names, and filters to 2021+. Most columns in the Crashes dataset are well populated, so relatively few are dropped. After cleaning, the date, location, road condition, weather, and crash outcome fields are all retained.

#### Potential Drawbacks of Cleaning
The 43,443 `UNKNOWN` weather records become missing values after cleaning. If bad weather is underreported in cases where the reporting officer did not know or did not fill in the weather field, and if bad weather also correlates  with crash severity, then models trained on this data may underestimate the effect of weather on injury outcomes. The 5,546 rows missing coordinates are also non random crashes in certain neighborhoods or with less thorough incident reports may be more likely to lack location data, which could introduce a geographic bias if location based features are used.

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

## Data cleaning:

### Shared Cleaning Pipeline
We applied the same base cleaning steps to all three datasets. We think this can help keep our process consistent and makes it easier to reproduce.
1.	Replacing placeholder strings. All empty entries were replaced with missing values. Without this step, these strings would appear as real categories and inflate group counts in any analysis. We did this because we found fields like weather_condition and driver_action used `UNKNOWN` as a placeholder, which would otherwise appear as a real category.
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

- any_injury: maximum across all people `person_id`: count, renamed to num_people 
- All other attributes: first value in the group
The resulting crash level dataset was used as model input, with `any_injury` as the target variable. 

### Feature Engineering
We simplified four variables to reduce the number of categories going into the model. Fewer, broader categories reduce sparsity and make results easier to interpret. 
For example, `weather_condition` originally had many categories, several of which became missing after placeholder replacement.

- weather_group: clear and cloudy to CLEAR; rain, snow, fog, sleet, and freezing conditions to BAD
- lighting_group: daylight to DAY; darkness with or without street lights to NIGHT; dusk and dawn to LOW_LIGHT 
- speed_bin: four bands, LOW (0–25 mph), MEDIUM (25–35 mph), HIGH (35–45 mph), VERY_HIGH (45+ mph)
- is_night: True if the crash occurred between 8 PM and 5 AM 

### Speed Camera Dataset
We applied the same shared pipeline to the Speed Camera dataset. The placeholder string is replaced with NaN, column names are in lowercase, violation_date is parsed to datetime, and records are filtered beyond 2021. We checked the missing rate of each column, and none exceeded the threshold of 80%.
For transparency and reproducibility, we recorded and cleaned this dataset, although it was not used in the model. The reason why we exclude it is structural, as we explianed in data quality section. There is no shared row level key between camera records and crash records.

### This link directs to Box where our cleaned datasets are stored: 
https://github.com/ChenxiZhang7/is477outliers.2/blob/main/Milestone%204%20Processed%20Datasets

---

## Findings:

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


## Future work: Brief discussion of any lessons learned and potential future work.

- This project was extremely insightful in regards to the limitations of working with real traffic data as well as the analytical process behind it. Although our analysis was somewhat successful in identifying important factors that increase the chance of harm, such as speed and lighting, it also showed a number of areas where the model's predictive performance and depth of insights could be greatly improved. We gathered several insights that would be informative for future work.
- The first insight is the limitation of depending on a comparatively small set of high level variables. The intricacy of real world traffic and crash level dynamics is not entirely captured by speed, lighting, and weather, despite the fact that these variables are obvious and pertinent. The model's relatively low ROC-AUC score indicates that these factors by themselves are not sufficient for comprehensive analysis. More detailed and behavior oriented features should be the main emphasis of future research. For instance, factors pertaining to driving conduct, such distraction or failure to yield, may have superior predictive power. Additionally, road features like lane layout, traffic intensity, intersection type, and signage can provide further context that the model does not yet provide.
- Another lesson is the importance of data granularity and structure. The need to aggregate person level data to the crash level simplified the modeling process but may have resulted in a loss of important detail. For instance, differences in injury severity across individuals within the same crash were reduced to a single binary outcome. Future work could dive further into modeling at the individual level instead, which would allow for a more comprehensive understanding of injury risk. This would enable for analysis of the way that factors such as age or safety equipment usage can influence crash outcomes. Alternatively, multi level modeling approaches could be used to preserve both crash level and person level versions.
- Additionally, further data integration offers an opportunity for more in depth analysis and more actionable insights. Although only a few datasets were used in this specific project, using more data sources could greatly improve the analysis. One way this could be potentially done is through the inclusion of traffic volume data which would normalize crash rates and present a more accurate picture of risk exposure. Similarly, incorporating geographic data such as road networks or relevant neighborhood features could allow for spatial analysis and the identification of high risk locations. This could be helpful for City of Chicago policymakers and planners in terms of finding specific interventions and implementing traffic policy. 
- In summary, this project highlights the intricacy of crash level analysis while providing a solid basis for understanding injury risk in traffic accidents. Expanding the feature set and enhancing modeling methods, while also incorporating new data sources should be the main goals of future initiatives. By expanding on these specific areas, experts in the future will be able to do more than just find patterns, they will be able to create more precise predictive models and practical insights for enhancing public safety.


---

## Challenges: Discuss the main challenges you encountered while working on the project.

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
  Step 1 — Clone the repository               
                                          
  Download the project files to your local machine.
                                                                                                                                                                                                                                               
  git clone https://github.com/ChenxiZhang7/is477outliers.2.git
  cd is477outliers.2                                                                                                                                                                                                                           
                                          
  ---                                                                                                                                                                                                                                          
  Step 2 — Set up environment                                                                                                                                                                                                                  
                                              
  Install required libraries:                                                                                                                                                                                                                  
  pip install pandas scikit-learn requests
                                                                                                                                                                                                                                               
  Create output directory for cleaned files:  
  data_dir = Path("data")                                                                                                                                                                                                                      
  data_dir.mkdir(exist_ok=True)
                                                                                                                                                                                                                                               
  ---                                     
  Step 3 — Load datasets                                                                                                                                                                                                                       
                                                                                                                                                                                                                                               
  No manual download is required. The notebook fetches all three datasets directly from the Chicago Data Portal API and filters to 2021 onward at download time. This may take a few minutes due to dataset size.
                                                                                                                                                                                                                                               
  Load the People dataset:                
  resp = requests.get(
      "https://data.cityofchicago.org/resource/u6pd-qa9d.csv",                                                                                                                                                                                 
      params={"$where": "crash_date >= '2021-01-01T00:00:00'", "$limit": 1500000},
      timeout=300                                                                                                                                                                                                                              
  )                                           
  df = pd.read_csv(io.StringIO(resp.text), low_memory=False)
                                                                                                                                                                                                                                               
  Load the Crashes dataset:                                                                                                                                                                                                                    
  resp = requests.get(                                                                                                                                                                                                                         
      "https://data.cityofchicago.org/resource/85ca-t3if.csv",                                                                                                                                                                                 
      params={"$where": "crash_date >= '2021-01-01T00:00:00'", "$limit": 700000},                                                                                                                                                              
      timeout=300                                                                
  )                                                                                                                                                                                                                                            
  df2 = pd.read_csv(io.StringIO(resp.text))
                                                                                                                                                                                                                                               
  Load the Speed Camera dataset:                                                                                                                                                                                                               
  resp = requests.get(                                                                                                                                                                                                                         
      "https://data.cityofchicago.org/resource/hhkd-xvj4.csv",                                                                                                                                                                                 
      params={"$where": "violation_date >= '2021-01-01T00:00:00'", "$limit": 300000},
      timeout=300                                                                                                                                                                                                                              
  )                                       
  df_cam = pd.read_csv(io.StringIO(resp.text), low_memory=False)                                                                                                                                                                               
                                                                                                                                                                                                                                               
  ---
  Step 4 — Standardize missing values                                                                                                                                                                                                          
                                              
  Many fields use strings like "UNKNOWN" or "NONE" as placeholders. Replace "UNKNOWN", "NONE", and empty strings "" with NaN in all three datasets:                                                                                            
                                              
  df     = df.replace(["UNKNOWN", "NONE", ""], pd.NA)                                                                                                                                                                                          
  df2    = df2.replace(["UNKNOWN", "NONE", ""], pd.NA)
  df_cam = df_cam.replace(["UNKNOWN", "NONE", ""], pd.NA)                                                                                                                                                                                      
                                          
  ---                                                                                                                                                                                                                                          
  Step 5 — Inspect missingness (optional)                                                                                                                                                                                                      
                                                                                                                                                                                                                                               
  df.isna().mean().sort_values(ascending=False)                                                                                                                                                                                                
  df2.isna().mean().sort_values(ascending=False)
  df_cam.isna().mean().sort_values(ascending=False)                                                                                                                                                                                            
                                          
  ---                                                                                                                                                                                                                                          
  Step 6 — Drop high-missingness columns                                                                                                                                                                                                       
                                        
  Drop columns with more than 80% missing values from the People and Camera datasets. This removes 10 columns from People, leaving 19. No columns are dropped from Crashes.                                                                    
                                              
  df     = df.drop(columns=[c for c in df.columns if df[c].isna().mean() > 0.8])
  df_cam = df_cam.drop(columns=[c for c in df_cam.columns if df_cam[c].isna().mean() > 0.8])
                                                                                                                                                                                                                                               
  ---                                     
  Step 7 — Clean column formats                                                                                                                                                                                                                
                                                                                                                                                                                                                                               
  Convert column names to lowercase:                                                                                                                                                                                                           
  df.columns     = df.columns.str.lower()                                                                                                                                                                                                      
  df2.columns    = df2.columns.str.lower()
  df_cam.columns = df_cam.columns.str.lower() 
                                             
  Convert key fields:
  - crash_record_id to string                                                                                                                                                                                                                  
  - crash_date and violation_date to datetime using pd.to_datetime(..., errors="coerce")
  - sex and person_type values to uppercase                                                                                                                                                                                                    
                                                                                                                                                                                                                                               
  ---                                                                                                                                                                                                                                          
  Step 8 — Filter data                                                                                                                                                                                                                         
                                                                                                                                                                                                                                               
  Remove rows with missing date fields. Keep only records from 2021 onward:                                                                                                                                                                    
                                              
  df     = df[df["crash_date"] >= "2021-01-01"]                                                                                                                                                                                                
  df2    = df2[df2["crash_date"] >= "2021-01-01"]
  df_cam = df_cam[df_cam["violation_date"] >= "2021-01-01"]                                                                                                                                                                                    
                                          
  ---                                                                                                                                                                                                                                          
  Step 9 — Merge datasets                                                                                                                                                                                                                      
                         
  Merge on crash_record_id using a left join:                                                                                                                                                                                                  
  dfm = df.merge(df2, on="crash_record_id", how="left")
                                          
  Remove duplicates:                          
  dfm = dfm.drop_duplicates()                                                                                                                                                                                                                  
                             
  Check missing join keys:                                                                                                                                                                                                                     
  dfm["crash_record_id"].isna().sum()  # Expected: 0
                                                                                                                                                                                                                                               
  ---                                         
  Step 10 — Create injury variables                                                                                                                                                                                                            
                                   
  Create any_injury indicator: 1 if any injury type is greater than 0                                                                                                                                                                          
                                              
  Create severe_injury indicator: 1 if incapacitating or fatal injury is greater than 0                                                                                                                                                        
                                          
  ---                                                                                                                                                                                                                                          
  Step 11 — Aggregate to crash-level dataset                                                                                                                                                                                                   
                                                                                                                                                                                                                                               
  Group by crash_record_id and aggregate:                                                                                                                                                                                                      
  - max of any_injury
  - count of person_id (renamed to num_people)                                                                                                                                                                                                 
  - first value for variables such as speed limit, weather, lighting, road surface, time, and location
                                                                                                                                                                                                                                               
  ---                                                                                                                                                                                                                                          
  Step 12 — Feature engineering           
                                                                                                                                                                                                                                               
  Create weather_group: CLEAR and CLOUDY/OVERCAST → CLEAR; rain, snow, fog, etc. → BAD                                                                                                                                                         
                                                                                                                                                                                                                                               
  Create lighting_group: daylight → DAY; darkness → NIGHT; dawn/dusk → LOW_LIGHT                                                                                                                                                               
                                          
  Create speed_bin: four bands from LOW (0–25 mph) to VERY_HIGH (45+ mph)                                                                                                                                                                      
                                                                                                                                                                                                                                               
  Create is_night indicator: True for hours between 8 PM and 5 AM                                                                                                                                                                              
                                                                                                                                                                                                                                               
  ---             
  Step 13 — Exploratory analysis                                                                                                                                                                                                               
                  
  Compute average injury rates using groupby:                                                                                                                                                                                                  
                  
  df2.groupby("weather_group")["any_injury"].mean()
  df2.groupby("lighting_group")["any_injury"].mean()
  df2.groupby("speed_bin")["any_injury"].mean()

  ---                                                                                                                                                                                                                                          
  Step 14 — Prepare data for modeling
                                                                                                                                                                                                                                               
  Select features: posted_speed_limit, weather_group, lighting_group, is_night

  Convert categorical variables using one-hot encoding:                                                                                                                                                                                        
  X = pd.get_dummies(df2[[
      "posted_speed_limit", "weather_group", "lighting_group", "is_night"                                                                                                                                                                      
  ]], drop_first=True)                        
                                          
  Set target variable:
  y = df2["any_injury"]                                                                                                                                                                                                                        
   
  ---                                                                                                                                                                                                                                          
  Step 15 — Train model                   

  model = LogisticRegression(max_iter=1000)                                                                                                                                                                                                    
  model.fit(X, y)
                                                                                                                                                                                                                                               
  ---             
  Step 16 — Generate predictions
                                              
  injury_prob = model.predict_proba(X)[:, 1]
  predict     = model.predict(X)
                                                                                                                                                                                                                                               
  ---
  Step 17 — Evaluate model                                                                                                                                                                                                                     
                  
  accuracy = accuracy_score(y, predict)
  auc      = roc_auc_score(y, injury_prob)    
                                          
  Expected results:
  - Accuracy ≈ 0.896                                                                                                                                                                                                                           
  - AUC-ROC ≈ 0.587 
                                                                                                                                                                                                                                               
  ---                                         
  Step 18 — Save outputs                  

  Save all cleaned and processed datasets as CSV files to the data/ folder:                                                                                                                                                                    
   
  dfm.to_csv(data_dir / "merged_dataset.csv", index=False)                                                                                                                                                                                     
  df2.to_csv(data_dir / "crashes_dataset.csv", index=False)
  df.to_csv(data_dir / "people_dataset.csv", index=False)
  df_cam.to_csv(data_dir / "speed_camera_violations.csv", index=False) 

### This link directs to Box where our cleaned datasets are stored: 
https://github.com/ChenxiZhang7/is477outliers.2/blob/main/Milestone%204%20Processed%20Datasets

### This link directs to our code file: 
https://github.com/ChenxiZhang7/is477outliers.2/blob/main/milestone%204%20cleaning%20and%20merging%20dataset%20code.ipynb

---

## References:

- City of Chicago. (2026). Traffic Crashes – Crashes. Chicago Data Portal. https://data.cityofchicago.org/Transportation/Traffic-Crashes-Crashes/85ca-t3if/about_data
- City of Chicago. (2026). Traffic Crashes – People. Chicago Data Portal. https://data.cityofchicago.org/Transportation/Traffic-Crashes-People/u6pd-qa9d/about_data
- City of Chicago. (2026). Speed Camera Violations. Chicago Data Portal. https://data.cityofchicago.org/Transportation/Speed-Camera-Violations/hhkd-xvj4/about_data

