---
title: Expense Reporting App using Redis
description: Create a Spring MVC App using Redis
tags:
    - spring
    - sts
    - redis
    - maven
    - tomcat
---

## Introduction
In this tutorial, you will use Spring MVC and Redis to create an Expense Reporting app and then deploy it on Cloud Foundry.
The Expense Reporting app will be used to create and manage expenses.

## Prerequisites
Before you begin this tutorial, you should:

1.  Be familiar with STS (Spring Tool Suite) or Eclipse.

### Setup: Exercise1-Starter
Import the source code from the downloaded folder Exercise1-Starter as follows. Open STS, Select **File > Import > Maven > Existing Maven Projects** and select springmvc-expensereport-redis folder.

  ![maven-import-step1](/images/spring_tutorial/import-maven-project-step1.png)

  ![maven-import-step2](/images/spring_redis_tutorial/import-maven-project-step2.png)

In the Expense Reporting App, a user can create an expense, view its status, edit it or delete it.

<img src="/images/spring_redis_tutorial/usecase.png" alt="normal-user-usecase.png" width="30%">

These expenses are created by the users and the data
is stored in Redis key-value store.

## Application Flow:
The below figure shows the runtime interaction of the application.

<img src="/images/spring_redis_tutorial/cf-tutorials-redis.png" width="80%" alt="sequence-flow">

1.  User creates, deletes or updates an expense using the jsp/jQuery frontend.

2.  The http request goes to the `ExpenseController` which forwards it to the `ExpenseService`.

3.  The `ExpenseService` persists, deletes or updates the expense state in Redis using `RedisTemplate`.

In order to work with Spring data Redis the following dependencies have been added in pom.xml.
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.0.2.RELEASE</version>
</dependency>
```

## **Step 1:** Adding Data Models
Redis is an advanced key-value store. Since Redis strings are binary safe, we will be storing the entities in our application as binary strings in Redis that we get from serializing java objects. Also, as we are using the default serailization, we don't anything special in our entities. They are POJOs without any annotations.
```java
public class Expense implements Serializable {
    private Long expenseId;
    private String description;
    private Double amount;
    private State state = State.NEW;
    private Date expenseDate;
    private ExpenseType expenseType;
}
```
Create a package for model class as `com.springsource.html5expense.model` and add the following classes `Expense`, `ExpenseType`.

You can find the Entity classes [here](/frameworks/java/spring/tutorials/springmvc-redis/code/entities.html).

## **Step 2:** Adding Services
`@Service` specifies a special component that provides the business services through an interface.

```java
public interface ExpenseService {
    Long save(Expense expense);

    Expense getExpense(Long expenseId);

    List<Expense> getAllExpenses();

    List<Expense> getPendingExpensesList();

    Expense changeExpenseStatus(Long expenseId, String state);

    void deleteExpense(Long expenseId);

    void updateExpense(Expense expense);
}
```
Business methods perform the following actions:

+ Create, Update or Delete an Expense.

+ Get all the expenses.

+ Update an expense's status.

Create a package for service interfaces as `com.springsource.html5expense.service` and add `ExpenseService` and `ExpenseTypeService` interfaces which defines the business methods.
You can get the service interfaces [here](/frameworks/java/spring/tutorials/springmvc-redis/code/services.html).

Create a package `com.springsource.html5expense.redis.services` for service implementation and add the following classes `ExpenseRepository`, `ExpenseTypeRepository`. The `RedisTemplate` has been autowired which is used for interacting with Redis. It has API for persisting, removing and finding domain objects that are mapped to the datastore as key-value pairs.

Following is a snippet of code that shows how to find an object for a given key in Redis using `RedisTemplate`.
```java
@Service
public class ExpenseTypeRepository implements ExpenseTypeService {

    @Autowired
    private RedisTemplate<String, Serializable> redisTemplate;

    @Autowired
    @Qualifier(KeyUtils.EXPENSE_TYPE_COUNT)
    private RedisAtomicLong expenseTypeCount;

