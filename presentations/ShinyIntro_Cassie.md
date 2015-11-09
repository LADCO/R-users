Using Shiny to Vizualize and Share Your Data
=============================================================

Duluth R Users Group
--------------------------------------

April 8, 2015 <br>
<a href="mailto:cassie.mcmahon@state.mn.us"> Cassie McMahon</a>, Minnestoa Pollution Control Agency

<br>
Modified from <a href="shiny.rstudio.com/tutorial/"> Shiny by RStudio Tutorial </a> 

What is Shiny?
======================================================== 
<p> Shiny is an R package that makes it easy to build interactive web applications (apps) straight from R. 

- `install.packages("shiny")`
- `library(shiny)`
- To minimize crashing <br> 
  - `update.packages("Rcpp")`

<br>Shiny is an <a href="www.rstudio.com"> RStudio </a> project. 

Getting started
========================================================
<p> Shiny Apps have two components
- ***a user interface script (ui.R)*** <br>
  - Controls the layout and appearance of your app
    <br>
    <br>
- ***a server script (server.R)***<br>
  - Contains the instructions that your computer needs to build your app
    <BR>
    
Create a project directory
========================================================
<img src="http://shiny.rstudio.com/tutorial/lesson1/images/example1-folder.png" style=" margin-left: 85px" alt="File Directory Screen shot"/> 
<br><center> **Set your working directory one level higher than your project directory**</center>
<br> <code> setwd("...//ShinyApps") </code>


Basic structure: User Interface (ui.R)
========================================================

```r
#ui.R

library(shiny)

shinyUI(fluidPage(
  titlePanel("title panel"),
  
  sidebarLayout(
    sidebarPanel( "sidebar panel"),
    
    mainPanel("main panel")
  )
))
```
 <center> <b> <code> sidebarLayout() </code> always takes two arguments </b>
  <code>sidebarPanel() </code> and <code> mainPanel() </code> </center>

Basic structure: Server (server.R)
========================================================

```r
#server.R

library(shiny)

shinyServer(function(input,output){
  
  
  })
```
Resulting Shiny app layout
========================================================
<img src="http://shiny.rstudio.com/tutorial/lesson2/images/sidebar-layout1.png"</>

Customizing your user interface
========================================================
<code> shinyUI(fluidPage( </code>


```r
titlePanel("2010 US Census: Demographic Maps"),
```


```r
sidebarLayout(position="right",
  sidebarPanel("Demographic Variables",
    helpText("Create 2010 US Census demographic maps"),
  ),
  mainPanel("Waiting for instructions")
```
<code> )) </code> 
  

Adding user controls
======================================================== 
<img src=http://shiny.rstudio.com/tutorial/lesson3/images/basic-widgets.png> 

Standard Shiny widgets
======================================================== 

|Function       | Description         |
|---------------|---------------------|
|<code>actionButton</code>|Action Button| 
|<code> checkboxGroupInput</code>|	A group of check boxes|
|<code>checkboxInput</code>|	A single check box|
|<code>dateInput</code>|	A calendar to aid date selection|
|<code>dateRangeInput </code>|	A pair of calendars for selecting a date range|
|<code> fileInput</code>|	A file upload control wizard|
|<code>helpText</code>|	Help text that can be added to an input form|
|<code>numericInput </code>|	A field to enter numbers|
|<code>radioButtons </code>|	A set of radio buttons|
|<code>selectInput</code>|	A box with choices to select from|
|<code>sliderInput</code>|	A slider bar|
|<code>submitButton </code>|	A submit button|
|<code>textInput</code>|	A field to enter text|

Adding widgets to your UI
======================================================== 
To add a widget to your app, place a widget function in <code> sidebarPanel() </code> or  <code>  mainPanel() </code>
<br>
<br> 
Each widget function requires several arguments
- <b> A name for the widget </b>

- <b> A label for the widget </b> 


```r
actionButton("action",label="Action")
```

Adding user controls to the US Census Demographic Map
========================================================

