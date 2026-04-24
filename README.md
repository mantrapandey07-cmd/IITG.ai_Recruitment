# IITG.ai_Recruitment

## Interpretation and Reporting

### Missing data and anomaly handling

First, I cleaned the PGCB demand data so it could be used as a proper time-series dataset. The `datetime` column was converted into actual datetime format, then the rows were sorted by time. I also removed duplicate timestamps because having more than one value for the same hour could confuse the model.

Some columns had mixed or non-numeric values, so I kept only the numeric columns. After that, the data was resampled into hourly intervals. For missing hourly values, I used nearest interpolation, which basically fills the missing hour using the closest available value.

The weather data was handled in a similar way. I converted the time column, sorted it, removed duplicates, kept numeric columns, and resampled it hourly.

For outliers in demand, I used the IQR method. Any `demand_mw` value that was too far below or above the normal range was treated as an anomaly and removed. This was done so that sudden unusual spikes or drops would not have too much effect on the model.

After cleaning, remaining missing values were filled with `0`. This is not perfect, but it keeps the dataset usable and avoids errors during merging and modelling.

### Temporal and external features engineered

Since electricity demand changes a lot depending on time, I created several time-based features. These included the hour of the day, day of the week, month, quarter, whether the day was a weekend, and whether the hour was during night time.

I also created sine and cosine versions of hour and month. This was done because time works in cycles. For example, 11 PM and 12 AM are close in real life, but simple numbers make them look far apart. The sine and cosine features help the model understand this circular pattern better.

Lag features were also added using previous demand values. For example, the model was given demand from 1 hour ago, 2 hours ago, 24 hours ago, and 168 hours ago. These are important because electricity demand usually depends strongly on what happened recently. The 24-hour lag captures the previous day’s pattern, while the 168-hour lag captures the same hour from the previous week.

Rolling average and rolling standard deviation features were also created. These help the model understand the recent trend in demand, not just one previous value. For example, a rolling average can show whether demand has generally been rising or falling over the last few hours.

Weather data was merged with the demand data using the nearest timestamp within 30 minutes. Economic indicators were merged by year, such as GDP, population, electricity access, energy use, CPI, and industry-related values. These were added to give the model some wider external context.

### Key insights from feature importance

From the feature importance plot, `generation_mw` was the most important feature. This is reasonable because electricity generation and electricity demand are closely related in the power system.

The next most important features were mostly lag features, such as previous demand from 1 hour ago, 2 hours ago, 3 hours ago, 24 hours ago, and 48 hours ago. This shows that the model depends heavily on past demand values. In simple terms, recent demand is one of the best clues for predicting the next hour’s demand.

Rolling demand features were also important. This means the model was not only looking at single past values, but also using recent average behavior and demand variation.

Some weather and fuel-related variables also appeared in the top features, but they were not as strong as the demand lag and generation features. So, weather and external variables helped the model, but the main signal still came from the historical demand pattern itself.

Time-based features like hour and hour-cycle features were also useful, which confirms that electricity demand has a clear daily pattern.

Overall, the model seems to be mainly driven by recent demand behavior, daily and weekly cycles, and generation level. Weather and economic features add extra information, but they are secondary compared to the time-series features.
