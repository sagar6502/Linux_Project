# Linux_Project
# Practice Project Overview
Scenario
You've been tasked by your team to create an automated Extract, Transform, Load (ETL) process to extract daily weather forecast and observed weather data and load it into a live report to be used for further analysis by the analytics team. As part of a larger prediction modelling project, the team wants to use the report to monitor and measure the historical accuracy of temperature forecasts by source and station.
As a proof-of-concept (POC), you are only required to do this for a single station and one source to begin with. For each day at noon (local time), you will gather both the actual temperature and the temperature forecasted for noon on the following day for Casablanca, Morocco.
At a later stage, the team anticipates extending the report to include lists of locations, different forecasting sources, different update frequencies, and other weather metrics such as wind speed and direction, precipitation, and visibility.
Data source
For this practice project, you'll use the weather data package provided by the open source project wttr.in, a web service that provides weather forecast information in a simple and text-based format. For further information, you can read more about the service on its GitHub Repo.
First, you'll use the curl command to scrape weather data via the wttr.in website. For example, to get data for Casablanca, enter:
1
curl wttr.in/casablanca
Copied!
which prints the following to stdout:





1.1 Create a text file called rx_poc.log.
1.2 Add a header to your weather report.
echo -e "year\tmonth\tday\thour\tobs_tmp\tfc_temp">rx_poc.log

