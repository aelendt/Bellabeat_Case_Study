# Bellabeat Case Study
Author: Austin Elendt

Date: 8/4/2023
## Company Information
Bellabeat is a woman-owned tech company that focuses on mindfulness, fitness, and health. They are relatively small company with potential to expand their influence in the global smart device market. They have a respectable catalog of products which include:
- Bellabeat App - Provides users with health data related to activity, stress, heart rate, and menstrual cycles. The app also connects to any smart devices that the customer owns.
- Leaf - A wellness tracker bracelet, necklace, or clip-on. The leaf  tracker connects to the Bellabeat app to track activity, sleep, and stress. 
- Time - This wellness watch that combines a timepiece with smart technology to track user activity, sleep, and stress. This also connects to the Bellabeat app to provide data insights.
- Spring - This is a water bottle that tracks your daily hydration using smart technology. This water bottle also connects to the Bellabeat app.
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

### Tools Used
- Excel - Most of the files and tables contain less than 10,000 rows. It is easier to condense it all into one spreadsheet than to primarily use SQL and R for analysis. All visualizations were created within Excel.
- SQL (BigQuery) - One of the files containing heart rate information had over 2 million rows and needed to be analyzed using SQL because excel could not handle the size of the data.
- R (Rstudio) - Used to help analyze the heart rate information mentioned above.
### Tables Used
- dailyActivity_merged
- hourlySteps_merged
### Data Concerns and Limitations
- The overall sample size for the dataset is too low to show major customer trends.
- Many of the tables had an incompatible datetime format which made upload and data aggregation difficult.
- Some of the participants in the dataset showed low product engagement (did not wear the Fitbit for that day) and needed to be excluded from the analysis.
- Many of the columns in the data do no specify what metric was used for measurement. I would have liked more clarification on this front.
## Process
  While familiarizing myself with the data I found that many of the entries in the dailyActivity_merged.csv file showed 0 for tracker distance, 0 for total steps, and 1440 for sedentary minutes. This indicated to me that the user did not wear their smart device at all that day as 1440 minutes equates to 24 hours. I sorted these columns in ascending order and removed anything that met that criteria. After removal I used a count function in a pivot table to count the number of days per user. The data yielded the graph below.
![user_engagement](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/059c74ec-712f-4d8b-aaf0-d47ac5643883)
The above graph shows me information about the overall health of the dataset and also where things like product engagment can be improved. The average product engagement over the 31 day period was 84%.

The other main change that was needed had to do with fixing an issue with the datetime for all the hourly and seconds tables. The format used during data collection would not properly upload for SQL analysis. This could be fixed by importing all the columns from the table as a string type and then recasting them using the following code.

``` SQL
SELECT
  CAST(Id AS INT64) AS Id,
  LEFT(Date,9) AS Date,
  RIGHT(Date,10) AS Time,
  CAST(Value AS INT64) AS Value
FROM
  `ionlyrunbigqueries.bellabeat.raw_seconds_heartRate`
WHERE
  Id <> 'Id';
```
The results are then exported to google sheets (not downloaded locally because BigQuery only allows up to 10 mb for file size, this resulted in data loss) and then redownloaded as a .csv file and imported again into Rstudio for analysis and vizualization. All other tables were small enough to be directly uploaded into excel and had their datetime column fixed manually.

Finally for this step I checked the number of participants in each table to ensure that the table had enough participants to be useful for analysis. The code below is a sample that was applied to each table.
``` SQL
SELECT
  COUNT(DISTINCT Id) AS num_of_id
FROM
 `ionlyrunbigqueries.bellabeat.daily_activity`;
```
**Results**
```
Row: 1
num_of_id: 33
```
The only table that was excluded from analysis due to low participants was weightLogInfo_merged.csv with only 8 participants.

## Analysis
### Activity Level Breakdown
Before anything else I want to get an idea for how much time was spent in at each level of intensity across all the participants.
![activity_level_breakdown](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/7d2e4807-04e5-4e66-9aba-5b6f97a256ae)
The main finding of interest was that the percentage of time spent very active was greater than the amount of time spent at a fairly active level. It is relatively expected that ~75% of the time was spent sedentary since most people that wear Fitbits are adults and a large percentage of adults work sedentary jobs.
### Active Minutes vs Calories Burned
After seeing a high level distribution of how much time was spent at each intensity level I wanted to get a better idea of how much time spent in each intensity level affected calories burned. The 3 graphs side by side show the 3 active intensities and how each affects calories burned. Relatively positive correlations are expected here.
| Lightly Active Minutes | Fairly Active Minutes | Very Active Minutes |
| ---------- | ---------- | ---------- |
| ![lightlyActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/e5b09db9-a9d6-4962-bdd9-1066a8e5e039) | ![fairlyActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/1def19b6-24da-4689-b930-ad91fe0eca9d) | ![veryActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/9145c1c1-91a1-4cdc-a14b-2c73744d6539) |

