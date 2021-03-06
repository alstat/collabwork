the Function
====================
The `hackJMA.dat` function below imports data from the JMA Best Track database. The argument `url` is a character string of the data URL. Of course, internet connection is required
```{coffee}
hackJMA.dat <- function(url){
  # Read the lines of the url
  raw.dat <- readLines(url)
  
  # Match all entries of rows with starting characters 66666
  storm.hdr <- grep("^66666", raw.dat)
  
  # Define the column headers
  colmn.hdr <- c('yymmddhh','indicator','cat',
                 'latt','lont','hPa','kts','L50',
                 'S50','L30','S30','l1hr')
  
  # Clean the raw data by removing the storm header names
  new.dat <- read.table(text = raw.dat[-storm.hdr], col.names = colmn.hdr, comment.char = '', fill = TRUE)
  
  # Take the distance between rows of storm header names, these distances
  # will be used for repetitions of the storm labels
  storm.hdist <- c(diff(storm.hdr) - 1, length(raw.dat) - storm.hdr[length(storm.hdr)])
  
  # Tabulate the storm names
  storms <- read.table(text = raw.dat[storm.hdr], 
                       col.names = c('indicator', 'interId', 'nRecs', 'tropCycNum',
                                     'interId2', 'dissExit', 'dHrs', 'name', 'revDate'), fill = TRUE)
  
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

# Check the first six entries of the data
head(tydf2013)

# OUTPUT
  yymmddhh indicator cat latt lont  hPa kts L50 S50 L30 S30 l1hr   name
1 13010100         2   2   41 1417 1006   0  NA  NA  NA  NA      SONAMU
2 13010106         2   2   44 1395 1004   0  NA  NA  NA  NA      SONAMU
3 13010112         2   2   46 1372 1006   0  NA  NA  NA  NA      SONAMU
4 13010118         2   2   53 1350 1006   0  NA  NA  NA  NA      SONAMU
5 13010200         2   2   61 1326 1006   0  NA  NA  NA  NA      SONAMU
6 13010206         2   2   67 1304 1004   0  NA  NA  NA  NA      SONAMU

# Check the last six entries of the data
tail(tydf2013)

# OUTPUT
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

# First 10 of the YOLANDA data
tydat[1:10, ]

# OUTPUT
    yymmddhh indicator cat latt lont  hPa kts   L50 S50   L30 S30 l1hr   name CLongitude CLatitude Category MaxSpeed
896 13110306         2   2   58 1572 1004   0    NA  NA    NA  NA      HAIYAN      157.2       5.8       TS        0
897 13110312         2   2   61 1555 1008   0    NA  NA    NA  NA      HAIYAN      155.5       6.1       TS        0
898 13110318         2   2   61 1533 1004   0    NA  NA    NA  NA      HAIYAN      153.3       6.1       TS        0
899 13110400         2   3   61 1522 1002  35     0   0 90060  60      HAIYAN      152.2       6.1      STS       35
900 13110406         2   3   62 1504 1000  35     0   0 90060  60      HAIYAN      150.4       6.2      STS       35
901 13110412         2   3   63 1488  998  40     0   0 90100 100      HAIYAN      148.8       6.3      STS       40
902 13110418         2   3   65 1472  992  45     0   0 90120 120      HAIYAN      147.2       6.5      STS       45
903 13110500         2   4   65 1459  985  55 90030  30 90120 120      HAIYAN      145.9       6.5       TY       55
904 13110506         2   4   65 1446  980  60 90040  40 90120 120      HAIYAN      144.6       6.5       TY       60
905 13110512         2   5   69 1429  975  65 90050  50 90150 150      HAIYAN      142.9       6.9        L       65
```
Then, we can use the ggplot codes [here](https://github.com/alstat/Analysis-with-Programming/blob/master/2013/R/R-Mapping-Super-Typhoon-Yolanda-Haiyan-Track/Yolanda.R) by ignoring lines 7 to 20 only.


Years end in 00 and 0N
====================================
For data with Year entry in `yymmddhh` divisible with 100 (e.g. 1900 and 2000), or ends with 0N, where N = 1, 2, ..., 12. The special function below is used for remedy,
```{coffee}
format.date <- function(dat){
  dat[, 1] <- as.character(dat[, 1])
  for(i in 1:nrow(dat)){
    if((nchar(dat[i, 1]) < nchar('yymmddhh')) &&
         ((nchar('yymmddhh') - nchar(dat[i, 1])) == 3)){
      dat[i, 1] <- paste('000', dat[i, 1], sep = '')
    }
    if((nchar(dat[i, 1]) < nchar('yymmddhh')) &&
         ((nchar('yymmddhh') - nchar(dat[i, 1])) == 2)){
      dat[i, 1] <- paste('00', dat[i, 1], sep = '')
    }
    if((nchar(dat[i, 1]) < nchar('yymmddhh')) &&
         ((nchar('yymmddhh') - nchar(dat[i, 1])) == 1)){
      dat[i, 1] <- paste('0', dat[i, 1], sep = '')
    }
  }
  return(dat)
}
```
So for example, the Best Track for the years [2000 - 2009](http://www.jma.go.jp/jma/jma-eng/jma-center/rsmc-hp-pub-eg/Besttracks/bst0009.txt), we have

```{coffee}
tydf20.09 <- hackJMA.dat(url = "http://www.jma.go.jp/jma/jma-eng/jma-center/rsmc-hp-pub-eg/Besttracks/bst0009.txt")
head(tydf20.09)

# OUTPUT
  yymmddhh indicator cat latt lont  hPa kts L50 S50 L30 S30 l1hr   name
1    50418         2   2   88 1375 1004   0  NA  NA  NA  NA      DAMREY
2    50500         2   2   97 1360 1004   0  NA  NA  NA  NA      DAMREY
3    50506         2   2   99 1350 1004   0  NA  NA  NA  NA      DAMREY
4    50512         2   2  102 1344 1004   0  NA  NA  NA  NA      DAMREY
5    50518         2   2  107 1337 1002   0  NA  NA  NA  NA      DAMREY
6    50600         2   2  111 1330 1000   0  NA  NA  NA  NA      DAMREY
```
Notice R excludes zeros in `yymmddhh`, that is 50418 should be 00050418. To get rid of this, we need to tell R to treat 50418 as character not as numeric, then supply the missing zeros to it. That's what `format.date` function above does,
```{coffee}
newtydf20.09 <- format.date(dat = tydf20.09)
head(newtydf20.09)

# OUTPUT
  yymmddhh indicator cat latt lont  hPa kts L50 S50 L30 S30 l1hr   name
1 00050418         2   2   88 1375 1004   0  NA  NA  NA  NA      DAMREY
2 00050500         2   2   97 1360 1004   0  NA  NA  NA  NA      DAMREY
3 00050506         2   2   99 1350 1004   0  NA  NA  NA  NA      DAMREY
4 00050512         2   2  102 1344 1004   0  NA  NA  NA  NA      DAMREY
5 00050518         2   2  107 1337 1002   0  NA  NA  NA  NA      DAMREY
6 00050600         2   2  111 1330 1000   0  NA  NA  NA  NA      DAMREY
```
The function also works on entries like 00110600 (November 6, 2000) and 01110600 (November 6, 2001); any date that starts with zero.
