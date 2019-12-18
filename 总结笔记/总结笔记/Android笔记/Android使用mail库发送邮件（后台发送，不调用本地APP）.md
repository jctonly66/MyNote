今天接到一个任务：在Android端实现邮箱注册账号的功能，so决定先实现一个邮件发送器。下面是实现该demo的具体步骤，在此分享出来。

在Andoird中，实现发送邮件主要有两种方式：

1. 调用其他APP（QQ邮箱、gmail邮箱、自带邮箱等）。
2. 直接使用第三方服务器发送邮件。(就是直接使用QQ邮箱它们的服务器)。

如果使用第一种方式实现的话，需要有其他APP的支持，用户体验极差。在本文中，将使用第三方服务器（163邮箱）进行邮件发送APP的实现。

首先，使用sun公司开源的支持android端发送邮件的开源库。导入方法如下：
	
	compile 'com.sun.mail:android-mail:1.6.0'
	compile 'com.sun.mail:android-activation:1.6.0'

主要代码如下：

**邮件实体类：**

	import java.util.Properties;
	
	public class MailSender {
	    private String mailServerHost;  //发送邮件的服务器IP
	    private String mailServerPort; //发送邮件的服务器端口
	    private String username;    //邮件服务器用户名
	    private String password;    //邮件服务器密码
	    private String fromAddress; //发送者地址
	    private String toAddress;   //接收者地址
	    private boolean validate = false;   //是否需要身份认证
	    private String subject; //邮件主题
	    private String content; //邮件内容
	    private String[] attachFileNames;   //附件
	
	    public MailSender(String mailServerHost, String mailServerPort, String username, String password,
	                          String fromAddress, String toAddress, boolean validate,
	                          String subject, String content, String[] attachFileNames) {
	        this.mailServerHost = mailServerHost;
	        this.mailServerPort = mailServerPort;
	        this.username = username;
	        this.password = password;
	        this.fromAddress = fromAddress;
	        this.toAddress = toAddress;
	        this.validate = validate;
	        this.subject = subject;
	        this.content = content;
	        this.attachFileNames = attachFileNames;
	    }
	
	    /**
	     * 获取邮件相关配置
	     * @return Properties
	     */
	    public Properties getProperties() {
	        Properties p = new Properties();
	        p.put("mail.smtp.host", this.mailServerHost);
	        p.put("mail.smtp.port", this.mailServerPort);
	        p.put("mail.smtp.auth", validate ? "true" : "false");
	        return p;
	    }
	
	    public String getMailServerHost() {
	        return mailServerHost;
	    }
	
	    public String getMailServerPort() {
	        return mailServerPort;
	    }
	
	    public String getUsername() {
	        return username;
	    }
	
	    public String getPassword() {
	        return password;
	    }
	
	    public String getFromAddress() {
	        return fromAddress;
	    }
	
	    public String getToAddress() {
	        return toAddress;
	    }
	
	    public boolean isValidate() {
	        return validate;
	    }
	
	    public String getSubject() {
	        return subject;
	    }
	
	    public String getContent() {
	        return content;
	    }
	
	    public String[] getAttachFileNames() {
	        return attachFileNames;
	    }
	}

**密码认证器类：**

	public class MyAuthenticator extends Authenticator {
	    String username = null;
	    String password = null;
	
	    public MyAuthenticator() {}
	
	    public MyAuthenticator(String username, String password) {
	        this.username = username;
	        this.password = password;
	    }
	
	    protected PasswordAuthentication getPasswordAuthentication() {
	        return new PasswordAuthentication(username, password);
	    }
	}

