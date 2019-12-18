在开发过程中，很多应用程序都需要通过邮件提醒用户，Flask的扩展包Flask-Mail通过包装了Python内置的smtplib包，可以用在Flask程序中发送邮件。

Flask-Mail连接到简单邮件协议（Simple Mail Transfer Protocol,SMTP）服务器，并把邮件交给服务器发送。

```python
from flask import Flask
from flask_mail import Mail, Message


app = Flask(__name__)
app.config.update(
    DEBUG=True,
    MAIL_SERVER='smtp.163.com',
    MAIL_PORT=465,
    MAIL_USE_SSL=True,  # 这里尤其注意加密方式是 SSL 还是 TLS
    MAIL_USERNAME='18435138818@163.com',
    MAIL_PASSWORD='jct1076417695JCT',
)

mail = Mail(app)


@app.route('/')
def index():
    msg = Message("This is a test", sender='18435138818@163.com', recipients=['chentao.jia@ff.com', ])
    msg.body = "Flask test mail"
    mail.send(msg)
    print("Mail Send")
    return "Send success"


if __name__ == '__main__':
    app.run()
```

