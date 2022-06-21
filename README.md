# Checking-your-Package-for-Compatibility-with-R-4.0.0

The details of the codeset and plots are included in the attached Microsoft Word Document (.docx) file in this repository. 
You need to view the file in "Read Mode" to see the contents properly after downloading the same


=========================================================================================================================

The Data
=============
Currently, the package offers the following functions to download data:

    download_jhu_csse_covid19_data(): Downloads and tidies Covid-19 data from the Johns Hopkins University CSSE Github Repo. This data has developed to a standard resource for researchers and the general audience interested in assessing the global spreading of the virus. The data is provided at country and sub-country levels.
    download_ecdc_covid19_data(): Downloads and tidies Covid-19 case data provided by the European Centre for Disease Prevention and Control. The data is updated weekly and contains the latest available public data on the number of new Covid-19 cases reported per week and per country.
    NEW: download_owid_data(): Downloads and tidies data collected by the ‘Our World in Data’ team. This team systematically collects data on hospitalizations, testing and vaccinations from multiple national sources.
    download_acaps_npi_data(): Downloads and tidies the Government measures dataset provided by the Assessment Capacities Project (ACAPS). These data allow researchers to study the effect of non-pharmaceutical interventions on the development of the virus.
    download_oxford_npi_data(): Downloads and tidies data from the Oxford Covid-19 Government Response Tracker, an alternative data source for governmental interventions. Currently, I do not include the financial measures of this data set and also do not include its data in the merged data file (see below) as I view the ACAPS data to be better suited (this blog post details why).
    download_apple_mtr_data(): Downloads Mobility Trends Reports provided by Apple related to Covid-19. The data is provided at country and sub-country levels.
    download_google_cmr_data(): Downloads Google COVID-19 Community Mobility Reports data. As of April 17, Google provides a nice and clean CSV file containing country-day and region-day data. This makes the PDF scraping code that used to be part of this package obsolete. If you are interested in it for didactic reasons, you can still find it in the git hstory. This data is available at the country, regional and U.S. county level.
    download_google_trends_data(): Downloads and tidies Google Trends data on the search volume for the term “coronavirus” (Thank you to Yan Ouaknine for bringing up that idea!). This data can be used to assess the public attention to Covid-19 across countries and over time within a given country. The data is available at the country, regional and city level but availability varies across countries as Google Trends provides only more granular data for regions with high search volumes.
    download_wbank_data(): Downloads and tidies additional country level information provided by the World Bank using the {wbstats} package. These data allow researchers to calculate per capita measures of the virus spread and to assess the association of macro-economic variables with the development of the virus.
    download_merged_data(): Downloads all data sources and creates a merged country-day panel.

How to Use the Package
========================
The idea is simple. Load the data using the functions above and code away. So, for example:

library(tidyverse)
library(tidycovid19)
library(zoo)

df <- download_merged_data(cached = TRUE, silent = TRUE)

df %>%
  filter(iso3c == "USA") %>%
  mutate(
    new_cases = confirmed - lag(confirmed),
    ave_new_cases = rollmean(new_cases, 7, na.pad=TRUE, align="right")
  ) %>%
  filter(!is.na(new_cases), !is.na(ave_new_cases)) %>%
  ggplot(aes(x = date)) +
  geom_bar(aes(y = new_cases), stat = "identity", fill = "lightblue") +
  geom_line(aes(y = ave_new_cases), color ="red") +
  theme_minimal()

The data comes with two meta data sets that describe the data. The data frame tidycovid19_data_sources provides short descriptions and links for each data source used by the package. The data frame tidycovid19_variable_defintions provides variable definitions for each variable included in the merged country-day data frame provided by download_merged_data():

df <- tidycovid19_variable_definitions %>%
  select(var_name, var_def)
kable(df) %>% kableExtra::kable_styling()


