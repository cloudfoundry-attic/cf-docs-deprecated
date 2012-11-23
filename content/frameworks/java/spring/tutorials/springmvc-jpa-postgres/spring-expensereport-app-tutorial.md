---
title: Expense Reporting App with JPA using PostgreSQL
description: Create a Spring MVC App with JPA using PostgreSQL
tags:
    - spring
    - sts
    - tomcat
    - maven
    - jpa
    - postgresql
---

## Introduction
In this tutorial, you will use Spring MVC and PostgreSQL to create an Expense Reporting app and then deploy it on Cloud Foundry.
The Expense Reporting app will be used to create and manage expenses.

## Prerequisites
Before you begin this tutorial, you should:

1.  Be familiar with STS (Spring Tool Suite) or Eclipse.

### Setup: Exercise2-Starter
Import the source code from the downloaded folder Exercise2-Starter as follows. Open STS, Select **File > Import > Maven > Existing Maven Projects** and select Exercise2-Starter folder.

  ![maven-import-step1](/images/spring_tutorial/import-maven-project-step1.png)

  ![maven-import-step2](/images/spring_tutorial/import-maven-project-step2.png)

In the Expense Reporting App, there are two roles:

1.   Normal User - Can create an expense, view its status, edit or delete it.

2.   Manager User -  Can Approve or Reject expenses created by a Normal user.

Once a Normal User submits an expense, it goes to the Manager's work items. The Manager can then Approve or Reject the expense.

<img src="/images/spring_tutorial/normal-user-usecase.png" alt="normal-user-usecase.png" width="30%">

<img src="/images/spring_tutorial/manager-user-use-case.png" alt="manager-user-usecase_admin.png" width="30%">

These expenses are created by the users and the data
is stored in PostgreSQL.

## Application Flow:
The below figure shows the runtime interaction of the application.

<img src="/images/spring_tutorial/sequence-flow.png" alt="sequence-flow" width="50%">


1.  User logs into the Expense Reporting system.

2.  User details are verified in the database.

3.  User creates or deletes or updates an expense.

4.  The expense goes to the ExpenseService where JPA will persist or delete or update its state in the database.

*Note:*
: Before building the application, make sure you add Entities, Services, Controllers,
    Application Configuration classes and Views.

## **Step 1:** Adding Entities
The `@Entity` annotation indicates that the JavaBean is a persistent entity. Typically, an entity represents a table in a relational database, and each entity instance corresponds to a row in that table. The persistent state of an entity is represented through either persistent fields or persistent properties. These fields or properties use object-relational mapping annotations to map the entities and entity relationships to the relational data in the underlying data store.

By default, JPA takes a class name as a table name. If you want to store it under a different table name, provide the `@Table` annotation with the name property. Create a package for entities as `com.springsource.html5expense.model` and add the following entities `Expense`, `User`, `Role`, `State`.
```java
@Entity
@Table(name = "EXPENSE")
public class Expense {

    @Id
    @GeneratedValue
    private Long id;

    private String description;

    private Double amount;

    @Enumerated(EnumType.STRING)
    private State state = State.NEW;

    private Date expenseDate;

    private User user;

    @OneToOne
    private ExpenseType expenseType;
}
```

The `@Id` annotation specifies the primary key of the entity and the `@GeneratedValue` annotation tells JPA that the value in that field should map to the primary key and that the primary key should use an auto-incrementing value strategy. The `@OneToOne` annotation defines a single-valued association to another entity that has one-to-one multiplicity.

You can get the Entity classes [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/entities.html).

## **Step 2:** Adding Services
`@Service` specifies a special component that provides the business services through interface. This annotation serves as a specialization of `@Component`, allowing for implementation classes to be autodetected through classpath scanning.

```java
public interface ExpenseService {
    Long createExpense(String description, ExpenseType expenseType, Date expenseDate,
               Double amount, User user);

    Expense getExpense(Long expenseId);

    List<Expense> getAllExpenses();

    List<Expense> getExpensesByUser(User user);

    List<Expense> getPendingExpensesList();

    List<Expense> getApprovedAndRejectedExpensesList();

    Expense changeExpenseStatus(Long expenseId, String state);

    void deleteExpense(Long expenseId);

    void updateExpense(Expense expense);
}
```
These are the business methods to:

