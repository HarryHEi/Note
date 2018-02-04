---
title: flask_send_mail
date: 2016-03-13 14:46:29
tags: [flask]
---

# 配置
## SMTP服务器配置

|配置| 默认值 | 说明 |
|-----|---------|--------|
| MAIL_SERVER | localhost | 电子邮件服务器的主机名或 IP 地址 |
| MAIL_PORT | 25 | 电子邮件服务器的端口 |
| MAIL_USE_TLS | False | 启用传输层安全(Transport Layer Security,TLS)协议 |
| MAIL_USE_SSL | False | 启用安全套接层(Secure Sockets Layer,SSL)协议 |
| MAIL_USERNAME | None | 邮件账户的用户名 |
| MAIL_PASSWORD | None | 邮件账户的密码 |

## 保存密码到环境变量
```
(venv) $ set MAIL_USERNAME=<Gmail username>
(venv) $ set MAIL_PASSWORD=<Gmail password>
```

## 使用gmail发送邮件配置
```
app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME')
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD')
app.config['FLASKY_MAIL_SUBJECT_PREFIX'] = '[Flasky]'
app.config['FLASKY_MAIL_SENDER'] = '1099143767@qq.com'
app.config['FLASKY_ADMIN'] = os.environ.get('FLASKY_ADMIN')
```
# 发送邮件
```
app.config['FLASKY_MAIL_SUBJECT_PREFIX'] = '[Flasky]'
app.config['FLASKY_MAIL_SENDER'] = 'Flasky Admin <flasky@example.com>'

msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + subject,
                          sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
msg.body = render_template(template + '.txt', **kwargs)
msg.html = render_template(template + '.html', **kwargs)
mail.send(msg)
```
## 异步发送邮件
```
def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)

def send_email(to, subject, template, **kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + ' ' + subject,
                  sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    thr = Thread(target=send_async_email, args=[app, msg])
    thr.start()
    return thr
```
## 发送邮件
```
    send_email(app.config['FLASKY_ADMIN'], 'New User',
               'mail/new_user', user=user)
```
目标地址是环境变量中的FLASKY_ADMIN，主题是[flasky]New User，
调用mail目录下的模板，传递参数user。

```
>>> from flask.ext.mail import Message
>>> from flask.ext.mail import Mail
>>> mail=Mail()
>>> msg=Message("test",sender="harryx520@qq.com",recipients=["harryx520@qq.com"])
>>> mail.send(msg)
```
