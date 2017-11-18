---
title: 用DWR comet+Spring实现服务器推送的例子--网页聊天室
tags:
  - comet
  - DWR
  - Java
  - Spring
id: 39
categories:
  - 笔记
date: 2014-08-18 21:06:47
---

最近网上看的代码 敲下来试试 行得通 于是拿到这里和大家分享下。
首先 下载DWR的JAR包 下载地址[http://directwebremoting.org/dwr/downloads/index.html
](http://directwebremoting.org/dwr/downloads/index.html)接下来 在web工程中加入相应的jar包，因为用到了spring的消息驱动机制 如下
![](http://img.blog.csdn.net/20130729121442156?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJhbms1OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
接下来是文件结构
![](http://img.blog.csdn.net/20130729121523046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJhbms1OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<!--more-->
Message.java                    单位信息的承载类

<pre class="lang:java decode:true ">import java.util.Date;  
/** 
 * 作为信息传递的基础类 
 * @author frank59 
 * 
 */  
public class Message {  
    private int id;  
    private String msg;  
    private Date time;  
    public int getId() {  
        return id;  
    }  
    public void setId(int id) {  
        this.id = id;  
    }  
    public String getMsg() {  
        return msg;  
    }  
    public void setMsg(String msg) {  
        this.msg = msg;  
    }  
    public Date getTime() {  
        return time;  
    }  
    public void setTime(Date time) {  
        this.time = time;  
    }  
}</pre>

ChatMessageEvent.java

<pre class="lang:java decode:true ">import org.springframework.context.ApplicationEvent;  
/** 
 * 自定义事件类 
 * @author frank59 
 * 
 */  
public class ChatMessageEvent extends ApplicationEvent{  

    public ChatMessageEvent(Object source) {  
        super(source);  
    }  
    private static final long serialVersionUID = -6626763488290315190L;  
}</pre>

ChatMessageClient.java

<pre class="lang:java decode:true ">import java.util.Collection;  
import java.util.Date;  
import javax.servlet.ServletContext;  
import org.directwebremoting.ScriptBuffer;  
import org.directwebremoting.ScriptSession;  
import org.directwebremoting.ServerContext;  
import org.directwebremoting.ServerContextFactory;  
import org.springframework.context.ApplicationEvent;  
import org.springframework.context.ApplicationListener;  
import org.springframework.web.context.ServletContextAware;  

import entity.Message;  
/** 
 * MessageEvent事件的监听类 
 * 需要spring的支持 利用spring的消息驱动机制 
 * @author frank59 
 * 
 */  
@SuppressWarnings("unchecked")  
public class ChatMessageClient implements ApplicationListener,  
        ServletContextAware {  
    private ServletContext ctx;  

    @SuppressWarnings("deprecation")  
    public void onApplicationEvent(ApplicationEvent event) {  
        // 如果事件类型是ChatMessageEvent就执行下面操作  
        if (event instanceof ChatMessageEvent) {  
            Message msg = (Message) event.getSource();  
            ServerContext context = ServerContextFactory.get();  
            // 获得客户端所有chat页面script session连接数  
            Collection&lt;ScriptSession&gt; sessions = context  
                    .getScriptSessionsByPage(ctx.getContextPath() + "/chat.jsp");  
            for (ScriptSession session : sessions) {  
                ScriptBuffer sb = new ScriptBuffer();  
                Date time = msg.getTime();  
                String s = time.getYear() + "-" + (time.getMonth() + 1) + "-"  
                        + time.getDate() + " " + time.getHours() + ":"  
                        + time.getMinutes() + ":" + time.getSeconds();  
                System.out.println(s);/////////////////s  
                // 执行setMessage方法  
                sb.appendScript("showMessage({msg: '").appendScript(  
                        msg.getMsg()).appendScript("', time: '")  
                        .appendScript(s).appendScript("'})");  
                System.out.println(sb.toString());/////////////////sb  
                //执行客户端script session方法，相当于浏览器执行JavaScript代码                     
                //上面就会执行客户端浏览器中的showMessage方法，并且传递一个对象过去    
                session.addScript(sb);  
            }  
        }  
    }  
    public void setServletContext(ServletContext arg0) {  
        this.ctx = arg0;  
    }  
}</pre>

ChatMessageClient.java

<pre class="lang:default decode:true ">import java.util.Collection;  
import java.util.Date;  
import javax.servlet.ServletContext;  
import org.directwebremoting.ScriptBuffer;  
import org.directwebremoting.ScriptSession;  
import org.directwebremoting.ServerContext;  
import org.directwebremoting.ServerContextFactory;  
import org.springframework.context.ApplicationEvent;  
import org.springframework.context.ApplicationListener;  
import org.springframework.web.context.ServletContextAware;  

import entity.Message;  
/** 
 * MessageEvent事件的监听类 
 * 需要spring的支持 利用spring的消息驱动机制 
 * @author frank59 
 * 
 */  
@SuppressWarnings("unchecked")  
public class ChatMessageClient implements ApplicationListener,  
        ServletContextAware {  
    private ServletContext ctx;  

    @SuppressWarnings("deprecation")  
    public void onApplicationEvent(ApplicationEvent event) {  
        // 如果事件类型是ChatMessageEvent就执行下面操作  
        if (event instanceof ChatMessageEvent) {  
            Message msg = (Message) event.getSource();  
            ServerContext context = ServerContextFactory.get();  
            // 获得客户端所有chat页面script session连接数  
            Collection&lt;ScriptSession&gt; sessions = context  
                    .getScriptSessionsByPage(ctx.getContextPath() + "/chat.jsp");  
            for (ScriptSession session : sessions) {  
                ScriptBuffer sb = new ScriptBuffer();  
                Date time = msg.getTime();  
                String s = time.getYear() + "-" + (time.getMonth() + 1) + "-"  
                        + time.getDate() + " " + time.getHours() + ":"  
                        + time.getMinutes() + ":" + time.getSeconds();  
                System.out.println(s);/////////////////s  
                // 执行setMessage方法  
                sb.appendScript("showMessage({msg: '").appendScript(  
                        msg.getMsg()).appendScript("', time: '")  
                        .appendScript(s).appendScript("'})");  
                System.out.println(sb.toString());/////////////////sb  
                //执行客户端script session方法，相当于浏览器执行JavaScript代码                     
                //上面就会执行客户端浏览器中的showMessage方法，并且传递一个对象过去    
                session.addScript(sb);  
            }  
        }  
    }  
    public void setServletContext(ServletContext arg0) {  
        this.ctx = arg0;  
    }  
}</pre>

applicationContext-beans.xml

<pre class="lang:xhtml decode:true">&lt;?xml version="1.0" encoding="UTF-8"?&gt;  
&lt;beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"  
    xmlns:tx="http://www.springframework.org/schema/tx" xmlns:util="http://www.springframework.org/schema/util"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd   
    http://www.springframework.org/schema/aop         
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd        
    http://www.springframework.org/schema/tx         
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd        
    http://www.springframework.org/schema/util         
    http://www.springframework.org/schema/util/spring-util-3.0.xsd        
    http://www.springframework.org/schema/context         
    http://www.springframework.org/schema/context/spring-context-3.0.xsd"&gt;  
    &lt;bean id="chatService" class="chat.ChatService"/&gt;   
    &lt;bean id="chatMessageClient" class="chat.ChatMessageClient"/&gt;  
&lt;/beans&gt;</pre>

dwr.xml

<pre class="lang:xhtml decode:true ">&lt;?xml version="1.0" encoding="UTF-8"?&gt;  
&lt;!DOCTYPE dwr PUBLIC "-//GetAhead Limited//DTD Direct Web Remoting 3.0//EN"   
    "http://getahead.org/dwr/dwr30.dtd"&gt;  
&lt;dwr&gt;  
    &lt;allow&gt;  
        &lt;convert match="entity.Message" converter="bean"&gt;  
            &lt;param name="include" value="msg,time" /&gt;  
        &lt;/convert&gt;  
        &lt;create creator="spring" javascript="ChatService"&gt;  
            &lt;param name="beanName" value="chatService" /&gt;  
        &lt;/create&gt;  
    &lt;/allow&gt;  
&lt;/dwr&gt;</pre>

web.xml

<pre class="lang:default decode:true ">&lt;?xml version="1.0" encoding="UTF-8"?&gt;  
&lt;web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee   
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"&gt;  
    &lt;!-- 加载Spring容器配置 --&gt;  
    &lt;listener&gt;  
        &lt;listener-class&gt;org.springframework.web.context.ContextLoaderListener&lt;/listener-class&gt;  
    &lt;/listener&gt;  
    &lt;!-- 设置Spring容器加载配置文件路径 --&gt;  
    &lt;context-param&gt;  
        &lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;  
        &lt;param-value&gt;classpath*:applicationContext-*.xml&lt;/param-value&gt;  
    &lt;/context-param&gt;  
    &lt;listener&gt;  
        &lt;listener-class&gt;org.directwebremoting.servlet.DwrListener&lt;/listener-class&gt;  
    &lt;/listener&gt;  
    &lt;servlet&gt;  
        &lt;servlet-name&gt;dwr-invoker&lt;/servlet-name&gt;  
        &lt;servlet-class&gt;org.directwebremoting.servlet.DwrServlet&lt;/servlet-class&gt;  
        &lt;init-param&gt;  
            &lt;param-name&gt;debug&lt;/param-name&gt;  
            &lt;param-value&gt;true&lt;/param-value&gt;  
        &lt;/init-param&gt;  
        &lt;!-- dwr的comet控制 --&gt;  
        &lt;init-param&gt;  
            &lt;param-name&gt;pollAndCometEnabled&lt;/param-name&gt;  
            &lt;param-value&gt;true&lt;/param-value&gt;  
        &lt;/init-param&gt;  
    &lt;/servlet&gt;  
    &lt;servlet-mapping&gt;  
        &lt;servlet-name&gt;dwr-invoker&lt;/servlet-name&gt;  
        &lt;url-pattern&gt;/dwr/*&lt;/url-pattern&gt;  
    &lt;/servlet-mapping&gt;  
    &lt;welcome-file-list&gt;  
        &lt;welcome-file&gt;index.jsp&lt;/welcome-file&gt;  
    &lt;/welcome-file-list&gt;  
&lt;/web-app&gt;</pre>

**ChatService.js文件为空白文件**
chat.jsp

<pre class="lang:default decode:true ">&lt;%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%&gt;  
&lt;%  
String path = request.getContextPath();  
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";  
%&gt;  

&lt;!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"&gt;  
&lt;html&gt;  
    &lt;head&gt;  
        &lt;base href="&lt;%=basePath%&gt;"&gt;  

        &lt;title&gt;My JSP 'chat.jsp' starting page&lt;/title&gt;  

        &lt;meta http-equiv="pragma" content="no-cache"&gt;  
        &lt;meta http-equiv="cache-control" content="no-cache"&gt;  
        &lt;meta http-equiv="expires" content="0"&gt;  
        &lt;meta http-equiv="keywords" content="keyword1,keyword2,keyword3"&gt;  
        &lt;meta http-equiv="description" content="This is my page"&gt;  
        &lt;!-- 
    &lt;link rel="stylesheet" type="text/css" href="styles.css"&gt; 
    --&gt;  
        &lt;script type="text/javascript"  
            src="${pageContext.request.contextPath }/dwr/engine.js"&gt;&lt;/script&gt;  
        &lt;script type="text/javascript"  
            src="${pageContext.request.contextPath }/dwr/util.js"&gt;&lt;/script&gt;  
        &lt;script type="text/javascript"  
            src="${pageContext.request.contextPath }/dwr/interface/ChatService.js"&gt;&lt;/script&gt;  
        &lt;script type="text/javascript"&gt;           
            function send() {                
                var time = new Date();                
                var content = dwr.util.getValue("content");                
                var name = dwr.util.getValue("userName");                
                var info = encodeURI(encodeURI(name + " say:\n" + content));                
                var msg = {"msg": info, "time": time};                
                dwr.util.setValue("content", "");                
                if (!!content) {                    
                    ChatService.sendMessage(msg);                
                } else {                    
                    alert("发送的内容不能为空！");                
                }            
            }               
            function showMessage(data) {                
                var message = decodeURI(decodeURI(data.msg));                
                var text = dwr.util.getValue("info");                
                if (!!text) {                      
                    dwr.util.setValue("info", text + "\n" + data.time + "  " + message);                
                } else {                    
                    dwr.util.setValue("info", data.time + "  " + message);                
                }            
            }        
        &lt;/script&gt;  
    &lt;/head&gt;  

    &lt;body onload="dwr.engine.setActiveReverseAjax(true);"&gt;  
        &lt;textarea rows="20" cols="60" id="info" readonly="readonly"&gt;&lt;/textarea&gt;  
        &lt;hr /&gt;  
        昵称：&lt;input type="text" id="userName" /&gt;  
        &lt;br /&gt;  
        消息：&lt;textarea rows="5" cols="30" id="content"&gt;&lt;/textarea&gt;  
        &lt;input type="button" value=" Send " onclick="send()" style="height: 85px; width: 85px;" /&gt;  
    &lt;/body&gt;  
&lt;/html&gt;</pre>

实现效果
![](http://img.blog.csdn.net/20130729122231546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJhbms1OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)