 # Working with OData standard
 
 1. Set Key Generation strategy of the REST Connection which you are using to „Current Record“
 2. Use Add REST Connector Wizard to create the proper script for reading the first data page for you
 3. Before first LOAD block
    - Create variable and start "DO" loop 
```
LET skiptoken = '';
DO
```
 4. In the first LOAD block, where it reads "RestConnectorMasterTable: SQL SELECT" 
    - (If missing add "odata.nextLink" as field)
    - Put right after the FROM JSON-line before the semicolon 
```
WITH CONNECTION (QUERY "$skiptoken" "$(skiptoken)")
```
 5. Before "DROP RestConnectorMasterTable"
    - Parse the "$skiptoken" argument from odata.nextLink field
```
  nextLink: LOAD Only(odata.nextLink) RESIDENT 'RestConnectorMasterTable';
  LET nextlink = FieldValue('Only(odata.nextLink)', 1);
  DROP TABLE nextLink;
  LET skiptoken = TextBetween(nextlink & '&', '$skiptoken=', '&');
  WHEN Len(nextlink) TRACE [nextLink $skiptoken=$(skiptoken)];
```    
 6. After "DROP RestConnectorMasterTable"
    - Close DO-Loop
```
LOOP WHILE Len(nextlink);
```
 

