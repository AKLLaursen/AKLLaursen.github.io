---
layout: post
author: andreas
---
Some time ago, I was made aware of the brilliant [3-D view of the US treasury yield curve](http://www.nytimes.com/interactive/2015/03/19/upshot/3d-yield-curve-economic-growth.html) that [Gregor Aisch](http://driven-by-data.net/) had made for [New York Times](http://www.nytimes.com). In my opinion this is a brilliant example of how 3-D graphs in rare cases can be a viable way to visualize data.

For some time, I have been wanting to see how such a 3-D plot of the US treasury yield curve could be constructed in R. Finally, the other day, I had the time to sit down and consider it. First I needed to get hands on the actual data. A number of sources are available, but actually the [Treasury Department](https://www.treasury.gov/Pages/default.aspx) itself provides a daily XML feed with the data available [here](https://www.treasury.gov/resource-center/data-chart-center/interest-rates/Pages/TextView.aspx?data=yield). Thus it was easy to write a script to get the data using [`xml2`](https://github.com/hadley/xml2), [`magrittr`](https://cran.r-project.org/web/packages/magrittr/index.html) and [`dplyr`](https://github.com/hadley/dplyr).

```R
import::from(magrittr, `%>%`, `%$%`, set_names)
import::from(xml2, read_xml, xml_find_all, xml_children, xml_text, xml_name)
import::from(dplyr, bind_rows, select, arrange)

yield_url <- "http://data.treasury.gov/feed.svc/DailyTreasuryYieldCurveRateData"

yield_data <- read_xml(yield_url) %>%
  xml_find_all("//*[name()='m:properties']") %>%
  lapply(xml_children) %>%
  lapply(function(x) {
    lapply(x, function(y) {
      y %>%
        xml_text() %>%
        {
          if (xml_name(y) == "Id") {
            as.integer(.)
          } else if (xml_name(y) == "NEW_DATE") {
            as.Date(.)
          } else {
            as.numeric(.)
          }
        }
    }) %>%
      set_names(xml_name(x)) %>%
      as.data.frame(stringsAsFactors = FALSE)
  }) %>% 
  bind_rows() %>%
  select(-BC_30YEARDISPLAY) %>%
  arrange(NEW_DATE)
```

Here I also used the package [`import`](https://cran.r-project.org/web/packages/import/vignettes/import.html), which makes it possible to import objects into a given R session in a very Python like way. I like this a lot as

1. Often you only need to utilize a limited number of functions from a given package. As such, there is really no reason to import the entire library.
2. More importantly, it gives the reader of the script a nice overview of which packages the functions in the script comes from.

Next, how to create the actual plot? There exists a number of R packages for plotting 3-D visualizations. I have given a number of them below.

* The [`persp()`](https://stat.ethz.ch/R-manual/R-devel/library/graphics/html/persp.html) function in the base R package.
* The [`plot3D`](https://cran.r-project.org/web/packages/plot3D/index.html) package.
* The [`scatterplot3d`](https://cran.r-project.org/web/packages/scatterplot3d/index.html) package.
* The [`rgl`](https://cran.r-project.org/web/packages/rgl/index.html) package.

Especially `plot3D` has traditionally been used to create static 3-D visualizations. However, in order to make the 3-D graphic as comprehensible as possible, it needs to be dynamic. That is the reader/user has to be able to turn and zoom in the graphic and have labels appearing when hovering over a given point. This means utilizing some sort of JavaScript graphic library. Here [D3](https://d3js.org/) is the obvious choice. Luckily in R, we can use the awesome integration of [Plotly](https://plot.ly/) in R, to ultimately create a vizualisation that is very close to the one given in the New York Times article. (Digging in the source code of the page seems to reveal that that the original visualization was made using [Adobe Illustrator](https://www.adobe.com/products/illustrator.html)).

Making the 3-D view of the yield curve in R turned out to be a simple exercise using the [`plotly`](https://plot.ly/r/) package. Actually one of the [founders](https://plot.ly/~chris/folder/home) of Plotly had already made an example, and I basically took his JavaScript code and implemented a very similar solution using the Plotly R API. The code is given below.

```R
import::from(plotly, plot_ly, layout)

Sys.setenv("plotly_username"="UserName")
Sys.setenv("plotly_api_key"="ApiKey")

y <- yield_data %$%
  NEW_DATE

x <- c("1 Month", "3 Month", "6 Month", "1 Year", "2 Year", "3 Year", "5 Year", 
       "7 Year", "10 Year", "20 Year", "30 Year")

z <- yield_data %>%
  select(BC_1MONTH:BC_30YEAR) %>%
  as.matrix()

plot <- plot_ly(yield_data,
                x = x,
                y = y,
                z = z,
                type = "surface",
                name = "yield",
                colorscale = list(
                  list(
                    0,
                    "rgb(231, 245, 255)"
                  ),
                  list (
                    1,
                    "rgb(31, 119, 180)"
                  )
                ),
                showscale = FALSE,
                lighting = list(
                  fresnel = 0.01,
                  specular = 0.01,
                  ambient = 0.95,
                  diffuse = 0.99,
                  roughness = 0.01
                ),
                zmax = 10.0, 
                zmin = 0)

plot <- layout(
  plot,
  title = "US Treasury Yield Curve",
  autosize = FALSE,
  scene = list(
    xaxis = list(
      zeroline = FALSE,
      showgrid = TRUE,
      type = "category",
      title = ""
    ),
    yaxis = list(
      zeroline = FALSE,
      showgrid = TRUE,
      type = "date",
      title = ""
    ),
    type = "surface",
    zaxis = list(
      zeroline = FALSE,
      showgrid = TRUE,
      ticksuffix = "%",
      title = "Yield"
    ),
    aspectmode = "manual",
    camera = list(
      eye = list(
        y = -0.03,
        x = -2.7,
        z = 1.3
      ),
      up = list(
        y = 0,
        x = 0,
        z = 1
      ),
      center = list(
        y = 0,
        x = 0,
        z = 0
      )
    ),
    aspectratio = list(
      y = 4,
      x = 1,
      z = 2
    )
  ),
  height = 350,
  width = 700,
  showlegend = FALSE,
  margin = list(
    r = 20,
    b = 60,
    l = 20,
    t = 60
  )
)

plot
```

Yielding the awesome graphic given below.

<a href="https://plot.ly/~AndreasKeller/65/" target="_blank" title="US Treasury Yield Curve" style="display: block; text-align: center;"><img src="https://plot.ly/~AndreasKeller/65.png" alt="US Treasury Yield Curve" style="max-width: 100%;width: 1200px;"  width="1200" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="AndreasKeller:65"  src="https://plot.ly/embed.js" async></script>