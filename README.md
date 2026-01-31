# Mobile Phone Activity Analysis - Milan CDR Data

This project looks at mobile phone activity in Milan, Italy, between November 1 and 7, 2013. The dataset includes SMS, phone calls, and internet usage, spread across 10,000 grid cells and linked to activity from 302 different countries.

## How I Approached the Analysis

### Getting the Data Ready

I downloaded the data directly from Kaggle using their API and worked in Google Colab. Instead of loading all seven days at once (which kept crashing Colab), I focused on three days: November 2, 4, and 6.

I merged the data using pandas' `merge()` function with `how='outer'` so that no records were dropped just because they didn't line up perfectly. First, I merged November 2 and 4, then merged that result with November 6. After everything was combined, I ended up with just over 6.5 million rows.

### Cleaning Things Up

Once everything was merged, I checked the structure of the data using `data.info()`. That's when it became clear that a large portion of the activity columns - smsin, smsout, callin, callout, and internet - had missing values. In some columns, more than half the data was missing.

To get a clearer picture, I calculated the percentage of missing values in each column. Most of them fell between 55% and 77%, which is a lot.

Instead of dropping those rows or replacing missing values with zeros, I filled them using the column averages. It's not perfect, but it keeps the dataset usable without completely distorting the results or throwing away millions of records.

The datetime column also needed some work since it was stored as text. I converted it into a proper datetime format and then split it into separate date, time, and hour columns. This made it much easier to look at how activity changes throughout the day.

### Creating Helpful Features

To keep things simple and readable, I created a few new columns:

* **totalsms**: incoming + outgoing SMS
* **totalcalls**: incoming + outgoing calls
* **totalactivity**: SMS plus calls

Doing this upfront meant I didn't have to keep recalculating the same values later, and it made the analysis cleaner overall.

### How I Did the Analysis

Most of the analysis relied on standard pandas and numpy operations:

* Grouping by hour to get statistics for different times of day
* Finding the busiest and quietest hours
* Separating daytime and nighttime activity with simple filters
* Using correlation to see how SMS and calls relate to each other
* Basic counts, averages, and totals

For visualisation, I used matplotlib to create simple line plots. Nothing fancy - just clear labels, grid lines, and hourly ticks so the patterns were easy to see.

### Tools Used

* **pandas** for loading, merging, cleaning, and analysing the data
* **numpy** for calculations and correlations
* **matplotlib** for plotting
* **seaborn** (imported but not used in the final version)
* **os** for basic file checks

Everything was done in Google Colab using Python 3.

## Key Decisions I Made

### Handling Missing Data

Between 60% and 77% of the activity data was missing, depending on the column.

**What I did:** Filled missing values with column averages.

**Why:** Dropping rows would've wiped out most of the dataset, and filling with zeros would've made it look like nothing was happening in large parts of the city. Since I'm focusing on overall trends across many locations, using averages was the most practical option.

**Trade-off:** This smooths out some real variation, but that's a better outcome than losing the majority of the data.

### Keeping Outliers

I initially explored outliers using boxplots but chose not to remove them.

**What I did:** Left all extreme values in the dataset.

**Why:** In a city like Milan, spikes in activity are often real - think office districts, transport hubs, or major events. Removing them would mean losing exactly the behaviour that makes the data interesting.

### Creating Aggregate Columns Early

Rather than repeatedly summing columns during analysis, I created total columns from the start.

**Why:** It made the code easier to read, reduced mistakes, and avoided unnecessary recalculations.

### Breaking Down Time

Splitting datetime into date, time, and hour made it much easier to analyse daily patterns.

**Why:** Grouping by hour or filtering for day versus night becomes straightforward, without needing complex datetime logic every time.

### Using Only Three Days

Although seven days were available, I worked with just three.

**Why:** Colab's memory limits made it impractical to load everything at once. Three days still gave me over 6.5 million records and included both weekday and weekend behaviour, which was enough to spot strong patterns.

## What the Data Shows

### When Phone Activity Peaks

* **Busiest hour:** Around noon(12pm)
* **Quietest hour:** Around 3 AM
* **Day vs night:** About 78% of activity happens between 6 AM and 8 PM, with only 22% happening overnight

This lines up neatly with normal daily routines.

### International vs Local Activity

Across all records:

* 89% involve international country codes
* 11% are domestic (Italy)
* Activity spans 302 different countries

Looking specifically at SMS:

* 75% are international
* 25% are domestic

For international calls:

* There are about 1.67 incoming calls for every outgoing one
* More calls are coming into Milan than going out

This suggests Milan acts more as a destination than a caller hub, which fits its role as a business and tourism centre.

### Daily Activity Patterns

When plotting activity by hour, both local and international calls follow the same curve:

* Activity picks up around 6-7 AM
* Peaks around lunchtime
* Gradually declines through the evening
* Hits a low point between 2 and 4 AM

The schedule is surprisingly consistent, regardless of where the call is coming from.

### Texting vs Calling

At the grid level, SMS and call activity are almost perfectly aligned. The correlation between total SMS and total calls across grid cells is **0.9862**.

In simple terms: places that text a lot also call a lot. Busy areas are busy across the board.

### A Closer Look at One Area

I zoomed in on a single grid cell (CellID = 5) and analysed its hourly activity. Even at this small scale, the same day-night pattern appeared. That helped confirm that the overall trends weren't just averaging effects - they show up at the local level too.

## Final Thoughts

Milan's mobile phone activity follows very clear and predictable daily rhythms. Usage rises in the morning, peaks around midday, and drops off overnight. The strong international presence - 302 countries and nearly 90% of records - highlights Milan's global role.

The imbalance in international calls, with far more coming into the city than going out, suggests Milan is a focal point rather than just another node in the network. Finally, the extremely high correlation between texting and calling shows that location matters more than communication type. Where people are drives how much they communicate, regardless of whether they're texting or calling.
