---
title: Expense Reporting App using MongoDB
description: Create a Spring MVC App using MongoDB
tags:
    - spring
    - sts
    - mongodb
    - maven
    - tomcat
---

## Introduction
In this tutorial, you will use Spring MVC and MongoDB to create an Expense Reporting app and then deploy it on Cloud Foundry.
The Expense Reporting app will be used to create and manage expenses.

## Prerequisites
Before you begin this tutorial, you should:

1.  Be familiar with STS (Spring Tool Suite) or Eclipse.

### Setup: Exercise1-Starter
Import the source code from the downloaded folder Exercise1-Starter as follows. Open STS, Select **File > Import > Maven > Existing Maven Projects** and select springmvc-expensereport-mongodb folder.

  ![maven-import-step1](/images/spring_tutorial/import-maven-project-step1.png)

  ![maven-import-step2](/images/spring_mongodb_tutorial/import-maven-project-step2.png)

In the Expense Reporting App, a user can create an expense, view its status, edit it or delete it.

<img src="/images/spring_mongodb_tutorial/usecase.png" alt="normal-user-usecase.png" width="30%">

These expenses are created by the users and the data
is stored in MongoDB document database.

## Application Flow:
The below figure shows the runtime interaction of the application.

<img src="/images/spring_mongodb_tutorial/cf-tutorials-mongo.png" width="70%" alt="sequence-flow">

1.  User creates or deletes or updates an expense.

2.  The expense goes to the ExpenseSevice where MongoTemplate will persist or delete or update its state in the MongoDB document database.

In order to work with Spring data MongoDB the following dependencies have been added in pom.xml.
```xml
 <dependency>
   <groupId>org.springframework.data</groupId>
   <artifactId>spring-data-mongodb</artifactId>
   <version>1.0.4.RELEASE</version>
 </dependency>
 <dependency>
   <groupId>org.mongodb</groupId>
   <artifactId>mongo-java-driver</artifactId>
   <version>2.9.1</version>
 </dependency>
```

## **Step 1:** Adding Data Models
MongoDB is a document oriented database, the java classes will be stored as BSON documents. The domain classes are marked with @Document annotation to indicate that this class is a candidate to map to the database. By default, the class name is choosen as the collection name. If you want to store it under a different name, set under collection property.
```java
@Document(collection = "Expense")
public class Expense implements Serializable {
    @Id
    private Long expenseId;
    private String description;
    private Double amount;
    private State state = State.NEW;
    private Date expenseDate;
    private ExpenseType expenseType;
}
```
The `@org.springframework.data.annotation.Id` annotation is applied to fields. The field annotated with @Id will be stored as `_id` in MongoDB and is used to uniquely identify the document in a MongoDB collection. A collection in MongoDB is similar to a table in RDBMS and the document is similar to a row in RDBMS.

Create a package for model class as `com.springsource.html5expense.model` and add the following classes `Expense`, `ExpenseType`.

You can find the Entity classes [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/entities.html)

## **Step 2:** Adding Services
`@Service` specifies a special component that provides the business services through interface.

```java
public interface ExpenseService {
    Long save(Expense expense);

    Expense getExpense(Long expenseId);

    List<Expense> getAllExpenses();

    List<Expense> getPendingExpensesList();

    Expense changeExpenseStatus(Long expenseId, String state);

    void deleteExpense(Long expenseId);

    void updateExpense(Long expenseId, Update update);
}
```
These are the business methods to:

+ Create, Update or Delete an Expense.

+ Get all the expenses.

+ Update an expense's status.

* Create a package for service interfaces as `com.springsource.html5expense.service` and add `ExpenseService` and `ExpenseTypeService` interfaces which defines the business methods.
You can get the service interfaces [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/services.html)

* Create a package `com.springsource.html5expense.mongodb.services` for service implementation and add the following classes
 `ExpenseRepository`, `ExpenseTypeRepository`. The MongoTemplate has been autowired. The MongoTemplate by itself implemented `MongoOperations` interface. It has API to persisting, removing and finding domain objects that are mapped to the database as documents.

Following is a snippet of code that shows how to find documents in a MongoDB collections using `MongoTemplate`.
```java
@Service
public class ExpenseRepository implements ExpenseService {
    @Autowired
    MongoTemplate mongoTemplate;
    public static String EXPENSE_COLLECTION_NAME = "Expense";

    public List<Expense> getAllExpenses() {
        return mongoTemplate.findAll(Expense.class,EXPENSE_COLLECTION_NAME);
    }
    public List<Expense> getPendingExpensesList() {
        List<State> stateList = new ArrayList<State>();
        stateList.add(State.NEW);
        stateList.add(State.OPEN);
        stateList.add(State.IN_REVIEW);
        return mongoTemplate.find(new Query(Criteria.where("state").in(stateList)), Expense.class, EXPENSE_COLLECTION_NAME);
    }
}
```