![image](https://user-images.githubusercontent.com/26252963/133880542-48d85e9d-fca5-4c7a-80b5-4cb61f03e365.png)



Visualization
===============
The focus of the package lies on data collection and not on visualization as there are already many great tools floating around. Regardless, there are three functions that allow you to visualize some of the key data that the package provides.
Plot Covid-19 Spread over Event Time

The function plot_covid19_spread() allows you to quickly visualize the spread of the virus in relation to governmental intervention measures. It is inspired by the insightful displays created by John Burn-Murdoch from the Financial Times and offers various customization options.



library(tidycovid19)

merged <- download_merged_data(cached = TRUE, silent = TRUE)
plot_covid19_spread(
  merged, highlight = c("ITA", "ESP", "GBR", "FRA", "DEU", "USA", "BRA", "MEX"),
  intervention = "lockdown", edate_cutoff = 330
)

![image](https://user-images.githubusercontent.com/26252963/133880599-4dd778c2-4b16-4428-965c-5cf0ad2e1a7b.png)


Plot Covid-19 Stripes
========================
Another option to visualize the spread of Covid-19, in particular if you want to compare many countries, is to produce a stripes-based visualization. Meet the Covid-19 stripes:

plot_covid19_stripes()

Again, the function comes with many options. As an example, you can easily switch to a per capita display:

plot_covid19_stripes(
  per_capita = TRUE, 
  population_cutoff = TRUE, 
  sort_countries = "magnitude"
)

![image](https://user-images.githubusercontent.com/26252963/133880629-6d8547e8-2791-4307-8040-8e6f73edf052.png)


Or single out countries that you are interested in

plot_covid19_stripes(
  type = "confirmed", 
  countries = c("ITA", "ESP", "FRA", "GBR", "DEU", "USA", "BRA", "MEX"),
  sort_countries = "countries"
)

![image](https://user-images.githubusercontent.com/26252963/133880645-2f29c6a7-48e8-4890-ae4f-7936d6553876.png)


Map Covid-19
=============

Finally, as Covid-19 has become a truly world-wide pandemic, I decided to also include a basic mapping function. map_covid19() allows you to map the spread of the virus at a certain date both world-wide …

map_covid19(merged, cumulative = TRUE)

… or for certain regions.

map_covid19(merged, type = "confirmed", region = "Europe") 

If you have enough time (takes several minutes), you can also create an animation to visualize the spread of the virus over time.

map_covid19(
  merged, type = "confirmed", per_capita = TRUE, dates = unique(merged$date)
)

Again, you can customize the data that you want to plot and of course you can also modify the plot itself by using normal ggplot syntax.

Shiny App
=============
Sorry, I could not resist. The options of the plot_covid19_spread() make the implementation of a shiny app a little bit to tempting to pass. The command shiny_covid19_spread() starts the app. Click on the image to be taken to the online app. You can use it to customize your plot_covid19_spread() display as it allows copying the plot generating code to the clipboard, thanks to the fine {rclipboard} package. You can now also customize the app by providing plot_covid19_spread() options as a list to the plot_options parameter.



As the shinyapps.io server has had some issues with exhausting connections, 

pkgdown.yml
================

template:
  params:
    ganalytics: UA-119053376-1
development:
  mode: auto
navbar:
  type: default
  left:
  - icon: fa-home fa-lg
    href: index.html
  - text: Reference
    href: reference/index.html
  - text: News
    href: news/index.html
  right:
  - text: Code repository
    icon: fa-github
    

reference:
- title: Data functions
  desc: Functions to download and tidy Covid-19 related data from authorative sources
  contents:
  - download_jhu_csse_covid19_data
  - download_ecdc_covid19_data
  - download_owid_testing_data
  - download_acaps_npi_data
  - download_oxford_npi_data
  - download_apple_mtr_data
  - download_google_cmr_data
  - download_google_trends_data
  - download_wbank_data
  - download_merged_data
- title: Visualization functions
  desc: Customizable functions to display the spread of Covid-19
  contents:
  - plot_covid19_spread
  - plot_covid19_stripes
  - map_covid19
- title: Shiny app
  desc: App for interactive customization of Covid 19 spread plots
  contents:
  - shiny_covid19_spread
- title: Data sets
  desc: Dats sets describing the data source and the variables contained in the merged data frame.
