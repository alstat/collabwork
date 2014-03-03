the Function
====================
The `hackJMA.dat` function below will import data from the JMA Best Track database, with argument `url` -- a character string of the link. Of course, make sure you have the internet connection,
```{coffee}
hackJMA.dat <- function(url){
  # Read the lines of the url
  raw.dat <- readLines(url)
  
  # Match the all entries of rows with starting characters 66666
  storm.hdr <- grep("^66666", raw.dat)
  
  # Define the column headers
  colmn.hdr <- c('yymmddhh','indicator','cat',
                 'latt','lont','hPa','kts','L50',
                 'S50','L30','S30','l1hr')
  
  # Clean the raw data by removing the storm header names
  new.dat <- read.table(text = raw.dat[-storm.hdr], col.names = colmn.hdr, comment.char = '', fill = TRUE)
  
  # Take distance between rows of storm header names, these distance
  # will be used for labels repitions of the storm
  storm.hdist <- c(diff(storm.hdr) - 1, length(raw.dat) - storm.hdr[length(storm.hdr)])
  
  # Tabulate the storm names
  storms <- read.table(text = raw.dat[storm.hdr], 
                       col.names = c('indicator', 'interId', 'nRecs', 'tropCycNum',
                                     'interId2', 'dissExit', 'dHrs', 'name', 'revDate'))
  
  # Add columns for the storm names and its ID numbers
  new.dat$name <- rep(storms$name, times = storm.hdist)
  new.dat$id <- rep(storms$yyid, times = storm.hdist)
  
  # Return the data
  return(new.dat)
}
```
## Example
Let us import the 2013 data,
```{coffee}
tydf2013 <- hackJMA.dat(url = "http://www.jma.go.jp/jma/jma-eng/jma-center/rsmc-hp-pub-eg/Besttracks/bst2013.txt")

# Check the first five entries of the data
head(tydf2013)

#OUTPUT
  yymmddhh indicator cat latt lont  hPa kts L50 S50 L30 S30 l1hr   name
1 13010100         2   2   41 1417 1006   0  NA  NA  NA  NA      SONAMU
2 13010106         2   2   44 1395 1004   0  NA  NA  NA  NA      SONAMU
3 13010112         2   2   46 1372 1006   0  NA  NA  NA  NA      SONAMU
4 13010118         2   2   53 1350 1006   0  NA  NA  NA  NA      SONAMU
5 13010200         2   2   61 1326 1006   0  NA  NA  NA  NA      SONAMU
6 13010206         2   2   67 1304 1004   0  NA  NA  NA  NA      SONAMU

# Check the last five entries of the data
tail(tydf2013)

#OUTPUT
    yymmddhh indicator cat latt lont  hPa kts L50 S50   L30 S30 l1hr  name
939 13111400         2   2  122 1152 1004   0  NA  NA    NA  NA      PODUL
940 13111406         2   2  119 1129 1004   0  NA  NA    NA  NA      PODUL
941 13111412         2   3  119 1117 1002  35   0   0 70120  90      PODUL
942 13111418         2   3  118 1104 1002  35   0   0 70120  90      PODUL
943 13111500         2   2  115 1084 1006   0  NA  NA    NA  NA      PODUL
944 13111506         2   2  111 1071 1006   0  NA  NA    NA  NA      PODUL
```
You can do this for other year as well. 

For plotting purposes, we can extract storm names. For example we can extract the data for Super Typhoon Haiyan (Yolanda) only,
```{coffee}
category <- c("TD", "TS", "STS", "TY", "L")
tydat <- subset(tydf2013, tydf2013$name == "HAIYAN")
tydat$CLongitude <- tydat$lont*0.1
tydat$CLatitude <- tydat$latt*0.1
tydat$Category <- category[tydat$cat]
tydat$MaxSpeed <- tydat$kts
```
Then, we can use the ggplot codes [here](https://github.com/alstat/Analysis-with-Programming/blob/master/2013/R/R-Mapping-Super-Typhoon-Yolanda-Haiyan-Track/Yolanda.R) by ignoring the lines 7 to 20 only.
