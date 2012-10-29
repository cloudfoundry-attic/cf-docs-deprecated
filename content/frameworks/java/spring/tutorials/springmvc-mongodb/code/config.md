---
title: Spring MVC ExpenseReport App Configuration Code
description: Spring MVC ExpenseReport App Configuration Code
tags:
    - Spring
    - Code
    - Java
    - Configuration
---

```java
package com.springsource.html5expense.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;
import org.springframework.data.mongodb.MongoDbFactory;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoDbFactory;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.web.multipart.MultipartResolver;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import com.mongodb.Mongo;
import com.springsource.html5expense.controller.ExpenseController;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.mongodb.services.AttachmentRepository;

@Configuration
@EnableTransactionManagement
@ComponentScan(basePackageClasses = {AttachmentRepository.class,
     ExpenseController.class,Expense.class})

public class ComponentConfig {

    @Bean
    public MongoDbFactory mongoDbFactory() throws Exception {
       return new SimpleMongoDbFactory(new Mongo("localhost",
         27017), "expensereport");
    }

    @Bean
    public MongoTemplate mongoTemplate() throws Exception {
       MongoTemplate mongoTemplate = new MongoTemplate(mongoDbFactory());
       return mongoTemplate;
    }

    @Bean
    public MultipartResolver multipartResolver() {
       CommonsMultipartResolver multipartResolver = new org.springframework.web.multipart.commons.CommonsMultipartResolver();
       multipartResolver.setMaxUploadSize(10000000);
       return multipartResolver;
    }

    @Bean
    public InternalResourceViewResolver internalResourceViewResolver() {
       InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
       internalResourceViewResolver.setPrefix("/WEB-INF/views/");
       internalResourceViewResolver.setSuffix(".jsp");
       return internalResourceViewResolver;
    }
}
```

```java
package com.springsource.html5expense.config;

import java.util.List;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.HandlerMethodReturnValueHandler;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
@Import(ComponentConfig.class)
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {
     @Override
     public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {

     }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Bean
    public static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Override
    public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> returnValueHandlers) {

    }
}
```