+ Create, Update or Delete an Expense.

+ Get pending expenses for a user. Get all the expenses for a user.

+ Update an expense's status.

* Create a package for service interfaces as `com.springsource.html5expense.services` and add `ExpenseService`, `ExpenseTypeService`, `UserService` and `RoleService` interfaces which define the business methods.
You can get the service interfaces [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/services.html).

* Add the following service implementation classes `JpaExpenseService`, `JpaExpenseTypeService`,
`JpaRoleService` and `JpaUserService` in the `com.springsource.html5expense.services` package. The EntityManager has been autowired. The `@Autowired` annotation is auto wired to the bean by matching data type. It is associated with a persistence context. A persistence context is a set of entity instances in which for any persistent entity identity there is a unique entity instance. Within the persistence context, the entity instances and their lifecycle are managed. The EntityManager API is used to create and remove persistent entity instances, to find entities by their primary key, and to query over entities. @PersistenceContext uses the EntityManagerFactory to create an EntityManager instance. We have marked the createExpense method with @Transactional annotation which will ensure the method is running in transaction.

```java
@Service
public class JpaExpenseService implements ExpenseService {

    private EntityManager entityManager;

    public EntityManager getEntityManager() {
        return entityManager;
    }

    @PersistenceContext
    public void setEntityManager(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    @Transactional
    public Long createExpense(String description, ExpenseType expenseType, Date expenseDate,
            Double amount, User user) {
        Expense expense = new Expense(description, expenseType, expenseDate, amount, user);
        entityManager.persist(expense);
        return expense.getId();
    }

    @Transactional
    public Expense getExpense(Long expenseId) {
        return entityManager.find(Expense.class, expenseId);
    }
}
```
You can get the service implementation classes [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/services-implementation.html).

Once you have added service implementation classes, ensure you aren't getting errors.

## **Step 3:** Adding the controller
The Controller is responsible for mapping requests. The `@RequestMapping` annotation is used to map requests onto specific handler methods. The following table explains the URLs we used in the ExpenseReport application:

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
In the method below, we have used `ModelMap`. It is a place where you can store key/value pairs to be used by the view in rendering a response.  It is through this context that you request specific information in the response.

```java
@RequestMapping(value = "/", method = RequestMethod.GET)
public String getExpenses(ModelMap model) {
     List<ExpenseType> expenseTypeList = expenseTypeService.getAllExpenseType();
     model.addAttribute("expenseTypeList", expenseTypeList);
     model.addAttribute("user", getUser());
     return "expensereports";
}
```
 In our controller, we have gathered some data about the expense types and made them available in the ModelMap using the autowired ExpenseTypeService reference. We can also use HTTP requests to store key/value pairs of data. To use HttpServletRequest, simply add another argument of type HttpServletRequest.

 Since HTML only supports two methods: POST and GET. To use the DELETE and PUT methods, add `HiddenHttpMethodFilter` into your web.xml. Now use the Spring form tag with method as `delete` or use Ajax. It inspects the request and looks for a request parameter (usually called `_method`) that informs how it’s to transform the request.

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

 You can get the controller class code [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/controllers.html).


## **Step 4:** Configuring the ExpenseReport application
In order to work with data from a PostgreSQL database, we need to obtain a connection to the database in order to define the dataSource bean. If your PostgreSQL server requires authentication, put the username and password of your PostgreSQL server in the dataSource bean. Define an interface `DataSourceConfiguration` in the `com.springsource.html5expense.config` package to get `dataSource`. Now implement this interface for your local datasource as `LocalDataSourceConfiguration` and use the `@Profile("local")` annotation.

