---
date: 2018-02-26
title: Dynamic weight tracking form and dashboard
commentIssueId: 9
tags: 
  - Scripts
  - Web
aliases: /blog/2018/Weight-tracker-form
---

This is a quick post, but if you want a quick way to track body weight, or any metric you choose, and have an automatically updating dashboard, this is it. I used Google Forms to easy record the information and then this information is read by a Google Apps Script that generates a chart with [Chart.js](http://www.chartjs.org/).

{{< figure src="/images/2018/02/weight-tracker/screenshot.png" caption="Example Screenshot" >}}

I roughly followed the instruction found here by Ben Collins: [Creating a d3 chart with data from Google Sheets](https://www.benlcollins.com/spreadsheets/d3-google-sheets/). The largest differences were using Chart.JS for simplicity and using time series data, since I wanted to track weight over time. Hopefully this project can serve as an example or base for whatever you'd like to do.

The general process was:

1. Create a Google Form to store the data you'd like to access, and add some of this data
2. Open up the script editor and create the following files in the Apps Script project:
    * code.gs
    * index.html
    * styles.css.html
    * main.js.html
    * chart.js.html
3. Copy and paste the code from GitHub for each of these files from the project repo: [Weight Tracker Form](https://github.com/seanlane/weight-tracker-form)
4. Publish the app to the web

Note that you'll need to modify various links and IDs within the code base, but it should be apparent when you look through each file.

Source Code: [Weight Tracker Form](https://github.com/seanlane/weight-tracker-form)