You can find the MongoDB service implementation classes [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/services-implementation.html).

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
                <td>/expenses</td>
                <td>Create a new expense</td>
            </tr>
            <tr>
                <td>3</td>
                <td>GET</td>
                <td>/expenses/{id}</td>
                <td>Return an expense whose expenseid is passed</td>
            </tr>
            <tr>
                <td>4</td>
                <td>PUT</td>
                <td>/expenses/{id}</td>
                <td>Update an expense whose expenseid is passed</td>
            </tr>
            <tr>
                <td>5</td>
                <td>DELETE</td>
                <td>/expenses/{id}</td>
                <td>Delete an expense whose expenseid is passed</td>
            </tr>
    </tbody>
</table>

Create a package for Controllers as `com.springsource.html5expense.controllers` and add `ExpenseController` class. The below method is used in ExpenseController to create a new expense. The `@ResponseBody` annotation indicates that this method return value should be bound to the web response body. We have added Jackson JSON marshaling library dependency in our project. Spring MVC detects this at startup and registers a `MappingJacksonHttpMessageConverter`, which will serialize java objects to JSON strings.
```java
@ResponseBody
@RequestMapping(value = "/expenses", method = RequestMethod.POST)
@ResponseStatus(value = HttpStatus.CREATED)
public Long createNewExpense(@RequestParam("description")String description
    , @RequestParam("amount")Double amount, @RequestParam("expenseTypeId")Long expenseTypeVal) {
    ExpenseType expenseType = expenseTypeService.getExpenseTypeById(expenseTypeVal);
    Long expenseId = expenseService.createExpense(description, expenseType, new Date(),
            amount);
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
You can get the controller classes code [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/controllers.html)

## **Step 4:** Configuring the ExpenseReport application for MongoDB
For working with MongoDB, we need to obtain a connection to the database in order to define the MangoDbFactory bean. The MongoFactory provides the container with an ExceptionTranslator implementation that translates MongoDB exceptions to exceptions in Spring's portable DataAccessException hierarchy for data access classes annotated with the @Repository annotation. Define an interface `MongoDbFactoryConfiguration` to get `mongoDbFactory`. Now implement this interface for your local mongoDbFactory as `LocalMongoDbFactoryConfiguration` and use `@Profile("local")` annotation.

```java
@Configuration
@PropertySource("/db.properties")
@Profile("local")
public class LocalMongoDbFactoryConfiguration
              implements MongoDbFactoryConfiguration {
    @Autowired private PropertyResolver propertyResolver;

    @Bean
    public MongoDbFactory mongoDbFactory() throws Exception {
        return new SimpleMongoDbFactory(new Mongo(
               propertyResolver.getProperty("mongo.host"),
               new Integer(propertyResolver.getProperty("mongo.port"))),
               propertyResolver.getProperty("mongo.db.name"));
    }
}
```
 Now implement `MongoDbFactoryConfiguration` for Cloud Foundry dataSource as `CloudMongoDbFactoryConfiguration` and use `@Profile("cloud")` annoation. To use the Cloud Foundry API add the following dependency has been added in `pom.xml`.
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
public class CloudMongoDbFactoryConfiguration implements MongoDbFactoryConfiguration {

    @Bean
    public MongoDbFactory mongoDbFactory() throws Exception {
        CloudMongoDbFactoryBean factory = new CloudMongoDbFactoryBean();
        factory.afterPropertiesSet();
        return factory.getObject();
    }
}
```

Now we have declared two mongoDbFactory, one for local environment and another one for Cloud Foundry environment. To avoid switching back from one database to another database, we can make selected profile as active. Based on the active profile only that particular bean should be created. We can select the profiles active based on the environment, create a class called `ExpenseReportAppContextInitializer` in `com.springsource.html5expense.web` and implement `ApplicationContextInitializer`. The ApplicationContextInitializer is used to set active profiles and register custom property sources.

