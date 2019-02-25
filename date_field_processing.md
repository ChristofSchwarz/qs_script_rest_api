# Working with date fields and APIs

Usually, date field detection works well when Qlik Sense uses the Add Data Wizard. But since we are writing script 
ourselves when communicating with REST APIs, the proper date/time interpretation is up to you. 

An ISO date format typically looks like this
```
2019-02-25T14:32:51.000Z
```
### Reading ISO date format
To read this you will parse the left 10 digits as date, skip one char, and then the next 6 digits as time:
```
Timestamp(Date#(Left([isoDate],10),'YYYY-MM-DD') + Time#(Mid([isoDate],12,8),'hh:mm:ss'), '$(TimestampFormat)') AS [isoDate]
```
