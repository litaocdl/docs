## Openssl sub command 


openssl通过子命令支持不同种类型的算法和工具 `openssl help`列出所有支持的子命令
## enc
 对称加密算法子命令
 
`-p`:打印本次加密的iv和salt  
加密的秘钥通过iv和密码生成,iv和salt是明文，会存储在file.enc中，解密的时候只需要密码就可以，salt的目的是为了确保相同的密码生成不同的秘钥
加密  
`openssl enc -des3 -salt -in file.txt -out file.enc -pass pass:mypass -p`
`openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass pass:mypass -p`
解密
`openssl enc -d -des3 -in file.enc `

## rand

生成随机数  
`openssl rand -base64 24`

## dgst  
消息验证码的生成  

`openssl dgst -sha1|md5|sha256 file.txt`   
`openssl dgst -sha1 -out file.hash file.txt`   

生成hmac值，hmac值给予给定的对称秘钥生成
`openssl dgst -sha1 -hmac "mykey" -out file.hmac file.txt`

## s_client

### Check certification expired date 
openssl s_client -servername NAME -connect HOST:PORT 2>/dev/null | openssl x509 -noout -dates

### Who the certification issue to 
openssl s_client -servername shellhacks.com -connect shellhacks.com:443 2>/dev/null | openssl x509 -noout -subject -issuer

### Show the sha1 finger print
openssl s_client -servername www.shellhacks.com -connect www.shellhacks.com:443 2>/dev/null | openssl x509 -noout -fingerprint
