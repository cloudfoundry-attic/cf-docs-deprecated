---
title: Config classes for Expense Reporting App using Redis
description: Config classes for Expense Reporting App using Redis
---
```java
package com.springsource.html5expense.config;

import java.io.Serializable;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.support.atomic.RedisAtomicLong;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

import com.springsource.html5expense.controllers.ExpenseController;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.redis.services.ExpenseRepository;
import com.springsource.html5expense.redis.services.KeyUtils;

@Configuration
@ComponentScan(basePackageClasses = { ExpenseRepository.class, ExpenseController.class,
        Expense.class })
@EnableWebMvc
public class ComponentConfig {

    @Autowired
    private RedisConnectionFactoryConfiguration redisConfiguration;

    @Bean
    public RedisTemplate<String, Serializable> redisTemplate() throws Exception {
        RedisTemplate<String, Serializable> redisTemplate = new
                RedisTemplate<String, Serializable>();
        redisTemplate.setConnectionFactory(redisConfiguration.redisConnectionFactory());
        return redisTemplate;
    }

    @Bean(name = KeyUtils.EXPENSE_COUNT)
    public RedisAtomicLong expenseCount() throws Exception {
        return new RedisAtomicLong(KeyUtils.EXPENSE_COUNT,
                redisConfiguration.redisConnectionFactory());
    }

    @Bean(name = KeyUtils.EXPENSE_TYPE_COUNT)
    public RedisAtomicLong expenseTypeCount() throws Exception {
        return new RedisAtomicLong(KeyUtils.EXPENSE_TYPE_COUNT,
                redisConfiguration.redisConnectionFactory());
    }

    @Bean
    public InternalResourceViewResolver internalResourceViewResolver() {
        InternalResourceViewResolver internalResourceViewResolver =
                new InternalResourceViewResolver();
        internalResourceViewResolver.setPrefix("/WEB-INF/views/");
        internalResourceViewResolver.setSuffix(".jsp");
        return internalResourceViewResolver;
    }
}
```

```java
package com.springsource.html5expense.config;

import org.springframework.data.redis.connection.RedisConnectionFactory;

public interface RedisConnectionFactoryConfiguration {
    RedisConnectionFactory redisConnectionFactory() throws Exception;
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
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;

@Configuration
@PropertySource("/db.properties")
@Profile("local")
public class LocalJedisConnectionFactoryConfiguration implements
        RedisConnectionFactoryConfiguration {

    @Autowired
    private PropertyResolver propertyResolver;

    @Override
    @Bean
    public RedisConnectionFactory redisConnectionFactory() throws Exception {
        JedisConnectionFactory factory = new JedisConnectionFactory();
        factory.setHostName(propertyResolver.getProperty("redis.host"));
        factory.setPort(Integer.parseInt(propertyResolver.getProperty("redis.port")));
        factory.setPassword(propertyResolver.getProperty("redis.pass"));
        return factory;
    }
}
```

```java
package com.springsource.html5expense.config;

import org.cloudfoundry.runtime.service.keyvalue.CloudRedisConnectionFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.data.redis.connection.RedisConnectionFactory;

@Configuration
@Profile("cloud")
public class CloudRedisConnectionFactoryConfiguration implements
        RedisConnectionFactoryConfiguration {

    @Override
    @Bean
    public RedisConnectionFactory redisConnectionFactory() throws Exception {
        CloudRedisConnectionFactoryBean factory = new CloudRedisConnectionFactoryBean();
        factory.afterPropertiesSet();
        return factory.getObject();
    }
}
```