year    month   day hour    obs_tmp fc_temp
#! /bin/bash
chmod u+x rx_poc.sh
2.1. Create a text file called rx_poc.sh and make it a bash script.
Remember to make it executable.
Click here for Hint 1
Click here for Hint 2
Click here for Solution 1
Include the bash ‘shebang’ on the first line of rx_poc.sh:
1
#! /bin/bash
Copied!
{: codeblock}
Click here for Solution 2
Make your script executable:
1
chmod u+x rx_poc.sh
Copied!
{: codeblock}
Verify your changes using the ls command with the -l option.
2.2 Download today’s weather report from wttr.in
Tip: It’s good practice to keep a copy of the raw data you are using to extract the data you need.
By appending a date or time stamp to the file name, you can ensure it’s name is unique.
This builds a history of the weather forecasts which you can revisit at any time to recover from errors or expand the scope of your reports
Using the prescribed date format ensures that when you sort the files, they will be sorted chronologically. It also enables searching for the report for any given date.
If needed, you can compress and archive the files periodically. You can even automate this process by scheduling it.
Follow the steps below to download and save your report as a datestamped file named raw_data_<YYYYMMDD>, where <YYYYMMDD> is today’s date in Year, Month, Day format.
2.2.1 Create the filename for today’s wttr.in weather report
Click here for Hint 1> Use the `date` command with the correct formatting option.Click here for Hint 2> Use command substitution to save the date to a variable called `today`.Click here for Hint 3> Include the variable's value in the name of the file.Click here for Solution to Hints 1 and 2
1
today=$(date +%Y%m%d)
Copied!
{: codeblock}
Click here for full Solution
1
2
today=$(date +%Y%m%d)
weather_report=raw_data_$today
Copied!
{: codeblock}
OR, more directly:
1
weather_report=raw_data_$(date +%Y%m%d)
Copied!
{: codeblock}
2.2.2 Download the wttr.in weather report for Casablanca and save it with the filename you created
Click here for Hint> Use the `curl` command with the `--output`` option.Click here for Solution
1
2
city=Casablanca
curl wttr.in/$city --output $weather_report
Copied!
{: codeblock}
2.3. Extract the required data from the raw data file and assign them to variables obs_tmp and fc_temp.
Extracting the required data is a process that will take some trial and error until you get it right.
Study the weather report you downloaded, and determine what you need to extract. Look for patterns.
You must find a way to ‘chip away’ at the weather report:
Use shell commands to extract only the data you need (the signal)
Filter everything else out (the noise)
Combine your filters into a pipeline (recall the use of pipes to chain filters together)
Tip: We introduce three more simple filters here that you will find very useful for your data extraction strategy.
tr - trim repeated characters to a single character.
For example, to remove extra spaces from text:
1
echo "There are    too    many spaces in this    sentence." | tr -s " "
Copied!
{: codeblock}
xargs - can be used to trim leading and trailing spaces from a string.
For example, to remove the spaces from the begining and the end of text:
1
echo " Never start or end a sentence with a space. " | xargs
Copied!
rev - reverse the order of characters on a line of text.
Try entering the following commmand:
1
echo ".sdrawkcab saw ecnetnes sihT" | rev
Copied!
{: codeblock}
You will find rev to be a useful operation to apply in combination with the cut command.
For example, suppose you want to access the last field in a delimited string of fields:
1
2
# print the last field of the string
echo "three two one" | rev | cut -d " " -f 1 | rev
Copied!
You will also find xargs to be a useful operation to apply in combination with the cut command.
For example, suppose you want to access the last word in a sentence as above, but there happens to be an extra space at the end:

# Unfortunately, this prints the last field of the string, which is empty:
echo "three two one " | rev | cut -d " " -f 1 | rev
# But if you trim the trailing space first, you get the expected result:
echo "three two one " | xargs | rev | cut -d " " -f 1 | rev
Copied!
Let’s now return to extracting the temperature data of interest.
Click here for a Hint to get started
Click here for another Hint to get started
Click here for Solution
1
grep °C $weather_report > temperatures.txt
Copied!
{: codeblock}
2.3.1. Extract the current temperature, and store it in a shell variable called obs_tmp

Here’s the full solution. Ensure you understand what each filter accomplishes.
Using the command line, try adding one filter at a time to see what the outcome is as you develop the pipeline.
1
obs_tmp=$(head -1 temperatures.txt | tr -s " " | xargs | rev | cut -d " " -f2 | rev)
Copied!
{: codeblock}
2.3.2. Extract tomorrow’s temperature forecast for noon, and store it in a shell variable called fc_tmp
Click here for Hint
Click here for Solution
1
fc_temp=$(head -3 temperatures.txt | tail -1 | tr -s " " | xargs | cut -d "C" -f2 | rev | cut -d " " -f2 | rev)
Copied!
{: codeblock}
Actually, the use of xargs may not be necessary here but it does no harm since it only removes spaces if they exist!
2.4. Store the current hour, day, month, and year in corresponding shell variables

hour=$(TZ='Morocco/Casablanca' date -u +%H)
day=$(TZ='Morocco/Casablanca' date -u +%d)
month=$(TZ='Morocco/Casablanca' date +%m)
year=$(TZ='Morocco/Casablanca' date +%Y)
Copied!
{: codeblock}
Tip: You might be wondering why we didn’t just set the hour to be 12. After all, noon is the time we are expecting.
But then we would lose the ability to verify that we are actually running the code at the correct local time.
Threrfore, you should think of the local time as a measurement.
2.5. Merge the fields into a tab-delimited record, corresponding to a single row in Table 1.
Append the resulting record as a row of data to your weather log file.
Click here for Hint
Click here for Solution
1
2
record=$(echo -e "$year\t$month\t$day\t$hour\t$obs_tmp\t$fc_temp")
echo $record>>rx_poc.log
Copied!
{: codeblock}
PreviousNext


3.1. Determine what time of day to run your script
Recall that you want to load the weather data coresponding to noon, local time in Casablanca, every day.
Check the time difference between your system’s default time zone and UTC.
Click here for Hint 1
Click here for Hint 2
Running the following commands gives you the info you need to get the time difference between your system and UTC.

$ date
Mon Feb 13 11:28:12 EST 2023
$ date -u
Mon Feb 13 16:28:16 UTC 2023
Copied!
Now you can determine how many hours difference there are between your system’s local time and that of Casablanca.
Click here for Solution
From the example above, we see that the system time relative to UTC is UTC+5 (i.e., 16 - 11 = 5).
We know Casablanca is UTC+1, so the system time relative to Casablanca is 4 hours earlier. Thus, to run your script at noon Casablanca time, you need to run it at 8 am.
3.2 Create a cron job that runs your script
Click here for Hint
Click here for Solution
Edit the crontab file
1
crontab -e
Copied!
Add the following line at the end of the file:
1
0 8 * * * /home/project/rx_poc.sh
Copied!
Save the file and quit the editor.
Full solution
For reference, here is a bash script that creates the required weather report.
Try not to peek until after you have attempted to work through every step!

#! /bin/bash


# create a datestamped filename for the raw wttr data:
today=$(date +%Y%m%d)
weather_report=raw_data_$today


# download today's weather report from wttr.in:
city=Casablanca
curl wttr.in/$city --output $weather_report


# use command substitution to store the current day, month, and year in corresponding shell variables:
hour=$(TZ='Morocco/Casablanca' date -u +%H)
day=$(TZ='Morocco/Casablanca' date -u +%d)
month=$(TZ='Morocco/Casablanca' date +%m)
year=$(TZ='Morocco/Casablanca' date +%Y)


# extract all lines containing temperatures from the weather report and write to file
grep °C $weather_report > temperatures.txt


# extract the current temperature
obs_tmp=$(head -1 temperatures.txt | tr -s " " | xargs | rev | cut -d " " -f2 | rev)


# extract the forecast for noon tomorrow
fc_temp=$(head -3 temperatures.txt | tail -1 | tr -s " " | xargs | cut -d "C" -f2 | rev | cut -d " " -f2 |rev)


# create a tab-delimited record
# recall the header was created as follows:
# header=$(echo -e "year\tmonth\tday\thour_UTC\tobs_tmp\tfc_temp")
# echo $header>rx_poc.log


record=$(echo -e "$year\t$month\t$day\t$obs_tmp\t$fc_temp")
# append the record to rx_poc.log
echo $record>>rx_poc.log
Copied!
Congratulations
Well done! You’ve just completed a challenging real world practice project using many of the concepts you’ve learned from this course. The knowledge you’ve gained has prepared you to solve many practcal real world problems! You are almost there, the final step in your journey is to complete the peer-reviewed Final Project.
Authors
Jeff Grossman
Other Contributors
Rav Ahuja
© IBM Corporation. All rights reserved.
Previous






Weather report: Bangalore

  [38;5;226m   \  /[0m       Partly cloudy
  [38;5;226m _ /""[38;5;250m.-.    [0m [38;5;208m+33[0m([38;5;202m34[0m) °C[0m     
  [38;5;226m   \_[38;5;250m(   ).  [0m [1m↘[0m [38;5;226m15[0m km/h[0m      
  [38;5;226m   /[38;5;250m(___(__) [0m 8 km[0m           
                0.0 mm[0m         
                                                       ┌─────────────┐                                                       
┌──────────────────────────────┬───────────────────────┤  Fri 31 May ├───────────────────────┬──────────────────────────────┐
│            Morning           │             Noon      └──────┬──────┘     Evening           │             Night            │
├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
│ [38;5;226m    \   /    [0m Sunny          │ [38;5;226m    \   /    [0m Sunny          │ [38;5;226m   \  /[0m       Partly Cloudy  │ [38;5;226m _`/""[38;5;250m.-.    [0m Patchy rain ne…│
│ [38;5;226m     .-.     [0m [38;5;214m+28[0m([38;5;214m30[0m) °C[0m     │ [38;5;226m     .-.     [0m [38;5;208m+32[0m([38;5;202m34[0m) °C[0m     │ [38;5;226m _ /""[38;5;250m.-.    [0m [38;5;208m+31[0m([38;5;208m32[0m) °C[0m     │ [38;5;226m  ,\_[38;5;250m(   ).  [0m [38;5;214m+29[0m([38;5;208m31[0m) °C[0m     │
│ [38;5;226m  ― (   ) ―  [0m [1m↘[0m [38;5;214m21[0m-[38;5;208m24[0m km/h[0m   │ [38;5;226m  ― (   ) ―  [0m [1m↘[0m [38;5;220m17[0m-[38;5;214m20[0m km/h[0m   │ [38;5;226m   \_[38;5;250m(   ).  [0m [1m↘[0m [38;5;226m15[0m-[38;5;214m22[0m km/h[0m   │ [38;5;226m   /[38;5;250m(___(__) [0m [1m↑[0m [38;5;220m16[0m-[38;5;214m21[0m km/h[0m   │
│ [38;5;226m     `-’     [0m 10 km[0m          │ [38;5;226m     `-’     [0m 10 km[0m          │ [38;5;226m   /[38;5;250m(___(__) [0m 10 km[0m          │ [38;5;111m     ‘ ‘ ‘ ‘ [0m 10 km[0m          │
│ [38;5;226m    /   \    [0m 0.0 mm | 0%[0m    │ [38;5;226m    /   \    [0m 0.0 mm | 0%[0m    │               0.0 mm | 0%[0m    │ [38;5;111m    ‘ ‘ ‘ ‘  [0m 0.0 mm | 61%[0m   │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
                                                       ┌─────────────┐                                                       
┌──────────────────────────────┬───────────────────────┤  Sat 01 Jun ├───────────────────────┬──────────────────────────────┐
│            Morning           │             Noon      └──────┬──────┘     Evening           │             Night            │
├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
│ [38;5;226m    \   /    [0m Sunny          │ [38;5;226m    \   /    [0m Sunny          │ [38;5;226m _`/""[38;5;240;1m.-.    [0m Moderate rain …│ [38;5;226m _`/""[38;5;240;1m.-.    [0m Moderate or he…│
│ [38;5;226m     .-.     [0m [38;5;214m+28[0m([38;5;214m30[0m) °C[0m     │ [38;5;226m     .-.     [0m [38;5;208m+32[0m([38;5;202m34[0m) °C[0m     │ [38;5;226m  ,\_[38;5;240;1m(   ).  [0m [38;5;220m+27[0m([38;5;214m29[0m) °C[0m     │ [38;5;226m  ,\_[38;5;240;1m(   ).  [0m [38;5;220m+25[0m([38;5;220m27[0m) °C[0m     │
│ [38;5;226m  ― (   ) ―  [0m [1m→[0m [38;5;226m15[0m-[38;5;220m17[0m km/h[0m   │ [38;5;226m  ― (   ) ―  [0m [1m→[0m [38;5;226m15[0m-[38;5;220m17[0m km/h[0m   │ [38;5;226m   /[38;5;240;1m(___(__) [0m [1m→[0m [38;5;190m10[0m-[38;5;226m15[0m km/h[0m   │ [38;5;226m   /[38;5;240;1m(___(__) [0m [1m↖[0m [38;5;220m17[0m-[38;5;196m33[0m km/h[0m   │
│ [38;5;226m     `-’     [0m 10 km[0m          │ [38;5;226m     `-’     [0m 10 km[0m          │ [38;5;21;1m   ‚‘‚‘‚‘‚‘  [0m 8 km[0m           │ [38;5;21;1m   ‚‘‚‘‚‘‚‘  [0m 7 km[0m           │
│ [38;5;226m    /   \    [0m 0.0 mm | 0%[0m    │ [38;5;226m    /   \    [0m 0.0 mm | 0%[0m    │ [38;5;21;1m   ‚’‚’‚’‚’  [0m 5.3 mm | 100%[0m  │ [38;5;21;1m   ‚’‚’‚’‚’  [0m 5.0 mm | 100%[0m  │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
                                                       ┌─────────────┐                                                       
┌──────────────────────────────┬───────────────────────┤  Sun 02 Jun ├───────────────────────┬──────────────────────────────┐
│            Morning           │             Noon      └──────┬──────┘     Evening           │             Night            │
├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
│ [38;5;226m    \   /    [0m Sunny          │ [38;5;226m _`/""[38;5;250m.-.    [0m Thundery outbr…│ [38;5;226m _`/""[38;5;240;1m.-.    [0m Moderate rain …│ [38;5;240;1m     .-.     [0m Torrential rai…│
│ [38;5;226m     .-.     [0m [38;5;220m+27[0m([38;5;214m29[0m) °C[0m     │ [38;5;226m  ,\_[38;5;250m(   ).  [0m [38;5;208m+31[0m([38;5;208m33[0m) °C[0m     │ [38;5;226m  ,\_[38;5;240;1m(   ).  [0m [38;5;220m+26[0m([38;5;214m28[0m) °C[0m     │ [38;5;240;1m    (   ).   [0m [38;5;190m21[0m °C[0m          │
│ [38;5;226m  ― (   ) ―  [0m [1m→[0m [38;5;226m14[0m-[38;5;220m17[0m km/h[0m   │ [38;5;226m   /[38;5;250m(___(__) [0m [1m→[0m [38;5;226m13[0m-[38;5;226m15[0m km/h[0m   │ [38;5;226m   /[38;5;240;1m(___(__) [0m [1m↑[0m [38;5;190m10[0m-[38;5;220m17[0m km/h[0m   │ [38;5;240;1m   (___(__)  [0m [1m↖[0m [38;5;220m17[0m-[38;5;196m32[0m km/h[0m   │
│ [38;5;226m     `-’     [0m 10 km[0m          │ [38;5;228;5m    ⚡[38;5;111;25m‘‘[38;5;228;5m⚡[38;5;111;25m‘‘ [0m 9 km[0m           │ [38;5;21;1m   ‚‘‚‘‚‘‚‘  [0m 8 km[0m           │ [38;5;21;1m  ‚‘‚‘‚‘‚‘   [0m 3 km[0m           │
│ [38;5;226m    /   \    [0m 0.0 mm | 0%[0m    │ [38;5;111m    ‘ ‘ ‘ ‘  [0m 0.0 mm | 0%[0m    │ [38;5;21;1m   ‚’‚’‚’‚’  [0m 5.4 mm | 100%[0m  │ [38;5;21;1m  ‚’‚’‚’‚’   [0m 8.5 mm | 100%[0m  │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
Location: Bengaluru, Bangalore Urban, Karnataka, 560001, India [12.9791198,77.5912997]

Follow [46m[30m@igor_chubin[0m for wttr.in updates



year    month   day hour    obs_tmp fc_temp
2024 05 31 12 [38;5;208m+33[0m([38;5;202m34[0m) [38;5;208m+32[0m([38;5;202m34[0m)




#! /bin/bash


# create a datestamped filename for the raw wttr data:
today=$(date +%Y%m%d)
weather_report=raw_data_$today


# download today's weather report from wttr.in:
city=Casablanca
curl wttr.in/$city --output $weather_report


# use command substitution to store the current day, month, and year in corresponding shell variables:
hour=$(TZ='Morocco/Casablanca' date -u +%H)
day=$(TZ='Morocco/Casablanca' date -u +%d)
month=$(TZ='Morocco/Casablanca' date +%m)
year=$(TZ='Morocco/Casablanca' date +%Y)


# extract all lines containing temperatures from the weather report and write to file
grep °C $weather_report > temperatures.txt


# extract the current temperature
obs_tmp=$(head -1 temperatures.txt | tr -s " " | xargs | rev | cut -d " " -f2 | rev)


# extract the forecast for noon tomorrow
fc_temp=$(head -3 temperatures.txt | tail -1 | tr -s " " | xargs | cut -d "C" -f2 | rev | cut -d " " -f2 |rev)


# create a tab-delimited record
# recall the header was created as follows:
# header=$(echo -e "year\tmonth\tday\thour_UTC\tobs_tmp\tfc_temp")
# echo $header>rx_poc.log


record=$(echo -e "$year\t$month\t$day\t$obs_tmp\t$fc_temp")
# append the record to rx_poc.log
echo $record>>rx_poc.log


  [38;5;226m _ /""[38;5;250m.-.    [0m [38;5;208m+33[0m([38;5;202m34[0m) °C[0m     
│ [38;5;226m     .-.     [0m [38;5;214m+28[0m([38;5;214m30[0m) °C[0m     │ [38;5;226m     .-.     [0m [38;5;208m+32[0m([38;5;202m34[0m) °C[0m     │ [38;5;226m _ /""[38;5;250m.-.    [0m [38;5;208m+31[0m([38;5;208m32[0m) °C[0m     │ [38;5;226m  ,\_[38;5;250m(   ).  [0m [38;5;214m+29[0m([38;5;208m31[0m) °C[0m     │
│ [38;5;226m     .-.     [0m [38;5;214m+28[0m([38;5;214m30[0m) °C[0m     │ [38;5;226m     .-.     [0m [38;5;208m+32[0m([38;5;202m34[0m) °C[0m     │ [38;5;226m  ,\_[38;5;240;1m(   ).  [0m [38;5;220m+27[0m([38;5;214m29[0m) °C[0m     │ [38;5;226m  ,\_[38;5;240;1m(   ).  [0m [38;5;220m+25[0m([38;5;220m27[0m) °C[0m     │
│ [38;5;226m     .-.     [0m [38;5;220m+27[0m([38;5;214m29[0m) °C[0m     │ [38;5;226m  ,\_[38;5;250m(   ).  [0m [38;5;208m+31[0m([38;5;208m33[0m) °C[0m     │ [38;5;226m  ,\_[38;5;240;1m(   ).  [0m [38;5;220m+26[0m([38;5;214m28[0m) °C[0m     │ [38;5;240;1m    (   ).   [0m [38;5;190m21[0m °C[0m          │





