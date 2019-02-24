# qs_script_rest_api
helpful script snippets working with REST connector

SET 
SQL SELECT 
 ...
FROM JSON (wrap on) "root" PK "__KEY_root"
WITH CONNECTION (
	URL "$(vBaseAPIurl)/endpint"
    //,QUERY "pagesize" "10000"
    ,HTTPHEADER "Cache-Control" "no-cache"
    ,HTTPHEADER "Authorization" "Bearer $(vToken)"
	,HTTPHEADER "X-HTTP-Method-Override" "PUT",
	,HTTPHEADER "cookie" "$(vCookie)")
    //,BODY "{""path"":""$(vAttribute)""}"
    //,BODY "$(vJsonBody)"
); 