    @Override
    public List<ExpenseType> getAllExpenseType() {
        List<ExpenseType> expenseTypeList = new ArrayList<ExpenseType>(
                expenseTypeCount.intValue());
        for (int i = 1; i <= expenseTypeCount.intValue(); i++) {
            ExpenseType expenseType = (ExpenseType) redisTemplate.opsForValue().get(
                    KeyUtils.expenseTypeKey(Long.valueOf(i)));
            if (expenseType != null) {
                expenseTypeList.add(expenseType);
            }
        }
        return expenseTypeList;
    }

    @Override
    public ExpenseType getExpenseTypeById(Long id) {
        return (ExpenseType) redisTemplate.opsForValue().get(KeyUtils.expenseTypeKey(id));
    }
}
```
Apart from `RedisTemplate` we are also autowiring an `RedisAtomicLong expenseTypeCount` which is used for generating sequence ids for `ExpenseType` entities. They are used in a analogous fashion to sequences in relational databases.

We will be generating those keys serially by using an entity specific prefix and an entity specific `RedisAtomicLong` counter. For example, for Expenses, the prefix we are using is `expense:` and the `RedisAtomicLong` we are using is `expenseCount`.

You can find the Redis service implementation classes [here](/frameworks/java/spring/tutorials/springmvc-redis/code/services-implementation.html).

## **Step 3:** Adding the controller
The Controller is responsible for mapping requests. The `@RequestMapping` annotation is used to map requests onto specific handler methods. The following table explains the urls which we used in ExpenseReport application.

<table class="spring-tutorial-index-table">
    <thead>
            <tr>
                <th>No</th>
                <th>HTTP Method</th>
                <th>URL Pattern</th>
                <th>Purpose</th>
            </tr>
    </thead>
    <tbody>
            <tr>
                <td>1</td>
                <td>GET</td>
                <td>/expenses</td>
                <td>Get all the expenses.</td>
            </tr>
            <tr>
                <td>2</td>
                <td>POST</td>
                <td>/expense</td>
                <td>Create a new expense</td>
            </tr>
            <tr>
                <td>3</td>
                <td>GET</td>
                <td>/expense/{id}</td>
                <td>Return an expense whose expenseid is passed</td>
            </tr>
            <tr>
                <td>4</td>
                <td>PUT</td>
                <td>/expense/{id}</td>
                <td>Update an expense whose expenseid is passed</td>
            </tr>
            <tr>
                <td>5</td>
                <td>DELETE</td>
                <td>/expense/{id}</td>
                <td>Delete an expense whose expenseid is passed</td>
            </tr>
    </tbody>
</table>

Create a package for Controllers as `com.springsource.html5expense.controllers` and add `ExpenseController` class. The below method is used in ExpenseController to create a new expense. The `@ResponseBody` annotation indicates that this method return value should be bound to the web response body. We have added Jackson JSON marshaling library dependency in our project. Spring MVC detects this at startup and registers a `MappingJacksonHttpMessageConverter`, which will serialize java objects to JSON strings.
```java
@ResponseBody
@RequestMapping(value = "/expense", method = RequestMethod.POST)
@ResponseStatus(value = HttpStatus.CREATED)
public Long createNewExpense(@RequestParam("description") String description,
        @RequestParam("amount") Double amount,
        @RequestParam("expenseTypeId") Long expenseTypeVal) {

    ExpenseType expenseType = expenseTypeService.getExpenseTypeById(expenseTypeVal);
    Expense expense = new Expense(description, expenseType, new Date(), amount);
    Long expenseId = expenseService.save(expense);
    return expenseId;
}
```
In the method below we have used `ModelMap`. It is a place where you can store key/value pairs to be used by the view in rendering a response.  It is through this context that you request specific information in the response.

```java
@RequestMapping(value = "/", method = RequestMethod.GET)
public String getExpenses(ModelMap model) {
     List<ExpenseType> expenseTypeList = expenseTypeService.getAllExpenseType();
     model.addAttribute("expenseTypeList", expenseTypeList);
     return "expensereports";
}
```
In our controller, we have fetched some data about the expense types and made them available in the ModelMap using the autowired ExpenseTypeService reference. We can also use HTTP request to store key/values pair of data.

Since html forms only supports two methods: POST and GET we have added `HiddenHttpMethodFilter` in web.xml to use DELETE and PUT methods. We can now use Spring form tag with method as `delete` or use Ajax. `HiddenHttpMethodFilter` inspects the request and looks for a request parameter(usually called `_method`) that informs how it’s to transform the request.

```xml
<filter>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <url-pattern>/</url-pattern>
    <servlet-name>appServlet</servlet-name>
