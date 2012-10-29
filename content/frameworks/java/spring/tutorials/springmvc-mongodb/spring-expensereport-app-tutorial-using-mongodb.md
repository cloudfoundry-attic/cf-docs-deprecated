---
title: Expense Reporting App using MongoDB
description: Create a Spring MVC App using MongoDB and Deploying on Cloud Foundry
tags:
    - Spring
    - STS
    - Tomcat
    - Maven
    - MongoDB
---

## Introduction
In this tutorial, you will use Spring MVC and MongoDB to create an Expense Reporting app and then deploy it on Cloud Foundry.
The Expense Reporting app will be used to create and manage expenses.

## Prerequisites
Before you begin this tutorial, you should:

1.  Be familiar with STS (Spring Tool Suite) or Eclipse.

### Setup: Exercise1-Starter
Import the source code from the downloaded folder Exercise1-Starter as follows. Open STS, Select **File > Import > Maven > Existing Maven Projects** and select Exercise1-Starter folder.

  ![maven-import-step1](/images/spring_tutorial/import-maven-project-step1.png)

  ![maven-import-step2](/images/spring_tutorial/import-maven-project-exercise1-starter.png)

In the Expense Reporting App, there are two roles:

1.   Normal User - Can create an expense, view its status, edit or delete it.

2.   Manager User -  Can Approve or Reject expenses created by a Normal user.

Once a Normal User submits an expense, it goes to the Manager's work items. The Manager can then Approve or Reject the expense.

<img src="/images/spring_tutorial/normal-user-usecase.png" alt="normal-user-usecase.png" width="30%">

<img src="/images/spring_tutorial/manager-user-use-case.png" alt="manager-user-usecase_admin.png" width="30%">

These expenses are created by the users and the data
is stored in MongoDB document database.

## Application Flow:
The below figure shows the runtime interaction of the application.

<img src="/images/spring_tutorial/sequence-flow-expense-report-app-using-mongo.png" width="70%" alt="sequence-flow">


1.  User logs in to the Expense Reporting system.

2.  User details are verified in database.

3.  User creates or deletes or updates an expense.

4.  The expense goes to the ExpenseSevice where MongoTemplate will persist or delete or update its state in the MongoDB document database.

In order to work with Spring data MongoDB add the following dependency in pom.xml.
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
MongoDB is a document oriented database, the java classes will be stored as BSON documents. The domain classes are marked with @Document annotation at class level to indicate that this class is a candidate to map the database. By default, the class name is choosen as a collection name, if you want to store it under a different name, set under collection property.
```java
@Document(collection = "Expense")
public class Expense implements Serializable {
 @Id
 private Long expenseId;
 private String description;
 private Double amount;
 private State state = State.NEW;
 private Date expenseDate;
 private User user;
 private ExpenseType expenseType;
}
```
The `@org.springframework.data.annotation.Id` annotation is applied to fields. The field which annotated with @Id will be stored as `_id` in MongoDB. This field is used to uniquely identify the document in a collection. The collection is similar to a table in RDBMS and the document is similar to a row in RDBMS.

Create a package for model class as `com.springsource.html5expense.model` and add the following classes `Expense`, `ExpenseType`,
`Role`, `User`.

You can find the Entity classes [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/documents.html)

## **Step 2:** Adding Services
`@Service` specifies a special component that provides the business services through interface.

```java
public interface ExpenseService {
    Long save(Expense expense);

    Expense getExpense(Long expenseId);

    List<Expense> getAllExpenses();

    List<Expense> getExpensesByUser(User user);

    List<Expense> getPendingExpensesList();

    Expense changeExpenseStatus(Long expenseId, String state);

    void deleteExpense(Long expenseId);

    void updateExpense(Long expenseId, Update update);
}
```
These are the business methods to:

+ Create, Update or Delete an Expense.

+ Get pending expenses for a user. Get all the expenses for a user.

+ Update an expense's status.

