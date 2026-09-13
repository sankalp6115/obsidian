**

Overview:

“Tropical cyclones are rapidly evolving weather systems that can cause severe winds, rainfall, flooding and storm surge, particularly in coastal regions. Although India already has an advanced cyclone forecasting system, accurate prediction remains challenging because cyclone behavior depends on many interacting atmospheric and oceanic factors and the available observations can be heterogeneous or incomplete.

Our objective is to build an AI-assisted system that uses multi-source satellite and meteorological data. First, a CNN analyzes satellite imagery to detect and classify developing cyclonic systems based on their spatial structure. We then use temporal information such as historical position, movement and intensity, along with relevant environmental information, with an LSTM to forecast the cyclone's future track and intensity.

Finally, these predictions can be converted into location-specific early warnings for authorities, coastal communities and the maritime sector. We see this as a decision-support system that complements existing forecasting infrastructure rather than replacing meteorologists.”

  

Complete Overview of Project in simple:

Comprehensive Project & Technical Reference Synthesis (TatvaX SIH 2026)

1. Executive Summary & Core Positioning

Primary Objective: Build an AI-assisted decision-support system utilizing multi-source satellite and meteorological data to improve early detection, classification, and track/intensity forecasting of tropical cyclones.

Operational Alignment: Designed strictly as an analytical decision-support tool that complements existing forecasting infrastructure (such as IMD and RSMC New Delhi) rather than replacing human meteorologists.

30-Second Core Summary: Multi-source satellite imagery and meteorological data are ingested and preprocessed -> A CNN extracts spatial features to detect and classify developing cyclonic systems -> An LSTM model processes historical temporal sequences (position, movement, wind speed, pressure, ambient weather) to predict future track and intensity -> Predictions are converted into location-specific early warnings for disaster management authorities, fishermen, and coastal communities.

2. Slide-by-Slide Presentation Walkthrough

Slide 1 — Problem Statement: High-impact tropical cyclones require faster detection and accurate track/intensity prediction so that vulnerable coastal populations and emergency response authorities can take timely action.

Slide 2 — Proposed Solution & Pipeline:

Data Pipeline: Satellite Imagery (INSAT-3D/3DR, NOAA) + Meteorological Data -> Preprocessing -> CNN (Detection & Classification) -> LSTM (Track & Intensity Forecasting) -> Alert & Warning Generation -> Mobile/SMS delivery to authorities and public.

Slide 3 — Technical Stack & Implementation:

Frontend: React, React Native, D3.js (interactive graphs & data visualization), Leaflet (map visualization).

Backend: FastAPI (REST APIs), gRPC (high-throughput binary streaming), Python, SQLAlchemy.

Database Architecture:

PostgreSQL: Relational database for application metadata, user management, and static geographic boundaries.

InfluxDB: Time-series database dedicated to continuously incoming weather metrics, pressure/wind readings, and timestamped sensor data.

AI/ML Frameworks: PyTorch, TensorFlow, Scikit-Learn.

Primary Datasets: INSAT-3D/3DR satellite imagery, IMD Best Track API, Digital Typhoon dataset, NOAA HURSAT, OpenWeatherMap API, NOAA HURDAT2.

DevOps & Infrastructure: Docker (containerization), Nginx (reverse proxy / load balancer), Grafana (system & pipeline health monitoring), cloud GPU resources (Google Colab / dedicated GPUs for training).

Slide 4 — Feasibility & Viability:

Feasibility: Utilizes publicly accessible satellite/met datasets and proven CNN/LSTM deep learning architectures.

Key Challenges: Prediction uncertainty, false alarms, missing/corrupted satellite observations, real-time inference latency, and system scalability.

Mitigations: Multi-source probabilistic modeling, robust data augmentation, confidence thresholds, and human-in-the-loop verification.

Slide 5 — Impact & Sectoral Benefits:

Government & Emergency Response: Rapid, data-driven evacuation planning and targeted resource allocation.

General Public: Timely, transparent, location-specific early warnings.

Maritime & Fishing Industry: Trajectory insights to keep vessels and coastal fishermen out of high-risk sea areas.

Socio-Economic: Minimizes infrastructure damage, protects coastal trade/tourism, and saves lives.

Slide 6 — Research & Academic Grounding:

Grounded in peer-reviewed research for CNN spatial feature extraction and LSTM time-series forecasting.

Integrates ISRO (MOSDAC) and NOAA (NHC) satellite and historical cyclone track data.

3. Deep Learning Architecture & Defense Q&A

Functional Separation of CNN & LSTM:

CNN (Spatial Intelligence): Analyzes individual satellite imagery frames at time t. Extracts cloud structure organization, eye/eyewall features, and spatial patterns to detect presence and classify developmental stage.

LSTM (Temporal Intelligence): Processes historical sequential state vectors (t_{-n} ... t_0) containing latitude, longitude, maximum sustained wind speed, central pressure, and atmospheric steering variables to predict continuous future states (t_{+1} ... t_{+k}).

Model Integration: Features extracted by CNN are concatenated with historical weather and trajectory time-series data -> fed as sequential input vectors into the LSTM network.

Sub-Task Formulations:

Detection (Classification): Binary determination of whether a cyclonic disturbance is present. Primary evaluation metric: Recall (critical to minimize false negatives/missed cyclones).

Classification (Categorization): Identifies the category/stage of the cyclone (e.g., Depression, Cyclonic Storm, Severe Cyclonic Storm).

Forecasting (Regression):

