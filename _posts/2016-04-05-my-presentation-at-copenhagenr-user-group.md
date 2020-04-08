---
layout: post
author: andreas
---
Last week I had the pleasure of presenting a side project in front of a bunch of R enthusiast at the monthly [CopenhagenR useR Group meeting](http://www.meetup.com/CopenhagenR-useR-Group/), this time hosted at Microsoft Denmark. I gave a description of the event in a [previous post](http://www.akll.io/2016/3/19/speaking-copenhagenr-user-group-revoluti/). As usual, it was a cool meetup, and I really enjoyed presenting in front of such skilled people. Presenting to a group of peers really gives one the change to "nerd out", and not dumb down on the technical stuff, which I also relished. The slides for my presentation are available for download [here](https://dl.dropboxusercontent.com/s/4h9hv9zho9vvqo5/mycroftr.pdf) and the app I showcased is hosted [here](http://bit.ly/1NINHEF). Below I give a short summary of the presentation.

## `mycroftr`

In the talk I wanted to highlight some things about R, that I find really cool, and that I thought might be unfamiliar to most people. Of course I had a time constraint, so I also had to stay within what could reasonably be presented in 30 minutes, (I might have ended up going a bit over time). I knew I wanted to talk about a Shiny app project that has been in ongoing development for quite some time, so I ended up picking the two coolest features of this app. These being

1. Easy implementation of the JavaScript graphing library [Highcharts](https://www.highcharts.com/) in a Shiny app using the package [`highcharter`](http://jkunst.com/highcharter/) by [Joshua Kunst](http://jkunst.com/).
2. How to build a dynamic UI in a [`Shiny`](http://shiny.rstudio.com/) App for easy scaling of input data.

The inspiration for the app hails from the financial industry, where the [Bloomberg Terminal]( http://www.bloomberg.com/professional/products-solutions/) is a popular tool. Using the custom desktop environment [Launchpad](http://www.bloomberg.com/professional/custom-desktop/), it is possible to build a dashboard with important data and select market news. However, not having $24,000 for a yearly license, I thought I would attempt to create my own using data available for free. Also, it would be nice to create something that doesn't look like an 80s desktop application.

![Bloomberg Launchpad - Those are some nice color](https://dl.dropboxusercontent.com/s/mlqhv2aiose47oq/bloomberg_launchpad_example_large.png)

Before I started working on the app, I made the following list of requirements. The app should be driven by free online data such as what is available from

* [`Quandl`](https://www.quandl.com/)
* [`quantmod`](http://www.quantmod.com/)
* Various APIs such as the [one offered by Statistics Denmark](http://www.dst.dk/da/Statistik/statistikbanken/api).

The charting should be done using a interactive charting framework such as

* [Highcharts](https://www.highcharts.com/)
* [D3](https://d3js.org/)
* [`plotly`](https://plot.ly/)

I should be able to feed the app a list of data, and it should generate the graphs automatically depending on the data it has been fed. The reason for this is

* It should be easy to scale the app using new found data.
* It should be easy to add data by just appending new data sets, such as a Quandl code, to a static file.

So that is a quick overview of the project. The current status is, that the The “template” ui has been completed. Currently needing to find data and feed it (and add graph types if needed). I have decided to call the app and accompanying package `mycroftr`, inspired by the [brother](https://en.wikipedia.org/wiki/Mycroft_Holmes) of [one of my favourite characters](https://en.wikipedia.org/wiki/Sherlock_Holmes) in literature, of whom it is said in The Adventure of the Bruce-Partington Plans

> He has the tidiest and most orderly brain, with the greatest capacity for storing facts, of any man living. The same great powers which I have turned to the detection of crime he has used for this particular business. The conclusions of every department are passed to him, and he is the central exchange, the clearinghouse, which makes out the balance. All other men are specialists, but his specialism is omniscience. We will suppose that a minister needs information as to a point which involves the Navy, India, Canada and the bimetallic question; he could get his separate advices from various departments upon each, but only Mycroft can focus them all, and say offhand how each factor would affect the other. They began by using him as a short-cut, a convenience; now he has made himself an essential. In that great brain of his everything is pigeon-holed and can be handed out in an instant.

## `highcharter`

As mentioned above I chose to integrate the [`highcharter`](http://jkunst.com/highcharter/) package in my app. But why use [Highcharts](https://www.highcharts.com/) at all? The reasons I see can be listed as

- The ability to easily create beautiful interactive plots.
- There are lots of different kinds of plot types available (also plots specialised for finance through [Highstocks](http://www.highcharts.com/products/highstock)).
- Highcharts had a very mature API, excellent documentation and customisation.

So how to implement Highcharts in R, and specifically in a Shiny app? Until now there has basically been three solutions to implement Highcharts
in R

- Use the package [`rCharts`](http://rcharts.io/), which uses a lattice style approach.
  - Last commit on Github was 2 years ago.
- Use the package [`highchartR`](https://github.com/jcizel/highchartR) of which I’m unfamiliar
  - Last commit on Github was 1 year ago.
- Write a JS wrapper yourself.

I started on the last part, but then `highcharter` was [released](http://jkunst.com/r/presenting-highcharter/) in January. `highcharter` is by far (in my opinion) the best Highcharts implementation so far. The package is essentially a wrapper around Highcharts. All plot types of Highcharts are available, though they might take some work to implement. And best of all it uses a familiar layering approach in building graphs. That is `highcharter` is a bit like [`ggplot2`](http://ggplot2.org/), but without non-standard evaluation (as the author puts it himself. The layering is done using the the [`magrittr`](https://cran.r-project.org/web/packages/magrittr/index.html) pibe. Finally it is possible to build custom themes, which is also very cool. I gave some examples of plots and how to create them, which I have presented below.

```r
# Get data

library(dplyr)
library(Quandl)

start_date <- as.Date("2015-01-01")
end_date <- as.Date("2015-12-31")

data <- list(
  DowJones = Quandl("YAHOO/INDEX_DJI", start_date = start_date,
                    end_date = end_date, collapse = "monthly") %>% 
    mutate(Name = "Dow Jones Industrial Average"),
  SP500 = Quandl("YAHOO/INDEX_GSPC", start_date = start_date,
                 end_date = end_date, collapse = "monthly") %>% 
    mutate(Name = "S&P 500"),
  DAX = Quandl("YAHOO/INDEX_GDAXI", start_date = start_date,
               end_date = end_date, collapse = "monthly") %>% 
    mutate(Name = "DAX")
) %>% 
  rbind_all() %>%
  transmute(Date = format(Date, "%b"), Name, Price = round(Close, 2))


# Line plot

library(highcharter)

chart <- highchart() %>% 
  hc_add_serie(name = "DAX",
               data = data %>% filter(Name == "DAX") %>% `$`(Price))

chart


# Line plot - multiple lines

library(highcharter)

chart %>% 
  hc_title(text = "Index values") %>% 
  hc_xAxis(categories = data$Date) %>%
  hc_add_serie(name = "Dow Jones Industrial Average",
               data = data %>%
                 filter(Name == "Dow Jones Industrial Average") %>%
                 `$`(Price)) %>% 
  hc_add_serie(name = "S&P 500",
               data = data %>% filter(Name == "S&P 500") %>% `$`(Price)) %>% 
  hc_yAxis(title = list(text = "Value")) %>%
  hc_add_theme(hc_theme_sandsignika())


# Line plot and column plot

highchart() %>% 
  hc_title(text = "Index values") %>% 
  hc_add_serie(name = "DAX",
               data = data %>% filter(Name == "DAX") %>% `$`(Price),
               type = "column") %>% 
  hc_xAxis(categories = data$Date) %>%
  hc_add_serie(name = "Dow Jones Industrial Average",
               data = data %>% filter(Name == "Dow Jones Industrial Average") %>% `$`(Price),
               type = "column") %>% 
  hc_add_serie(name = "S&P 500",
               data = data %>% filter(Name == "S&P 500") %>% `$`(Price)) %>% 
  hc_yAxis(title = list(text = "Value")) %>%
  hc_add_theme(hc_theme_sandsignika())


# Tree map

library(treemap)
library(viridisLite)

data(GNI2014)

tm <- treemap(GNI2014, index = c("continent", "iso3"),
              vSize = "population", vColor = "GNI",
              type = "value", palette = viridis(6),
              draw = FALSE)
hc_tm <- highchart() %>% 
  hc_add_serie_treemap(tm, allowDrillToNode = TRUE,
                       layoutAlgorithm = "squarified",
                       name = "tmdata") %>% 
  hc_title(text = "Gross National Income World Data - 2014") %>% 
  hc_tooltip(pointFormat = "<b>{point.name}</b>:<br>
             Pop: {point.value:,.0f}<br>
             GNI: {point.valuecolor:,.0f}")

hc_tm
```

Every container in the Highcharts API has its own separate `highcharter` function. As such, every graph imaginable from the Highcharts website can be recreated in R. Thus it is easy to leverage the extremely powerful chart library that is Highcharts. The charts are easy to implement in a Shiny app through the `renderHighchart()` and `highchartOutput()` functions.

## Dynamic UI in Shiny
If you are familiar with writing applications in Shiny, you are familiar with the `ui.R` and `server.R` approach to building apps.

- `ui.R` is where the interface is defined.
- `server.R` handles the server side of the app. Usually data gathering, data manipulation, data modeling and data visualisation.

This approach is however very static, that is for each plot you write a server side component and a UI component. This makes it difficult to scale when you increase the amount of input data. Below is an example of how you would build an app traditionally using this approach. The example is taken from the [RStudio Shiny tutoriual](http://shiny.rstudio.com/tutorial/lesson1/).

```r
library(shiny)

# ui.R
shinyUI(fluidPage(

  titlePanel("Hello Shiny!"),

  sidebarLayout(
    sidebarPanel(
      sliderInput("bins", "Number of bins:", min = 1, max = 50, value = 30)
    ),

    # Static output displayed
    mainPanel(
      plotOutput("distPlot")
    )
  )
))

# server.R
shinyServer(function(input, output) {

  # Static output generated
  output$distPlot <- renderPlot({
    x    <- faithful[, 2]  # Old Faithful Geyser data
    bins <- seq(min(x), max(x), length.out = input$bins + 1)

    # draw the histogram with the specified number of bins
    hist(x, breaks = bins, col = 'darkgray', border = 'white')
  })
})
```

I however want to be able to serve the app a list of data, and have the interface (the plots) adjust accordingly. This is in part inspired by [HackMyResume](http://please.hackmyresume.com/), a project making you able to generate CV from information in a JSON file. So in `Shiny` I want to serve the server a list of data, that can be subset through the inputs from the UI, making me able to

- Generate the UI on the server side though the `renderUI()` function.
- Display the UI through the `uiOutput()` and leave the rest of the `ui.R`
file as a template.

An example of how to implement this is given below. This code is inspired by [this gist](https://gist.github.com/wch/5436415/).

```r
library(shiny)

# ui.R
shinyUI(fluidPage(

  titlePanel("Hello Shiny!"),

  sidebarLayout(
    sidebarPanel(
      selectInput("stock_geography",
                  "Select area",
                  names(get_stock_list())),
      selectInput("stock_type",
                  "Select chart type",
                  c("Line Plot", "Candlestick"),
                  "Candlestick")
    ),

    # Dynamic UI displayed
    mainPanel(
      uiOutput("plots")
    )
  )
))

# server.R
shinyServer(function(input, output) {

  data_stock <- reactive({
    # Get data from Quandl based on security type and geographic
    lapply(get_stock_list() %>% `[[`(input$stock_geography), function(x) {
      Quandl(x, type = "xts")
    })
  })
  
  output$plots <- renderUI({

    # Generate list of Highcharts outputs
    plot_output_list <- lapply(1:(data_stock() %>% length), function(x) {
      plotname <- paste("plot", x, sep = "_")
      highchartOutput(plotname, height = 500)
    })

    # Convert the list to a tagList
    do.call(tagList, plot_output_list)
  })

  observe({
    for (x in 1:(data_stock() %>% length)) {

      # Need local so that each item gets its own number.
      local({
        x_local <- x

        plotname <- paste("plot", x_local, sep = "_")

        output[[plotname]] <- renderHighchart({

          if (input$stock_type == "Candlestick") {
            plot_candlestick(data_stock()[x_local])
          } else if (input$stock_type == "Line Plot") {
            plot_line(data_stock()[x_local])
          }
        })
      })
    }
  })
})
```

This finishes the examples from my presentations. The app is currently in development (and very infrequently updated), but a working version of the app can be found [here](www.apps.akll.io/shiny/mycroftr) and the code is available at my [Github](https://github.com/AKLLaursen/mycroftr).