</filter-mapping>
```
You can get the controller classes code [here](/frameworks/java/spring/tutorials/springmvc-redis/code/controllers.html).

## **Step 4:** Configuring the ExpenseReport application for Redis
For working with Redis, we need to obtain a connection to the datastore in order to define the `RedisConnectionFactory` bean. The RedisConnectionFactory provides the container with an ExceptionTranslator implementation that translates Redis exceptions to exceptions in Spring's portable `DataAccessException` hierarchy for data access classes annotated with the `@Repository` annotation. Define an interface `RedisConnectionFactoryConfiguration` to get `redisConnectionFactory`. Now implement this interface for your local `JedisConnectionFactory` as `LocalJedisConnectionFactoryConfiguration` and use `@Profile("local")` annotation.

```java
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
 Now implement `RedisConnectionFactoryConfiguration` for Cloud Foundry dataSource as `CloudRedisConnectionFactoryConfiguration` and use `@Profile("cloud")` annotation. To use the Cloud Foundry API add the following dependency has been added in `pom.xml`.
```xml
<dependency>
   <groupId>org.cloudfoundry</groupId>
   <artifactId>cloudfoundry-runtime</artifactId>
   <version>0.8.2</version>
</dependency>
```

```java
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

Now we have declared two redisConnectionFactory, one for local environment and another one for Cloud Foundry environment. To avoid switching back from one datastore to the other, we can make selected profile as active. Based on the active profile only that particular bean should be created. We can select the profiles active based on the environment, create a class called `ExpenseReportAppContextInitializer` in `com.springsource.html5expense.web` and implement `ApplicationContextInitializer`. The ApplicationContextInitializer is used to set active profiles and register custom property sources.

```java
public class ExpenseReportAppContextInitializer implements
        ApplicationContextInitializer<AnnotationConfigWebApplicationContext> {

    private CloudEnvironment cloudEnvironment = new CloudEnvironment();

    private boolean isCloudFoundry() {
        return cloudEnvironment.isCloudFoundry();
    }

    @Override
    public void initialize(AnnotationConfigWebApplicationContext applicationContext) {
        String profile = "local";
        if (isCloudFoundry()) {
            profile = "cloud";
        }
        applicationContext.getEnvironment().setActiveProfiles(profile);
        applicationContext.refresh();
    }
}
```


The application needs to be configured in order to make it ready to deploy. Add the following classes: `ComponentConfig` in `com.springsource.html5expense.config` package. The ComponentConfig class has been marked with `@Configuration` annotation to configure beans. This is an alternative to XML configuration for bean definition. We can pass this class as an argument to Spring container as a source for bean creation.
```java
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

To declare a bean, simply annotate a method with the `@Bean` annotation in your config class. This method is used to register a bean definition within an ApplicationContext of the type specified as the method's return value. By default, the bean name will be the same as the method name.

Next, configure the components using `@ComponentScan`. It will Configure component scanning directives for use with `@Configuration` classes and provides parallel support with Spring XML's `<context:component-scan>` element, passing a basePackageClasses() or basePackages() value.

Note that we are also defining two `RedisAtomicLong` beans to be used as sequence generators for `Expense` and `ExpenseType` entities.

Modify the `db.properties` for hostname, port and password according to your local Redis setup.

To load ComponentConfig class as part of contextConfigLocation, add contextClass as `AnnotationConfigWebApplicationContext` and contextConfigLocation as `com.springsource.html5expense.config`. Then set contextInitializerClasses as `ExpenseReportAppContextInitializer`.
```xml
<context-param>
    <param-name>contextInitializerClasses</param-name>
    <param-value>
       com.springsource.html5expense.web.ExpenseReportAppContextInitializer
    </param-value>
</context-param>
<context-param>
    <param-name>contextClass</param-name>
    <param-value>
      org.springframework.web.context.support.AnnotationConfigWebApplicationContext
    </param-value>
</context-param>
<context-param>
   <param-name>contextConfigLocation</param-name>
   <param-value>com.springsource.html5expense.config</param-value>
 </context-param>
```

