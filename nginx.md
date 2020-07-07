

### 1. Why nginx if is evi
https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/


### 2. Configure nginx to create request_id if not exists in request header

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

[reference unique id for nginx](https://www.jianshu.com/p/5e103e1eb017)

### 3. Nginx location matching rule.

`location [ = | ~ | ~* | ^~ ] uri { ... }`  

To find a location request matching, nginx will do the following:  
1. looking for if location defined with prefix can be found, the longest matching prefix is selected and remembered.
If the longest matching prefix location has the “^~” modifier then regular expressions are not checked and use the that prefix directly.  

2. regular expression will be checked in the order of their appearance in configuration, if match, the first regular expression will be choose. if not, the remembered prefix will be used. 


Prefix Check:  
` `   no modifier, it is a prefix check, if request matches this rule then will be remembered and continue check regular expression.  
`=`   define the exactly match rule, if exactly match rule found, search `terminates`.    
`^~`  if longest request match this location, will not further check regular expression, search will `terminates`.   

Regular Expression Check:  
`~*`: Regular expression matching, case-insensitive matching  
`~` : Regular expression matching, case-sentitive matching  

Example:
```
    location = / {
        [ configuration A ]
    }

    location / {
        [ configuration B ]
    }

    location /documents/ {
        [ configuration C ]
    }

    location ^~ /images/ {
        [ configuration D ]
    }

    location ~* \.(gif|jpg|jpeg)$ {
        [ configuration E ]
    }
```
The “/” request will match configuration A (Exactly match)  
The “/index.html” request will match configuration B (prefix match and no regular expression match)  
The “/documents/document.html” request will match configuration C (prefix match and no regular expression match)
The “/images/1.gif” request will match configuration D (prefix match, no need regular expression match)
The “/documents/1.jpg” request will match configuration E.(prefix match but regular expression match, use regular expression)

Reference: http://nginx.org/en/docs/http/ngx_http_core_module.html#location
