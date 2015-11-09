Introduction to Air Data Analysis in R
========================================================
Cassie McMahon <br>
Minnesota Pollution Control Agency <br>
August 12, 2015

Retrieving AQS data using raqdm
========================================================
<br>
**raqdm** is an R package for directly accessing data from U.S. EPA's Air Quality Data Mart (AQDM). It uses a web interface to query the AQDM and returns the results as a data.frame.
<br>
<br>
You can install **raqdm** from GitHub with **devtools**: 

```r
install.packages("devtools") 

library("devtools")
devtools::install_github("ebailey78/raqdm")
```

```r
library("raqdm") 
```
<br>
You can view documentation for **raqdm** on <a href="https://github.com/ebailey78/raqdm"> GitHub </a>


Retrieving AQS data using raqdm
========================================================
<br>
- The AQS Data Mart requires a username and password to retrieve data.
- This username and password is **not** the same as your AQS username and password. 
- To request an Air Quality Data Mart user account, follow the instructions found <a href="http://www.epa.gov/airdata/tas_Data_Mart_Registration.html"> here </a>.

Data output options
========================================================
- ***DMCSV***
  - Data Mart file includes geographic location and sample record data
- ***AQS***:
  - Identical to AMP 501
- ***AQCSV***: 
  - AirNow data format - smallest file with combined fields
  <br>
  <br>


Configuring raqdm
========================================================
 
Set user name and password

```r
setAQDMuser("my username", 
            "my_password", 
            save = TRUE)
```
 
Set default query options

```r
setAQDMdefaults(user = "myemail@example.com",
                pw = "my_password", 
                param = "44201", 
                frmonly = TRUE,
                pc="CRITERIA")
```

  
Retrieving data using raqdm
========================================================
Currently, the Data Mart only offers asynchronous data retrievals. This means that you must request and retrieve your data in two steps. 

```r
request=getAQDMdata(bdate="20140401",
                    edate="20141031",
                    state="27",
                    county="003",
                    site="1002",
                    param="44201",
                    format="AQS")
```
  
  
Wait for request to be processed on the server - You will receive an email when complete


```r
data=getAQDMrequest(request,
                    stringAsFactors=FALSE)
```
  
  
Retrieving data using raqdm
========================================================
You can also use loops to make multiple requests 

```r
# Set defaults for Wisconsin in 2014
  setAQDMdefaults(state = "55", bdate = "20140101", edate = "20141231")

# Create a vector with the parameters you are interested in
params <- c("45201", "42602", "44201")

# Use lapply to loop through the params vector, requesting each one from AQDM. A list of requests will be returned to the x variable
x <- lapply(params, function(p) {
  return(getAQDMdata(param=p))
})
```



```r
# now loop through the requests to retrieve the data
y <- lapply(x, function(r) {
  return(getAQDMrequest(r))
})

# You could then use do.call and rbind to combine them into one data.frame
d <- do.call(rbind, y)
```

 

Viewing your data
========================================================


```r
colnames(data)
head(data) displays first 6 rows
#summary(data) displays summary statistics
```

Creating working variables
========================================================

```r
#Create AQSID Field
data$aqsid=paste(sprintf("%02d",data$State.Code),
                 (sprintf("%03d",data$County.Code)),
                 (sprintf("%04d",data$Site.ID)),
                  sep="-")
unique(data$aqsid)
```



```r
#Check reported units and convert to PPB
unique(data$Unit)
data$Result.PPB=ifelse(
  data$Unit==7,
  data$Sample.Value*1000,
  data$Sample.Value)
```
 

```r
data$date=as.POSIXct(
  paste(data$Date,data$Start.Time,sep=" "),
  format="%Y%m%d %H:%M",tz="GMT")

data$date[1]

data$shortdate=strftime(
  data$date,format="%Y-%m-%d",tz="GMT")

data$shortdate[1]
```

Analysis with openair
========================================================
**openair** is an R package developed by the Natural Environment Research Council (NERC) that provides open source tools for the analysis of air pollution data. 

The **openair** <a href="http://www.openair-project.org/Downloads/OpenAirManual.aspx">user manual</a> provides an introduction to R and detailed examples of each **openair** function

For more info, visit the <a href="http://www.openair-project.org/"> **openair** website </a>

Install and load opeanair
========================================================

```r
install.packages("openair")
```

```r
library(openair)
```
 
 
