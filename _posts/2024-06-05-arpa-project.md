Weather, air and water pollution, and other data from ARPA Lombardy 
----------------------------------------------

## Data Gathering

#### Weather, air and water pollution, and other data from ARPA Lombardy 

<a href="https://dati.lombardia.it/stories/s/Meteo-inquinamento-aria-e-acqua-e-altri-dati-da-AR/auv9-c2sj/"><img src="https://raw.githubusercontent.com/Data-Analytics-Boolean/assets/main/arpa/arpa_logo.jpg" alt="ARPA Website - Data" width=200 /></a>

Several datasets concerning weather monitoring activities and air and water pollution levels in Lombardy (Italy) have been published on the [website](https://dati.lombardia.it/stories/s/Meteo-inquinamento-aria-e-acqua-e-altri-dati-da-AR/auv9-c2sj/) of ARPA Lombardy - the Regional Agency for the Protection of the Environment.

This is a considerable amount of data, as it covers the history of several decades' worth of readings - up to the present day - of all the survey probes on the territory, as well as some interpolations made directly by ARPA.

#### The data from 2018 to present

ARPA Lombardia's air quality detection network consists of fixed stations that, by means of automatic analyzers, provide continuous data at regular time intervals. The pollutant species monitored continuously are NOX, SO2, CO, O3, PM10, PM2.5 and benzene. 

At [this page](https://www.dati.lombardia.it/Ambiente/Dati-sensori-aria-dal-2018/g2hp-ar79/about_data) you can find the air quality data from 2018 to the present day. You can preview the dataset by clicking on the `Dati` tab at the top of the screen. Notice that the entire dataset contains 5 columns and well over 17 million rows. 

<img src="https://raw.githubusercontent.com/Data-Analytics-Boolean/assets/main/arpa/arpa_website_01.png" alt="ARPA Website - Data" width=600 />

#### Accessing the data via API

As you can see from the screenshot below, if you click on the button `Azione`, a list of options to access, visualize and share the data will popup. 

<img src="https://raw.githubusercontent.com/Data-Analytics-Boolean/assets/main/arpa/arpa_website_02.png" alt="ARPA Website - API" width=600 />

If you click on the `API` option, a new window will appear, allowing you to choose the data format (make sure you select CSV) and access the API endpoint (ie: the URL), which you can copy to your clipboard using the dedicated button (or by copying it manually). 

<img src="https://raw.githubusercontent.com/Data-Analytics-Boolean/assets/main/arpa/arpa_website_03.png" alt="ARPA Website - CSV" width=600 />

#### API parameters

You can already start testing the API, if you paste the link in your browser and hit return, you’ll effectively make the API call to the service and get back the data (you’ll find the data in the downloads folder). However, if you open the csv file, you’ll notice that the dataset only has 1000 rows, quite far from the 17 million rows we previously saw. 

In order [to retrieve more rows](https://support.socrata.com/hc/en-us/articles/202949268-How-to-query-more-than-1000-rows-of-a-dataset), we can take advantage of some additional parameters that we can add to the URL: 
- the **limit parameter** '$limit=' allows you to set a limit on how many rows you query from your dataset. You can set it to a max limit of 50,000 records. 
- the **offset parameter** '$offset=' allows you to offset your results; for instance, if you set it to 1000, it will skip the first 1000 records. 

You can **combine the limit and offset parameters** to cycle through and download the entire dataset. 

For example, the following endpoint downloads the first 50k rows (setting an offset to 0 is equivalent to having no offset): 

`https://www.dati.lombardia.it/resource/g2hp-ar79.csv?$limit=50000&$offset=0`

The following endpoint will get the following 50k records (setting the offset parameter to 50,000 means skip the first 50k records): 

`https://www.dati.lombardia.it/resource/g2hp-ar79.csv?$limit=50000&$offset=50000`

#### Your first task

Your first task is to download the data from the following two pages: 
- Sensor data since 2018: [link](https://www.dati.lombardia.it/Ambiente/Dati-sensori-aria-dal-2018/g2hp-ar79/about_data)
- Sensor IDs Lookup Table: [link](https://www.dati.lombardia.it/Ambiente/Stazioni-qualit-dell-aria/ib47-atvt/about_data)

If you're up to the challenge, feel free to dive straight into the task without reading the following section. Viceversa, if you're stuck or if you need a starting point, then keep reading the following section, which gives you some hints on how to download the data (especially for the first link). 

#### Instructions to download the data

If we parametrise the offset we can initialise it at zero and keep increasing it by increments of 50k, while we loop through the whole dataset and retrieve every single record with multiple API calls. Below you'll find a template that you'll need to complete to retrieve all the data. 

> Note: it may take a while - over 10 minutes - to download the millions of rows from the source repository.

# Incomplete template code to download the data 

<!-- -->
    url = "https://www.dati.lombardia.it/resource/g2hp-ar79.csv "
    list_data = [] # initialise an empty list where you
    offset_n = 0   # initialise the offset parameter to zero
    large_number = ... # replace ... with a large number (it should be larger than the rows in the source dataset)

    while offset_n < large_number: 
        new_url = ...   # cunstruct the url: concatenate your base url with "?$limit=50000&$offset=offset_n"
        temp_df = ...   # get data from url endpoint and save it to a temporary dataframe (hint: read_csv() can get data from a URL)
        # ... at each cycle, append the temp_df to list_data
        # ... at the end of each cycle, increment offset_n by 50k

At this point, you'll have a series of DataFrames stacked up into your list `list_data`, so you'll need to concatenate those DataFrames in a new DataFrame object that should look like the one in the screenshot below. You should also save your data locally to a csv file. 

<img src="https://raw.githubusercontent.com/Data-Analytics-Boolean/assets/main/arpa/arpa_dataset_01.png" alt="ARPA dataset" width=500/>

The data you just downloaded should have 5 columns, whose content is summarized below: 

| Column Name | Description | Data Type |
| -------- | -------- | --------|
| idSensore | Unique identifier for the sensor | text |
| Data | Date and time | datetime |
| Valore<sup>1</sup> | Value detected by the sensor | number |
| Stato<sup>2</sup> | `VA` = valid data; `NA` = invalid data | text |
| idOperatore | `1` = Mean value | text |

Notes: 
1. *`-9999` = invalid data. The absence of a record indicates that the data is not available*
2. *The data in this archive from the previous year still contain uncertain values that may change as a result of validation processes based on statistical analysis of the measured series. The data validation process includes a final evaluation phase that is completed by 3/30 of the year following the year of measurement. Therefore, prior to that date, data should be considered non-final.*

As you can see, the variable `idSensore` is just a code, so we don't know yet what kind of sensor each value is referring to. To find out, we'll need to **enrich our dataset** with more data. At [this page](https://www.dati.lombardia.it/Ambiente/Dati-sensori-aria-NRT/ykhg-b8rs/about_data) you'll find another dataset from ARPA containing a lookup table with a whole set of information on each `idSensore`. With less than 1000 rows, this table is much smaller, so you can easily download the data using the URL of the API without worrying about limits or offsets. 

Once you've retrieved and saved the data, your dataframe should look like the one in the following screenshot. 

<img src="https://raw.githubusercontent.com/Data-Analytics-Boolean/assets/main/arpa/arpa_dataset_02.png" alt="Sensor Data"/>

As you have probably already guessed, the next step will be to join the two DataFrames using `idSensore` as the common key. This will give you the final DataFrame with which you'll be working for the rest of the project. 


## Data Analysis

This project focuses on identifying the optimal location and operation times for an Air Purifier in the Italian region of Lombardy. 

## Introduction

You are part of a governmental agency of the Lombardy region, Italy. Your team was tasked with deploying a highly efficient but costly and energy-intensive air purifier (such as [this one](https://www.purevento.com/en/purevento-city-air-cleaner/)) to improve urban air quality. Given the device's operational costs, **it must be activated only for a few critical hours each day**. 

Your job is to analyze air quality data to **determine the most effective location and time** for the air purifier's operation, focusing on maximizing its impact on public health and air quality.

## Considerations for the Project Objective

### The data source

You will be using air quality data collected by [ARPA Lombardy](https://dati.lombardia.it/stories/s/Meteo-inquinamento-aria-e-acqua-e-altri-dati-da-AR/auv9-c2sj/) - the Regional Agency for the Protection of the Environment. 

You will need to analyse some historical data (at least the past 5 years) to determine the existence of any trends, but ultimately your final decision should be informed by the latest available data (past year). 

### Focusing on Key Pollutants

Limiting the study to certain key pollutants can make the analysis more manageable and relevant. Also, most outdoor air purifiers are designed to be effective on specific air pollutants. Consider focusing on:
- Particulate matter (PM2.5 and PM10) due to their significant health impacts.
- Nitrogen Dioxide (NO2) and Ozone (O3) as they are common in urban environments and have immediate health effects.
- Any other pollutant that is particularly relevant to the region being studied, such as Sulfur Dioxide (SO2) in areas near industrial facilities.

You can choose one or more of the above pollutants from the list above for your analysis. 

### A framework for your analysis

It is useful to have a framework to refer to when embarking in such a venture and the [CRISP-DM](https://en.wikipedia.org/wiki/Cross-industry_standard_process_for_data_mining) guideline is usually a good reference to have in mind. 

<img src="https://raw.githubusercontent.com/Data-Analytics-Boolean/assets/main/data-analytics-cycle-boolean.png" alt="Data analytics lifecycle" width=650 />

Here is how we can adapt the Data Analytics Lifecycle to our specific problem:

1. **Objective**: your job is to analyze air quality data to **determine the most effective location and time** for the air purifier's operation, focusing on maximizing its impact on public health and air quality.
2. **Data Understanding**: make sure you familiarise yourself with the data 
3. **Data Processing**: the dataset is quite big, so after you have imported, cleaned and enriched your data, you will need to perform a preliminary analysis to determine how much data to retain
    - do you need invalid sensor readings?
    - how many years of data do you need?
    - do you need all the variables in your dataset
    - which key pollutants should you keep in your dataset for your analysis? 
4. **Data Analysis**: here are some ideas for your exploratory data analysis
    - *Seasonal Variation Analysis*: analyze how seasonal changes affect pollutant levels and determine if the purifier's operation should be adjusted seasonally.
    - *Geographical Analysis*: Use the data to identify the most polluted areas. GIS (Geographic Information Systems) tools may be helpful in this step for visualizing pollution hotspots.
    - *Operational Time Analysis*: Determine when pollutant levels are consistently at their highest to propose operational hours for the air purifier.
5. **Data Presentation**: your final presentation should include your methodology for choosing the location and time, supported by data visualizations and a discussion on the expected impact of the air purifier's operation.

## Final considerations

You are free to include any additional data that may help you in your analysis and there are no limits to the software, tools or algorithms that you can use. 