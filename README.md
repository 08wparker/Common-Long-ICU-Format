# ICU rationing simulation

This script accepts a clean, long form patient dataset and generates a set of inputs for a discrete event microsimulation model of ICU rationing.




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
Ignore respiratory support, i.e. make no distinction between mechanical ventilation, NIPPV (CPAP/BiPAP), high-flow, or low-flow nasal cannula

* P/F <=100 -> 4
* P/F 100-200 -> 3
* P/F 200-300 ->  2
* P/F 300-400 -> 1
* P/F >400 -> 0

If PaO2/FiO2 is not available *or is more than 4 hours old*, use the SaO2/FiO2:
* SF<=150 -> 4
* SF 150-235 -> 3
* SF 235-315 ->  2
* SF 315-400 -> 1
* SF >400 -> 0

In other words, use the respiratory SOFA calculated from a blood gas for 4 hours after collection, then default back to SaO2/FiO2 ratio (unless a new blood gas has been drawn)

To estimate the FiO2 for a patient on low-flow nasal cannula, use the following formula

Fi02 = 0.24 + 0.04*(LPM)

where LPM = liters per minute of low-flow oxygen

Finally, for patients on room air, set FiO2 = 0.21. Patients on room air should almost always have a respiratory SOFA of 0.


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
* >12 -> 4

### SOFA Coagulation

Platelet count in 10^3 per uL

* >= 150 -> 0
* 100-150 -> 1
* 50-100 -> 2
* 20-50 -> 3
* <20 -> 4


### SOFA Central Nervous System
By recorded Glascow Coma Scale (GCS). If GCS is missing, a score of zero is assigned
* GCS = 15 ->0
* GCS 13-14 -> 1,
* GCS 10-12 -> 2,
* GCS 6-9 -> 3,
* GCS 0-5 -> 4