```java
@Configuration
@Profile("local")
@PropertySource("/db.properties")
public class LocalDataSourceConfiguration implements DataSourceConfiguration {

    @Autowired
    private PropertyResolver propertyResolver;

    @Bean
    public DataSource dataSource() {
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
        dataSource.setUrl(String.format("jdbc:postgresql://%s:%s/%s",
                 propertyResolver.getProperty("postgres.host"),
                 propertyResolver.getProperty("postgres.port"),
                 propertyResolver.getProperty("postgres.db.name")));
        dataSource.setDriverClass(org.postgresql.Driver.class);
        dataSource.setUsername(propertyResolver.getProperty("postgres.username"));
        dataSource.setPassword(propertyResolver.getProperty("postgres.password"));
        return dataSource;
    }
}
```
 Now implement `DataSourceConfiguration` for your Cloud Foundry datasource as `CloudDataSourceConfiguration` and use `@Profile("cloud")` annotation. To use the Cloud Foundry API, add the following dependency in your `pom.xml`:
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
public class CloudDataSourceConfiguration implements DataSourceConfiguration {

    private CloudEnvironment cloudEnvironment = new CloudEnvironment();

    @Bean
    public DataSource dataSource() {
        Collection<RdbmsServiceInfo> psqlServiceInfo = cloudEnvironment.
                                   getServiceInfos(RdbmsServiceInfo.class);
        RdbmsServiceCreator dataSourceCreator = new RdbmsServiceCreator();
        return dataSourceCreator.createService(psqlServiceInfo.iterator().next());
    }
}
```

 Now we have declared two datasources, one for your local environment and another for the Cloud Foundry environment. To avoid switching back from one database to another, we can make the selected profile active. Based on the active profile, only that particular bean should be created. We can select the active profiles based on the environment, create a class called `ExpenseReportAppContextInitializer` in `com.springsource.html5expense.web` and implement `ApplicationContextInitializer`. The ApplicationContextInitializer is used to set active profiles and register custom property sources.

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

The application needs to be configured in order to make it ready to deploy. Add the following classes: `ComponentConfig`, `WebConfig` in the `com.springsource.html5expense.config` package. The ComponentConfig class has been marked with a `@Configuration` annotation to configure beans. This is an alternative to XML configuration for bean definition. We can pass this class as an argument to the Spring container as a source for bean creation.
```java
@Configuration
@EnableTransactionManagement
@ComponentScan(basePackageClasses = {JpaExpenseService.class,
        ExpenseController.class, Expense.class })
public class ComponentConfig {

    @Autowired private DataSourceConfiguration dataSourceConfiguration;

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean emfb =
                     new LocalContainerEntityManagerFactoryBean();
        emfb.setJpaVendorAdapter(jpaAdapter());
        emfb.setDataSource(dataSourceConfiguration.dataSource());
        emfb.setJpaPropertyMap(jpaPropertyMap());
        emfb.setJpaDialect(new HibernateJpaDialect());
        emfb.setPackagesToScan(new String[]{Expense.class.getPackage().getName()});
        return emfb;
    }

    @Bean
    public JpaVendorAdapter jpaAdapter() {
        HibernateJpaVendorAdapter hibernateJpaVendorAdapter = new HibernateJpaVendorAdapter();
        hibernateJpaVendorAdapter.setShowSql(true);
        hibernateJpaVendorAdapter.setDatabase(Database.POSTGRESQL);
        hibernateJpaVendorAdapter.setShowSql(true);
        hibernateJpaVendorAdapter.setGenerateDdl(true);
        return hibernateJpaVendorAdapter;
    }

    public Map<String, String> jpaPropertyMap() {
        Map<String, String> map = new HashMap<String, String>();
        map.put(org.hibernate.cfg.Environment.HBM2DDL_AUTO, "create");
        map.put(org.hibernate.cfg.Environment.HBM2DDL_IMPORT_FILES, "import.sql");
        map.put("hibernate.c3p0.min_size", "5");
        map.put("hibernate.c3p0.max_size", "20");
        map.put("hibernate.c3p0.timeout", "360000");
        map.put("hibernate.dialect", "org.hibernate.dialect.PostgreSQLDialect");
        return map;
    }
}

