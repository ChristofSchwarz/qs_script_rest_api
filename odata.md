 # Qlik REST Connector works with OData Standard
 
There is an OData connector in Qlik Web Connector Package https://help.qlik.com/en-US/connectors/Subsystems/Web_Connectors_help/Content/Connectors_QWC/Data-Source-Connectors/OData-Connector.htm which is chargeable. However, the OData Standard can be followed with a few step using the free REST Connector of Qlik Sense/QlikView.

 1. Set Key Generation strategy of the REST Connection which you are using to „Current Record“
 2. Use Add REST Connector Wizard to create the proper script for reading the first data page for you
 3. Before first LOAD block
    - Create variable and start "DO" loop 
```
LET vSkiptoken = '';
DO
```
 4. In the first LOAD block, where it reads "RestConnectorMasterTable: SQL SELECT" 
    - (If missing add "odata.nextLink" as field)
    - Put right after the FROM JSON-line before the semicolon 
```
WITH CONNECTION (
    // maybe you have to add other params, like URL, HTTPHEADER, BODY
    QUERY "$skiptoken" "$(vSkiptoken)"
)
```
 5. Before "DROP RestConnectorMasterTable"
    - Parse the "$skiptoken" argument from odata.nextLink field
```
  tNextLink: LOAD Only(odata.nextLink) AS __nextLink 
  RESIDENT RestConnectorMasterTable;
  LET vNextlink = FieldValue('__nextLink', 1);
  DROP TABLE tNextLink;
  LET vSkiptoken = TextBetween(vNextlink & '&', '$skiptoken=', '&');
  WHEN Len(vSkiptoken) TRACE [nextLink $skiptoken=$(vSkiptoken)];
```    
 6. After "DROP RestConnectorMasterTable"
    - Close DO-Loop
```
LOOP WHILE Len(vSkiptoken);
```
 

