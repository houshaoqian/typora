## MySQL５.７

1. 初始化root密码

   ~~~shell
   # 方法1
   mysql>update mysql.user set authentication_string=password("新密码");
   mysql>flush privileges;
   
   # 方法2 该方法支持root远程登录
   mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';  
   mysql> flush privileges;
   ~~~

   