* Create a package for service interfaces as `com.springsource.html5expense.service` and add `AttachmentService`, `ExpenseService`, `ExpenseTypeService`, `UserService` interfaces which define the business methods.
You can get the service interfaces [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/services.html)

* Create a package `com.springsource.html5expense.mongodb.services` for service implementation and add the following classes
 `AttachmentRepository`, `ExpenseRepository`, `ExpenseTypeRepository`,
`RoleRepository` and `UserRepository`. The MongoTemplate has been autowired. The MongoTemplate by itself implemented `MongoOperations` interface. It has API to persisting, removing and finding domain objects that are mapped to the database as documents.

```java
@Service
public class ExpenseRepository implements ExpenseService {
    @Autowired
    MongoTemplate mongoTemplate;
    public static String EXPENSE_COLLECTION_NAME = "Expense";

    public List<Expense> getAllExpenses() {
        return mongoTemplate.findAll(Expense.class,EXPENSE_COLLECTION_NAME);
    }
    public List<Expense> getExpensesByUser(User user) {
        return mongoTemplate.find(new Query(Criteria.where("user").is(user)) , Expense.class, EXPENSE_COLLECTION_NAME);
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
You can find the MongoDB service implementation classes [here].(/frameworks/java/spring/tutorials/springmvc-mongodb/code/services-implementation.html)

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
                <td>Get all the expenses for the user</td>
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

Create a package for Controllers as `com.springsource.html5expense.controllers` and add the following class: `ExpenseController`. The below method is used in ExpenseController to create a new expense. The `@ResponseBody` annotation indicates that this method return value should be bound to the web response body. We have added Jackson JSON marshaling library dependency in our project. Spring MVC detects this at startup and registers a `MappingJacksonHttpMessageConverter`, which will convert any java.lang.Objects to Strings.
```java
@ResponseBody
@RequestMapping(value = "/expenses", method = RequestMethod.POST)
@ResponseStatus(value = HttpStatus.CREATED)
public Long createNewExpense(@RequestParam("description")String description
    , @RequestParam("amount")Double amount, @RequestParam("expenseTypeId")Long expenseTypeVal) {
    ExpenseType expenseType = expenseTypeService.getExpenseTypeById(expenseTypeVal);
    User user = getUser();
    Long expenseId = expenseService.createExpense(description, expenseType, new Date(),
            amount, user);
    return expenseId;
}
```
In the method below we have used `ModelMap`. It is a place where you can store key/value pairs to be used by the view in rendering a response.  It is through this context that you request specific information in the response.

```java
@RequestMapping(value = "/", method = RequestMethod.GET)
public String getExpenses(ModelMap model) {
     List<ExpenseType> expenseTypeList = expenseTypeService.getAllExpenseType();
     model.addAttribute("expenseTypeList", expenseTypeList);
     model.addAttribute("user", getUser());
     return "expensereports";
}
```
 In our controller, we have gathered some data about the expense types and made them available in the ModelMap using the autowired ExpenseTypeService reference. We can also use HTTP request to store key/values pair of data. To use HttpServletRequest, simply add another argument of type HttpServletRequest.

 Since html only supports two methods: POST and GET. To use DELETE and PUT method add `HiddenHttpMethodFilter` in web.xml. Now use Spring form tag with method as `delete` or use Ajax. It inspects the request and looks for a request parameter(usually called `_method`) that informs how it’s to transform the request.

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

## **Step 4:** Configuring the ExpenseReport application
In order to work with data from a MongoDB database, we need to obtain a connection to the database in order to define the MangoDbFactory bean. The MongoFactory provides the container with an ExceptionTranslator implementation that translates Mongo exceptions to exceptions in Spring's portable DataAccessException hierarchy for data access classes annoated with the @Repository annotation. Define an interface `MongoDbFactoryConfiguration` to get `mongoDbFactory`. Now implement this interface for your local mongoDbFactory as `LocalMongoDbFactoryConfiguration` and use `@Profile("local")` annotation.

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
 Now implement `MongoDbFactoryConfiguration` for Cloud Foudndry dataSource as `CloudMongoDbFactoryConfiguration` and use `@Profile("cloud")` annoation. To use the Cloud Foundry API add the following depdendency in your `pom.xml`.
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
to initialize default values in database. The method annotated with `PostConstruct` annotation needs to be executed after dependency injection is done to perform initialization. In this method below we just create user, role and expenesetype object and saved into MongoDb.

```java
package com.springsource.html5expense.mongodb.services;

