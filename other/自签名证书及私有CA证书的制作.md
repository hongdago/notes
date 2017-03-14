# 自签名证书和私有CA证书的制作
## 基本概念
### 证书类型
* PEM(privacy-enhanced Electronic Mail)：明文格式的，以----BEGING CERTIFICATE ----开头，以
----END CERTIFICATE ----结尾，中间是经过basee64编码的内容。
  + 查看证书信息命令：`openssl x509 -noout -text -in some.pem`

* DER：二进制格式证书
  + 查看证书信息命令：`openssl x509 -noout -text -inform der -in some.der`

### 证书后缀名
* `.crt`：证书文件，可以是DER编码的，也可以是PEM编码的。
* `.cer`：证书文件，可以是DER编码的，也可以是PEM编码的，常见于windows系统。
* `.csr`：证书签名请求，一般是生成后发送给CA，然后CA会给你签名并返回证书。
* `.key`：公钥或密钥，可以是DER编码的，也可以是PEM编码的
  + 查看DER格式：`openssl rsa -inform DER -noout -text -in some.key`
  + 查看PEM格式：`openssl rsa -inform PEM -noour -text -in some.key`

## 操作步骤
### 自签名证书的制作
1. 生成服务器私钥
  + `openssl genrsa -des3 -out server.key 4096`
2. 生成证书签名请求
  + `openssl req -new -key server.key -out server.csr`
  + `保证Common name跟你的域名或者IP相同`
3. 对证书签名请求进行签名
  + `openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt`
4. 生成无需密码的服务器私钥
  + `openssl rsa -in server.key -out server.key.insecure`
  + `mv server.key server.key.secure`
  + `mv server.key.insecure server.key`
5. 简单的方法一步生成私钥及自签名证书
  + `openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout server.key -out server.crt`
### 私有CA签名证书的制作
1. 创建CA私钥
  + `openssl genrsa -des3 -out ca.key 4096`
  + `可选步骤，删除私钥中的密码：openssl rsa -in ca.key -out ca.key`
2. 生成CA的自签名证书
  + `openssl req -new -x509 -days 365 -key ca.key -out ca.crt` 
  + 其实CA证书就是一个自签名证书
3. 生成服务端私钥
  + `openssl genres -des3 -out server 4096`
4. 需要签名的证书(服务端)生成证书签名请求
  + `openssl req -new -key server.key -out server.csr`
  + 证书签名请求中的Common Name必须区别与CA的证书里面的Common Name
5. 用CA证书给步骤四中的签名请求进行签名
  + `openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt`

## 查看命令
* 查看私钥信息
  + `openssl rsa -noout -text -in server.key`
* 查看签名请求信息
  + `openssl req -noout -text -in server.csr`
* 查看证书信息
  + `openssl x509 -noout -text -in server.crt`
* 将crt文件转换为pem
  + `openssl x509 -in server.crt -out server.pem -outform PEM`
* 验证一个证书是否是某一个CA签发
  + `openssl verify -CAfile ca.crt server.crt`
* 模拟一个ssl客户端访问ssl服务器
  + `openssl s_client -connect 192.168.0.1:443 -cert client.crt -key client.key`

