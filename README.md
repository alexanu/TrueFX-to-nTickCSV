# TrueFX-to-nTickCSV
Convert TrueFX.com tick data to n-tick OHLC CSV format, with [fasttime](https://cran.r-project.org/web/packages/fasttime/)-friendly datetime field. The purpose is to check if n-tick charts can be produced, meeting the requirements of Bob Volman's [Forex Price Action Scalping](https://infofpas.wordpress.com/) method.
  
Standard input and output are used, the only **parameter** is number of ticks to aggregate:

    funzip EURUSD-2016-05.zip | tfx2ntick 1000 > EURUSD-2016-05_T1000.csv

## In and Out

######Download URLs
[http://truefx.com/dev/data/2016/MAY-2016/EURUSD-2016-05.zip](http://truefx.com/dev/data/2016/MAY-2016/EURUSD-2016-05.zip) (requires login after free registration)  
Alternatively, same files are linked with https from [Pepperstone](https://pepperstone.com/en/client-resources/historical-tick-data) (no need to register).  
Here is a TrueFX bulk download [script](https://github.com/webradio/TrueFX-by-AutoIt3).

######Content of TrueFX tick files
    EUR/USD,20160502 00:00:00.959,1.14601,1.14605
    EUR/USD,20160502 00:00:00.991,1.14599,1.14605
    EUR/USD,20160502 00:00:01.018,1.14599,1.14604
    EUR/USD,20160502 00:00:01.078,1.14599,1.14605

######Target format
    2016-05-02 00:01:02.345,1.1460,1.1462,1.1459,1.1461
Note **full pips** are used, as opposed to usual for FX pipettes. Separators after years and months should make it easier to read with [**R**](https://www.r-project.org/) by `fastPOSIXct()`. 

## Aggregation
- In time: Each bar contains **predefined number of ticks**, like 1000. Time increment is then irregular by definition. Begin a fresh bar also after weekends, holidays. This way a **weekend gap** is visible as such, and not like a huge bar.
- In price: So far, I could not find a source of times-and-sales (last traded) tick feed for FX, if such one exists at all, especially with data from more than one broker consolidated. This is why a mean between so called best bid and ask (top of the book) is used. Now, to avoid full pip jumps in output data where in reality the price change was a mere pipette, a staircase function with **hysteresis** is used: one pip more if pip fractional rises beyond 3/4, one pip less if pip fractional drops below 1/4.    

###### Example of Spread Increase Before Weekend
    USD/JPY,20160506 20:59:56.266,107.089996,107.166  
    USD/JPY,20160506 20:59:56.291,107.089996,107.209999  
    USD/JPY,20160506 20:59:56.414,**106.986,107.265999**  
    USD/JPY,20160506 20:59:58.136,**106.986,107.265999**  
    USD/JPY,20160506 20:59:59.085,**106.978996,107.273003**  
    USD/JPY,20160506 20:59:59.203,**106.978996,107.273003**  
    USD/JPY,20160506 20:59:59.437,**106.917999,107.488998**  
    USD/JPY,20160509 00:00:00.026,107.359001,107.366997  
    USD/JPY,20160509 00:00:00.140,107.359001,107.366997  

## Miscellaneous
**Limitation**: with EUR/USD in mind, the expected price format is hard-coded to `x.xxxPp` (e.g. `1.14601`). This quick-and-dirty lazy short-cut makes this tool unsuitable for e.g. USD/JPY.
   
If drive space is a concern, one can (in Windows) first create an empty target file, set compress attribute and then let operating system deflate it on the fly, using `>>` (append) instead of `>` (redirect):

    funzip EURUSD-2016-05.zip | tfx2ntick 1000 >> EURUSD-2016-05_T1000.csv

## Links
[fUnZip](http://www.info-zip.org/) for extracting .zip to stdout.  
[gzip](http://www.gzip.org/#exe) is able to create .gz from stdin  
  
