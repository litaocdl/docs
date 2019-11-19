
## Check certification expired date 
openssl s_client -servername NAME -connect HOST:PORT 2>/dev/null | openssl x509 -noout -dates

## Who the certification issue to 
openssl s_client -servername shellhacks.com -connect shellhacks.com:443 2>/dev/null | openssl x509 -noout -subject -issuer

## Show the sha1 finger print
openssl s_client -servername www.shellhacks.com -connect www.shellhacks.com:443 2>/dev/null | openssl x509 -noout -fingerprint
