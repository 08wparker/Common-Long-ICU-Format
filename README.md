# ICU rationing simulation

This repository will contain two scripts, both of which accepts a clean, long form patient dataset in a standardized format (detailed below)

1. quality_control_check.rmd - performs various quality control checks on the data
2. simulation_inputs.rmd - generates a set of inputs for a discrete event microsimulation model of ICU rationing.


## Data format 

| encounter | time_icu | sofa_total | age_years | race               |  elix_vw    | elix_ahrq  |charlson    | vent | status | covid |  zip       |
|-----------|----------|------------|-----------|--------------------|-------------|------------|------------|------|--------|-------|------------|
| 1         | 0        | 6          | 75        | Non-Hispanic White |     22      |  19        | 3          | 0    | icu    |  1    | XXXXX-YYYY |
| 1         | 1        | 6          | 75        | Non-Hispanic White |     22      |  19        | 3          | 0    | icu    |  1    | XXXXX-YYYY |
| 1         | 2        | 7          | 75        | Non-Hispanic White |     22      |  19        | 3          | 0    | icu    |  1    | XXXXX-YYYY |

Only key columns for transition matrices included in sample above, feel free to include more variables (SOFA sub-components).

**encounter** is an ID variable for each ICU stay (so a given patient can have multiple values).

## Race/ethnicity cateogries

* Non-Hispanic White
* Non-Hispanic Black
* Hispanic
* Other              

## Comorbidity calculation

Please report the ARHQ Elixhauser score (Moore et al., 2017), weighted VW Elixhauser (van Walraven et al. 2009), and Charlson index (Charlson 1987) calculated from ICD codes with the ![comorbidity](https://cran.r-project.org/web/packages/comorbidity/index.html) package in R

The simulation_inputs.Rmd file will assign comoribidity category cutoffs for the simulation matrices in a standardized way across datasets

## Status variable

Factor with three levels **(icu, recovered, died)**

* icu - currently admitted to ICU
* recovered - last observation in ICU with successful discharge to floor
* died - death in ICU or discharge to hospice (including floor transfers that went to hospice) 

Notes: 
1. Code transfers to nursing facilities or home as **recovered**
2. Code deaths that occur after transfer to the wards but prior to discharge as **recovered**
3. Code discharge to hospice from the ICU are coded as **died**, regardless of exactly when/where the patient dies 

## SOFA Coding Details

To standardize SOFA coding between sites, please follow the best practices below. 

### General

* If there are missing values, code 0 for that item until the lab/vital sign appears
* Carryforward values from previous observations. For example, if the Creatine was 1.5 at 9:00 AM earning a Renal Score of 1, the patient's Renal Score remains 1 until a new creatinine value is recorded.
    * SOFA respiratory score from P/F and SOFA renal score from dialysis have carryforward time-limits, see below for details

### SOFA CARDS
Only number of pressors matters, not dose.

* 2 or more pressors -> 4
* 1 pressor -> 3
* Dobutamine alone -> 2
* Map < 70 -> 1
* MAP >70, no pressors -> 0


### SOFA respiratory



* P/F <=100 -> 4 (must be on respiratory support)
* P/F 100-200 -> 3 (must be on respiratory support)
* P/F 200-300 ->  2
* P/F 300-400 -> 1
* P/F >400 -> 0

If PaO2/FiO2 is not available *or is more than 4 hours old*, use the SaO2/FiO2:
* SF<=150 -> 4 (must be on respiratory support)
* SF 150-235 -> 3 (must be on respiratory support)
* SF 235-315 ->  2
* SF 315-400 -> 1
* SF >400 -> 0

In other words, use the respiratory SOFA calculated from a blood gas for 4 hours after collection, then default back to SaO2/FiO2 ratio (unless a new blood gas has been drawn)


#### Notes:
* Treat all high-level ICU respiratory support equivalently, i.e. make no distinction between mechanical ventilation, NIPPV (CPAP/BiPAP), high-flow
* patients on low-flow nasal cannula can get at most a resp SOFA of 2
* To estimate the FiO2 for a patient on low-flow nasal cannula, use the following formula where LPM = liters per minute of low-flow oxygen
      * Fi02 = 0.24 + 0.04*(LPM)

* Max SOFA score on low-flow nasal cannula is 2.

* for patients on room air, set FiO2 = 0.21. Patients on room air should almost always have a respiratory SOFA of 0.


### SOFA renal 
Ignore urine output, use creatine criteria only 
* Cr < 1.2 -> 0
* Cr 1.2-1.9 -> 1
* Cr 2.0 - 3.4 -> 2
* Cr 3.5 - 4.9 -> 3
* Cr > 5.0 or on dialysis -> 4

After a dialysis session, the patient's SOFA renal score of 4 carries forward for 72 hours.

### SOFA liver

total bilirubin in mg/dl

* < 1.2 -> 0
* 1.2-1.9 -> 1
* 2.0-5.9 -> 2
* 6 - 11.9 -> 3
* > 12 -> 4

### SOFA Coagulation

Platelet count in 10^3 per uL

* > 150 -> 0
* 100-150 -> 1
* 50-100 -> 2
* 20-50 -> 3
* < 20 -> 4

### SOFA Central Nervous System
By recorded Glascow Coma Scale (GCS). If GCS is missing, a score of zero is assigned
* GCS = 15 ->0
* GCS 13-14 -> 1,
* GCS 10-12 -> 2,
* GCS 6-9 -> 3,
* GCS 0-5 -> 4


## Other notes
* Do not need to filter out patients who are still in the hospital at the end of follow-up. Can use their censored data in the transition matrices
* 9-digit zip code is preferred for more granular mapping to measures like the the Area Deprivation Index (https://rdrr.io/cran/sociome/man/get_adi.html)
