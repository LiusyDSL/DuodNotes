---
title: Cookie学习
author: LiusyDD
date: 2020-03-11 10:46
---

#### 1.什么是Cookie

​	Cookie是存储在客户端的一段很小(不超过4KB)的文本信息，以Key-Value的形式存储。它具有Expires(过期时间)、Path、Domain、Secure、HTTPOnly 等属性信息。

#### 2.为什么使用Cookie

​	Web客户端与Web服务器之间是通过Http协议进行信息发送和响应，而Httpq请求是无状态的，也就是Web客户端与Web服务器之间不需要建立持久的连接，Web客户端发送请求(Request)，Web服务端进行请求响应(Response)并返回，连接就被关闭了，在服务器端没有留下任何有关联的信息。

​	当Web服务端需要记住客服端用户的相关信息的时候，服务端就通过Response向客户端返回一个Cookie记录客户端用户信息。客户端收到后就把Cookie保存起来，客户端下次发起请求的时候会自动的带上Cookie信息一同提交给服务器。服务器通过检查Cookie来检测用户的信息。

#### 3.Cookie的属性

| 属性名称   | 描述                                                         |
| :--------- | ------------------------------------------------------------ |
| Name/Value | 设置Cookie的名称及相对应的值，对于认证Cookie，Value值包括Web服务器所提供的访问令牌 |
| Expires    | 设置Cookie的生存期。有两种存储类型的Cookie：会话性与持久性。Expires缺省时为会话性存错，存储于客户端的内存中，随着浏览器的关闭而失效。持久性的保存在用户电脑的硬盘里随着手动删除和到达指定的过期时间而失效。 |
| Path       | 设置Cookie在那个路径下生成。                                 |
| Domain     | 设置Cookie在那个域名下可以访问。                             |
| Secure     | 设置Cookie是否使用Https安全协议发送。                        |
| HTTPOnly   | 防止客户端通过脚本访问Cookie信息。                           |

#### 4.Cookie生成机制

​	用户第一次登陆某个网站的时候，cookie得到设置及发送会经历以下步骤：

<img src="/images/cookie-jz.png" style="zoom:70%;" />



1. 客户端发送Http Request请求到服务端。
2. 服务端对客户端发送的请求进行响应并通过`response.addCookie()`方法设置相应的Cookie信息。
3. 客户端接收带有`Set-Cookie`属性的响应头信息，并把信息保存到Cookie。
4. 客户端再次发起Http Request请求时，客户端会自动带上cookie信息一起提交给服务器，请求头里包含`Cookie`属性。
5. 客户端在对请求进行响应返回。

​    纯靠的理论的知识是不能很好的记忆和理解的，只有通过堂堂正正的敲代码才能加深印象和理解。接下来我们通过代码来演示上面的流程。

1. 环境搭建

   开发工具：idea    jdk:1.8

   * 使用idea创建webappmaven工程

   <img src="/images/cookie-环境.jpg" style="zoom:43%;" />

   * pom.xml配置信息

     ```xml
     <!--添加Servlet依赖-->
     <dependency>
     	<groupId>javax.servlet</groupId>
     	<artifactId>javax.servlet-api</artifactId>
     	<version>3.1.0</version>
     	<scope>provided</scope>
     </dependency>
     <!--   tomcat插件   -->
     <plugin>
     	<groupId>org.apache.tomcat.maven</groupId>
     	<artifactId>tomcat7-maven-plugin</artifactId>
     	<version>2.2</version>
     	<configuration>
     		<path>/</path> <!-- 项目访问路径 本例：localhost:9090, 如果配置的aa，则访问路径为localhost:9090/aa -->
     		<port>8888</port>
     		<uriEncoding>UTF-8</uriEncoding><!-- 非必需项 -->
     	</configuration>
     </plugin>
     ```

2. 编写Servlet

   ```java
   public class MyCookieServlet extends HttpServlet {
       @Override
       protected void doGet(HttpServletRequest req, HttpServletResponse resp)
               throws ServletException, IOException {
           Cookie cookie = new Cookie("myCookie", "This is Hello Cookie");
           resp.addCookie(cookie);
           PrintWriter writer = resp.getWriter();
           writer.write("Test Cookie");
       }
   
       @Override
       protected void doPost(HttpServletRequest req, HttpServletResponse resp)
               throws ServletException, IOException {
   
       }
   }
   ```

3. 在Web.xml中配置Servlet

   ```xml
   <servlet>
       <servlet-name>MyCookieServlet</servlet-name>
       <servlet-class>org.example.servlet.MyCookieServlet</servlet-class>
     </servlet>
     <servlet-mapping>
       <servlet-name>MyCookieServlet</servlet-name>
       <url-pattern>/cookie</url-pattern>
     </servlet-mapping>
   ```

4. 启动服务进行测试

   在浏览器输入：`http://localhost:8888/cookie`

   我们通过浏览器的开发者工具进行请求信息查看

   <img src="/images/cookie-request-start.jpg" style="zoom:50%;" />

   可以看见在Response Headers中有Set-Cookie属性，显示的值是以Key-Value形式存在。对应着我们前面设置的属性。

   ```java
   Cookie cookie = new Cookie("myCookie", "This is Hello Cookie");
   resp.addCookie(cookie);
   ```

   在Request Headers中存在Cookie属性。

   当我们再次刷新当前页面，我们再来看当前请求。

   <img src="F:\Study notes\images\cookie-request-restart.png" style="zoom:50%;" />

   可以看见Request Headers中的Cookie属性自动带上了前面设置的Cookie。

5. 后端服务获取Cookie

   ```java
   Cookie[] cookies = request.getCookies();
   for (Cookie c: cookies) {
   	System.out.println(c.getValue());
   }
   ```

#### 5.Cookie的Expires属性

​	通过Cookie的`setMaxAge()`方法可以设置Cookie的过期时间。

* 0：带表删除Cookie，删除Cookie的时候要确保Cookie的其它属性都相同如：path、Domain等属性，不然就不能删除而是代表当前这个Cookie在浏览器端不进行存储。
* 负值或者不设置：表示当前Cookie不会永久的存储，当浏览器进行关闭的时候就删除。
* 正值：表示永久的将Cookie存储在用户磁盘，当用户手动删除或者到达过期时间就删除。