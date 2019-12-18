#####使用python连接AD

1. 代码：https://github.com/raykuan/msldap

   使用python的ldap模块连接AD服务器，有两种方式：

   非加密：con = ldap.initialize('ldap://myhost.com:389)

   加密(SSL)：con = ldap.initialize('ldaps://myhost.com:636)

   使用非加密的ldap时，只能对AD域账号信息查询。如果要对AD域用户信息进行修改和新增操作，必须使用（SSL）加密方式连接AD，需满足如下几个条件：

   ① AD上需要安装证书服务

   ② 连接AD的主机上使用http://myhost.com/certsrv/打开浏览器申请证书

   ③ 如果连接AD的主机是Linux，需要安装openssl包，制作CA证书

   \# 使用openssl命令把申请到的AD主机上的CA证书格式转换为.pem

   openssl x509 -in master.cer -inform der -outform pem -out master.pem

   \# 使用openssl命令指定CA证书连接到AD主机

   openssl s_client -connect myhost:636 -CAfile /path/to/master.pem

   \# 查看CN subject信息，python中ldap初始化的时间需要保证和CN的域名一致

   openssl x509 -noout -text -in imsva_cert.pem | grep Subject

   Subject: C=en, ST=xx, O=yy, OU=zz, CN=test.com

   con = ldap.initialize('ldaps://myhost.com')

   可以在/etc/hosts文件下增加域名解析记录

   ④ 在Python的ldap初始化连接AD前，设置CA证书文件加载

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
   import ldap
   # point to the cert
   cert_file='/path/to/master.pem'
   ldap.set_option(ldap.OPT_X_TLS_CACERTFILE, cert_file)
   con = ldap.initialize('ldaps://myhost.com')
   dn = 'CN=me,DC=myhost,DC=com'
   pw = 'password'
   con.simple_bind_s(dn, pw)
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

    注意：此处dn是有管理员权限的AD用户，通过绑定此用户才能使用ldaps对域用户的信息进行增删改，查询的时间一般使用非SSL连接，普通用户即可。