You can get the Config classes [here](/frameworks/java/spring/tutorials/springmvc-redis/code/config.html).

## **Step 5:** Initialize datastore with default values
Create a new class called `DataInitializer` in `com.springsource.html5expense.redis.services`, and add a init method with the `@PostConstruct` annotation to initialize default values in database. The method annotated with `PostConstruct` annotation needs to be executed after dependency injection is done to perform initialization. In this method we just create and persist ExpenseType objects if there aren't any and save them into Redis.

```java
@Service
public class DataInitalizer {

    @Autowired
    private RedisTemplate<String, Serializable> redisTemplate;

    @Autowired
    @Qualifier(KeyUtils.EXPENSE_TYPE_COUNT)
    private RedisAtomicLong expenseTypeCount;

    @PostConstruct
    private void init() {

        if (expenseTypeCount.intValue() > 0) {
            logger.debug("ExpenseTypes are already defined.");
            return;
        }

        String[] expenseTypes = { "GYM", "TELEPHONE", "MEDICARE", "TRAVEL" };

        for (String expensType : expenseTypes) {
            long nextId = expenseTypeCount.incrementAndGet();
            ExpenseType type = new ExpenseType();
            type.setExpenseTypeId(nextId);
            type.setName(expensType);
            String expenseKey = KeyUtils.expenseTypeKey(nextId);
            redisTemplate.opsForValue().set(expenseKey, type);
        }
    }
}
```

## **Step 6:** Adding View Files
The ExpenseReport app service is ready. To get users to interact with it, it needs a user interface. To resolve the Controllers return `view` value, we have defined InternalViewResolver.

```java
@Bean
public InternalResourceViewResolver internalResourceViewResolver() {
    InternalResourceViewResolver internalResourceViewResolver =
            new InternalResourceViewResolver();
    internalResourceViewResolver.setPrefix("/WEB-INF/views/");
    internalResourceViewResolver.setSuffix(".jsp");
    return internalResourceViewResolver;
}
```

The application requires the following files to be added, `expensereports.jsp` in `webapp/WEB-INF/view` folder.

You can download the jsp files [here](/frameworks/java/spring/tutorials/springmvc-redis/code/views.html).

*Note:*
: Before running the application, make sure your Redis instance is running.

## Check Point
1. Select your project, drag it down and drop into your VMware vFabric tc server listed in the bottom left Server window.

2. Select VMware vFabric tc server and click the start button to run the server.

3. Once the server starts, open your browser and enter the application url : `http://localhost:8080/springmvc-expensereport-redis/`. Click on New Expense to create a new expense.
  ![local_deploy.png](/images/spring_redis_tutorial/local_deploy.png)

4. Go to your Redis client and give the command **keys ***. It will list all the entities in the datastore. You can see that we have expenseTypeCount, expenseCount and a few keys for expenseTypes that we created while initializing the application

```bash
$ keys *
1) "expenseTypeCount"
2) "\xac\xed\x00\x05t\x00\rexpenseType:1"
3) "\xac\xed\x00\x05t\x00\rexpenseType:2"
4) "\xac\xed\x00\x05t\x00\rexpenseType:3"
5) "\xac\xed\x00\x05t\x00\rexpenseType:4"
6) "expenseCount"
$
```

## Deploying to Cloud Foundry
* To learn how to deploy a Spring App using Redis to Cloud Foundry, please refer [here](/frameworks/java/spring/tutorials/springmvc-redis/springmvc-app-with-redis-deployment-to-cloudfoundry.html).

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-redis/springmvc-with-redis.html">Prev</a> <a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/java/spring/tutorials/springmvc-redis/springmvc-app-with-redis-deployment-to-cloudfoundry.html">Next</a>
