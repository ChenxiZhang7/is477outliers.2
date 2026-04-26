# Interim Status Report  
### Team Outliers: Chenxi Zhang, Mustafa El Zayyat

---


## Update on each of the tasks described on the project plan:

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


---


## Updated timeline indicating the status of each task and when they will be completed:

### Phase 1 — Project Planning (Completed)

All planning tasks, repository setup, dataset review, and `ProjectPlan.md` were completed on schedule.

- Finalize research questions — Completed
- Identify and evaluate datasets — Completed
- Review licenses and terms of use — Completed
- Explore dataset structures — Completed
- Set up GitHub repository and project structure — Completed
- Prepare and submit `ProjectPlan.md` — Completed

---

### Phase 2 — Data Cleaning and Crash-Level Integration (Apr 13 – Apr 25)

**Download and inspect datasets — Completed**  
Collected selected Chicago transportation datasets and reviewed data structure.

**Clean crashes dataset — Completed**  
Standardized date/time variables, cleaned environmental condition fields, removed low-quality values.

**Clean people dataset — Completed**  
Standardized injury classifications, removed invalid IDs, handled missing values.

**Aggregate people-level data to crash level — Completed**  
Grouped people records using `CRASH_RECORD_ID`.

**Merge crashes and people datasets — Completed**  
Created `merged_dataset.csv` for crash-level analysis.

**Initial exploratory analysis — In Progress**  
Reviewed injury rates by speed, lighting, and weather.

**Data integration review between current datasets — In Progress**  
Comparing crash-level and person-level units of observation and evaluating whether aggregation is the most appropriate method.

**Speed camera integration feasibility review — In Progress**  
Reviewing structural differences between the camera dataset and crash datasets, including missing shared identifiers and many-to-many matching risks.

**Prepare and submit `Interim Status Report.md` — In Progress**

---

### Phase 3 — Analysis and Final Development (Apr 23 – May 3)

**Finalize feature engineering — Apr 25**  
Complete variables such as:

- `speed_bin`
- `severe_injury`
- `weather_group`
- `lighting_group`
- `is_night`

**Develop predictive models — Apr 26**  
Complete logistic regression and compare model performance.

**Create visualizations — Apr 28**  
Produce charts showing injury risk by major variables.

**Optional speed camera integration — Apr 29**  
If feasible, aggregate violations by date and compare with crash outcomes.

**Evaluate location-based camera integration methods — Apr 29**  
Explore whether camera coordinates can be used for proximity-based comparisons with crash locations.

**Write data dictionary / metadata — Apr 30**  
Document datasets, variables, and transformations.

**Finalize workflow / reproducibility files — May 1**  
Update scripts, requirements, and repository organization.

**Write final README / report — May 2**

**Create final release and submit — May 3**


---


## Description of any changes to your project plan itself, in particular about your progress so far. Also include changes you made to your plan based on feedback you may have received for Milestone 2.

---

## Summarize any challenges or problems you have encountered so far. For each issue, explain how you resolved it or describe your plan to address it in the near future.


---


## Team member contribution:

## Chenxi

### Project Planning and Direction Revision

- Supported the transition from the original airline pricing proposal to the Chicago traffic safety datasets after identifying integration limitations in the initial project plan.
- Helped refine the updated research focus toward crash-level injury severity analysis using the crashes and people datasets.

### Milestone Documentation and Reporting

- Prepared updates on planned tasks for the interim status report.
- Created the revised project timeline based on completed work, current progress, and remaining deliverables.
- Wrote and edited sections of `StatusReport.md` to ensure milestone requirements were clearly addressed.

### Methodology and Project Structure

- Reviewed Milestone 2 feedback and helped update the analytical strategy.
- Supported prioritizing crash-level integration before optional speed camera integration.
- Assisted with identifying relevant variables for future analysis and modeling.

### Data Integration Review

- Reviewed the relationship between the crashes dataset and people dataset beyond the basic merge process.
- Compared crash-level and person-level units of observation and evaluated one-to-many integration structure.
- Evaluated whether aggregating people-level records to the crash level is more appropriate than direct person-level merging for the current research question.
- Reviewed the possible integration of the Speed Camera Violations dataset and identified challenges such as lack of shared identifiers, different units of observation, and many-to-many date matching risks.
- Explored future integration approaches including daily aggregation and possible location-based comparisons using coordinates.

### Repository Coordination

- Helped organize project deliverables and report structure in the shared GitHub repository.
- Added commits related to milestone documentation and project updates.

---

## Mustafa