```

 To declare a bean, simply annotate a method with the `@Bean` annotation in your config class. This method is used to register a bean definition within an ApplicationContext of the type specified as the method's return value. By default, the bean name will be the same as the method name.
   
To enable declarative transaction management, add a `@EnableTransactionManagement` annotation to the ComponentConfig class. The `@EnableTransactionManagement` annotation is responsible for Spring's annotation-driven transaction management capability, similar to the support found in Spring's <tx:*> XML namespace. This annotation will search for a bean of type PlatformTransactionManager registered in the context so define a bean `PlatformTransactionManager` in ComponentConfig class.
```java
@Bean
public PlatformTransactionManager transactionManager() {
    final JpaTransactionManager transactionManager = new JpaTransactionManager();
    transactionManager.setEntityManagerFactory(entityManagerFactory().getObject());
    Map<String, String> jpaProperties = new HashMap<String, String>();
    jpaProperties.put("transactionTimeout", "43200");
    transactionManager.setJpaPropertyMap(jpaProperties);
    return transactionManager;
}
```

Next, configure the components using `@ComponentScan`. It will Configure component scanning directives for use with `@Configuration` classes and provides parallel support with Spring XML's `<context:component-scan>` element, passing a basePackageClasses() or basePackages() value.

  Create a new file `import.sql` in `src/main/resources` and add sql statements.
```sql
insert into role(roleid,rolename) values(nextval('hibernate_sequence'),'ROLE_USER');
insert into role(roleid,rolename) values(nextval('hibernate_sequence'),'ROLE_MANAGER');
insert into users(userid,emailid,enabled,password,username,role_roleid) values(nextval('hibernate_sequence'),'admin@expense.com',true,'admin','admin',2);
insert into expenseType(id,name) values(nextval('hibernate_sequence'),'MEDICAL');
insert into expenseType(id,name) values(nextval('hibernate_sequence'),'TRAVEL');
insert into expenseType(id,name) values(nextval('hibernate_sequence'),'TELEPHONE');
insert into expenseType(id,name) values(nextval('hibernate_sequence'),'GYM');
insert into expenseType(id,name) values(nextval('hibernate_sequence'),'EDUCATION');
```

  To execute these statements automatically in the database when the app starts, we have set
 these two properties in the entityManagerFactory bean.
```java
map.put(org.hibernate.cfg.Environment.HBM2DDL_AUTO, "create");
map.put(org.hibernate.cfg.Environment.HBM2DDL_IMPORT_FILES, "import.sql");
```

  Modify the `db.properties` file properties database username, password, and database name according to your local database.

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

You can get the Config classes [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/config.html).

## **Step 5:** Adding View Files
The ExpenseReport app service is ready. To get users to interact with it, it needs a user interface. To resolve the Controller's return `view` value, we have defined InternalViewResolver.

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

The application requires the following files to be added: `login.jsp`, `expensereports.jsp`, and `signup.jsp` (in the `webapp/WEB-INF/view` folder). You can download the jsp files [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/views.html).

## Check Point
1. Select your project, drag it down and drop into your VMware vFabric tc server listed in the bottom left Server window.

2. Select VMware vFabric tc server and click the start button to run the server.

3. Once the server starts, open your browser and enter the application url : `http://localhost:8080/html5expense/expenses/1`. You will get an empty response since we haven't created any expense.

4. Go to your PostgreSQL client and give the command **\dt**. It will list all the tables in the database so you can check that new tables have been created for the Entities.
```sql
postgres=# \dt
                 List of relations
 Schema |          Name          | Type  |  Owner
--------+------------------------+-------+----------
 public | expense                | table | postgres
 public | expensetype            | table | postgres
 public | role                   | table | postgres
 public | users                  | table | postgres
(4 rows)

```

## Complete Application Code
If you are getting any errors, download the Exercise2-Complete code and import into STS.

## Deploying to Cloud Foundry
* To learn how to deploy a Spring App using PostgreSQL to Cloud Foundry, please refer [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/springmvc-app-with-postgresql-deployment-to-cloudfoundry.html).

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/springmvc-template-project.html">Prev</a><a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/expensereport-app-with-spring-security.html">Next</a>

