---
title: SpringMVC 中文乱码解决
tags:
  - Java
  - SpringMVC
id: 271
categories:
  - 笔记
date: 2016-12-26 10:48:06
---

使用以下配置

    &lt;?xml version="1.0" encoding="UTF-8"?&gt;
    &lt;beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:p="http://www.springframework.org/schema/p"
           xmlns:mvc="http://www.springframework.org/schema/mvc"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.2.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd"
           default-autowire="byName"&gt;
        &lt;mvc:annotation-driven &gt;
            &lt;mvc:message-converters register-defaults="true"&gt;
                &lt;bean class="org.springframework.http.converter.StringHttpMessageConverter"&gt;
                    &lt;property name="supportedMediaTypes" value="text/html;charset=UTF-8"/&gt;
                &lt;/bean&gt;
            &lt;/mvc:message-converters&gt;
        &lt;/mvc:annotation-driven&gt;
        &lt;context:component-scan base-package="com.youku.direct.store.controller"&gt;&lt;/context:component-scan&gt;
        &lt;bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
              p:viewClass="org.springframework.web.servlet.view.JstlView"
              p:prefix="/WEB-INF/jsp/"
              p:suffix=".jsp"/&gt;

    &lt;/beans&gt;
    