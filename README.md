# Amazon-Data-Final-Project
# What Drives Logistics Efficiency in Last Mile Delivery Operations?
**Group: Roberto Nunez & WestLee Parker**

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

## Repository Contents

| File | Description |
|------|-------------|
| `Vis_Final_Project.twbx` | Packaged Tableau workbook containing all dashboards and worksheets |
| `amazon_delivery_eda.ipynb` | Python notebook documenting the full data cleaning, validation, and exploratory analysis |
| `amazon_delivery_clean.csv` | The cleaned dataset produced by the notebook and used as the source for every Tableau dashboard |
| `Data_Visualization_Final_Project_Proposal.pdf` | Our original project proposal |
| `README.md` | This file |

**Tableau workbook link:** *(paste your published Tableau Public / course Tableau site link here)*

---

## The Dataset

**Source:** [Amazon Delivery Dataset on Kaggle](https://www.kaggle.com/datasets/sujalsuthar/amazon-delivery-dataset)

The raw file contains 43,739 individual deliveries across more than 20 Indian cities, recorded between February and April 2022. Each row captures order and pickup timestamps, store and drop-off GPS coordinates, delivery agent attributes, the weather and traffic conditions at the time of the order, the vehicle and area type, the product category, and the total delivery time in minutes.

This comfortably clears the "sufficient heft" bar: 16 raw columns and over 43,000 rows, expanded to 24 columns and 40,153 rows after cleaning and feature engineering.

---

## Data Cleaning and Validation

The raw export was not analysis-ready. Before building a single chart, we profiled the data, documented every issue, and applied a cleaning pipeline. The full process is in `amazon_delivery_eda.ipynb`. Summary of what we did:

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
A high-level summary of system performance: average delivery time, average distance, and average delivery speed. This gives a manager a quick read on the network before drilling into the detailed views.

### D2: Distance and Time
A scatter plot of distance against delivery time with a trend line, supported by average time per distance bucket and a severe-delay rate breakdown. This view tests Hypothesis 1 and shows that distance and time move together but the relationship flattens out, meaning distance alone is a weak predictor.

### D3: Geographic Performance
An interactive map of India paired with a city ranking bar chart. City dots are sized by delivery volume and colored by P90 delivery time, the "bad day" experience that 90 percent of deliveries finish within. A sample-size filter restricts the comparison to cities with at least 1,000 deliveries so the rankings are statistically reliable. This view tests Hypotheses 2 and 4.

### D6: Weather Impact
A breakdown of how weather conditions affect delivery duration and severe-delay rates, comparing performance across weather categories.

All dashboards share a consistent set of interactive filters (Traffic, Weather, Vehicle, Area) so a viewer can slice the data and watch every view respond together.

---

## Key Findings

**Traffic is the strongest categorical driver of delivery time.** Median delivery time climbs steadily from about 100 minutes in Low traffic to roughly 150 minutes in Jam conditions.

**Distance matters, but only modestly.** Delivery time rises with distance and then plateaus, with an overall correlation of about 0.28. Distance sets a rough floor for delivery time, but conditions like traffic drive most of the variation around it.

**Average performance is nearly uniform across cities, but tail performance is not.** Every major city averages around 125 minutes, which makes averages look identical everywhere. The P90 metric tells the real story: among high-volume cities, Vadodara and Jaipur stand out with worst-case delivery times above their peers. Operational weaknesses show up in tail variability, not in the average.

**Time of day has a large effect.** Mornings are the fastest at around 90 minutes, while the evening rush window of 7 to 9 pm is the slowest at close to 150 minutes, a swing of roughly 60 minutes based purely on when an order is placed.

**Agent rating is the strongest agent-level signal,** with a correlation of about -0.31 with delivery time, while agent age shows only a weak relationship.

---

## How These Findings Should Drive Decisions

1. **Measure tail performance, not just averages.** A manager looking only at average delivery time sees no variation across India and concludes the network is balanced. P90 reveals which cities produce slower bad-day experiences. Amazon should add P90 and P95 to standard operational dashboards.

2. **Investigate Vadodara and Jaipur specifically.** These are the operational weak points among high-volume markets and should be the priority targets for root-cause analysis.

3. **Do not over-invest in small-volume markets without more data.** We deliberately filtered out cities with fewer than 1,000 deliveries because their P90 values were dominated by statistical noise. Apparent problems in those markets should be confirmed with more data before any operational investment.

---

## Limitations and Assumptions

- **Time window is short.** The data covers only February through April 2022, so seasonal effects like monsoon, summer heat, and festival surges are not captured. Findings should be re-validated against current data before action.
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

## Use of Large Language Models

We used an LLM assistant (Claude) throughout this project, and being honest about how it helped is part of the writeup.

The biggest contribution was working past the limitations of our own data skills. Our initial EDA went reasonably well on its own. We could load the data, see the missing values, and start working with it. But the LLM pushed the analysis further than we would have taken it ourselves. It helped us think critically about the data and flagged checks we had not even considered. We would not have thought to verify that delivery agent ages fell outside a realistic range, or that some of the geographic coordinates were genuinely wrong and needed to be corrected rather than just dropped. It also helped with the city labels. A lot of deliveries were only loosely located, and without help many would have ended up labeled as "Other." Cross-referencing the latitude and longitude coordinates against known city centers let us map those out properly. In short, the LLM filled gaps in our foundational skills in a way that let us keep building, and we learned from each step rather than just accepting an answer.

The second area where it helped was working through pain points in Tableau. When we got stuck building a specific visual, we could describe what we were trying to make and what was going wrong, and work through the fix step by step. That kept the project moving when we would otherwise have been blocked.

Our honest takeaway is that an LLM is a genuinely useful tool, but only when you already know where you want to go and how you intend to analyze the data. It accelerated our learning and troubleshooting, but the analytical direction, the cleaning decisions, the dashboard design, and the final interpretations were ours. The tool helped us execute and learn faster, not think for us.

---

## Design Approach

The dashboards apply course concepts in deliberate ways. We use a consistent sequential color scale across the geographic views so that color always means the same thing (darker equals slower). Marks are sized by volume so the eye is drawn to the highest-impact locations first, applying the Gestalt principle of visual hierarchy. On-graph text, axis labels, and tooltips are written so each dashboard can be understood without a presenter. Colors and fonts are kept consistent across all four dashboards and the supporting notebook so the project reads as one professional artifact.
