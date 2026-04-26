# Interim Status Report  
### Team Outliers: Chenxi Zhang, Mustafa El Zayyat

--


## Update on each of the tasks described on your project plan including references and links to specific artifacts in your repository (such as datasets, scripts, workflows, workflow diagrams, etc).

### 1. Data Collection and Dataset Selection

We finalized the datasets used in the project after revising our original proposal. The current project uses public transportation safety datasets from the City of Chicago:

- `crashes_dataset.csv`
- `people_dataset.csv`
- `merged_dataset.csv`

These datasets provide crash-level and people-level information that support injury severity analysis.

Repository artifacts:

- `/data/crashes_dataset.csv`
- `/data/people_dataset.csv`
- `/data/merged_dataset.csv`

---

### 2. Data Cleaning and Standardization

We cleaned and standardized the datasets to improve consistency and usability.

Completed tasks include:

- Converted date/time columns into datetime format
- Standardized categorical values
- Replaced `"UNKNOWN"` and `"NOT APPLICABLE"` with null values
- Removed unnecessary or low-quality variables
- Checked missing values across major columns

Repository artifact:

- `/scripts/cleaning_and_merging_dataset_code.ipynb`

---

### 3. Dataset Integration

The crashes dataset is stored at the crash level, while the people dataset is stored at the person level. To combine them correctly, we aggregated people-level records to the crash level using `CRASH_RECORD_ID`.

We then merged the two cleaned datasets into a structured analytical dataset.

Repository artifact:

- `/data/merged_dataset.csv`

In addition to the current merge process, we are reviewing the integration relationship between the two active datasets: Traffic Crashes – Crashes and Traffic Crashes – People. These two datasets are stored at different levels of observation. The crashes dataset contains one row per crash event, while the people dataset contains one row per person involved in a crash.

We are comparing whether direct merging or aggregated merging is more appropriate for the current research question. A direct merge is possible using `CRASH_RECORD_ID`, but it duplicates crash-level information for each person involved in the same crash. Aggregating people-level records to the crash level helps preserve the crash as the primary unit of analysis.

We are also reviewing the possible integration of the Speed Camera Violations dataset. This dataset is recorded at the camera-day level and does not share a common identifier with the crash datasets. Because of this structural difference, direct row-level integration may not be reliable.

If merged only by date, the camera dataset and crash dataset may create many-to-many matching problems because one date can contain many camera records and many crash records. We are evaluating future integration through daily aggregated comparisons or location-based methods using coordinates.

---

### 4. Feature Engineering

We created additional variables to support analysis:

- `any_injury`
- `severe_injury`
- `speed_bin`
- `weather_group`
- `lighting_group`
- `is_night`

These variables improve interpretability and model performance.

---

### 5. Exploratory Analysis

We began comparing injury outcomes across groups such as:

- Speed limit categories
- Weather conditions
- Lighting conditions
- Time of day

Initial findings suggest environmental conditions may be associated with injury severity.

---

### 6. Model Development

We developed an initial logistic regression model to estimate injury probability.

The first version used only speed-related variables. We later improved the model by adding:

- Weather variables
- Lighting variables
- Time-based indicators

This created a stronger baseline model.

We are also reviewing how future models may incorporate aggregated speed camera enforcement variables, such as daily violations or camera-zone activity levels, after resolving integration challenges.
