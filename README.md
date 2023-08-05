# Bellabeat Case Study
Author: Austin Elendt

Date: 8/4/2023
## Company Information
Bellabeat is a woman-owned tech company that focuses on mindfulness, fitness, and health. They are relatively small company with potential to expand their influence in the global smart device market. They have a respectable catalog of products which include:
- Bellabeat App - provides users with health data related to activity, stress, heart rate, and menstrual cycles. The app also connects to any smart devices that the customer owns.
- Leaf - a wellness tracker bracelet, necklace, or clip-on. The leaf  tracker connects to the Bellabeat app to track activity, sleep, and stress. 
- Time - this wellness watch that combines a timepiece with smart technology to track user activity, sleep, and stress. This also connects to the Bellabeat app to provide data insights.
- Spring - this is a water bottle that tracks your daily hydration using smart technology. This water bottle also connects to the Bellabeat app.
- Bellabeat membership - This is a subscription based membership program that gives 24/7 personalized guidance and analytics for nutrition, activity, sleep, health, beauty, and mindfullness.
## Ask
### What is the Business Task?
- Identify trends in the data that shows how customers are interacting with the product (Fitbit)
- Identify trends that can be used to improve the user experience for any of the Bellabeat products
- Find correlations that can be used to verify the effectiveness of Bellabeat (comparable) products
### Who are the Stakeholders?
- Urska Srsen - Chief Creative Officer
- Sando Mur - Co-founder
- Bellabeat Marketing and Analytics team

## Prepare
### Data Source
  The data is publically available from Kaggle.com. It shows the user data from 33 Fitbit users over a 31 day period. The date ranges from April 2016 to May 2016. The data shows daily, hourly, and second data on a number or metrics. These metrics track heart rate, sleep time, GPS location (for distance traveled), number of steps taken, minutes spent at certain levels of intensity, and calories burned. The data can be downloaded from the link below.

[Fitbit Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit).
### 
### Tools Used
- Excel - Most of the files and tables contain less than 10,000 rows. It is easier to condense it all into one spreadsheet than to primarily use SQL and R for analysis.
- SQL (BigQuery) - One of the files containing heart rate information had over 2 million rows and needed to be analyzed using SQL because excel could not handle the size of the data.
- R (Rstudio) - Used to help analyze the heart rate information mentioned above.
### Data Concerns and Limitations
- The overall sample size for the dataset is too low to show major customer trends.
- Many of the tables had an incompatible datetime format which made upload and data aggregation difficult.
- Some of the participants in the dataset showed low product engagement (did not wear the Fitbit for that day) and needed to be excluded from the analysis.
## Process
  While familiarizing myself with the data I found that many of the entries in the dailyActivity_merged.csv file showed 0 for tracker distance, 0 for total steps, and 1440 for sedentary minutes. This indicated to me that the user did not wear their smart device at all that day as 1440 minutes equates to 24 hours. I sorted these columns in ascending order and removed anything that met that criteria. After removal I used a count function in a pivot table to count the number of days per user. The data yielded the graph below.

![user_engagement](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/c7b0e4fc-dc3c-4e31-b705-052e6e85e6f6)

The above graph shows me information about the overall health of the dataset and also where things like product engagment can be improved. The average product engagement over the 31 day period was 84%.
## Analysis
### Activity Level Breakdown
![activity_level_breakdown](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/5d97ec9c-7ec4-4f90-8a18-0d1b16701725)
### Active Minutes vs Calories Burned
| Lightly Active Minutes | Fairly Active Minutes | Very Active Minutes |
| ---------- | ---------- | ---------- |
| ![lightlyActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/940f9d32-b16e-4b7b-a7ec-ade4878e979e) | ![fairlyActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/545c68fa-4469-45bb-bb8e-d0495cee7cd0) | ![veryActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/69fa8657-c5ab-4cfa-a5ec-4eac2cfee865) |

![aggregatedActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/5c65eaee-8294-472c-8179-40ad7e70dc01)
### Calories vs Steps
![calories_vs_totalSteps](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/53f02593-90f0-4664-a68a-f4798f665612)
![calories_vs_stepsByIndividual](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/3d0a9e3f-57a9-48a7-a617-af4ad9252122)
### Activity by Time
![steps_by_dayofWeek](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/8332e415-6410-48fd-b6eb-1edd7fdb8af3)
![steps_by_timeofDay](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/fe478014-adca-4446-b751-dee8c260e77e)
## Recommendations