Track Forecast: Predicts latitude/longitude sequence. Evaluation metric: Haversine distance / mean positional error (km).

Intensity Forecast: Predicts maximum wind speed and minimum central pressure. Evaluation metrics: MAE and RMSE.

Prototyping & Defense Strategy: Acknowledge that the system is a proposed architecture under active design and validation. Focus on explaining dataset selection, pipeline architecture, and physical justification rather than claiming unverified metrics.

4. Meteorological Science & Domain Fundamentals

Geographic Scope & Basins:

7 global tropical cyclone basins (~69% Northern Hemisphere, ~31% Southern Hemisphere).

Indian region focuses on Bay of Bengal (Eastern coast: Odisha, Andhra Pradesh, West Bengal, Tamil Nadu) and Arabian Sea (Western coast: Gujarat).

Cyclone Formation Mechanics (5 Steps):

Thermal Engine: Ocean sea surface temperatures >= 26-26.5 deg C supply heat and moisture.

Convective Lift & Latent Heat: Warm, moist air rises; condensation releases latent heat, warming the air further and decreasing surface pressure.

Surface Convergence: Surrounding air rushes into the central low-pressure zone.

Coriolis Deflection: Earth's rotation induces spin (counter-clockwise in North, clockwise in South; active broadly between 5–30 deg latitude).

Positive Feedback: Sustained inflow and latent heat release amplify circulation strength into a mature cyclone.

Structural Features:

Eye: Low-pressure, cloud-free, calm central axis.

Eyewall: Surrounding ring of violent updrafts, maximum wind speeds, and heavy rainfall.

Spiral Rainbands: Outward curved bands of convective thunderstorms.

IMD Intensity Classification (Wind Speed Thresholds):

Low Pressure Area: <31 km/h

Depression: 31–49 km/h

Deep Depression: 50–61 km/h

Cyclonic Storm: 62–88 km/h

Severe Cyclonic Storm: 89–117 km/h

Very Severe Cyclonic Storm: 118–167 km/h

Extremely Severe Cyclonic Storm: 168–221 km/h

Super Cyclonic Storm: >= 222 km/h

Movement Physics: Cyclones are guided by surrounding large-scale atmospheric steering flows (predominantly northwestward in the North Indian Ocean, with potential northeastward recurvature).

5. Ten Key Core Takeaways

Problem: Early detection, classification, and track/intensity prediction for tropical cyclones.

Inputs: Multi-source satellite imagery (INSAT, NOAA) combined with time-series meteorological and historical track data.

CNN Function: Spatial feature extraction and structural classification.

LSTM Function: Sequential trajectory (lat/long) and intensity (wind/pressure) regression forecasting.

Database Roles: PostgreSQL handles relational metadata/users/geography; InfluxDB handles high-frequency time-series weather streams.

Data Fusion: Multi-source ingestion protects against noise, cloud cover interference, and missing sensor data.

Key Evaluation Metrics: Precision/Recall/F1 for detection; Haversine distance, MAE, and RMSE for trajectory and intensity forecasting.

False Alarm Policy: Confidence thresholding and human-in-the-loop verification before triggering public warnings.

Primary End-Users: Disaster management authorities, coastal citizens, port operators, and commercial fishermen.

Core Identity: AI-driven decision-support tool complementing IMD/RSMC operational forecasting.

  
  
  

Project Implementation in detail:

Slide 1 — Problem Statement

What to say:

“Our problem statement is to develop an AI/ML-based system for tropical cyclone identification, classification, and prediction using multi-source satellite data.

The main objective is to improve early detection and prediction of cyclones so that authorities and people in vulnerable coastal regions can take timely action.”

---

## Slide 2 — Proposed Solution

What to say:

“Our solution consists of four major stages.

First, we collect multi-source satellite and atmospheric data, such as INSAT and NOAA, to continuously monitor cyclone activity.

Second, we use a CNN to analyze satellite images and detect and classify developing cyclonic systems.

Third, once a cyclone is detected, an LSTM model uses historical sequences to predict its future track and intensity.

Finally, based on the predicted impact, our system generates location-specific early warnings and communicates them directly to citizens and authorities through mobile or SMS.”

This slide is particularly important because it explains your complete pipeline:  
Satellite Data → Preprocessing → CNN → LSTM → Warning → Users/Authorities.

---

## Slide 3 — Technical Implementation

What to say:

“For the frontend, we are using React and React Native along with D3.js and Leaflet for visualization and maps.

React is a js library for UI

