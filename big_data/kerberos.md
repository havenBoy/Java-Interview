## kerberos
> 认证（kerberos）+授权（那些人有权限访问，那些人没有权限访问--ranger）
> 是一种计算机网络认证协议。

- 一些术语
  - KDC：秘钥分发中心，存储用户信息，管理发放票据（用户访问某个服务的认证信息）
    1. AS，认证服务器
    2. DataBase，存储各个用户与服务的Principle
    3. TGS，票证授权服务器，拿到这个服务的票据才可以访问
  - Realm：所管理的一个领域或者范围
  - Principal：所管理的一个用户或者服务，格式通常为primary/instance@realm
    1. 用户信息如xiong@163.com
    2. 服务信息如2nn/node1@163.com
  - keytab：用户的认证信息，是指秘钥文件。
- 认证原理
  1. 进行认证，kinit,输入用户与密码，
  2. 访问数据库信息，判断认证信息是否合法，如果合法返回TGT
  3. 客户端会拿着TGT去访问TGS服务，判断TGT是否有效合法，如果合法，返回server ticket
  4. 这样客户端就可以去相关服务访问
- 安装
  1. yum install -y krb5-server，安装在服务端
  2. yum install -y krb5-workstation krb5-libs，安装在客户端
  3. 修改服务的配置文件在主节点，路径为/var/kerberos/krb5kdc/kdc.conf，需要修改realms，为公司网址地址
  4. 修改客户端的配置文件在各个节点，路径为/etc/krb5
    - 加参数dns_look_up=false
    - default_realm=EXAMPLE.COM，这个是realm默认值
    - 修改realms的内容，EXAMPLE.COM = {  
         kdc = node1  
         admin_server = node1    
    }
  5. 分发配置项到每个节点
  6. 初始化KDC数据库，执行kdb5_util create - s ,输入数据库的密码
  7. 修改kerberos的管理员配置文件，路径为/etc/kerberos/krb5kdc/kadm5.acl
  7. 启动主节点的KDC与kadmin，配置开机自启,分别为systemctl start/enable krbkdc,systemctl start/enable kadmin
  8. 在主节点创建管理员用户，kadmin.local -q "addprinc admin/admin"
  9. 
- 使用
  1. 注册操作，在数据库写入一个用户数据
    - 本地登录下为kadmin.local-> addprinc test(增加用户)-> list_principals(查询所有用户)->cpw test（修改用户的密码）-> delprinc test(删除用户)
    - 远程登录，kadmin，需要输入管理员用户与密码，使用quit退出
  2. 认证操作
    - kinit test -> klist(查看所有认证完毕的用户)，使用密码来进行认证
    - kadmin.local -q "-xst -norandkey -k /root/test.keytab test@EXAMPLE.com"
    - kinit -kt /root/test.keytab test  -> klist
    - kdestroy 销毁登录的票证

- 为hadoop开启认证
  > 为不同的服务创建不同的用户与组，在每个节点执行
  1. hdfs:hadoop --  namenode-snn-journalNode,datanode
  2. yarn:hadoop  --  resource_manager,node_manager
  3. mapred:hadoop -- jobHistory,mapreduce

- 为hadoop各个服务创建kerberos服务主体
  1. namenode
  2. secondnamenode
  3. resource_manager
  4. node_manager
  5. jobHistory_Server
  6. Web_UI
- 修改hadoop配置文件
  - core-site.xml
  - 



