Looking at these three graphs yields the following R<sup>2</sup> scores:
- Lightly Active: 0.1946 = **19%**
- Fairly Active: 0.1151 = **12%**
- Very Active: 0.4445 = **44%**

This shows that the Very Active Minutes graph shows the strongest postive correlation between very active minutes and calories burned. However, 44% is still a relatively weak score.

In order to try and find a stronger correlation I aggregated the activity level minute data using the following formula:
```
=SUM(C2+(D2*2)+(E2*3))
```
This formula gives more weight to the fairly active minutes (D2) and the most weight to very active minutes (E2). Once this formula was run it yielded the graph below.
![aggregatedActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/568b3154-100c-48be-a4de-1b2028684542)
The R<sup>2</sup> Score for this relationship was **57%**. A stronger correlation than the 44% from very active minutes.
### Calories vs Steps
Looking further for different trends in the data I wanted to analyze the relationship between the total number of steps in a day and the effect it had on the amount of calories burned. The data yielded the graph below. 
![calories_vs_totalSteps](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/a2222476-bc6e-414d-b69f-29bf861af02d)
The R<sup>2</sup> score for this relationship was **31%**. Honestly, I was expecting a higher correlation score due to the CDC recommendation to get at least 10,000 steps daily. However, it now makes more sense that the updated recommendation is to get 150 minutes of moderate intensity exercise weekly. This is consistent with the correlation data I have found to this point. 

Because the data seemed messy in the visualization I averaged the calories burned per day and the average amount of steps per day by individual. The idea being that perhaps this might show a stronger relationship. The graph below shows the results.
![calories_vs_stepsByIndividual](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/53fea44f-72e8-4239-9831-8f9fc28b92e1)
The R<sup>2</sup> score for this graph was a weaker **15%**. Indicating to me that I was more right to use the raw data as opposed to the averaged data.
### Activity by Time
Finally I wanted to see the high level trend between time and steps taken. First by the day of the week, to see if people would be more active during the weekday vs the weekend. Then second by the time of the day to see at what times during the day are people most active. Below is the graph showing total steps taken by every participant by the day of the week.
![steps_by_dayofWeek](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/d7e75126-1000-4d49-9827-e47441875f2b)
The finding of interest here is that overall people are more active during the weekday than the weekend with their activity peaking on Tuesday and gradually falling off from that until Friday. The other finding is that people appeared to be more active on Saturday than on Sunday.

The graph below shows steps taken during certain times of the day. 
![steps_by_timeofDay](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/e96e1cc5-c080-4a5b-9e9b-1e851ca4c9f9)
We see 2 major peaks during lunchtime at **12:00-2:00 PM** and then the largest peak from **5:00-7:00 PM**. This tells me that people were most active during the time they were not at work.
## Recommendations
- Due to the heavier correlation between time spent active and calories burned as opposed to number of steps taken you could add reminders about CDC recommendations about exercise and active living.
  - 150 minutes of moderate physical activity a week
  - 10,000 steps a day for weight loss
- Due to the low number of people participating in the weight log, you could look into developing a smart device (scale) that could automatically upload your weight information and sync it with all the metrics uploaded into the Bellabeat app through the variety of other Bellabeat products.
  - You could even implement a design in which this information could be remotely shared with a physician, like an OBGYN or other health professional that could benefit from this information.
  - This could open up a market into advertising the Bellabeat product line for medical applications like pregnancy or eating disorders.
- Due to the low product engagement from some individuals, you could implement a notification that could remind you if you are leaving behind your smart device. Similar to the way Apple utilizes the AirTag.
  - This should result in a positive change in product engagement.  
- Since you are already tracking heart rate information through the other Bellabeat products you could train them to track your normal heart rate patterns and abnormal heart rate patterns. This can give advanced warnings about heart attacks or other heart arrhytmias.