```java
public class ExpenseReportAppContextInitializer implements ApplicationContextInitializer<AnnotationConfigWebApplicationContext> {

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
@EnableTransactionManagement
@ComponentScan(basePackageClasses = {ExpenseRepository.class,
       ExpenseController.class, Expense.class })
@EnableWebMvc
public class ComponentConfig {
    @Autowired private MongoDbFactoryConfiguration mongoDbConfiguration;

    @Bean
    public MongoTemplate mongoTemplate() throws Exception {
        MongoTemplate mongoTemplate = new MongoTemplate(
        mongoDbConfiguration.mongoDbFactory());
        return mongoTemplate;
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

Modify the `db.properties` file properties database username, password, database name according to your local MongoDB database.

To load ComponentConfig class as part of contextConfigLocation, add contextClass as `AnnotationConfigWebApplicationContext` and `contextConfigLocation` as
com.springsource.html5expense.config. Then set `contextInitializerClasses` as ExpenseReportAppContextInitializer.
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

You can get the Config classes [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/config.html).

## **Step 5:** Initialize database with default values
Create a new class called `DataInitializer` in `com.springsource.html5expense.mogodb.services`, and add a init method with the `@PostConstruct` annotation
to initialize default values in database. The method annotated with `PostConstruct` annotation needs to be executed after dependency injection is done to perform initialization. In this method we just create ExpenseType objects and save them into MongoDb.

```java
@Service
public class DataInitalizer {

    @Autowired
    MongoTemplate mongoTemplate;

    @PostConstruct
    private void init() {

        mongoTemplate.getCollection("ExpenseType").drop();
        mongoTemplate.getCollection("Expense").drop();
        DBCollection expenseTypeCollection = mongoTemplate.getCollection("ExpenseType");

        BasicDBObject expenseTypeDoc = new BasicDBObject();

        expenseTypeDoc.put("_id", new Long(1));
        expenseTypeDoc.put("expenseTypeId", new Long(1));
        expenseTypeDoc.put("name", "GYM");
        expenseTypeCollection.insert(expenseTypeDoc);

        expenseTypeDoc.clear();
        expenseTypeDoc.put("_id", new Long(2));
        expenseTypeDoc.put("expenseTypeId", new Long(2));
        expenseTypeDoc.put("name", "TELEPHONE");
        expenseTypeCollection.insert(expenseTypeDoc);

        expenseTypeDoc.clear();
        expenseTypeDoc.put("_id", new Long(3));
        expenseTypeDoc.put("expenseTypeId", new Long(3));
        expenseTypeDoc.put("name", "MEDICARE");
        expenseTypeCollection.insert(expenseTypeDoc);

        expenseTypeDoc.clear();
        expenseTypeDoc.put("_id", new Long(4));
        expenseTypeDoc.put("expenseTypeId", new Long(4));
        expenseTypeDoc.put("name", "TRAVEL");
        expenseTypeCollection.insert(expenseTypeDoc);
    }
}
```

## **Step 6:** Adding View Files
The ExpenseReport app service is ready. To get users to interact with it, it needs a user interface. To resolve the Controllers return `view` value, we have defined InternalViewResolver.

```java
@Bean
public InternalResourceViewResolver internalResourceViewResolver() {
    InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
    internalResourceViewResolver.setPrefix("/WEB-INF/views/");
    internalResourceViewResolver.setSuffix(".jsp");
    return internalResourceViewResolver;
}
```

The application requires the following files to be added, `expensereports.jsp` in `webapp/WEB-INF/view` folder.

You can download the jsp files [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/views.html).

*Note:*
: Before running the application, make sure your MongoDB instance is running.

## Check Point
1. Select your project, drag it down and drop into your VMware vFabric tc server listed in the bottom left Server window.

2. Select VMware vFabric tc server and click the start button to run the server.

3. Once the server starts, open your browser and enter the application url : `http://localhost:8080/springmvc-expensereport-mongodb/`. Click on New Expense to create a new expense.
  ![local_deploy.png](/images/spring_mongodb_tutorial/local_deploy.png)

4. Go to your MongoDB client and give the command **show collections**. It will list all the collections in the database so you can check that new collections have been created for the Entities.

```bash
$ show collections
Expense
ExpenseType
system.indexes
$
```

## Deploying to Cloud Foundry
* To learn how to deploy a Spring App using MongoDB to Cloud Foundry, please refer [here](/frameworks/java/spring/tutorials/springmvc-mongodb/springmvc-app-with-mongodb-deployment-to-cloudfoundry.html).

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-mongodb/springmvc-with-mongodb.html">Prev</a> <a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/java/spring/tutorials/springmvc-mongodb/springmvc-app-with-mongodb-deployment-to-cloudfoundry.html">Next</a>