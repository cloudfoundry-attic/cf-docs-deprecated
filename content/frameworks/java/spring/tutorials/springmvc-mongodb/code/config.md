---
title: Config classes for Expense Reporting App using MongoDB
description: Config classes for Expense Reporting App using MongoDB
---
```java
package com.springsource.html5expense.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

import com.springsource.html5expense.controllers.ExpenseController;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.mongodb.services.ExpenseRepository;

@Configuration
@ComponentScan(basePackageClasses = { ExpenseRepository.class, ExpenseController.class,
        Expense.class })
@EnableWebMvc
public class ComponentConfig {
    @Autowired
    private MongoDbFactoryConfiguration mongoDbConfiguration;

    @Bean
    public MongoTemplate mongoTemplate() throws Exception {
        MongoTemplate mongoTemplate = new MongoTemplate(mongoDbConfiguration.mongoDbFactory());
        return mongoTemplate;
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

import org.springframework.data.mongodb.MongoDbFactory;

public interface MongoDbFactoryConfiguration {
    MongoDbFactory mongoDbFactory() throws Exception;
}
```

```java
package com.springsource.html5expense.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.PropertyResolver;
import org.springframework.data.mongodb.MongoDbFactory;
import org.springframework.data.mongodb.core.SimpleMongoDbFactory;

import com.mongodb.Mongo;

@Configuration
@PropertySource("/db.properties")
@Profile("local")
public class LocalMongoDbFactoryConfiguration implements MongoDbFactoryConfiguration {
    @Autowired
    private PropertyResolver propertyResolver;

    @Override
    @Bean
    public MongoDbFactory mongoDbFactory() throws Exception {
        return new SimpleMongoDbFactory(new Mongo(propertyResolver.getProperty("mongo.host"),
                new Integer(propertyResolver.getProperty("mongo.port"))),
                propertyResolver.getProperty("mongo.db.name"));
    }
}
```

```java
package com.springsource.html5expense.config;

import org.cloudfoundry.runtime.service.document.CloudMongoDbFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.data.mongodb.MongoDbFactory;

@Configuration
@Profile("cloud")
public class CloudMongoDbFactoryConfiguration implements MongoDbFactoryConfiguration {

    @Override
    @Bean
    public MongoDbFactory mongoDbFactory() throws Exception {
        CloudMongoDbFactoryBean factory = new CloudMongoDbFactoryBean();
        factory.afterPropertiesSet();
        return factory.getObject();
    }
}
```
