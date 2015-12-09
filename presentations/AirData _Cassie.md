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
 
A Minnesota example
========================================================

 ```r
##Load Custom Functions
round2 = function(x, n) {
  posneg = sign(x)
  z = abs(x)*10^n
  z = z + 0.51
  z = trunc(z)
  z = z/10^n
  z*posneg
}

trunc2=function(x,n){
  posneg=sign(x)
  z=abs(x)*10^n
  z=trunc(z)
  z=z/10^n
  z*posneg
}

aqsid=function(state,county,site){
  x=paste(sprintf("%02d",state),(sprintf("%03d",county)),(sprintf("%04d",site)),sep="-")
  x
}

aqsidpoc=function(state,county,site,poc){
  x=paste(sprintf("%02d",state),(sprintf("%03d",county)),(sprintf("%04d",site)),poc,sep="-")
  x
}

##Run RAQDM
library("raqdm")
setAQDMuser("your_username","your_password",save=T)
setAQDMdefaults(pc="CRITERIA",format="AQS")

request=getAQDMdata(bdate="20140401",
                    edate="20141031",
                    state="27",
                    param="44201",
                    format="AQS")

##Do not run until you receive email indicating file is ready
MNO3=getAQDMrequest(request,stringAsFactors=FALSE)

##Create custom variables

#Do I neeed to include POCS in my site id?
unique(MNO3$POC) #In this example only 1 POC - do not need POC in Site.Id

MNO3$aqsid=aqsid(MNO3$State.Code,MNO3$County.Code,MNO3$Site.ID) #Create an AQSID field
unique(MNO3$aqsid) #View each unique AQSID in the file

#Check reported units and convert to PPB
unique(MNO3$Unit) #In this example all data is reported in PPM (unit code 7)

#I want to work with PPB, so I will create a new variable with the data stored as PPB
MNO3$Result.PPB=ifelse(MNO3$Unit==7,MNO3$Sample.Value*1000,MNO3$Sample.Value) #If my file also had data in PPB, this would convert the results in PPM to PPB and Retain the PPB results

#Create date fields for use in openair
MNO3$date=as.POSIXct(paste(MNO3$Date,MNO3$Start.Time,sep=" "),format="%Y%m%d %H:%M",tz="GMT") #Format = the current format of the data
MNO3$date[1]

MNO3$shortdate=strftime(MNO3$date,format="%Y-%m-%d",tz="GMT") #When using strftime format= the format you would like to convert your data to
MNO3$shortdate[1]

#Use the help function to learn more about functions. The strftime help file includes a library of codes for formatting date variables
help(strftime)



###Tidy up data frame to variables I want to use

#Trim MNO3 frame
MNO3_2=subset(MNO3,select=c("aqsid","Result.PPB","date"))

#Rename variables
names(MNO3_2)=c("site","ozone","date")

#Done with the original?
rm(MNO3)

####Load openair
install.packages("openair")
library("openair")

##Quickview of our MNO3
summaryPlot(MNO3_2,pollutant="ozone",period="months")
help(summaryPlot)

#Subset the data to include only one monitoring site
blaine=subset(MNO3_2,MNO3_2$site=="27-003-1002")
summaryPlot(blaine,pollutant="ozone",period="months",main="Blaine 2014")

##Calendar View

#View daily results at all sites based on hourly data
calendarPlot(MNO3_2,pollutant="ozone",year=2014,month=4:10,statistic="mean")
help(calendarPlot)

#View daily results at Blaine based on hourly data
calendarPlot(blaine,pollutant="ozone",year=2014,month=4:10,statistic="mean",
             breaks=c(0,20,40,60,65,70,75,80,85,90),annotate="value", 
             main="Blaine O3 in 2014 \n Daily Mean of Hourly Values (ppb)",
             key.header="24-hour Avg Ozone", key.footer="ppb")



#Calculate 8-Hour Rolling Means at Blaine
rolling8HR=rollingMean(blaine,pollutant="ozone",width=8,new.name="Rolling8HR",MNO3.thresh=75,align="left")
rolling8HR$shortdate=as.Date(strftime(rolling8HR$date,format="%Y-%m-%d",tz="GMT"))
dailymax8HR=aggregate(rolling8HR,by=list(rolling8HR$shortdate),FUN="max")

#Trim data frame
dailymax8HR=subset(dailymax8HR,select=c("shortdate","site","Rolling8HR"))
names(dailymax8HR)=c("date","site","ozone")

calendarPlot(dailymax8HR,pollutant="ozone",year=2014,month=4:10,
             statistic="mean",breaks=c(0,50,60,65,70,75,80),
             cols="heat",annotate="value",
             main="Daily Maximum 8-hour Ozone at Blaine 2014",
             key.header="Daily Max\n8-HR Ozone",
             key.footer="ppb")

 ```
