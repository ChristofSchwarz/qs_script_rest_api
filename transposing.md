 # Transposing Array Results into Tables
 ## "Pivoting" in Script
 
If your Json response from an REST API returns a data matrix, it may come back like this, that the first array element 
has headers and the following elements have the data for it. 
 
Example

If you do nothing else but use the Add Data wizard of the REST Connector you will get this:

We need to re-transform the data into columns (pivot it). What is missing is an auto-increment number which restarts when the main key 
(here: __FK_values_u0) changes ...

Add this to the block of your RestConnectorMasterTable just before SQL SELECT "__Key_root", ...
```
LOAD *,
    If(Len(__FK_values_u0)
    	,If(Peek('__FK_values_u0')=__FK_values_u0, Peek('__X')+1, 1)
        ) AS __X,
;
```
This introduces a new field "__X" with an increment.
