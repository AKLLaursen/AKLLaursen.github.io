---
layout: post
author: andreas
---
Yesterday, I finished hosting a short two-day workshop focused on [R](https://www.r-project.org/) and [RStudio](https://www.rstudio.com/), which I had prepared for my coworkers at my [job](http://www.nationalbanken.dk/en/about_danmarks_nationalbank/organisation/Pages/Statistics.aspx). It was a great experience with very positive feedback from my colleagues, and something I look forward to repeating. 

It is no secret that I am very much a proponent of using R, not only for doing statistical analysis, but for doing everything in, what one could label the statistics "stack." The statistics stack being a representation of everything a skilled statistician or data scientist needs to be able to do. In my mind, one can partition the statistics stack into:

1. Reading data, be it from local files (csv, excel, sas, etc.), a database (SQL, NoSQL, etc.), online sources (Quandl,  web scraping, online APIs, etc.) or something else.
2. The transformation of data, popular known as data munging or data wrangling.
3. Modeling data using both tried and tested models as well as the latest cutting edge models developed by eg. academic researchers.
4. Visualising data.
5. Sharing the data using dynamic reporting or shared dashboards.

When I made the presentation for the workshop, I decided that I wanted to start the first session, an introduction to R, by giving a small example that covers all aforementioned 5 points. The point was to showcase how it is possible to do everything in R and more importantly, how easy it is. Visually in a presentation, I think that it works very well to showcase ease of use, by showing small code snippets and the respective outputs of these snippets. The following examples are what I choose in order to cover the five points.

For showcasing how to read data into R, I call two time series containing the daily historical open, high, low and close prices as well as the volume of the Apple and Microsoft stocks. I do this using the [Quandl package](https://www.quandl.com/tools/r), and do a bit of fiddling with the data using [dplyr](https://github.com/hadley/dplyr). The code is given below.

```R
library(dplyr)
library(Quandl)

data <- list(
  apple = Quandl("WIKI/AAPL") %>%
    mutate(Name = "Apple"),
  microsoft = Quandl("WIKI/MSFT") %>%
    mutate(Name = "Microsoft")
) %>%
  bind_rows() %>%
  select(Date, Open, High, Low, Close, Volume, Name) %>% 
  arrange(Date %>% desc())

head(data)
```

Here the sharp minded reader will notice that I have employed the [magrittr](https://cran.r-project.org/web/packages/magrittr/index.html) forward pibe operator as well. The output of running the code is displayed below.

```
Source: local data frame [6 x 7]

        Date  Open    High   Low Close   Volume      Name
      (date) (dbl)   (dbl) (dbl) (dbl)    (dbl)     (chr)
1 2016-02-26 97.20 98.0237 96.58 96.91 28691422     Apple
2 2016-02-26 52.60 52.6800 51.10 51.30 35687833 Microsoft
3 2016-02-25 96.05 96.7600 95.25 96.76 27217966     Apple
4 2016-02-25 51.73 52.1000 50.61 52.10 26537823 Microsoft
5 2016-02-24 93.98 96.3800 93.32 96.10 35824629     Apple
6 2016-02-24 50.69 51.5000 50.20 51.36 31825224 Microsoft
```

To showcase data munging, I give a small example of how descriptive statistics can be computed using [dplyr](https://github.com/hadley/dplyr)	.

```R
library(moments)

data %>%
  group_by(Name) %>% 
  summarise(Mean = mean(Close),
            StandardDeviation = sd(Close),
            Maximum = max(Close),
            Minimum = min(Close),
            Skewness = skewness(Close),
            Kurtosis = kurtosis(Close))
			
```

Yielding the output given below.

```
Source: local data frame [2 x 7]

       Name     Mean StandardDeviation Maximum Minimum  Skewness kurtosis
      (chr)    (dbl)             (dbl)   (dbl)   (dbl)     (dbl)    (dbl)
1     Apple 99.22821         138.53090  702.10   11.00 2.4106307 8.003144
2 Microsoft 59.35221          33.44367  179.94   15.15 0.8116662 2.829117
```

For modeling, I choose to give the simplest example possible: How to run a linear regression model in R. Here I also used [tidyr](http://vita.had.co.nz/papers/tidy-data.html) to show how to pivot data in R.

```R
library(tidyr)

data %>%
  select(Date, Name, Close) %>%
  spread(Name, Close) %>%
  filter(Date >= "1990-01-01", Date < "2014-01-01") %>%
  lm(Apple ~ Microsoft, data = .) %>%
  summary()
```

Of course this model makes little sense considering the data, but that is not the point. Here `summary()` exemplifies how easy it is to obtain the standard statistics, you would want from a model fit, as seen from the output below.

```
Call:
lm(formula = Apple ~ Microsoft, data = .)

Residuals:
    Min      1Q  Median      3Q     Max 
-170.30  -90.12  -29.63   32.98  532.78 

Coefficients:
             Estimate Std. Error t value Pr(>|t|)    
(Intercept) 226.43396    3.38662   66.86   <2e-16 ***
Microsoft    -1.83794    0.04811  -38.20   <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 134.6 on 6047 degrees of freedom
Multiple R-squared:  0.1944,	Adjusted R-squared:  0.1943 
F-statistic:  1460 on 1 and 6047 DF,  p-value: < 2.2e-16
```

For visualisation of data, I choose the simplest example possible using the definite graphics library in R, [ggplot2](http://ggplot2.org/).

```R
library(ggplot2)

qplot(x = Date, y = Close, data = data, color = Name, geom = "line")
```

Showing how beautiful ggplot2 graphs are even when using the standard unaltered theme.

![Plot of Apple and Microsoft daily stock prices](https://dl.dropboxusercontent.com/s/wrxxef4ff7gskcd/apple-microsoft.svg)

Finally, I turned to sharing the analysis of the data, first by dynamic reporting using [rmarkdown](http://rmarkdown.rstudio.com/). Here I show the R Markdown document.

```R
---
title: "Example of Dynamic Reporting"
output:
  pdf_document
---

# Plot of Apple and Microsoft
  
Here we plot the closing prices for Apple and Microsoft for 
**`r data %>% filter(Name == "Apple") %$% Date %>% min()`** to 
**`r data %>% filter(Name == "Microsoft") %$% Date %>% max()`** and 
**`r data %>% filter(Name == "Apple") %$% Date %>% min()`** to 
**`r data %>% filter(Name == "Microsoft") %$% Date %>% max()`** respectively.

```{r, echo = FALSE, fig.width = 10, fig.height = 7.5}
qplot(x = Date, y = Close, data = data,
      color = Name, geom = "line")
    ```
```

I also present the call of the rendering function.

```R
library(rmarkdown)

render("stock-example.rmd")
```

Yielding the pdf file that can be found [here](https://www.dropbox.com/s/xed2g68myb1f532/stock-example.pdf?dl=0), where the dynamic in-line text has been bolded.

Finally, I present a [Shiny](https://www.rstudio.com/products/shiny/) app, showcasing an implementation of the [Highcharts](http://www.highcharts.com/) interavtive JavaScript charts in addition to the standard ggplot2 graph. The Highcharts implementation was just a quick hack based on my previous experience working in Highcharts, and won't be presented here.

Instead I present the ui.R file.

```R
dashboardPage(
  dashboardHeader(title = "Aktie app"),
  dashboardSidebar(
    sidebarMenu(
      
      menuItem("R graphics",
               tabName = "r-graphics",
               icon = icon("dashboard")),
      menuItem("Highchart Graphics",
               tabName = "highchart-graphics",
               icon = icon("th")))
    ),
  dashboardBody(
    
    singleton(tags$head(tags$script(
      src = "https://code.highcharts.com/stock/highstock.js"))),
    singleton(tags$head(tags$script(
      src = "https://code.highcharts.com/stock/1.2.5/modules/exporting.js"))),
    
    tabItems(
      
      tabItem(tabName = "r-graphics",
              fluidRow(
                h1("Stock app with ggplot2", align = "center"),
                br(),
                column(4,
                       textInput(inputId = "in_symbol",
                                 label = "Symbol",
                                 value = "AAPL"),
                       dateRangeInput(inputId = "in_dates", 
                                      label = "Date range",
                                      start = "1997-02-01", 
                                      end = as.character(Sys.Date())),
                       checkboxInput("in_log", "Plot y axen på en log skala", 
                                     value = FALSE)),
                column(8,
                       plotOutput("stock_gg", height = 250))
                )),
      
      tabItem(tabName = "highchart-graphics",
              fluidRow(
                h1("Stock app with ggplot2", align = "center"),
                br(),
                column(3,
                       textInput(inputId = "in_symbol",
                                 label = "Symbol",
                                 value = "AAPL"),
                       dateRangeInput(inputId = "in_dates", 
                                      label = "Date range",
                                      start = "1997-02-01", 
                                      end = as.character(Sys.Date()))),
                column(9,
                       htmlOutput("stock_hc"))))
    )
  )
)
```

And the server.R file.

```R
shinyServer(function(input, output) {
  
  data <- reactive({
    Quandl(paste0("WIKI/", input$in_symbol),
           start_date = input$in_dates[1],
           end_date = input$in_dates[2]) %>%
      mutate(Name = input$in_symbol) %>%
      select(Date, Open, High, Low, Close, Volume, Name)
    
  })
  
  output$stock_gg <- renderPlot({
    qplot(x = Date, y = Close, data = data(), color = Name, geom = "line") %>% 
      `+`(., if (input$in_log == TRUE) scale_y_log10())
  })
  
  output$stock_hc <- renderList({
    highchart_line_chart(data() %>%
                           select(Date, Close) %>%
                           mutate(Date = as.integer(as.POSIXct(Date)) * 1000))
  })

})
```

I'll add a link to the app running from my server at [DigitalOcean](https://www.digitalocean.com/), but for now, I present the app as two static images.

![Shiny app example with ggplot2](https://dl.dropboxusercontent.com/s/50ud6ruy8xprmzn/stock-app-gg.png)

![Shiny app example with Highcharts](https://dl.dropboxusercontent.com/s/bauvxzqtbjap5a6/stock-app-hc.png)

Overall I am very happy with these small examples, and the fact that it only took very limited time to make.