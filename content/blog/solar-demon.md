+++
title = 'Scraping Solar Demon'
date = 2025-01-29
description = "Retrieve a list of flares from Solar Demon as a CSV file."

[taxonomies]
tags = ["solar", "data", "programming"]

[extra]
toc = false
+++

## What is it?

Solar Demon is a tool to detect flares, dimmings, and EUV waves in SDO/AIA images.

[This is the main website](https://www.sidc.be/solardemon/).

The associated paper is ['Solar Demon - an approach to detecting flares, dimmings, and EUV waves on SDO AIA images' by Kraaikamp and Verbeeck(2015)](https://doi.org/10.1051/swsc/2015019). 

## Scraping the data

I wanted a way of converting all the data to a Pandas dataframe to export as a CSV. Here's the code:

```py
import pandas as pd
from bs4 import BeautifulSoup
import requests
from dateutil.parser import parse as parse_datetime_str

def find_month_breaks(rows):
    breaks = []
    for i, row in enumerate(rows):
        if i >= 1:
            first_td = row.find("td")
            if first_td and first_td.has_attr("colspan"):
                breaks.append(i)
    return breaks

def parse_row(rows, index):
    return [e.text for e in rows[index].find_all("td")]

if __name__ == "__main__":
    url = "https://www.sidc.be/solardemon/science/flares.php?min_seq=1&min_flux_est=0.000000001&days=0&science=1"

    page = requests.get(url)  # this takes a few minutes to run
    soup = BeautifulSoup(page.text, 'html.parser')

    rows = soup.find_all("tr")

    column_headers = ["day of month", 
                        "est. class",
                        "start", 
                        "peak", 
                        "end", 
                        "#", 
                        "lat", 
                        "lon", 
                        "dist. R☉", 
                        "AR" ,
                        "est. flux", 
                        "GOES flux",
                        "GOES peak time",
                        "extra time",
                        "COMESEP", 
                        "# det.", 
                        "dimming"]
                    
    month_frames = []
    month_breaks = find_month_breaks(rows) + [len(rows)]

    for i in range(len(month_breaks)-1):
        month_yr_str = rows[month_breaks[i]].text
        this_month_frame = pd.DataFrame([parse_row(rows, i) for i in range(month_breaks[i]+1, month_breaks[i+1])], columns=column_headers)
        this_month_frame['month'] = month_yr_str.split(",")[0]
        this_month_frame['year'] = int(month_yr_str.split(",")[1])
        month_frames.append(this_month_frame)
        
    df = pd.concat(month_frames, ignore_index=True)

    # convert the month, day of month, and year columns into one date column
    df['date'] = (df['month'] + " " + df['day of month'] + ", " + df['year'].astype(str)).map(lambda r: parse_datetime_str(r.replace("&nbsp", "")))
    df = df.drop(columns=["month", "day of month", "year"])  # drop the old columns
    new_columns = ['date', 'est. class', 'start', 'peak', 'end', '#', 'lat', 'lon', 'dist. R☉',
            'AR', 'est. flux', 'GOES flux', 'GOES peak time', 'extra time',
            'COMESEP', '# det.', 'dimming'] 
    df = df[new_columns]  # reorder the columns

    # do whatever you want with the pandas dataframe now, e.g. write it to CSV
    df.to_csv("solar_demon_data.csv")
```

Enjoy!
