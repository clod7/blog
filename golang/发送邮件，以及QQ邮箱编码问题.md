# golang发送邮件，以及QQ邮箱编码问题

## 发送邮件
### 代码如下。

```
package main

import (
	"bytes"
	"encoding/base64"
	"fmt"
	"io/ioutil"
	"log"
	"mime"
	"net/smtp"
	"strings"
	"time"
)

type Mail interface {
	Auth()
	Send(message Message) error
}

type SendMail struct {
	user     string
	password string
	host     string
	port     string
	auth     smtp.Auth
}

type Attachment struct {
	name        string
	contentType string
	withFile    bool
}

type Message struct {
	from        string    // 发送者邮箱
	to          []string  // 接收者邮箱
	cc          []string  // 抄送
	bcc         []string
	subject     string    // 标题
	body        string    // 内容
	contentType string
	attachment  Attachment // 附件
}

func main() {
	log.SetFlags(log.LstdFlags | log.Lshortfile)

	user := "xxxxx@aliyun.com"
	password := "xxxxxx"
	host := "smtpdm.aliyun.com"
	port := "25"

	var mail Mail
	mail = &SendMail{user: user, password: password, host: host, port: port}

	message := Message{
		from:        user,
		to:          []string{"xxxx@qq.com", "xxxx@163.com"},
		cc:          []string{},
		bcc:         []string{},
		subject:     "subject",
		body:        "body",
		contentType: "text/plain;charset=UTF-8",
		// 添加附件
		attachment: Attachment{
			name:        "xxxx.text",
			contentType: "application/octet-stream",
			withFile:    true,
		},
	}

	err := mail.Send(message)
	if err != nil {
		log.Println("Send mail error!")
		log.Println(err)
	} else {
		log.Println("Send mail success!")
	}
}
func (mail *SendMail) Auth() {
	mail.auth = LoginAuth(mail.user, mail.password)
}

func (mail SendMail) Send(message Message) error {
	mail.Auth()

	buffer := bytes.NewBuffer(nil)
	// 信头信息
	boundary := "GoBoundary"
	Header := make(map[string]string)
	Header["From"] = message.fromContent
	Header["To"] = strings.Join(message.to, ";")
	Header["Cc"] = strings.Join(message.cc, ";")
	Header["Bcc"] = strings.Join(message.bcc, ";")
	Header["Subject"] = message.subjectContent
	Header["Content-Type"] = "multipart/mixed;boundary=" + boundary
	Header["Mime-Version"] = "1.0"
	Header["Date"] = time.Now().String()
	mail.writeHeader(buffer, Header)

	// 内容信息
	body := "\r\n--" + boundary + "\r\n"
	body += "Content-Type:" + message.contentType + "\r\n"
	body += "\r\n" + message.body + "\r\n"
	buffer.WriteString(body)

	// 附件信息
	if message.attachment.withFile {
		attachment := "\r\n--" + boundary + "\r\n"
		attachment += "Content-Transfer-Encoding:base64\r\n"
		attachment += "Content-Disposition:attachment\r\n"
		attachment += "Content-Type:" + message.attachment.contentType + ";name=\"" + mime.BEncoding.Encode("UTF-8", message.attachment.name) + "\"\r\n"
		buffer.WriteString(attachment)
		defer func() {
			if err := recover(); err != nil {
				log.Fatalln(err)
			}
		}()
		mail.writeFile(buffer, message.attachment.name)
	}

	to_address := MergeSlice(message.to, message.cc)
	to_address = MergeSlice(to_address, message.bcc)
	buffer.WriteString("\r\n--" + boundary + "--")
	
	// 发送邮件
	err := smtp.SendMail(mail.host+":"+mail.port, mail.auth, message.from, to_address, buffer.Bytes())
	return err
}

// 合并切片
func MergeSlice(s1 []string, s2 []string) []string {
	slice := make([]string, len(s1)+len(s2))
	copy(slice, s1)
	copy(slice[len(s1):], s2)
	return slice
}

func (mail SendMail) writeHeader(buffer *bytes.Buffer, Header map[string]string) string {
	header := ""
	for key, value := range Header {
		header += key + ":" + value + "\r\n"
	}
	header += "\r\n"
	buffer.WriteString(header)
	return header
}

// 读写文件到缓冲区
func (mail SendMail) writeFile(buffer *bytes.Buffer, fileName string) {
	file, err := ioutil.ReadFile(fileName)
	if err != nil {
		panic(err.Error())
	}
	
	payload := make([]byte, base64.StdEncoding.EncodedLen(len(file)))
	base64.StdEncoding.Encode(payload, file)
	
	buffer.WriteString("\r\n")
	for index, line := 0, len(payload); index < line; index++ {
		buffer.WriteByte(payload[index])
		if (index+1)%76 == 0 {
			buffer.WriteString("\r\n")
		}
	}
}

type loginAuth struct {
	username, password string
}

func LoginAuth(username, password string) smtp.Auth {
	return &loginAuth{username, password}
}

func (a *loginAuth) Start(server *smtp.ServerInfo) (string, []byte, error) {
	return "LOGIN", []byte(a.username), nil
}

func (a *loginAuth) Next(fromServer []byte, more bool) ([]byte, error) {
	if more {
		switch string(fromServer) {
		case "Username:":
			return []byte(a.username), nil
		case "Password:":
			return []byte(a.password), nil
		}
	}
	return nil, nil
}

```