[D3.js](http://d3.js) is js library for graph and data visualization.

Leaflet is js library for showing maps easily

The backend is based on FastAPI and gRPC, with Python and SQLAlchemy.

FastAPI is python framework for developing backend apis

gRPC is a type of API where data is transported in compact binary formats and server sends continuos binary streams.

For data storage, we are using PostgreSQL and InfluxDB.

PostgreSQL is a RDBMS

InfluxDB is a time-series DB where current weather data with timestamp is stored.

For the AI and ML pipeline, we are using TensorFlow, PyTorch and scikit-learn.

Our major datasets include INSAT-3D and INSAT-3DR satellite imagery, IMD best-track data, the Digital Typhoon dataset, NOAA HURSAT and OpenWeatherMap data.

For deployment and monitoring, we are using Docker, Nginx and Grafana, with GPU resources such as Google Colab for model training.”

Docker -> containerization of app

Nginx -> reverse proxy / load balancer (Not much important)

Grafana -> Visualizations dashboard for system monitoring

Important: Don't spend too much time listing every technology. The CNN + LSTM + multi-source data pipeline is what judges will care about most.

---

## Slide 4 — Feasibility & Viability

“The solution is feasible because the required satellite, meteorological and geographic datasets are publicly available, and CNN and LSTM are already proven techniques for image analysis and sequential forecasting.

However, there are challenges such as prediction uncertainty, false alarms, missing or noisy satellite data, real-time processing and scalability.

To address these, we propose multi-source probabilistic modeling, robust validation, data augmentation, an optimized architecture and human-in-the-loop verification.

Most importantly, our system is designed as a decision-support tool for meteorologists and authorities, rather than replacing human decision-making.”

That “AI as decision support, not replacement” point is worth emphasizing. It makes the system sound much more realistic.

---

## Slide 5 — Impact & Benefits

What to say:

“Our system can benefit multiple groups.

For government and disaster management authorities, it can provide faster, data-driven decision-making and better resource allocation.

For the general public, timely and transparent information can improve awareness and preparedness.

Fishermen and the maritime sector can use the predictions to avoid dangerous sea conditions.

More broadly, the system can help reduce infrastructure damage, minimize disruption to fisheries, trade and tourism, and most importantly, save lives through earlier warnings.”

---

## Slide 6 — Research & References

What to say:

“We have based our approach on existing research in deep learning for tropical cyclone detection and track forecasting.

The CNN research supports satellite-image-based cyclone detection and classification, while the LSTM research supports sequential prediction of cyclone tracks and intensity.

We are also using data sources such as MOSDAC/ISRO and NOAA/NHC for satellite imagery and historical cyclone track and intensity data.

These references helped us design our CNN detection pipeline and LSTM forecasting approach.”

---

# The 30-second overall explanation

If a judge suddenly asks:

“Explain your solution in simple terms.”

Say:

“We are building an AI-based early warning system for tropical cyclones. We first collect satellite imagery and meteorological data from multiple sources. A CNN analyzes the satellite images to detect and classify cyclonic systems. Once a cyclone is detected, an LSTM uses its historical movement and intensity patterns along with relevant data to predict its future track and intensity. Based on the predicted impact, the system can generate location-specific alerts for authorities and the public.”

---

# One thing you should prepare for

The most likely technical follow-up is:

“Why are you using CNN and LSTM? Why not one model?”

Your short answer:

“CNN is particularly suited for extracting spatial features from satellite images, such as the structure and cloud patterns of a cyclone. LSTM is designed for sequential data, so it can learn how the cyclone's position and intensity evolve over time. Therefore, CNN handles the spatial information while LSTM handles the temporal prediction.”

Also be ready for:

- What exact data will LSTM receive?
    
- What exactly does the CNN classify?
    
- How will you handle missing satellite data?
    
- How will you measure prediction accuracy?
    
- How far into the future will you predict the track?
    
- How will you reduce false alarms?
    
- Why multi-source data instead of only INSAT?
    
- How is your system different from existing cyclone warning systems?
    
- Can CNN perform regression?
    
- Why LSTM instead of Transformer/GRU?
    

If you want, I can also prepare a rapid-fire list of 20–30 questions the SIH judges are likely to ask specifically about your CNN + LSTM + satellite-data architecture, with short answers you can memorize tonight.

  
  

Rapid Fire Questions:

# 1. Understand the project in one diagram

Your entire project can be understood as:

Satellite + Weather + Historical Data

                ↓

        Data Preprocessing

                ↓

             CNN

                ↓

    Cyclone Detection / Classification

                ↓

       Cyclone Characteristics

                ↓

             LSTM

                ↓

    Future Track + Intensity Prediction

                ↓

       Location-specific Alert

                ↓

     Authorities + Citizens

The key concept is:

CNN understands what the cyclone looks like. LSTM understands how the cyclone changes over time.

That's the sentence I would remember.

---

# 2. CNN + Satellite Data Questions

### Q1. Why are you using CNN?

Answer:

Satellite data contains spatial patterns. CNNs are good at extracting spatial features such as cloud structures, cyclone shape and organization from images.

---

### Q2. What exactly does the CNN do?

Your PPT says the CNN is responsible for detecting and classifying developing cyclonic systems.

You can say:

“The CNN takes preprocessed satellite imagery as input and determines whether a cyclonic system is present and, depending on our classification scheme, identifies its cyclone category or development stage.”

Important: Your PPT doesn't define the exact classes. You should decide these before presenting.

For example:

No Cyclone

↓

Tropical Disturbance

↓

Developing Cyclone

↓

Mature Cyclone

Or you could classify according to a recognized intensity classification.

---

### Q3. Is CNN only for classification?

Answer:

“No. CNNs can also be used for regression. In our architecture, however, we primarily use CNN for extracting spatial features and cyclone detection/classification.”

This distinction is important.

---

### Q4. What is the input to your CNN?

Answer:

“The primary input is satellite imagery from sources such as INSAT-3D/3DR and NOAA datasets. The images need to be cleaned, normalized and standardized before being given to the CNN.”

Your PPT explicitly lists INSAT-3D/3DR and NOAA HURSAT among the datasets.

---

### Q5. What preprocessing will you perform?

Say:

“We would perform operations such as resizing, normalization, handling missing or corrupted observations, and aligning the satellite images spatially and temporally.”

Don't claim a specific preprocessing pipeline unless you actually decide to implement it.

---

### Q6. Why use multiple satellite sources?

Answer:

“Different sources can provide complementary information and improve robustness. If one source has missing, noisy or low-quality observations, other sources can potentially provide additional information.”

This directly connects to the multi-source modeling and data redundancy mentioned in your PPT.

---

# 3. The Most Important Question: LSTM

### Q7. Why do you need LSTM after CNN?

This is probably one of the most important questions.

“A satellite image gives us spatial information at a particular point in time. But cyclone prediction requires understanding how the cyclone moves and changes over time. LSTM is designed to learn temporal patterns from sequential data, so we use it for track and intensity forecasting.”

Example:

t-3                    t-2      t-1                       t

 ↓                       ↓             ↓                        ↓

Cyclone → Cyclone → Cyclone → Cyclone

                         ↓

                       LSTM

                         ↓

                  Future position

                  Future intensity

---

### Q8. What goes into the LSTM?

This is where your team needs to be very clear.

Your PPT says:

LSTM predicts the cyclone's track and intensity.

It also says you have historical track/intensity data from IMD and NOAA/NHC.

A reasonable architecture would be:

Time t-5:

latitude, longitude, intensity, pressure, wind, weather features

        ↓

Time t-4

        ↓

Time t-3

        ↓

Time t-2

        ↓

Time t-1

        ↓

LSTM

        ↓

Time t+1 prediction

So the LSTM learns:

Given the cyclone's previous states, where and how strong will it be next?

---

### Q9. What is cyclone track prediction?

Answer:

“Track prediction means predicting the future geographical position of the cyclone, primarily represented by latitude and longitude over future time steps.”

For example:

Current:

15.2° N, 82.1° E

  

Prediction:

15.8° N, 81.3° E

16.5° N, 80.6° E

17.2° N, 79.8° E

These points form the predicted trajectory.

---

### Q10. What is intensity prediction?

Answer:

“Intensity prediction means estimating how the cyclone's strength will evolve over time, using parameters such as maximum wind speed or central pressure.”

Your NOAA/HURDAT2 reference specifically mentions position, maximum wind and central pressure.

---

### Q11. Is LSTM classification or regression?

For your proposed system, primarily regression.

“Track prediction is a regression problem because latitude and longitude are continuous values. Intensity prediction is also regression when predicting continuous quantities such as wind speed or pressure.”

This is a good technical point to know.

---

# 4. What Actually Connects CNN and LSTM?

### Q12. How will CNN and LSTM communicate?

This is a question you absolutely should prepare for.

A simple conceptual answer:

“The CNN extracts meaningful spatial features from satellite images. These features can be combined with meteorological and historical cyclone parameters and arranged as a time sequence. The resulting sequence is given to the LSTM for temporal prediction.”

Conceptually:

Satellite Image t1 ──→ CNN ──→ Features t1 ──┐

Satellite Image t2 ──→ CNN ──→ Features t2 ──┤

Satellite Image t3 ──→ CNN ──→ Features t3 ──┤→ LSTM → Prediction

Satellite Image t4 ──→ CNN ──→ Features t4 ──┤

                                                                                  │

Weather / Historical Data ────────────────┘

You don't need to claim that you've already implemented this if you haven't.

Say:

“This is our proposed architecture.”

That's completely defensible at this stage.

---

# 5. Data Questions

### Q13. What datasets are you using?

Your PPT lists:

- INSAT-3D / INSAT-3DR satellite imagery
    
- IMD Best Track API
    
- Digital Typhoon Dataset
    
- NOAA HURSAT
    
- OpenWeatherMap API
    

And your references specifically mention MOSDAC/ISRO and NOAA/NHC.

---

### Q14. What is "historical track data"?

“It is historical information about how cyclones moved and evolved, including their positions and intensity measurements at different times.”

Think:

Time     Latitude    Longitude    Wind

----------------------------------------

00:00    14.2        85.1         50

06:00    14.8        84.4         55

12:00    15.3        83.8         65

18:00    15.9        83.1         75

That becomes a time series for the LSTM.

---

### Q15. What is HURDAT2?

If they ask:

“HURDAT2 is a historical tropical cyclone database maintained by NOAA/NHC containing cyclone track and intensity information.”

Your PPT references HURDAT2 specifically for historical track and intensity data.

---

### Q16. What problems can occur with satellite data?

- Missing data
    
- Cloud cover
    
- Low resolution
    
- Sensor limitations
    
- Delays
    
- Noisy data
    

Answer:

“Satellite observations may contain missing, noisy or low-quality data. We plan to address this using multiple sources, preprocessing, interpolation where appropriate and data augmentation.”

---

# 6. Prediction Questions

### Q17. How far ahead will you predict?

Your PPT only says:

“the next hours/days.”

So don't invent a number.

Say:

“Our target is short-term forecasting over the next several prediction intervals, with the exact forecast horizon determined during model development and validation.”

---

### Q18. How do you measure track prediction accuracy?

This is something you should know.

For predicted and actual geographic positions, one common metric is Haversine distance.

Predicted location

       ●

       |

       |  error

       |

       ●

Actual location

You can say:

“For track prediction, we can evaluate the geographic distance between predicted and actual cyclone positions using metrics such as Haversine distance or mean positional error.”

---

### Q19. How do you evaluate intensity prediction?

“We can treat intensity prediction as regression and use metrics such as MAE, RMSE and potentially correlation between predicted and observed intensity.”

---

### Q20. How do you evaluate CNN detection?

For classification:

“We can use accuracy, precision, recall, F1-score and confusion matrix. For a disaster-warning application, recall is particularly important because missing an actual cyclone can be more dangerous than generating an additional false alarm.”

That's a very good answer for judges.

---

# 7. False Alarm Questions

### Q21. What happens if your model predicts a cyclone incorrectly?

Your PPT explicitly recognizes false positives and false negatives as a challenge.

Answer:

“We don't want the AI prediction to directly trigger critical decisions without validation. We propose confidence thresholds, robust validation and human-in-the-loop verification before high-impact warnings are issued.”

---

### Q22. Why not automatically send an alert whenever CNN detects a cyclone?

“Detection alone isn't sufficient. We need to consider confidence, predicted trajectory, intensity and potential geographical impact before generating a warning.”

This is an important distinction:

Detection ≠ Warning

---

# 8. Multi-source Data

### Q23. What does "multi-source" actually mean?

“It means combining information from different sources rather than relying on a single satellite or dataset—for example, satellite imagery, historical cyclone tracks, meteorological data and geographic information.”

---

### Q24. What if two data sources disagree?

Good answer:

“We would first align them temporally and spatially and assess their quality. Depending on the model design, we can assign appropriate weights or confidence to different sources rather than blindly treating all observations as equally reliable.”

Your PPT proposes probabilistic and multi-source modeling for exactly this reason.

---

# 9. Real-time System Questions

### Q25. How does this become a real-time system?

Conceptually:

New satellite observation

          ↓

     Preprocessing

          ↓

     CNN inference

          ↓

 Update cyclone state

          ↓

     LSTM forecast

          ↓

   Impact assessment

          ↓

      Alert users

Say:

“When new satellite or meteorological observations arrive, the pipeline processes them, updates the cyclone state and generates an updated forecast.”

---

### Q26. Why FastAPI?

“FastAPI provides a lightweight Python backend for exposing our ML inference and application services through APIs.”

Your PPT lists FastAPI and Python for the backend.

---

### Q27. Why PostgreSQL and InfluxDB?

This is another one you should understand.

PostgreSQL:

Structured application data, users, cyclone metadata, geographic information, etc.

InfluxDB:

Time-series data such as continuously changing weather and cyclone measurements.

So:

PostgreSQL

→ structured relational information

  

InfluxDB

→ time-series observations

Your PPT specifically proposes both.

---

# 10. Architecture / Deployment

### Q28. Why Docker?

“Docker allows us to package the different components of the system with their dependencies, making deployment and scaling more consistent.”

---

### Q29. Why Nginx?

“Nginx can act as a reverse proxy and efficiently route incoming requests to our backend services.”

---

### Q30. Why Grafana?

“Grafana can be used for monitoring system and pipeline metrics, helping us observe performance and resource usage.”

Your PPT lists Docker, Nginx and Grafana under deployment and monitoring.

---

# 11. The "AI Replacement" Question

### Q31. Can your AI replace meteorologists?

Do NOT say yes.

Your PPT explicitly frames the system as decision support, not replacement.

Say:

“No. Our system is intended as a decision-support tool. It provides fast, consistent predictions and confidence information, while final decisions remain with meteorologists and disaster-management authorities.”

---

# 12. The Difficult Questions

These are the questions where you need to understand your project rather than memorize answers.

### Q32. Why LSTM instead of a Transformer?

A safe answer:

“LSTM is a well-established architecture for sequential forecasting and is suitable as our initial temporal model. Transformer-based architectures could be explored later if the dataset size and computational requirements justify them.”

Don't claim LSTM is universally better.

---

### Q33. Why not use only satellite images?

“Satellite imagery provides strong spatial information, but cyclone evolution also depends on temporal and meteorological variables. Combining imagery with historical track, intensity and weather information can provide a more complete representation.”

---

### Q34. What if the cyclone suddenly changes direction?

“The model will have uncertainty because cyclone behavior is inherently difficult to predict. That's why we propose probabilistic predictions and continuous updating as new observations arrive.”

Your PPT specifically identifies prediction uncertainty as a major challenge.

---

### Q35. What is the biggest limitation of your system?

A very honest answer:

“The biggest limitation is prediction uncertainty and data quality. Cyclone evolution is complex, and satellite observations can be incomplete or noisy. Therefore, our system should provide probabilistic, continuously updated forecasts rather than treating predictions as certain.”

That's much better than claiming your AI will predict cyclones perfectly.

---

# 13. One Question You REALLY Need to Prepare

A judge may look at your architecture and ask:

“You have CNN for detection and LSTM for prediction. But what exactly is the input-output of each model?”

You should be able to answer this immediately:

### CNN

Input:

Satellite image

Output:

Cyclone detected?

+

Cyclone class / features

Potentially:

Image

 ↓

CNN

 ↓

Spatial features

 ↓

Cyclone classification

### LSTM

Input:

Sequence of cyclone states

+

weather/meteorological features

+

CNN-derived features (if included)

Output:

Future latitude

Future longitude

Future intensity

Then:

CNN → "What is happening?"

LSTM → "How will it evolve?"

That distinction will make your architecture much easier to explain.

---

# 14. And Because You Don't Have a Prototype...

This is actually important.

Don't try to bluff the judges into thinking you have implemented everything in the PPT.

If they ask:

“Have you trained the complete CNN-LSTM model?”

If you haven't:

“The complete integrated prototype is currently under development. Our current focus has been on validating the architecture, identifying suitable datasets and designing the end-to-end pipeline. The proposed implementation is based on established CNN-based detection and LSTM-based forecasting approaches.”

That's much safer than inventing accuracy numbers.

Similarly, don't invent:

- CNN accuracy
    
- LSTM RMSE
    
- number of training samples
    
- inference latency
    
- prediction horizon
    
- API response time
    
- number of cyclones detected
    

unless you actually have those results.

---

# 15. The 10 Things Your Entire Team Should Know Tonight

If you have limited time, learn these 10 things extremely well:

1. What problem are we solving?  
    Early detection + track/intensity prediction of tropical cyclones.
    
2. What is the input?  
    Satellite imagery + meteorological/historical data.
    
3. What does CNN do?  
    Spatial feature extraction + cyclone detection/classification.
    
4. What does LSTM do?  
    Temporal forecasting of cyclone movement and intensity.
    
5. What is track prediction?  
    Future latitude/longitude.
    
6. What is intensity prediction?  
    Future wind speed/pressure or another defined intensity measure.
    
7. Why multiple data sources?  
    Complementary information + robustness against missing/noisy data.
    
8. How do you evaluate it?  
    CNN → precision/recall/F1 etc.; LSTM → MAE/RMSE + positional error.
    
9. What are the major challenges?  
    Uncertainty, false alarms, missing data, real-time processing and scalability.
    
10. Is AI replacing meteorologists?  
    No. It's a decision-support system.
    

---

## One final mental model

## Satellite images tell us the cyclone's current visual structure. CNN extracts information from those images. Historical and meteorological data tell us how the cyclone has been evolving. LSTM learns that temporal evolution and predicts where the cyclone will go and how its intensity will change. The system then converts those predictions into location-specific early warnings.

  

One important point first: don't claim that existing cyclone forecasting is ineffective. In India, IMD already operates a sophisticated system using satellites, radar, surface/upper-air observations, numerical weather prediction, statistical/dynamical models and human forecasters. IMD issues track and intensity forecasts up to 120 hours. ([Mausam](https://mausam.imd.gov.in/Forecast/mcmarq/mcmarq_data/IMD%20Vision_2047_10-01-2025.pdf?utm_source=chatgpt.com))

---

  

Problem Statement Related Questions:

# 1. What is the actual problem?

A tropical cyclone is not just a big cloud visible on a satellite image.

It is a dynamic weather system whose:

- location changes with time
    
- intensity changes with time
    
- structure changes with time
    
- rainfall and wind fields change
    
- landfall location can change
    
- intensity can rapidly increase or decrease
    

So the actual problem is:

Given continuously arriving satellite, atmospheric, oceanic and historical data, can we automatically detect a developing cyclone and estimate how its location and intensity will evolve over the coming hours/days?

Detection → Classification → Track & Intensity Prediction → Warning.

---

# 2. Where do tropical cyclones happen?

Cyclones primarily form over tropical and subtropical oceans.

There are seven major tropical-cyclone basins:

|   |   |
|---|---|
|Region|Common name|
|North Atlantic|Hurricanes|
|Northeast Pacific|Hurricanes|
|Northwest Pacific|Typhoons|
|North Indian Ocean|Cyclones|
|Southwest Indian Ocean|Tropical cyclones|
|Southeast Indian/Australian|Tropical cyclones|
|Southwest Pacific|Tropical cyclones|

Approximately 69% occur in the Northern Hemisphere and 31% in the Southern Hemisphere; the Pacific accounts for the largest share of global tropical-cyclone activity.

### For India, the important regions are:

Bay of Bengal

India

  |

  |   Bay of Bengal

  |       ↓

  |    Cyclones

  |

  ↓

Eastern Coast

and

Arabian Sea

India

  |

  | ← Arabian Sea

  |

  ↓

Western Coast

This is why places such as Odisha, Andhra Pradesh, West Bengal, Tamil Nadu and parts of the Gujarat coast are more relevant to cyclone landfall risk than inland northern India.

So your system is particularly relevant to coastal populations, fishermen, ports, disaster-management authorities and infrastructure.

# 3. Why do cyclones form?

Think of a tropical cyclone as a heat engine powered by the warm ocean.

### Step 1 — Warm ocean

The ocean surface needs to be sufficiently warm. Around 26–26.5°C or higher over a significant depth provides the energy and moisture needed for development.

Warm water → evaporation → lots of water vapour.

---

### Step 2 — Air rises

The warm, moist air rises.

As it rises, the water vapour condenses into clouds.

Condensation releases latent heat.

That makes the surrounding air warmer and encourages further rising.

So you get:

Warm ocean

    ↓

Evaporation

    ↓

Moist air rises

    ↓

Condensation

    ↓

Heat released

    ↓

More rising air

    ↓

Lower pressure

---

### Step 3 — Pressure decreases

As air rises from the surface, pressure near the surface decreases.

Air from surrounding regions starts flowing toward the low-pressure area.

---

### Step 4 — Rotation develops

Because Earth rotates, the Coriolis effect causes the inflowing air to rotate.

Therefore:

- Northern Hemisphere → counter-clockwise
    
- Southern Hemisphere → clockwise
    

A cyclone generally doesn't form right at the equator because the Coriolis effect is too weak there. Tropical cyclones commonly form several degrees away from the equator, broadly around 5–30° latitude. ([National Hurricane Center](https://www.nhc.noaa.gov/climo/?utm_source=chatgpt.com))

---

### Step 5 — Feedback loop

Now we get a positive feedback mechanism:

Warm ocean

    ↓

Evaporation

    ↓

Moist air

    ↓

Thunderstorms

    ↓

Condensation → Heat

    ↓

More rising air

    ↓

Lower pressure

    ↓

More inflow

    ↓

Stronger circulation

    ↓

Stronger cyclone

But this doesn't happen automatically. Conditions such as warm water, moisture, a pre-existing disturbance and relatively low vertical wind shear need to be favorable. ([National Ocean Service](https://oceanservice.noaa.gov/facts/cyclone.html?utm_source=chatgpt.com))

---

# 4. What are the components of a cyclone?

This is especially relevant because you're working with satellite imagery.

The three major visible structural components are:

### 1. Eye

The center.

Usually:

- relatively calm
    
- comparatively cloud-free
    
- lowest surface pressure
    
- warmer aloft
    

---

### 2. Eyewall

The ring surrounding the eye.

This is where you generally find:

- strongest winds
    
- intense thunderstorms
    
- heavy rainfall
    
- strong upward motion
    

([National Ocean Service](https://oceanservice.noaa.gov/facts/cyclone.html?utm_source=chatgpt.com))

---

### 3. Spiral rainbands

These are curved bands of thunderstorms extending outward from the center.

They produce:

- heavy rain
    
- strong winds
    
- thunderstorms
    

Conceptually:

       Spiral rainbands

      ╱       │       ╲

  

   ─────── Eyewall ───────

             ↓

          ┌─────┐

          │ EYE │

          └─────┘

### Why does this matter for your CNN?

Because the visual structure of these features changes as a cyclone develops.

A CNN can learn patterns such as:

Disorganized clouds

        ↓

Organized circulation

        ↓

Curved rainbands

        ↓

Stronger central structure

        ↓

Well-defined eye/eyewall

So the satellite image contains information about the cyclone's development stage and intensity.

---

# 5. How does a cyclone evolve?

This is another thing you should understand extremely well.

A simplified lifecycle is:

Tropical Disturbance

        ↓

Depression

        ↓

Deep Depression

        ↓

Cyclonic Storm

        ↓

Severe Cyclonic Storm

        ↓

Very Severe Cyclonic Storm

        ↓

Extremely Severe Cyclonic Storm

        ↓

Super Cyclonic Storm

For the North Indian Ocean, these are the categories used by IMD. ([Mausam](https://mausam.imd.gov.in/met-oly/Met-Olympiad-Study-Material-Junior.pdf?utm_source=chatgpt.com))

The corresponding maximum sustained wind ranges are:

|   |   |
|---|---|
|Stage|Wind speed|
|Low Pressure Area|<31 km/h|
|Depression|31–49 km/h|
|Deep Depression|50–61 km/h|
|Cyclonic Storm|62–88 km/h|
|Severe Cyclonic Storm|89–117 km/h|
|Very Severe Cyclonic Storm|118–167 km/h|
|Extremely Severe Cyclonic Storm|168–221 km/h|
|Super Cyclonic Storm|≥222 km/h|

([Mausam](https://mausam.imd.gov.in/met-oly/Met-Olympiad-Study-Material-Junior.pdf?utm_source=chatgpt.com))

---

# 6. What happens during intensification?

Suppose your satellite system observes this:

### T₀

Weak disturbance.

☁ ☁   ☁

   ☁ ☁

☁    ☁

### T₁

Clouds become more organized.

 ☁☁☁

 ☁   ☁

☁  ↻  ☁

 ☁   ☁

  ☁☁☁

### T₂

Strong circulation develops.

### T₃

Eye and eyewall become visible.

### T₄

Cyclone reaches high intensity.

So your CNN isn't merely answering "cyclone / no cyclone."

Ideally, you're trying to extract information about the structure and development of the system.

---

# 7. How does a cyclone move?

This is where your LSTM becomes important.

A cyclone doesn't simply move randomly.

Its movement is influenced strongly by the large-scale atmospheric flow around it, often called the steering flow.

For example:

Atmospheric steering winds

          ↓

       CYCLONE

          ↓

   Future position

In the North Indian Ocean, many systems move generally northwestward, although some later recurve toward the northeast depending on the surrounding atmospheric circulation. ([RSMC New Delhi](https://rsmcnewdelhi.imd.gov.in/movement.php?utm_source=chatgpt.com))

That's why you shouldn't think:

"LSTM predicts the cyclone's path from the previous location alone."

Instead:

The model should learn the relationship between the cyclone's historical movement and the surrounding environmental conditions.

---

# 8. Why is prediction difficult?

Imagine you have:

Time 1 → (15.0°N, 85.0°E)

Time 2 → (15.5°N, 84.2°E)

Time 3 → (16.0°N, 83.5°E)

You could extrapolate the movement.

But suddenly:

- steering winds change
    
- ocean conditions change
    
- wind shear increases
    
- interaction with land occurs
    
- internal cyclone structure changes
    

The trajectory can change.

Therefore:

Cyclone prediction is inherently uncertain.

IMD itself uses a combination of observations, numerical/dynamical models, statistical techniques and forecaster expertise rather than relying on one model. ([Mausam](https://mausam.imd.gov.in/Forecast/mcmarq/mcmarq_data/IMD%20Vision_2047_10-01-2025.pdf?utm_source=chatgpt.com))

This is why your PPT's idea of probabilistic/multi-source modelling is important.

---

# 9. What are existing solutions?

This is where I would be careful during SIH.

## Existing solution: IMD

India already has a very sophisticated operational cyclone warning system.

IMD uses:

- satellite observations
    
- Doppler weather radars
    
- surface observations
    
- upper-air observations
    
- ocean observations
    
- numerical weather prediction
    
- dynamical/statistical models
    
- forecaster expertise
    

([Mausam](https://mausam.imd.gov.in/Forecast/mcmarq/mcmarq_data/IMD%20Vision_2047_10-01-2025.pdf?utm_source=chatgpt.com))

RSMC New Delhi also provides:

- cyclone detection
    
- tracking
    
- track forecasts
    
- intensity forecasts
    
- storm-surge guidance
    
- marine warnings
    
- cyclone bulletins
    

([RSMC New Delhi](https://rsmcnewdelhi.imd.gov.in/?utm_source=chatgpt.com))

And IMD already issues 120-hour track and intensity forecasts from the appropriate cyclone stage. ([RSMC New Delhi](https://rsmcnewdelhi.imd.gov.in/bulletins-products.php?utm_source=chatgpt.com))

So if a judge asks:

"Why do we need your system if IMD already does this?"

This is probably your most important answer.

---

# 10. Don't say "IMD is bad"

Instead say:

“We are not trying to replace the existing operational forecasting system. Our objective is to develop an AI-assisted analytical layer that can complement existing forecasting by automatically learning patterns from multi-source satellite and historical data and providing rapid, continuously updated predictions and decision support.”

That is a much stronger position.

---

# 11. So what is YOUR contribution?

You need to be able to articulate this clearly.

### Existing operational approach

Many observations

       +

Numerical models

       +

Statistical models

       +

Meteorological expertise

       ↓

Official forecast

       ↓

Warning

### Your proposed AI layer

Satellite imagery ────────┐

                              │

Historical cyclone data    ───┤

                           ↓

                     AI pipeline

                           ↓

                 Detection + Classification

                           ↓

                 Track + Intensity Forecast

                           ↓

                    Decision Support

Your PPT specifically proposes combining satellite, meteorological, oceanic and historical information and using probabilistic modelling, robust validation and human-in-the-loop verification.

---

# 12. What is the problem with existing approaches?

Be careful with this wording.

Don't say:

"Existing systems cannot predict cyclones."

That's false.

Instead, talk about challenges that remain in cyclone forecasting:

### 1. Prediction uncertainty

Even sophisticated models have uncertainty in:

- track
    
- intensity
    
- landfall
    
- rapid intensification
    

---

### 2. Data heterogeneity

You have different sources:

Satellite

Weather stations

Radar

Ocean

Historical tracks

Numerical models

They have different:

- resolutions
    
- formats
    
- update frequencies
    
- spatial coverage
    
- uncertainties
    

Combining them effectively is difficult.

---

### 3. Missing/noisy data

Satellite observations can be affected by:

- clouds
    
- sensor limitations
    
- resolution
    
- missing observations
    
- processing delays
    

Your PPT explicitly identifies this as a challenge.

---

### 4. False alarms

If you issue too many warnings:

False alarm

     ↓

People stop trusting warnings

But if you miss a real cyclone:

Missed cyclone

     ↓

Potentially severe consequences

Therefore your system needs a confidence-aware warning mechanism.

---

### 5. Real-time processing

Satellite data can arrive continuously.

Your system needs to process:

New observation

      ↓

Preprocess

      ↓

Inference

      ↓

Update forecast

      ↓

Update warning

with sufficiently low latency.

Your PPT explicitly identifies real-time processing and scalability as challenges.

---

# 13. Why could your solution be useful?

I'd frame your benefits into three levels.

### Level 1 — Detection

Automatically identify developing cyclonic systems from satellite imagery.

### Level 2 — Prediction

Estimate future location and intensity using temporal information.

### Level 3 — Decision support

Convert predictions into understandable, location-specific information for authorities, fishermen and communities.

So:

Raw Data

   ↓

Information

   ↓

Prediction

   ↓

Action

That's the real value of your project.

---

# 14. How does this help a fisherman?

This is an excellent example to use during presentation.

Imagine a fisherman operating near the coast.

Instead of only knowing:

"There is a cyclone somewhere in the Bay of Bengal."

the system could eventually provide:

"A cyclonic system is moving northwest. Its predicted trajectory indicates increasing risk for this coastal region within the forecast period."

Then the fisherman can return to safety earlier.

Similarly:

### Government

Can prepare:

- evacuation
    
- emergency teams
    
- shelters
    
- food
    
- medical resources
    

### Ports

Can prepare for dangerous marine conditions.

### Coastal communities

Can receive earlier warnings.

### Researchers

Can use the historical predictions/data for climate and cyclone analysis.

These are also the target groups identified in your PPT.

---

# 15. One subtle but VERY important distinction

Your project has three different prediction problems.

### Problem A — Detection

Is there a cyclone?

Image → CNN → Cyclone / No Cyclone

### Problem B — Classification

What stage/category is it?

Image → CNN → Depression / Cyclonic Storm / etc.

### Problem C — Forecasting

Where will it go and how strong will it become?

Historical sequence + current information

                 ↓

               LSTM

                 ↓

Future location + intensity

Don't mix these three when answering judges.

And one final point: your project's strongest story isn't “we made an AI that predicts cyclones better than IMD.” You don't have evidence for that yet. The stronger and defensible story is “we propose an AI-based, multi-source decision-support pipeline that can complement existing cyclone monitoring and forecasting by learning spatial patterns from satellite imagery and temporal patterns from historical/environmental data.” ([Mausam](https://mausam.imd.gov.in/Forecast/mcmarq/mcmarq_data/IMD%20Vision_2047_10-01-2025.pdf?utm_source=chatgpt.com))

  
  
  
  
  
  
  
  
**