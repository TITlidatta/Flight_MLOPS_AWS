# Flight Delay Prediction - Documentation

## Overview
This project implements a machine learning pipeline for predicting flight delays using AWS SageMaker.

The script `data_pipeline_npn.py`:
- Loads flight & airport data from S3
- Cleans and preprocesses data
- Trains a RandomForest classifier
- Saves model (`model.pkl`) for backend
- Logs accuracy (`latest.txt`) to S3

---

## Expected Input Data

### flights.csv
- YEAR: Year of flight (int)
- MONTH: Month (1–12)
- DAY: Day of month (1–31)
- DAY_OF_WEEK: 1 = Monday, …, 7 = Sunday
- AIRLINE: Airline code (string, e.g., "AA", "UA")
- FLIGHT_NUMBER: Flight number (int)
- TAIL_NUMBER: Aircraft identifier (string)
- ORIGIN_AIRPORT, DESTINATION_AIRPORT: 3-letter codes
- SCHEDULED_DEPARTURE, ARRIVAL_TIME, etc. (HHMM format → converted to minutes)
- ARRIVAL_DELAY: Target variable (minutes; later converted into binary class)

### airports.csv
- IATA_CODE: 3-letter code
- CITY, STATE: Location metadata
- LATITUDE, LONGITUDE: Coordinates

---

## Pipeline Steps
1. Data loading from S3
2. Cleaning irrelevant columns
3. Time feature engineering (HHMM → minutes since midnight)
4. Encoding categorical features
5. Train/test split
6. Missing value imputation (KNN & median)
7. Scaling (Robust + Standard)
8. Model training (RandomForestClassifier)
9. Outputs:
   - `/opt/ml/model/model.pkl` (inside SageMaker model.tar.gz)
   - `s3://model-npn/latest.txt` (accuracy log)

