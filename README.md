# What Drives Logistics Efficiency in Last Mile Delivery Operations?

---

## Overview

This project analyzes Amazon's last mile delivery operations across India to understand what actually drives delivery efficiency. We started with a raw operational dataset of roughly 43,700 deliveries, cleaned and validated it in Python, engineered new analytical fields, and built a set of interactive Tableau dashboards that test our hypotheses and translate the findings into operational recommendations.

Our core question: **of all the factors recorded in a delivery (distance, traffic, weather, area type, agent quality, time of day, and location), which ones genuinely explain how long a delivery takes, and where should an operations manager focus to improve performance?**

---

## Target Audience

This project is built for **logistics and operations managers** responsible for regional delivery performance, **supply chain analysts** who monitor efficiency metrics, and **students or professionals** studying how large technology companies run last mile delivery networks.

The dashboards are designed to be read by a decision-maker, not a data scientist. Each one answers a specific operational question and is built to stand on its own without narration.

---

## Questions We Set Out to Answer

The project is organized around four hypotheses from our original proposal:

1. **Distance and time:** Delivery time increases with distance, but not in a perfectly linear way because of real-world constraints like traffic and routing.
2. **Geographic variation:** There are meaningful differences in delivery performance across regions, pointing to uneven logistics efficiency.
3. **Diminishing returns:** Reducing delivery time gets harder beyond a certain point.
4. **Outliers as bottlenecks:** Extreme delivery times represent operational bottlenecks, not just long distances.

---

## The Dataset

