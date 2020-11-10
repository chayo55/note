**Tomcat是**Apache开源的**Servlet容器**
Tomcat接受客户请求并作出响应的过程如下：
1). 客户端 发送HTTP请求，访问web服务器
2).web服务器接受到请求，传递给**Servlet容器(Tomcat)**
3). **Servlet容器(Tomcat)**接受到请求，加载**Servlet(用于处理业务的Servlet)**，产生Servlet实例对象，构建**HttpServlet和HttpResponse对象**，传递给Servlet实例
4).** Servlet实例**接受到**HttpServlet和HttpResponse对象**，进行业务处理，将处理结果返回给**Servlet容器(Tomcat)**
5). **Servlet容器(Tomcat)**接受到Servlet实例返回的结果，通过Response对象返回结果给客户端，