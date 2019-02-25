## Try a REST request and get new token if needed

Rather than each time requesting a new bearer token it is better to first try whether the old token is still valid. 
If not, enter the sub which retrieves a new token and retry the call. To do so, you need 
 * a sub with the logic to get the token
 * put the REST-calls for data inside a DO-UNTIL loop 
 
 ### Sub to get a new token
 ```
 
SUB CheckIfSuccessful(vErr)
	// Sub is testing if there was an Error with previous statement
    
	SET ErrorMode = 1; // break if there's an error
	IF vErr = 0 THEN
    	LET vSuccessful = 1;
    ELSEIF vErr = 12 THEN
        TRACE Token seems to be invalid. Get new token ...;
        
        LIB CONNECT TO 'REST POST Request';
        Authentication:
        SQL SELECT 
            "access_token",
            "token_type",
            "expires_in",
            "expiration_time",
            "error_message"
        FROM JSON (wrap on) "root"
        WITH CONNECTION (
            URL "$(vBaseAPIurl)/api/Authentication"
            ,QUERY "username" "#####"
            ,QUERY "password" "#####"
        ); 
        
        LET vToken = FieldValue('access_token', 1);
        DROP TABLE Authentication;
        TRACE New Token is $(vToken);	
    ELSE
       	LET vErr = Num(vErr) & ' (' & Text(vErr) & ')';
		TRACE Error $(vErr); 
        EXIT SCRIPT;
	END IF
END SUB
 ```
 ### Try and repeat logic to retrieve data
 ```
 
 // First call of API, put in a do-until loop with error handling so if the
 // the REST call was unsuccessful, a new token is requested and the call is retried
 
DO

    SET ErrorMode = 0;  // ignore if there's an error
    LIB CONNECT TO 'REST GET Request';

    buildings:
    LOAD 
        id as buildingId
        ,name as building
    ;
    SQL SELECT 
        "id",
        "name"
    FROM JSON (wrap on) "root"
    WITH CONNECTION (
        URL "$(vBaseAPIurl)/api/Buildings"
        ,HTTPHEADER "Authorization" "Bearer $(vToken)"
        ,QUERY "mandantId" "A9FC4B24E0C8A4AA70B4E8C2F4C418A7"
    ); 

	CALL CheckIfSuccessful(ScriptError);
LOOP UNTIL vSuccessful;    
 ```
