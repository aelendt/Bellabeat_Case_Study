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

![user_engagement](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/c7b0e4fc-dc3c-4e31-b705-052e6e85e6f6)

The above graph shows me information about the overall health of the dataset and also where things like product engagment can be improved. The average product engagement over the 31 day period was 84%.

The other main change that was needed to make to the files had to do with fixing an issue with the datetime for all the hourly and seconds tables. The format used during data collection would not properly upload for SQL analysis. This could be fixed by importing all the columns from the table as a string type and then recasting them using the following code.

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
The only table that was excluded from analysis due to low participants was weightLongInfo_merged.csv with only 8 participants.

## Analysis
### Activity Level Breakdown
Before anything else I want to get an idea for how much time was spent in at each level of intensity across all the participants.
![activity_level_breakdown](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/5d97ec9c-7ec4-4f90-8a18-0d1b16701725)
The main finding of interest was that the percentage of time spent very active was greater than the amount of time spent at a fairly active level. It is relatively expected that ~75% of the time was spent sedentary since most people that wear Fitbits are adults and a large percentage of adults work sedentary jobs.
### Active Minutes vs Calories Burned
After seeing a high level distribution of how much time was spent at each intensity level I wanted to get a better idea of how much time spent in each intensity level affected calories burned. The 3 graphs side by side show the 3 active intensities and how each affects calories burned. Relatively positive correlations are expected here.
| Lightly Active Minutes | Fairly Active Minutes | Very Active Minutes |
| ---------- | ---------- | ---------- |
| ![lightlyActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/940f9d32-b16e-4b7b-a7ec-ade4878e979e) | ![fairlyActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/545c68fa-4469-45bb-bb8e-d0495cee7cd0) | ![veryActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/69fa8657-c5ab-4cfa-a5ec-4eac2cfee865) |

Looking at these three graphs yields the following R<sup>2</sup> scores:
- Lightly Active: 0.1946 = **19%**
- Fairly Active: 0.1151 = **12%**
- Very Active: 0.4445 = **44%**

This shows that the Very Active Minutes graph shows the strongest postive correlation between very active minutes and calories burned. However, 44% is still a relatively weak score.

In order to try and find a stronger correlation I aggregated the activity level minute data using the following formula:
```
=SUM(C2+(D2*2)+(E2*3))
```
This formula gives more weight to the fairly active minutes (D2) and the most weight to very active minutes (E2). Once this formula was run it yielded the below graph.
![aggregatedActiveMinutes_vs_calories](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/5c65eaee-8294-472c-8179-40ad7e70dc01)

The R<sup>2</sup> Score for this relationship was **57%**. A stronger correlation than the 44% from very active minutes.
### Calories vs Steps
Looking further for different trends in the data I wanted to analyze the relationship between the total number of steps in a day and the effect it had on the amount of calories burned. The data yielded the graph below. 
![calories_vs_totalSteps](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/53f02593-90f0-4664-a68a-f4798f665612)

The R<sup>2</sup> score for this relationship was **31%**. Honestly I was expecting a higher correlation score due to the CDC recommendation to get at least 10,000 steps daily. However, it now makes more sense that the updated recommendation is to get 150 minutes of moderate intensity exercise weekly. This is consistent with the correlation data I have found to this point. 

Because the data seemed messy in the visualization I averaged the calories burned per day and the average amount of steps per day by individual. The idea being that perhaps this might show a stronger relationship. The graph below shows the results.
![calories_vs_stepsByIndividual](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/3d0a9e3f-57a9-48a7-a617-af4ad9252122)

The R<sup>2</sup> score for this graph was a weaker **15%**. Indicating to me that I was more right to use the raw data as opposed to the averaged data.
### Activity by Time
Finally I wanted to see the high level trend between time and steps taken. First by the day of the week, to see if people would be more active during the weekday vs the weekend. Then second by the time of the day to see at what times during the day are people most active. Below is the graph showing total steps taken by every participant by the day of the week.
![steps_by_dayofWeek](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/8332e415-6410-48fd-b6eb-1edd7fdb8af3)

The finding of interest here is that overall people are more active during the weekday than the weekend with their activity peaking on Tuesday and gradually falling off from that until Friday. The other finding is that people appeared to be more active on Saturday than on Sunday.

The graph below shows steps taken during certain times of the day. 
![steps_by_timeofDay](https://github.com/aelendt/Bellabeat-Case-Study/assets/136762105/fe478014-adca-4446-b751-dee8c260e77e)

We see 2 major peaks during lunchtime at **12:00-2:00 PM** and then the largest peak from **5:00-7:00 PM**. This tells me that people were most active during the time they were not at work.
## Recommendations
- 

