<code> sidebarPanel( </code>

```r
selectInput("var", 
    label = "Choose a variable to map",
      choices = list("Percent White",
                     "Percent Black",
                     "Percent Hispanic", 
                     "Percent Asian"),
      selected = "Percent White"),
      
sliderInput("range", 
    label = "Range of interest (%):",
    min = 0, max = 100, value = c(0, 100))
    ),
```



Resulting layout
========================================================
<img src="http://shiny.rstudio.com/tutorial/lesson3/images/censusvis.png">

Creating reactive output
======================================================== 
Creating reactive output is a two step process

  1. Add an R object to your user-interface <code> ui.R </code>
  2. Tell Shiny how to build the object in <code> server.R </code>
    - The object will be reactive if the code that builds it calls a widget value

Example R objects for UI
======================================================== 

|Output function | creates |
|----------------|---------|
|<code>htmlOutput</code>	|raw HTML|
|<code>imageOutput|	</code>image|
|<code>plotOutput|</code>	plot|
|<code>tableOutput|</code>	table|
|<code>textOutput|</code>	text|
|<code>uiOutput|</code>	raw HTML|
|<code>verbatimTextOutput|</code>	text|

Adding an R object to your UI
=============================================

```r
#ui.R

shinyUI(fluidPage(... 
                  
   mainPanel(
      textOutput("text1"),
      textOutput("text2"),
      plotOutput("map")
    )
```


Provide R code to build the object
======================================================== 
Use <code> server.R </code> to tell Shiny how to build the object.


```r
# server.R

shinyServer(function(input, output) {

     output$text1=renderText({ 
          "You have selected this..." })
     
     output$text2=renderText({"You have 
       selected a range that goes from..."})
     
     output$map=renderPlot({...})
  }
)
```

======================================================== 

The unamed function builds a list-like object named <code> output </code> that contains all of the code needed to update the R objects in your app. Each R object needs to have it's own entry in the list. 

|server.R| ui.R|
|--------|-----|
|<code>output$text1 </code> |<code> textOutput("text1") </code> |
|<code>output$text2 </code>| <code> textOutput("text2") </code> |
|<code>output$map </code> | <code> plotOutput("map") </code> |

Use render* functions to build an object
======================================================== 
Within server.R 

|Render function| creates|
|---------------|---------|
|<code>renderImage</code>|	images (saved as a link to a source file)|
|<code>renderPlot</code>|	plots|
|<code>renderPrint</code>|	any printed output|
|<code>renderTable</code>|	data frame, matrix, other table |like structures|
|<code>renderText</code>|	character strings|
|<code>renderUI</code>|	a Shiny tag object or HTML|

<small> Each render* function takes a single argument: an R expression surrounded by braces {}. </small> 

Making the R object reactive
======================================================== 
Use the <b> input </b> argument in the unamed function


```r
# server.R

shinyServer(
  function(input, output) {
  
    output$text1 <- renderText({ 
      paste("You have selected", input$var)
    })    
  }
)
```
 
<small> Use the widget name to call the user input in the server script </small> 
 

Review the ui.R and server.R scripts
======================================================== 

```r
#ui.R
selectInput("var", ...,
      
sliderInput("range",...
 
mainPanel(
   textOutput("text1"),
   textOutput("text2")
```


```r
#server.R
output$text1 <- renderText({ 
 paste("You have selected:", input$var)})
  
output$text2<-renderText({
  paste("You have selected a range that goes from",input$range[1],"to",input$range[2], sep=" ")})
```

Adding data and scripts
========================================================
Our app will dynamically map county level demographic data from the 2010 U.S. Census

- <b>Data</b>: We will be using data included in the <code>UScensus2010 </code> R package. You can download it <a href:"http://shiny.rstudio.com/tutorial/lesson5/census-app/data/counties.rds"> here </a>

- <b> Script</b>:<code> helpers.R </code> is an R script that can help you make choropleth maps. You can download helpers.R <a href="http://shiny.rstudio.com/tutorial/lesson5/census-app/helpers.R"> here </a> 
 
 
Where to store the data file
========================================================
 - Create a new folder named <b> data </b> in your app directory
 - Move the <b> counties.rds </b>  into the data folder
 

```r
counties=readRDS("ShinyDemoApp/data/counties.rds")

shinyServer(
  function(input, output) {...
```

Where to store the helpers.R script
========================================================
 
Save <code> helpers.R </code> inside your app directory
- Within server.R call the script using...

```r
source("ShinyDemoApp/helpers.R")

shinyServer(
  function(input, output) {...
```
Dependencies for <code> helpers.R </code>
  - <code> maps </code>
  - <code> mapsproj </code> 
  

```r
install.packages(c("maps","mapproj"))
```

Creating the script to map the data
========================================================
We will use the <code> percent_map </code> function in <code> helpers.R </code>

Percent map takes 5 arguments
  - <b> var</b>: a column vector from counties.rds 
  - <b> color</b>: any character string you see in the ouput of colors()
  - <b> legend.title</b>: a character string to use as the title of the plots legend
  - <b> max</b>: a parameter for controlling shade range (100)
  - <b> min</b>: a paramter for cocontrolling shade range (0)

Static mapping script
========================================================

```r
library(maps
library(mapproj)
source("ShinyDemoApp/helpers.R")
counties=readRDS("ShinyDemoApp/data/counties.rds")
percent_map(counties$white, "darkgreen", "% white")
```

Adding reactive output to the mapping script
========================================================

```r
output$map <- renderPlot({
  data <- switch(input$var, 
    "Percent White" = counties$white,
    "Percent Black" = counties$black,
    "Percent Hispanic" = counties$hispanic,
    "Percent Asian" = counties$asian)
      
  color <- switch(input$var, 
    "Percent White" = "darkgreen",
    "Percent Black" = "darkblue",
    "Percent Hispanic" ="darkorange",
    "Percent Asian" = "darkviolet")
  
  ...
```
Continued 
========================================================

```r
...

legend <- switch(input$var, 
    "Percent White" = "% White",
    "Percent Black" = "% Black",
    "Percent Hispanic" = "% Hispanic",
    "Percent Asian" = "% Asian")
      
   percent_map(var = data, 
   color = color, 
   legend.title = legend, 
   max = input$range[2], 
   min = input$range[1])

 })
```

Running your app
========================================================
1. Save changes to ui.R and server.R
2. Set your working directory 
  - <code> setwd("...//Shiny Presentation") </code> 
3. Load the shiny package
  - library(shiny)
4. Run <code> runApp("ShinyDemoApp") </code>
  - The app is called using the name of your app directory

Sharing your app
========================================================
Two options for sharing

1) Share your shiny app as two files: <code> server.R </code> and <code> ui.R </code>
  - You will also need to send any supplementary files 
  - The user will then run the app directly from R (runApp)
  - Options to store files at URL, GitHub, or Gist
      - runUrl, runGitHub, runGist

 2) Share your Shiny app as a web page
  - Most user friendly option
  - DIY if you are familiar with web hosting
  - Shiny also offers 3 levels of hosting
      - Shinyapps.io
      - Shiny Server
      - Shiny server Pro
