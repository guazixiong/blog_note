# Mysql8 连接Authentication plugin 'caching_sha2_password' cannot be loaded

通过navaciat远程连接mysql,提示

```bash
Authentication plugin 'caching_sha2_password' cannot be loaded
```



## 解决方式一: 修改为mysql_native_password模式

将 MySQL 8.0 配置为在mysql_native_password模式下运行，则可以解决此问题;

修改my.ini配置，重启mysql

```bash
[mysqld]

default_authentication_plugin=mysql_native_password
```



## 解决方式二: 修改加密方式

修改密码加密方式,重启mysql

```bash
ALTER USER 'yourusername'@'localhost' IDENTIFIED WITH mysql_native_password BY 'youpassword';
```



> mysql - 无法加载身份验证插件“caching_sha2_password”- 堆栈溢出 --- mysql - Authentication plugin 'caching_sha2_password' cannot be loaded - Stack Overflow: https://stackoverflow.com/questions/49194719/authentication-plugin-caching-sha2-password-cannot-be-loaded