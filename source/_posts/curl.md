---
title: curl命令的使用
date: 2021-02-24 20:32:14
tags:
  - Linux
  - CMD
---

## curl

[curl命令](http://curl.haxx.se/) 是一个利用URL规则在命令行下工作的文件传输工具。它支持文件的上传和下载，所以是综合传输工具，但按传统，习惯称curl为下载工具。作为一款强力工具，curl支持包括HTTP、HTTPS、ftp等众多协议，还支持POST、cookies、认证、从指定偏移处下载部分文件、用户代理字符串、限速、文件大小、进度条等特征。

## 语法

```shell
curl(选项)(参数)
```

## 常用命令

### 1. 查看网页源码

直接在curl命令后加上网址，就可以看到网页源码。以百度为例：

```shell
λ curl baidu.com

<html>
<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
</html>
```

如果要把这个网页保存下来，可以使用`-o`参数，这就相当于使用wget命令了。

```shell
λ curl -o [文件名] baidu.com
```

### 2. 自动跳转

有的网址是自动跳转的，使用`-L`参数，curl就会跳转到新的网址。

```shell
λ curl -L www.sina.com
```

 www.sina.com会跳转到 www.sina.com.cn。

### 3. 显示头信息

`-i`参数可以显示http response的头信息，连同网页代码一起。

```shell
λ curl -i baidu.com

HTTP/1.1 200 OK
Date: Wed, 24 Feb 2021 13:22:34 GMT
Server: Apache
Last-Modified: Tue, 12 Jan 2010 13:48:00 GMT
ETag: "51-47cf7e6ee8400"
Accept-Ranges: bytes
Content-Length: 81
Cache-Control: max-age=86400
Expires: Thu, 25 Feb 2021 13:22:34 GMT
Connection: Keep-Alive
Content-Type: text/html

<html>
<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
</html>
```

`-I`参数则是只显示http response的头信息。

```shell
λ curl -I baidu.com

HTTP/1.1 200 OK
Date: Wed, 24 Feb 2021 13:23:37 GMT
Server: Apache
Last-Modified: Tue, 12 Jan 2010 13:48:00 GMT
ETag: "51-47cf7e6ee8400"
Accept-Ranges: bytes
Content-Length: 81
Cache-Control: max-age=86400
Expires: Thu, 25 Feb 2021 13:23:37 GMT
Connection: Keep-Alive
Content-Type: text/html
```

### 4. 显示通信过程

`-v`参数可以显示一次http通信的整个过程，包括端口连接和http request头信息。

```shell
λ curl -v baidu.com

* Rebuilt URL to: baidu.com/
*   Trying 220.181.38.148...
* TCP_NODELAY set
* Connected to baidu.com (220.181.38.148) port 80 (#0)
> GET / HTTP/1.1
> Host: baidu.com
> User-Agent: curl/7.55.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Wed, 24 Feb 2021 13:24:27 GMT
< Server: Apache
< Last-Modified: Tue, 12 Jan 2010 13:48:00 GMT
< ETag: "51-47cf7e6ee8400"
< Accept-Ranges: bytes
< Content-Length: 81
< Cache-Control: max-age=86400
< Expires: Thu, 25 Feb 2021 13:24:27 GMT
< Connection: Keep-Alive
< Content-Type: text/html
<
<html>
<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
</html>
* Connection #0 to host baidu.com left intact
```

下面的命令可以查看更详细的通信过程

```shell
λ curl --trace output.txt baidu.com
```

或者

```shell
λ curl --trace-ascii output.txt baidu.com
```

运行后，请打开output.txt文件查看。

### 5. 发送表单信息

发送表单信息有GET和POST两种方法。GET方法相对简单，只要把数据附在网址后面就行。

```shell
λ curl example.com/form.cgi?data=xxx
```

POST方法必须把数据和网址分开，curl就要用到--data参数。

```shell
λ curl -X POST --data "data=xxx" example.com/form.cgi
```

可以让curl为你编码，参数是`--data-urlencode`。

```shell
λ curl -X POST--data-urlencode "date=April 1" example.com/form.cgi
```

### 6. HTTP动词

curl默认的HTTP动词是GET，使用`-X`参数可以支持其他动词。

```shell
λ curl -X POST www.example.com
```

```shell
λ curl -X DELETE www.example.com
```

### 7. 文件上传

假定文件上传的表单是下面这样：

```html
<form method="POST" enctype='multipart/form-data' action="upload.cgi">
    <input type=file name=upload>
    <input type=submit name=press value="OK">
</form>
```

可以用curl这样上传文件:

```shell
λ curl --form upload=@localfilename --form press=OK [URL]
```

### 8. Referer字段

有时需要在http request头信息中，提供一个referer字段，表示是从哪里跳转过来的。

```shell
λ curl --referer http://www.example.com http://www.example.com
```

### 9. User Agent字段

服务器有时会根据这个字段，针对不同设备，返回不同格式的网页，比如手机版和桌面版。

curl可以这样模拟：

```shell
λ curl --user-agent "[User Agent]" [URL]
```

### 10. cookie

使用`--cookie`参数，可以让curl发送cookie。

```shell
λ curl --cookie "name=xxx" www.example.com
```

至于具体的cookie的值，可以从http response头信息的`Set-Cookie`字段中得到。

`-c cookie-file`可以保存服务器返回的cookie到文件，`-b cookie-file`可以使用这个文件作为cookie信息，进行后续的请求。

```shell
λ curl -c cookies http://example.com
λ curl -b cookies http://example.com
```

### 11.  增加头信息

有时需要在http request之中，自行增加一个头信息。`--header`参数就可以起到这个作用。

```shell
λ curl --header "Content-Type:application/json" http://example.com
```

### 12. HTTP认证

有些网域需要HTTP认证，这时curl需要用到`--user`参数。

```shell
λ curl --user name:password example.com
```

## 选项

| 参数                                    | 说明                                                      |
| --------------------------------------- | --------------------------------------------------------- |
| -a/--append                             | 上传文件时，附加到目标文件                                |
| -A/--user-agent                         | 设置用户代理发送给服务器                                  |
| -anyauth                                | 可以使用“任何”身份验证方法                                |
| -b/--cookie                             | cookie字符串或文件读取位置                                |
| --basic                                 | 使用HTTP基本验证                                          |
| -B/--use-ascii                          | 使用ASCII /文本传输                                       |
| -c/--cookie-jar                         | 操作结束后把cookie写入到这个文件中                        |
| -C/--continue-at                        | 断点续传                                                  |
| -d/--data                               | HTTP POST方式传送数据                                     |
| --data-ascii                            | 以ascii的方式post数据                                     |
| --data-binary                           | 以二进制的方式post数据                                    |
| --negotiate                             | 使用HTTP身份验证                                          |
| --digest                                | 使用数字身份验证                                          |
| --disable-eprt                          | 禁止使用EPRT或LPRT                                        |
| --disable-epsv                          | 禁止使用EPSV                                              |
| -D/--dump-header                        | 把header信息写入到该文件中                                |
| --egd-file                              | 为随机数据(SSL)设置EGD socket路径                         |
| --tcp-nodelay                           | 使用TCP_NODELAY选项                                       |
| -e/--referer                            | 来源网址                                                  |
| -E/--cert                               | 客户端证书文件和密码 (SSL)                                |
| --cert-type                             | 证书文件类型 (DER/PEM/ENG) (SSL)                          |
| --key                                   | 私钥文件名 (SSL)                                          |
| --key-type                              | 私钥文件类型 (DER/PEM/ENG) (SSL)                          |
| --pass                                  | 私钥密码 (SSL)                                            |
| --engine                                | 加密引擎使用 (SSL). "--engine list" for list              |
| --cacert                                | CA证书 (SSL)                                              |
| --capath                                | CA目录 (made using c_rehash) to verify peer against (SSL) |
| --ciphers                               | SSL密码                                                   |
| --compressed                            | 要求返回是压缩的形势 (using deflate or gzip)              |
| --connect-timeout                       | 设置最大请求时间                                          |
| --create-dirs                           | 建立本地目录的目录层次结构                                |
| --crlf                                  | 上传是把LF转变成CRLF                                      |
| -f/--fail                               | 连接失败时不显示http错误                                  |
| --ftp-create-dirs                       | 如果远程目录不存在，创建远程目录                          |
| --ftp-method [multicwd/nocwd/singlecwd] | 控制CWD的使用                                             |
| --ftp-pasv                              | 使用 PASV/EPSV 代替端口                                   |
| --ftp-skip-pasv-ip                      | 使用PASV的时候,忽略该IP地址                               |
| --ftp-ssl                               | 尝试用 SSL/TLS 来进行ftp数据传输                          |
| --ftp-ssl-reqd                          | 要求用 SSL/TLS 来进行ftp数据传输                          |
| -F/--form                               | 模拟http表单提交数据                                      |
| --form-string                           | 模拟http表单提交数据                                      |
| -g/--globoff                            | 禁用网址序列和范围使用{}和[]                              |
| -G/--get                                | 以get的方式来发送数据                                     |
| -H/--header                             | 自定义头信息传递给服务器                                  |
| --ignore-content-length                 | 忽略的HTTP头信息的长度                                    |
| -i/--include                            | 输出时包括protocol头信息                                  |
| -I/--head                               | 只显示请求头信息                                          |
| -j/--junk-session-cookies               | 读取文件进忽略session cookie                              |
| --interface                             | 使用指定网络接口/地址                                     |
| --krb4                                  | 使用指定安全级别的krb4                                    |
| -k/--insecure                           | 允许不使用证书到SSL站点                                   |
| -K/--config                             | 指定的配置文件读取                                        |
| -l/--list-only                          | 列出ftp目录下的文件名称                                   |
| --limit-rate                            | 设置传输速度                                              |
| --local-port                            | 强制使用本地端口号                                        |
| -m/--max-time                           | 设置最大传输时间                                          |
| --max-redirs                            | 设置最大读取的目录数                                      |
| --max-filesize                          | 设置最大下载的文件总量                                    |
| -M/--manual                             | 显示全手动                                                |
| -n/--netrc                              | 从netrc文件中读取用户名和密码                             |
| --netrc-optional                        | 使用 .netrc 或者 URL来覆盖-n                              |
| --ntlm                                  | 使用 HTTP NTLM 身份验证                                   |
| -N/--no-buffer                          | 禁用缓冲输出                                              |
| -o/--output                             | 把输出写到该文件中                                        |
| -O/--remote-name                        | 把输出写到该文件中，保留远程文件的文件名                  |
| -p/--proxytunnel                        | 使用HTTP代理                                              |
| --proxy-anyauth                         | 选择任一代理身份验证方法                                  |
| --proxy-basic                           | 在代理上使用基本身份验证                                  |
| --proxy-digest                          | 在代理上使用数字身份验证                                  |
| --proxy-ntlm                            | 在代理上使用ntlm身份验证                                  |
| -P/--ftp-port                           | 使用端口地址，而不是使用PASV                              |
| -q                                      | 作为第一个参数，关闭 .curlrc                              |
| -Q/--quote                              | 文件传输前，发送命令到服务器                              |
| -r/--range                              | 检索来自HTTP/1.1或FTP服务器字节范围                       |
| --range-file                            | 读取（SSL）的随机文件                                     |
| -R/--remote-time                        | 在本地生成文件时，保留远程文件时间                        |
| --retry                                 | 传输出现问题时，重试的次数                                |
| --retry-delay                           | 传输出现问题时，设置重试间隔时间                          |
| --retry-max-time                        | 传输出现问题时，设置最大重试时间                          |
| -s/--silent                             | 静默模式。不输出任何东西                                  |
| -S/--show-error                         | 显示错误                                                  |
| --socks4                                | 用socks4代理给定主机和端口                                |
| --socks5                                | 用socks5代理给定主机和端口                                |
| --stderr                                |                                                           |
| -t/--telnet-option                      | Telnet选项设置                                            |
| --trace                                 | 对指定文件进行debug                                       |
| --trace-ascii                           | Like --跟踪但没有hex输出                                  |
| --trace-time                            | 跟踪/详细输出时，添加时间戳                               |
| -T/--upload-file                        | 上传文件                                                  |
| --url                                   | Spet URL to work with                                     |
| -u/--user                               | 设置服务器的用户和密码                                    |
| -U/--proxy-user                         | 设置代理用户名和密码                                      |
| -w/--write-out [format]                 | 什么输出完成后                                            |
| -x/--proxy                              | 在给定的端口上使用HTTP代理                                |
| -X/--request                            | 指定什么命令                                              |
| -y/--speed-time                         | 放弃限速所要的时间，默认为30                              |
| -Y/--speed-limit                        | 停止传输速度的限制，速度时间                              |