import javax.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Service;
import com.mongodb.BasicDBObject;
import com.mongodb.DBCollection;

@Service
public class DataInitializer {

    @Autowired
    MongoTemplate mongoTemplate;

    @Autowired
    UserService userService;

    @Autowired
    RoleService roleService;

    @Autowired
    ExpenseTypeService expenseTypeService;

    @PostConstruct
    private void init() {
        mongoTemplate.getCollection("ExpenseType").drop();
        mongoTemplate.getCollection("User").drop();
        mongoTemplate.getCollection("ExpenseType").drop();
        mongoTemplate.getCollection("Expense").drop();
        mongoTemplate.getCollection("Role").drop();
        Role userRole = new Role();
        userRole.setRoleId(new Long(1));
        userRole.setRoleName("ROLE_USER");
        roleService.save(userRole);
        Role managerRole = new Role();
        managerRole.setRoleId(new Long(2));
        managerRole.setRoleName("ROLE_MANAGER");
        roleService.save(managerRole);
        userService.createUser("admin", "admin", "admin@expensereport.com", managerRole);
        ExpenseType expenseType = new ExpenseType();
        expenseType.setExpenseTypeId(new Long(1));
        expenseType.setName("TRAVEL");
        expenseTypeService.save(expenseType);
        expenseType.setExpenseTypeId(new Long(2));
        expenseType.setName("GYM");
        expenseTypeService.save(expenseType);
        expenseType.setExpenseTypeId(new Long(3));
        expenseType.setName("MEDICAL");
        expenseTypeService.save(expenseType);
        expenseType.setExpenseTypeId(new Long(4));
        expenseType.setName("TELEPHONE");
        expenseTypeService.save(expenseType);
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

The application requires the following files to be added, `login.jsp`, `expenseapproval.jsp`, `myexpense.jsp`, `newexpense.jsp`, `signup.jsp` in `webapp/WEB-INF/view` folder.

You can download the jsp files [here].(/frameworks/java/spring/tutorials/springmvc-mongodb/code/views.html).

*Note:*
: Before run the application, make sure your MongoDB instance is running.

## Check Point
1. Let's build the app and test it locally. Right click on the app and select **Run As -> Maven clean**.

    ![maven_clean.png](/images/spring_tutorial/maven_clean.png)

2. Once Maven clean completes, select **Run As -> Maven install**. Dependencies will be downloaded from pom.xml.

    ![maven_install.png](/images/spring_tutorial/maven_install.png)

3. Next, select **Run As -> Maven build**.

    ![maven_build.png](/images/spring_tutorial/maven_build.png)

4. Enter **tomcat:run** for Goals.

    ![maven_run.png](/images/spring_tutorial/maven_run.png)

5. Once the server starts, open your browser and enter the application url : `http://localhost:8080/html5expense/expensereports/new`. A form to create a new expense will open.

    ![create new expense](/images/spring_tutorial/create_new_expense.png)

6. Go to your MongoDB client and give the command **show collections**. It will list all the collections in the database so you can check that new collections have been created for the Entities.

```bash
$ show collections
Attachment
Expense
ExpenseType
Role
User
system.indexes
$
```

## Deploying to Cloud Foundry
* To learn how to deploy a Spring App using MongoDB to Cloud Foundry, please refer [here](/frameworks/java/spring/tutorials/springmvc-mongodb/springmvc-app-with-mongodb-deployment-to-cloudfoundry.html).

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-mongodb/spring-getting-started-with-STS.html">Prev</a> <a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/java/spring/tutorials/springmvc-mongodb/expensereport-app-with-spring-security.html">Next</a>