**邮件发送工具类**

	public class MailUtils {
	
	    /**
	     * 发送文本类型的邮件
	     * @param mailSender 待发送的mailSender对象
	     * @return 是否发送成功
	     */
	    public static boolean sendTextMail(MailSender mailSender) {
	
	        // 身份认证器类
	        MyAuthenticator authenticator = null;
	        if (mailSender.isValidate()) {
	            authenticator = new MyAuthenticator(mailSender.getUsername(),
	                    mailSender.getPassword());
	        }
	
	        // 邮件相关配置
	        Properties properties = mailSender.getProperties();
	        // 根根配置以及验证器构造一个发送邮件的session
	        Session sendMailSession = Session.getDefaultInstance(properties, authenticator);
	
	        try {
	            // 根据生成的session创建一个待发送的消息
	            Message mailMessage = new MimeMessage(sendMailSession);
	            Address from = new InternetAddress(mailSender.getFromAddress());
	            // 设置邮件发送者的地址
	            mailMessage.setFrom(from);
	            Address to = new InternetAddress(mailSender.getToAddress());
	            // 设置邮件接收者的地址
	            mailMessage.setRecipient(Message.RecipientType.TO, to);
	            // 设置邮件消息的标题
	            mailMessage.setSubject(mailSender.getSubject());
	            // 设置邮件消息发送的时间
	            mailMessage.setSentDate(new Date());
	            String mailContent = mailSender.getContent();
	            // 设置发送的正文文本
	            mailMessage.setText(mailContent);
	            // 发送邮件
	            Transport.send(mailMessage);
	            return true;
	        } catch (AddressException e) {
	            e.printStackTrace();
	        } catch (MessagingException e) {
	            e.printStackTrace();
	        }
	        return false;
	    }
	
	    /**
	     * 发送HTML格式的邮件
	     * @param mailSender 待发送的邮件对象
	     * @return 是否发送成功
	     */
	    public static boolean sendHtmlMail(MailSender mailSender) {
	        // 身份认证器类
	        MyAuthenticator authenticator = null;
	        // 如果需要身份认证，则创建一个密码验证器
	        if (mailSender.isValidate()) {
	            authenticator = new MyAuthenticator(mailSender.getUsername(), mailSender.getPassword());
	        }
	        // 相关配置
	        Properties pro = mailSender.getProperties();
	        // 根据邮件相关配置和密码验证器构造一个发送邮件的session
	        Session sendMailSession = Session.getDefaultInstance(pro, authenticator);
	        try {
	            // 根据session创建一个待发送的邮件消息
	            Message mailMessage = new MimeMessage(sendMailSession);
	            Address from = new InternetAddress(mailSender.getFromAddress());
	            // 设置邮件发送者的地址
	            mailMessage.setFrom(from);
	            Address to = new InternetAddress(mailSender.getToAddress());
	            // 设置邮件接收者的地址
	            mailMessage.setRecipient(Message.RecipientType.TO, to);
	            // 设置邮件消息的标题
	            mailMessage.setSubject(mailSender.getSubject());
	            // 设置邮件消息发送的时间
	            mailMessage.setSentDate(new Date());
	            // MiniMultipart类是一个容器类，包含MimeBodyPart类型的对象
	            Multipart mainPart = new MimeMultipart();
	            // 创建一个包含HTML内容的MimeBodyPart
	            BodyPart html = new MimeBodyPart();
	            // 设置HTML内容
	            html.setContent(mailSender.getContent(), "text/html; charset=utf-8");
	            mainPart.addBodyPart(html);
	            // 将MiniMultipart对象设置为邮件内容
	            mailMessage.setContent(mainPart);
	            // 发送邮件
	            Transport.send(mailMessage);
	            return true;
	        } catch (MessagingException ex) {
	            ex.printStackTrace();
	        }
	        return false;
	    }
	}

至此，主要的代码都已完成。接下来完成测试：

	public class MainActivity extends AppCompatActivity {
	    private Button mBtSendText;
	    private Button mBtSendHtml;
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        mBtSendText = (Button) findViewById(R.id.bt_send_text);
	        mBtSendHtml = (Button) findViewById(R.id.bt_send_html);
	
	        mBtSendText.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                sendMessage(0, "请看这个文本邮件");
	            }
	        });
	
	        mBtSendHtml.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                //如果发送HTML类型的邮件，需要时HTML格式的文本，否则会出错。（自行百度HTML邮件与文本邮件的区别）
	                sendMessage(1, "<html><head>请看这个HTML邮件</head></html>");
	            }
	        });
	    }
	
	    /**
	     * 发送邮件
	     * @param type 0：文本邮件   1：HTML邮件
	     * @param msg   发送内容
	     */
	    private void sendMessage(final int type, final String msg) {
	
	        Log.d("TAG--", "发送邮件");
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	
	                MailSender mailSender = new MailSender(
	                        "smtp.163.com",     //服务器地址
	                        "25",               //服务器端口
	                        "xxxxxx@163.com",   //你的邮箱
	                        "xxxxxx",           //你的邮箱密码或者授权码
	                        "xxxxxx@163.com",   //你的地址 （一般与你的邮箱相同）
	                        "xxxxxx@qq.com",    //目的地址
	                        true,               //是否身份认证
	                        "邮件",              //邮件标题
	                        msg,                //邮件正文
	                        null);              //附件
	
	                boolean isSuccess = false;
	                if (type == 0) {
	                    isSuccess = MailUtils.sendTextMail(mailSender);// 发送文体格式
	                } else if (type == 1) {
	                    isSuccess = MailUtils.sendHtmlMail(mailSender);// 发送文体格式
	                }
	                if (isSuccess) {
	                    Log.i("TAG--", "发送成功");
	                } else {
	                    Log.i("TAG--", "发送失败");
	                }
	            }
	        }).start();
	    }
	}

**注意：**邮箱密码可能需要填写授权码。一般在邮箱设置里面开启SMTP/POP3等服务时会进行设置。如下图：
![](https://i.imgur.com/sLvYcR7.png)

>若发送HTML邮件时出现错误，请确保发送内容为HTML格式！

至此，一个简单的支持后台发送邮件的demo已经完成。有许多不足，比如抄送，群发等，以后有机会的会一定补上。

>demo的开发借鉴了[该网友的博文，点击传送](http://blog.csdn.net/aowowolf/article/details/53432561)

如果报错的话记得添加网络权限：
`<uses-permission android:name="android.permission.INTERNET"/>`


10/20/2017 2:02:38 PM 