# java 邮件


## 概念

* 电子邮箱: 邮件服务器上的一片空间
* 服务器: 提供邮件服务
* 协议: 规定数据格式
    - `pop3`
    - `imap`
    - `smtp`


## Java发送邮件

* 依赖库
    - [JavaMail](http://www.oracle.com/technetwork/java/javamail/index.html)
    - [JavaBean Activation Framework(JAF)](http://www.oracle.com/technetwork/articles/java/index-135046.html)
* 依赖服务
    - Java只提供构成email的API, 并没有提供邮件服务器, 所以需要自己准备邮件服务器, 可以自己搭建或使用第三方


### 发送邮件

```java
// 用户验证信息
props.setProperty("mail.user", "username");
props.setProperty("mail.password", "password");


// 发送纯文本邮件
String to = "abc@gmail.com";
String from = "me@lixpolor.com";
String host = "localhost";
Properties p = System.getProperties();
p.setProperty("smtp.lixplor.com", host);
Seesion mailSession = Session.getDefaultInstance(p);

try {
    MimeMessage message = new MimeMessage(mailSession);
    message.setFrom(new InternetAddress(from));
    message.addRecipient(Message.RecipienType.TO, new InternetAddress(to));
    message.setSubject("这是标题");
    message.setText("这是内容");
    Transport.send(message);
} catch (MessagingException e) {

}


// 发送HTML邮件
String to = "abc@gmail.com";
String from = "me@lixpolor.com";
String host = "localhost";
Properties p = System.getProperties();
p.setProperty("smtp.lixplor.com", host);
Seesion mailSession = Session.getDefaultInstance(p);

try {
    MimeMessage message = new MimeMessage(mailSession);
    message.setFrom(new InternetAddress(from));
    message.addRecipient(Message.RecipienType.TO, new InternetAddress(to));
    message.setSubject("这是标题");
    message.setContent("<h1>HTML消息标题</h1>", "text/html");
    Transport.send(message);
} catch (MessagingException e) {

}


// 发送附件
String to = "abc@gmail.com";
String from = "me@lixpolor.com";
String host = "localhost";
Properties p = System.getProperties();
p.setProperty("smtp.lixplor.com", host);
Seesion mailSession = Session.getDefaultInstance(p);

try {
    MimeMessage message = new MimeMessage(mailSession);
    message.setFrom(new InternetAddress(from));
    message.addRecipient(Message.RecipienType.TO, new InternetAddress(to));
    message.setSubject("这是标题");

    // 创建附件
    Multipart m = new MimeMultipart();
    // 内容部分
    BodyPart messageBodyPart = new MimeBodyPart();
    messageBodyPart.setText("sdfsdaf");
    m.addBodyPart(messageBodyPart);
    // 附件部分
    messageBodyPart = new MimeBodyPart();
    String filename = "adb.txt";
    DataSource source = new FileDataSource(filename);
    messageBodyPart.setDataHandler(new DataHandler(source));
    messageBodyPart.setFileName(filename);
    m.addBodyPart(messageBodyPart);
    // 设置附件
    message.setContent(m);

    Transport.send(message);
} catch (MessagingException e) {

}
```
