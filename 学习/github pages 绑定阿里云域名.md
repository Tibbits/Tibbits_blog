# github pages 绑定阿里云域名

- 购买好域名后对该域名进行解析

![image-20230901162124954](/media/alicloud.png)

- 向你的 DNS 配置中添加 3 条记录

  > @          A             192.30.252.153
  > @          A             192.30.252.154
  > www      CNAME    username.github.io.（用你自己的 Github 用户名替换 username）

![image-20230901162405165](/media/alicloud2.png)

- 通过ping -4 username.github.io（IPV4）获取ip地址
- ![image-20230901162812390](/media/alicloud3.png)

- 仓库添加一个CNAME(一定要**大写**)文件其中只能包含一个顶级域名：

  > example.com



- 在github pages 添加弄好的域名

![](/media/github.png)