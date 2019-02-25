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

### Working with dates in APIs (read/write) 
<a href="https://github.com/ChristofSchwarz/qs_script_rest_api/blob/master/date_field_processing.md">See this snippets</a>

Enjoy.
