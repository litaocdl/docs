

### Why nginx if is evi
https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/


### Configure nginx to create request_id if not exists in request header

if x-request-id exists in request header, it will set req_id to $x-request-id
if not exists, will use nginx variable $request_id to generate a new one. 

notes, to visit variable in request header, nginx will need to use `http_<header key>`, while header key need convert 
all dash to underline and convert all capital to lower case. like `x-request-id` --> `x_request_id`

```
    map $http_x_request_id   $req_id {
        default          $http_x_request_id;
        ""               $request_id;
    }
    
    proxy_set_header X-Request-ID $req_id;
    
```
