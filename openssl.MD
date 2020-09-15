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

## 公开秘钥算法RSA genrsa, rsautl，rsa

生成rsa 秘钥对  
`openssl genrsa -out mykey.pem 2048`
将公钥从秘钥对中取出，便于分享
`openssl rsa in mykey.pem -pubout -out mypubkey.pem`

### rsa用来加密是公钥加密，私钥解密
使用秘钥对加密文件 
`openssl rsautl -encrypt -inkey mykey.pem -in plain.txt -out cipher.txt`  
使用公钥加密文件 `-pubin` 表示`-inkey`接收的是一个公钥  
`openssl rsautl -encrypt -pubin -inkey mypubkey.pem -in plain.txt -out cipher.txt`  
使用私钥解密  
`openssl rsautl -decrypt -inkey mykey.pem -in cipher.txt`  

### rsa用来签名是私钥`签名`，公钥验证签名，这里是签名，不是加密  

使用私钥签名  
`openssl dgst -sha256 -sign mykey.pem -out sign.txt file.txt`  
使用公钥验证签名  
```
openssl dgst -sha256 -verify mykey.pub -signature sign.txt file.txt 
Verified OK
```  

## DSA 是NIST提出的签名算法，只能用于签名，不能用于加密

生成参数文件  
`openssl dsaparam -out dsaparam.pem 1024`  
通过参数文件生成秘钥对  
`openssl gendsa -out mydsa.pem dsaparam.pem`  
提取出公钥  
`openssl dsa -in mydsa.pem -pubout -out mydsapub.pem`  
签名  
`openssl dgst -sha256 -sign mydsa.pem -out sign.txt plain.txt`  
公钥验证签名  
`openssl dgst -sha256 -verify mydsapub.pem -signature sign.txt plain.txt`  

## s_client  模拟http客户端

### Check certification expired date 
`openssl s_client -servername NAME -connect HOST:PORT 2>/dev/null | openssl x509 -noout -dates`

### Check starttls certs

`openssl s_client -connect host:port -starttls smtp`

### Who the certification issue to
```
openssl s_client -servername shellhacks.com -connect shellhacks.com:443 2>/dev/null | openssl x509 -noout -subject -issuer
```
### Show the sha1 finger print
```
openssl s_client -servername www.shellhacks.com -connect www.shellhacks.com:443 2>/dev/null | openssl x509 -noout -fingerprint
```

## show public key

`openssl s_client -connect localhost:55000 | openssl x509 -pubkey -noout`

## extract certification from db2 p12/kdb and use in jcc

```
 gsk8capicmd_64 -cert -export -db sourcedb.kdb -stashed source.sth -label "xxx" -type p12 -target mycert.p12 
 openssl pkcs12 -in mycert.p12 -out test.pem -nokeys
```
 
 use test.pem in jcc connection. sslCertLocation=test.pem
 
 ## Get certification of a site  
 
 if server support sni
 ```
 openssl s_client -showcerts -servername <servername> -connect servername:443 2>/dev/null | openssl x509 > cert.pem.pem
 
 openssl s_client -showcerts -servername <servername> -connect servername:443 2>/dev/null | openssl x509  -outform DER > cert.pem.pem
 
 
 ```
 
 if server not support sni
 
 ```
 openssl s_client -showcerts -connect iam.cloud.ibm.com:443 2>/dev/null | openssl x509 > iamcert.pem
 ```
 
 add in keystore
 
 ```
 keytool -import -noprompt -trustcacerts -alias iamcerts -file ./iamcert.pem -keystore /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.91-0.b14.el6_7.x86_64/jre/lib/security/cacerts -storepass changeit
 ```
 
 ## Generate certification with `extendedKeyUsage` openssl-111
 
 ```
 openssl req -new -newkey rsa:2048 -x509 -days 730 -nodes  -out cert.crt -keyout cert.key -addext extendedKeyUsage=serverAuth -subj "/C=US/ST=CA/L=Silicon Valley Lab/O=Data and AI/OU=IBM Data Platform/emailAddress=xxxx"
 ```
 
## debug options 

```
openssl s_client -tlsextdebug -msg -connect xxxx:xxx
```
