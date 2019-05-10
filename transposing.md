 # Transposing Array Results into Tables
 ## "Pivoting" in Script
 
If your Json response from an REST API returns a data matrix, it may come back like this, that the first array element 
has headers and the following elements have the data for it. 
 
Example

If you do nothing else but use the Add Data wizard of the REST Connector you will get this:

We need to re-transform the data into columns (pivot it). What is missing is an auto-increment number which restarts when the element key changes (in my example, the element key was __FK_values_u0, adjust it accordingly) ...

Add this to the block of your RestConnectorMasterTable just before SQL SELECT "__Key_root", ...
```
LOAD *,
    If(Len(__FK_values_u0)
    	,If(Peek('__FK_values_u0')=__FK_values_u0, Peek('__X')+1, 1)
        ) AS __X,
;
```
This introduces a new field "__X" with an restarting increment for each element.

 ## Fieldnames are delivered in the 1st element of response?

Then you are lucky. Just add this after your RestConnectorMasterTable load-block: (in my example, the element key was __FK_values_u0, adjust it accordingly)
```
__FieldNames:
MAPPING LOAD __X, @Value
RESIDENT RestConnectorMasterTable
WHERE __FK_values_u0 = 1 AND Len(@Value);

GENERIC LOAD
	__FK_values_u0, ApplyMap('__FieldNames', __X), @Value
RESIDENT RestConnectorMasterTable
WHERE __FK_values_u0 > 1;
```
 ## Fieldnames are not part of the response?
Well, then you have to specify them here in a mapping table (in my example, the element key was __FK_values_u0, adjust it accordingly):
```
__FieldNames:
MAPPING LOAD * INLINE [
    1, Timestamp
    2, Passenger
    3, From Airport
    4, To Airport
    5, Date
    6, Operator
] (no labels);

GENERIC LOAD
	__FK_values_u0, ApplyMap('__FieldNames', __X), @Value
RESIDENT RestConnectorMasterTable
WHERE __FK_values_u0 > 0;
```
GENERIC LOAD creates a data model structure like this. Each field gets a new table, which isn't unperformant, but maybe irritating for the developer. It is a bit of more scripting to get all fields back into >one< table.

Enjoy.
