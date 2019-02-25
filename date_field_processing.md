# Working with date fields and APIs

Usually, date field detection works well when Qlik Sense uses the Add Data Wizard. But since we are writing script 
ourselves when communicating with REST APIs, the proper date/time interpretation is up to you. 
 
 <a href="https://github.com/ChristofSchwarz/qs_script_rest_api">Back to main</a>

An ISO date format typically looks like this
```
2019-02-25T14:32:51.000Z
```
### Reading ISO date format
To read this you will parse the left 10 digits as date, skip one char, and then the next 6 digits as time:
```
Timestamp(
    Date#(Left([isoDate],10),'YYYY-MM-DD') 
    + Time#(Mid([isoDate],12,8),'hh:mm:ss')
    , '$(TimestampFormat)'
) AS [isoDate]
```
You would put that function in the block(s) AFTER _RestConnectorMasterTable: SQL SELECT .... FROM JSON ... WITH CONNECTION ();_ where 
the Select-Data wizard has placed the RESIDENT loads like this:

![alttext](https://github.com/ChristofSchwarz/pics/raw/master/ISODateTimeScript.png "screenshot")

Note: While **Date#()** and **Time#()** are text-to-number functions, **Timestamp()** is a number-to-text function. The purpose is to get 
a nice format of the parsed date/time. The first page of the script (the part which is automatically created when you
create the app) contains a variable with the TimestampFormat. That definition is reused here.

### Writing ISO date format
Put above string together with the **Date()** and **Time()** number-to-text functions, put a 'T' inbetween and add a 'Z' (or '.000Z' if the time has to have a milliseconds component). Below, we are putting the current date and time into a variable.
```
LET vDateTo = Date(Today(),'YYYY-MM-DD') & 'T' & Time(Now(), 'hh:mm:ss') & '.000Z';
```
Now lets send this vDateTo in a request-body to the API. The Select Block of the script is coming from the Select Data wizard, we just look at the relevant WITH CONNECTION part:
```
WITH CONNECTION (
    URL "$(vBaseAPIurl)/api/endpoint"
    ,HTTPHEADER "Authorization" "Bearer $(vToken)"
    ,BODY "{""dateFrom"":""2018-01-01T00:00:00.000Z"",""dateTo":""$(vDateTo)""}"
); 
```

### Autocalendar Fields Code Snippet
If you write script yourself (not using the Add Data wizard), your date fields won't get the auto-calendar dimensions (.Year, .YearMonth, .Month, .Week ...) unless you copy/paste below block at the end of your script. Edit the line that says "DERIVE FIELDS FROM" and put your date field names there.

```
[autoCalendar]: 
DECLARE FIELD DEFINITION Tagged ('$date')
FIELDS
    Dual(Year($1), YearStart($1)) AS [Year] Tagged ('$axis', '$year')
    ,Dual('Q'&Num(Ceil(Num(Month($1))/3)),Num(Ceil(NUM(Month($1))/3),00)) AS [Quarter] Tagged ('$quarter', '$cyclic')
    ,Dual(Year($1)&'-Q'&Num(Ceil(Num(Month($1))/3)),QuarterStart($1)) AS [YearQuarter] Tagged ('$yearquarter', '$qualified')
    ,Dual('Q'&Num(Ceil(Num(Month($1))/3)),QuarterStart($1)) AS [_YearQuarter] Tagged ('$yearquarter', '$hidden', '$simplified')
    ,Month($1) AS [Month] Tagged ('$month', '$cyclic')
    ,Dual(Year($1)&'-'&Month($1), monthstart($1)) AS [YearMonth] Tagged ('$axis', '$yearmonth', '$qualified')
    ,Dual(Month($1), monthstart($1)) AS [_YearMonth] Tagged ('$axis', '$yearmonth', '$simplified', '$hidden')
    ,Dual('W'&Num(Week($1),00), Num(Week($1),00)) AS [Week] Tagged ('$weeknumber', '$cyclic')
    ,Date(Floor($1)) AS [Date] Tagged ('$axis', '$date', '$qualified')
    ,Date(Floor($1), 'D') AS [_Date] Tagged ('$axis', '$date', '$hidden', '$simplified')
//   ,If (DayNumberOfYear($1) <= DayNumberOfYear(Today()), 1, 0) AS [InYTD] 
//   ,Year(Today())-Year($1) AS [YearsAgo] 
//   ,If (DayNumberOfQuarter($1) <= DayNumberOfQuarter(Today()),1,0) AS [InQTD] 
//   ,4*Year(Today())+Ceil(Month(Today())/3)-4*Year($1)-Ceil(Month($1)/3) AS [QuartersAgo] 
//   ,Ceil(Month(Today())/3)-Ceil(Month($1)/3) AS [QuarterRelNo] 
//   ,If(Day($1)<=Day(Today()),1,0) AS [InMTD] 
//   ,12*Year(Today())+Month(Today())-12*Year($1)-Month($1) AS [MonthsAgo] 
//   ,Month(Today())-Month($1) AS [MonthRelNo] 
//   ,If(WeekDay($1)<=WeekDay(Today()),1,0) AS [InWTD] 
//   ,(WeekStart(Today())-WeekStart($1))/7 AS [WeeksAgo] 
//   ,Week(Today())-Week($1) AS [WeekRelNo] 
;

DERIVE FIELDS FROM FIELDS myDateField1, [My Date Field 2] USING [autoCalendar] ;
```
Note: I commented out some of the sub-dimensions. Feel free to comment in what you need. 