**Source:** [Amazon Delivery Dataset on Kaggle](https://www.kaggle.com/datasets/sujalsuthar/amazon-delivery-dataset)

The raw file contains 43,739 individual deliveries across more than 20 Indian cities, recorded between February and April 2022. Each row captures order and pickup timestamps, store and drop-off GPS coordinates, delivery agent attributes, the weather and traffic conditions at the time of the order, the vehicle and area type, the product category, and the total delivery time in minutes.

After cleaning the data we were left with 24 columns and 40,153 rows.

---

## Data Cleaning and Validation

The raw export was not analysis ready. Before building a single chart, we profiled the data, documented every issue, and applied a cleaning pipeline. The full process is in `amazon_delivery_eda.ipynb`. Summary of what we did:

**Standardized categorical text.** Stripped trailing whitespace from every text column, fixed inconsistent casing (for example "motorcycle" became "Motorcycle"), and corrected the misspelling "Metropolitian" to "Metropolitan."

**Converted disguised missing values.** Some cells contained the literal text string "NaN" rather than a real null. We converted these into actual missing values so they could be handled properly.

**Removed corrupt records.** We dropped 91 rows that were missing Weather, Traffic, and Order_Time simultaneously. These same 91 rows also carried impossible sentinel values (an Agent_Rating of 6.0 on a 1-5 scale, paired with boundary ages of 15 or 50), confirming they were a single upstream pipeline failure rather than scattered errors.

**Fixed GPS coordinates.** Some store coordinates were sign-flipped (negative values mirroring their correct positive counterparts). Because India sits entirely in the positive latitude and longitude quadrant, we took the absolute value to correct them. We then dropped roughly 3,500 rows with placeholder near-zero coordinates, which were failed geocodes that could not be located on a map.

**Validated the result.** The cleaned dataset has 40,153 rows and only 44 remaining missing values, all in Agent_Rating, which we intentionally kept since rating is not critical to the analysis and we preferred to preserve those rows.

### Feature Engineering

On top of cleaning, we engineered new columns the raw data did not have. Operational data is recorded for transactions, not for analysis, so this step was necessary to make the data usable in Tableau:

- **Order_Datetime and Pickup_Datetime:** proper timestamps built from the separate date and time text fields, with a fix for pickups that crossed midnight.
- **Pickup_Lag_Min:** the minutes between order placement and pickup.
- **Distance_KM:** the great-circle distance between store and drop-off, calculated with the Haversine formula. The raw data gave us four coordinate values but never the distance between them, which is the central variable for testing Hypothesis 1.
- **Day_of_Week, Hour_of_Day, Week_of_Year:** calendar fields extracted from the order timestamp so the data could be grouped by time.
- **City:** each delivery mapped to its nearest major Indian city, which collapsed 40,000 coordinate pairs into about 22 meaningful groups and made geographic analysis possible.

This took the dataset from 16 columns up to 24.

---

## The Dashboards

The Tableau workbook contains four dashboards, each answering one part of the central question.

### Executive Dashboard
A single-screen view designed for a manager who needs the headline story without clicking into anything. The dashboard is built in four sections that read top to bottom.

**Top KPI Summary** gives the high-level numbers at a glance: Overall Average Delivery Time (128 mins vs a 180-min threshold), Severe Delay Rate (16.3% vs an 18% threshold), Longest Route Average Time (139 mins for 20-23 km trips), Highest Risk Weather (Fog, with a 27.3% severe delay rate), and the Top High-Risk City (Dehradun at 205 min average).

**Geographic Overview** plots every city on a map of India, with circle size showing delivery volume and color showing average delivery time. This lets a viewer immediately spot where the network's weight sits and which cities are running slower than peers.

**Key Drivers of Delays** sits to the right of the map and breaks down *why* delays happen along three dimensions: Severe Delay Risk by Weather (Fog and Cloudy lead at 27.3% and 26.9%), Severe Delay Risk by Distance (severe-delay rate climbs sharply from 3% on short trips to 29% on the longest), and Average Delivery Time by Distance bucket (showing the relationship between trip length and time).

**Where Delays Are Highest** finishes the dashboard with a Top 5 Cities by Average Delivery Time bar chart and a Key Insight callout box flagging Dehradun as roughly 60 minutes above the overall average.

The dashboard answers four questions in one view: *how is the system performing overall, where does delivery happen, what drives delays, and which locations are worst affected.*

---

## Key Findings

**Traffic is the strongest categorical driver of delivery time.** Median delivery time climbs steadily from about 100 minutes in Low traffic to roughly 150 minutes in Jam conditions.

**Distance matters, but only modestly.** Delivery time rises with distance and then plateaus, with an overall correlation of about 0.28. Distance sets a rough floor for delivery time, but conditions like traffic drive most of the variation around it.

**Average performance is nearly uniform across cities, but tail performance is not.** Every major city averages around 125 minutes, which makes averages look identical everywhere. The P90 metric tells the real story: among high-volume cities, Vadodara and Jaipur stand out with worst-case delivery times above their peers. Operational weaknesses show up in tail variability, not in the average.

**Time of day has a large effect.** Mornings are the fastest at around 90 minutes, while the evening rush window of 7 to 9 pm is the slowest at close to 150 minutes, a swing of roughly 60 minutes based purely on when an order is placed.

**Agent rating is the strongest agent-level signal,** with a correlation of about -0.31 with delivery time, while agent age shows only a weak relationship.

**Weather is the strongest categorical driver of severe delays.** Fog conditions push the severe-delay rate to 27.3% and Cloudy to 26.9%, well above the overall rate of 16.3%. Sunny conditions, by contrast, hold the severe-delay rate to 6.8%. Weather sensitivity matters more for *failure rate* than for typical delivery time.

---

## How These Findings Should Drive Decisions

1. **Measure tail performance, not just averages.** A manager looking only at average delivery time sees no variation across India and concludes the network is balanced. P90 reveals which cities produce slower bad-day experiences. Amazon should add P90 and P95 to standard operational dashboards.

2. **Investigate Dehradun, Vadodara, and Jaipur.** Dehradun stands out on average delivery time (205 min, roughly 60 min above the overall mean of 128 min). Vadodara and Jaipur stand out on tail performance (P90 above peer cities). These are different signals from different metrics, and both warrant root-cause analysis.

3. **Do not over-invest in small-volume markets without more data.** We deliberately filtered out cities with fewer than 1,000 deliveries because their P90 values were dominated by statistical noise. Apparent problems in those markets should be confirmed with more data before any operational investment.

---

## Limitations and Assumptions

- **Time window is short.** The data covers only February through April 2022, so seasonal effects like monsoon, summer heat, and festival surges are not captured. Findings should be re-validated against current data before action.
- **Delivery volumes are highly imbalanced across cities.** Comparing performance metrics between a massive metropolitan hub and a low-volume regional city is prone to statistical noise. Managers should be cautious about drawing firm conclusions or making operational changes in low-volume markets without gathering more data.
- **City labels are approximate.** We assigned each delivery to the nearest major city within roughly 55 km. Suburbs and satellite towns may be miscategorized, though the high-volume cities that carry most of the data are confidently mapped.
- **Distance is straight-line, not road distance.** The Haversine formula gives the shortest possible distance between two points. Actual driving distance is longer, so Distance_KM understates how far an agent traveled. It is a sound proxy for comparing deliveries but not an exact route measurement.
- **Bicycle sample is tiny.** Only 15 deliveries used a bicycle, so any analysis filtered to that vehicle type is directional, not conclusive.
- **The maps show where, not why.** The geographic dashboard identifies where variability concentrates but cannot explain the cause. Confirming the root cause would require additional data on local infrastructure and routing.

---

## Tools Used

- **Tableau:** all four dashboards and nine supporting worksheets. Primary visualization tool.
- **Python (pandas, NumPy, Matplotlib, Seaborn):** data cleaning, validation, feature engineering, and exploratory analysis in a documented Jupyter notebook.
- **GitHub:** project documentation and version control.

---

## Use of LLMs & AI

In this project, LLMs were utilized as an analytical partner to accelerate data processing, enforce data quality, and expand the scope of exploratory data analysis (EDA). While initial EDA and baseline data cleaning were handled manually, AI was instrumental in identifying subtle anomalies and scaling complex transformations.

**Key contributions included:**
* **Advanced Data Validation:** The AI prompted critical logic checks **we** had not initially considered, such as flagging delivery agent ages that fell outside legal parameters and identifying mathematically invalid geographic coordinates for correction rather than deletion.
* **Feature Engineering:** To prevent loosely located deliveries from being dumped into an "Other" category, **we** used the LLM to write scripts that cross-referenced latitude and longitude coordinates against known city centers, mapping them accurately.
* **Tool Mastery & Debugging:** AI served as a dynamic debugging tool, accelerating the resolution of complex Tableau visualization issues.

Integrating AI into this workflow allowed us to optimize code, practice efficient prompt engineering, and build a highly validated data pipeline. Ultimately, it transformed a standard analysis into a much more robust project while simultaneously deepening our foundational technical skills.

---

## Design Approach

The dashboards apply course concepts in deliberate ways. We use a consistent sequential color scale across the geographic views so that color always means the same thing (darker equals slower). Marks are sized by volume so the eye is drawn to the highest-impact locations first, applying the Gestalt principle of visual hierarchy. On-graph text, axis labels, and tooltips are written so each dashboard can be understood without a presenter. Colors and fonts are kept consistent across all four dashboards and the supporting notebook so the project reads as one professional artifact.