如果邮件发送者或者邮件标题有中文，则除QQ邮箱之外别的邮箱都可以完整收到邮件，发送至QQ邮箱中的邮件只会出现在你的垃圾箱中。这肯定是不能接受的。

## 原因

### 与其它邮件比较
在QQ邮箱中与别的邮件比较，外表看不出任何问题。
问题只能出现在信头中了。

### 如何查看信头
在QQ邮箱中，点击右上的>>符号 -> 显示邮件原文则可以看见原文。

### 比较信头
查看信头发现：
垃圾箱中的邮件信头：
```
To:xxx@qq.com;xxx@163.com
Subject:HELLO WORLD
From:鏃╁畨 <noreply@email.letabc.com>
```
完整邮件信头：
```
Subject:=?UTF-8?B?5pep5aS6K6y5LmJ?=
Content-Type:multipart/mixed;boundary=GoBoundary
From:=?UTF-8?B?5pea2m5b6S?= <xxx@aliyun.com>
To:xxx@qq.com;xxx@163.com
```

有变化的只有subject和from这两个值，完整的信头都是有一定的格式所构成的。

### 格式
```
Subject:=?UTF-8?B?base64编码后的字符串?=
From:=?UTF-8?B?base64编码后的字符串?= <xxx@aliyun.com>
```


## 修改代码
在Message中加入两个值，一个是编码后的接收者邮箱，另外一个是编码后的标题
```
type Message struct {
	from        string    // 发送者邮箱
	to          []string  // 接收者邮箱
	cc          []string  // 抄送
	bcc         []string
	subject     string    // 标题
	body        string    // 内容
	contentType string
	attachment  Attachment // 附件

	fromContent    string  // 编码后的接收者邮箱
	subjectContent string  // 编码后的标题
}
```

在Main函数中加上这两行代码
```
message.fromContent = fmt.Sprintf("=?UTF-8?B?%s?= <%s>", base64.StdEncoding.EncodeToString([]byte(message.from)), user)
message.subjectContent = fmt.Sprintf("=?UTF-8?B?%s?=", base64.StdEncoding.EncodeToString([]byte(message.subject)))
```

替换Send方法中编写信头时的值
```
Header["From"] = message.fromContent
Header["Subject"] = message.subjectContent
```

# 结束
这样修改后就可以完美的兼容QQ邮箱了。
在信头中不一定要求使用UTF-8来做编码，也可以使用GBK来编码也是可以的。