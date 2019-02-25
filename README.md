# Qlik script snippets working with REST connector

Helpful Qlik script snippets in combination with REST connector. For a demo visit https://youtu.be/7m9ZejlzkkY

Put your API base URL into a variable at the beginning of your script to be used in all the REST calls.
```
LET vBaseAPIurl = 'https://jsonplaceholder.typicode.com/todos';
```
### WITH CONNECTION block for copy/pasting
The WITH CONNECTION block has the magic to make a REST call dynamic and to pass variables defined before. After you inserted your LOAD statement with the "Select Data" wizard, search for the end of the SQL SELECT block, it ends with FROM JSON (...); and copy/paste this:
```
WITH CONNECTION (
    URL "$(vBaseAPIurl)/endpoint/method"
    ,QUERY "pagesize" "1000"
    //,HTTPHEADER "Cache-Control" "no-cache"
    ,HTTPHEADER "Authorization" "Bearer $(vToken)"
    ,HTTPHEADER "Content-Type" "application/json"
    //,HTTPHEADER "X-HTTP-Method-Override" "PUT",
    ,BODY "{""key"":""value""}"
    //,BODY "$(vJsonBody)"
); 
```

### Working with JSON Body 
It is easier syntax to first put the Json block (APIs typically expect strict notation, that means 
double-quotes around keys and values) into a script variable and then to clean and format the variable:
```
// Define your Json-Body here
SET vJsonFilter = {
  "dateFrom": "2018-01-01T00:00:00.000Z",
  "dateTo": "$(vDateTo)"
};
// replace 1xquote with 2xquote and remove line-break and tabulator chars
LET vJsonFilter = PurgeChar(Replace(vJsonFilter,'"','""'),CHR(13)&CHR(10)&CHR(8));
```
### Logic to try previous bearer token and get new one if needed
<a href="https://github.com/ChristofSchwarz/qs_script_rest_api/blob/master/sub_try_request.md">See this script</a>

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

Enjoy.
