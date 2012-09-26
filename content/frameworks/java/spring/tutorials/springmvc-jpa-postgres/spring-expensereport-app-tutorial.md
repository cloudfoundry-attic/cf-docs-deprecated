---
title: Expense Reporting App with JPA using PostgreSQL
description: Create a Spring MVC App with JPA using PostgreSQL
tags:
    - Spring
    - STS
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

![normal-user-usecase.png](/images/spring_tutorial/normal-user-usecase.png)

![manager-user-usecase_admin.png](/images/spring_tutorial/manager-user-use-case.png)

These expenses are created by the users and the data
is stored in PostgreSQL.

## Application Flow:
The below figure shows the runtime interaction of the application.

![sequence-flow](/images/spring_tutorial/sequence-flow.png)


1.  User logs in to the Expense Reporting system.

2.  User creates or deletes or updates an expense.

3.  The expense goes to the ExpenseSevice where JPA will persist or delete or update its state in the database.

*  The class diagram of the application is :

	![spring-expensereport-class_diagram.png](/images/spring_tutorial/class-diagram.png)

Before creating Entities, replace pom.xml with [this](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/pom.html).

*Note:*
: Before building the application, make sure you add Entities, Services, Controllers,
    Application Configuration classes and Views.


## **Step 1:** Adding Entities
An entity is a lightweight persistence domain object. Typically, an entity represents a table in a relational database, and each entity instance corresponds to a row in that table. The persistent state of an entity is represented through either persistent fields or persistent properties. These fields or properties use object-relational mapping annotations to map the entities and entity relationships to the relational data in the underlying data store.
The application requires the following entities to be added.

+ Expense
+ User
+ Role
+ Attachment
+ State

These classes are POJOs with getters and setters for its properties and annotated with `@Entity`. By default, JPA takes a class name as a table name. If you want to store it under a different table name, then provide the `@Table` annotation with the name property. Create a package for entities as `com.springsource.html5expense.model` and add all entities to it.
```java
@Entity
@Table(name = "EXPENSE")
public class Expense implements Serializable{
    @Id
    @GeneratedValue
    private Long id;
    private String description;
    private Double amount;
    private State state = State.NEW;
    private Date expenseDate;
    @OneToOne
    private Attachment attachment;
}
```

The `@Id` annotation specifies the primary key of the entity and the `@GeneratedValue` annotation tells JPA that the value in that field should map to the primary key and that the primary key should use an auto-incrementing value strategy. The `@OneToOne` annotation defines a single-valued association
 to another entity that has one-to-one multiplicity.

You can get the Entity classes [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/entities.html).

## **Step 2:** Adding Services
`@Service` specifies a special component that provides the business services through interface.

```java
public interface ExpenseService {
    public Long createExpense(String description,ExpenseType expenseType,Date expenseDate,
     Double amount,User user,Attachment attachment);
    public Expense getExpense(Long expenseId);
    public List getAllExpenses();
    public List<Expense> getExpensesByUser(User user);
    public List<Expense> getPendingExpensesList();
    public List<Expense> getApprovedAndRejectedExpensesList();
    public Expense changeExpenseStatus(Long expenseId,String state);
    public void deleteExpense(Long expenseId);
    public void updateExpense(Long expenseId,String description,Double amount,
     ExpenseType expenseType);
}
```
These are the business methods to:

+ Create, Update or Delete an Expense.

+ Get pending expenses for a user. Get all the expenses for a user.

+ Update an expense's status.

* Create a package for service interfaces as `com.springsource.html5expense.service` and add `AttachmentService`, `ExpenseService`, `ExpenseTypeService`, `UserService` and `RoleService` interfaces which define the business methods.
You can get the service interfaces [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/services.html).

* Create a package `com.springsource.html5expense.serviceImpl` for service implementation and add the following classes
 `JpaAttachmentServiceImpl`, `JpaExpenseServiceImpl`, `JpaExpenseTypeServiceImpl`,
`JpaRoleServiceImpl` and `JpaUserServiceImpl`. The EntityManager has been autowired. It handles transactions and is also responsible
 for persisting, removing and finding entities that are mapped by JPA to the database. @PersistenceContext uses the EntityManagerFactory to create an EntityManager instance.

```java
@Service
public class JpaExpenseServiceImpl implements ExpenseService {
    private EntityManager entityManager;
    public EntityManager getEntityManager() {
    return entityManager;
}
@PersistenceContext
public void setEntityManager(EntityManager entityManager) {
    this.entityManager = entityManager;
}
@Override
@Transactional
public List<Expense> getExpensesByUser(User user){
    Query query = getEntityManager().createQuery("select e from Expense e where
     e.user.userId =:userId ORDER BY e.expenseDate DESC",Expense.class);
    query.setParameter("userId", user.getUserId());
    return query.getResultList();
}
```
You can get the service implementation classes [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/services-implementation.html).

Once you have added service implementation classes, ensure you aren't getting errors.

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
                <td>/expensereports</td>
                <td>Get all the expensereports for the user</td>
            </tr>
            <tr>
                <td>2</td>
                <td>POST</td>
                <td>/expensereports</td>
                <td>Create a new expensereport</td>
            </tr>
            <tr>
                <td>3</td>
                <td>GET</td>
                <td>/expensereports/{id}</td>
                <td>Return an expensereport whose expenseid is passed</td>
            </tr>
            <tr>
                <td>4</td>
                <td>POST</td>
                <td>/expensereports/{id}</td>
                <td>Update an expensereport whose expenseid is passed</td>
            </tr>
            <tr>
                <td>5</td>
                <td>DELETE</td>
                <td>/expensereports/{id}</td>
                <td>Delete an expensereport whose expenseid is passed</td>
            </tr>
    </tbody>
</table>

Create a package for Controllers as `com.springsource.html5expense.controller` and add the following class: `ExpenseController`.
To support uploading attachments of expenses, we have added a createNewExpenseReport function in ExpenseController which calls for a file upload mechanism. To automatically detect the presence of the fileupload simply add a `@RequestParam` argument of type MultipartFile.
```java
@RequestMapping(value="/expensereports",method = RequestMethod.POST)
public String createNewExpenseReport(@RequestParam("file") MultipartFile file,HttpServletRequest request){
    String description = request.getParameter("description");
    String expenseTypeVal =request.getParameter("expenseTypeId");
    ExpenseType expenseType = expenseTypeService.getExpenseTypeById(new Long(expenseTypeVal));
    String amount = request.getParameter("amount");
    Date expenseDate = new Date();
    Double amountVal = new Double(amount);
    User user = (User)request.getSession().getAttribute("user");
    String fileName = "";
    String contentType = "";
    if(file!=null){
       try{
            fileName =file.getOriginalFilename();
            contentType = file.getContentType();
            Attachment attachment = new Attachment(fileName, contentType,  file.getBytes());
            attachmentService.save(attachment);
            Long id = expenseService.createExpense(description,expenseType,expenseDate,
            amountVal,user,attachment);
        }catch(Exception e){
         }
    }
    return "redirect:/expensereports";
}
```
In the method below we have used `ModelMap`. It is a place where you can store key/value pairs to be used by the view in rendering a response.  It is through this context that you request specific information in the response.

```java
    @RequestMapping(value="/expensereports", method = RequestMethod.GET)
    public String getAllExpenseReports(ModelMap model, Principal principal,HttpServletRequest request ) {
         String name = principal.getName();
        model.addAttribute("username", name);
        User user = getUserService().getUserByUserName(name);
        List<Expense> pendingExpenseList = getExpenseService().getExpensesByUser(user);
        model.addAttribute("pendingExpenseList",pendingExpenseList);
        request.getSession().setAttribute("user", user);
        return "myexpense";
    }
```
 In our controller, we have gathered some data about the pending expenses and made them available in the ModelMap using the autowired ExpenseService reference. We can also use HTTP request to store key/values pair of data. To use HttpServletRequest, simply add another argument of type HttpServletRequest.

 Since html only supports two methods: POST and GET. To use DELETE and PUT method add `HiddenHttpMethodFilter` in web.xml. Now use Spring form tag with method as `delete`. The `HiddenHttpMethodFilter` will create an `_method` parameter that will be converted into the corresponding HTTP method request.

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

 You can get the controller classe code [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/controllers.html).


## **Step 4:** Configuring the ExpenseReport application
The application needs to be configured in order to make it ready to deploy. Create a new package `com.springsource.html5expense.config` and add the following classes: `ComponentConfig`, `WebConfig`. The ComponentConfig class has been marked with `@Configuration` annotation to configure beans. This is an alternative to XML configuration for bean definition. We can pass this class as an argument to Spring container as a source for bean creation.
 To declare a bean, simply annotate a method with the `@Bean` annotation in your config class. This method is used to register a bean definition within an ApplicationContext of the type specified as the method's return value. By default, the bean name will be the same as the method name.

In order to work with data from a PostgreSQL database, we need to obtain a connection to the database in order to define the dataSource bean. If your PostgreSQL serevr requires authentication, put the username and password of your PostgreSQL server in dataSource bean. Otherwise, don't set these two properties. We have to set the `org.hibernate.cfg.Environment.HBM2DDL_AUTO` JPA property, to automatically create tables for our Entities.

*Note:*
: While deploying to Cloud Foundry, do not keep database properties such as username, password, database name, host name in a separate file. Doing so will cause
 the auto-reconfiguration mechanism will not work.
```java
@Configuration
@EnableTransactionManagement
@ComponentScan(basePackageClasses = {JpaExpenseServiceImpl.class,ExpenseController.class,
Expense.class})
public class ComponentConfig {

  @Bean
  public DataSource dataSource()  {
    SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
    dataSource.setUrl("jdbc:postgresql://localhost:5432/postgres");
    dataSource.setDriverClass(org.postgresql.Driver.class);
    dataSource.setUsername("postgres");
    dataSource.setPassword("postgres");
    return dataSource;
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(){
    LocalContainerEntityManagerFactoryBean emfb = new LocalContainerEntityManagerFactoryBean();
    emfb.setJpaVendorAdapter( jpaAdapter());
    emfb.setDataSource(dataSource());
    emfb.setJpaPropertyMap(createPropertyMap());
    emfb.setJpaDialect(new HibernateJpaDialect());
    emfb.setPackagesToScan(new String[]{Expense.class.getPackage().getName()});
    return emfb;
  }

  public Map<String,String> createPropertyMap(){
    Map<String,String> map= new HashedMap();
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

To identify multipart requests, we need to define the `multipartResolver` bean.
```java
@Bean
public MultipartResolver multipartResolver(){
    CommonsMultipartResolver multipartResolver = CommonsMultipartResolver();
    multipartResolver.setMaxUploadSize(10000000);
    return multipartResolver;
}
```
Next, configure the components using `@ComponentScan`. It will Configure component scanning directives for use with `@Configuration` classes and provides parallel support with Spring XML's `<context:component-scan>` element,passing a basePackageClasses() or basePackages() value.

  The `@EnableTransactionManagement` annotation is responsible for Spring's annotation-driven transaction management capability, similar to the support found in Spring's <tx:*> XML namespace.

* Create a new file `import.sql` in `src/main/resources` and add sql statements.

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

* To execute these statements automatically in the database while the app starts, we have set
 these two JPA properties in the entityManagerFactory bean.
```java
map.put(org.hibernate.cfg.Environment.HBM2DDL_AUTO, "create");
map.put(org.hibernate.cfg.Environment.HBM2DDL_IMPORT_FILES, "import.sql");
```

* To load `ComponentConfig` class as part of contextConfiglocation, add contextClass as `AnnotationConfigWebApplicationContext` and `contextConfigLocation` as
`com.springsource.html5expense.config`.
```xml
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

The application requires the following files to be added, `login.jsp`, `expenseapproval.jsp`, `myexpense.jsp`, `newexpense.jsp`, `signup.jsp` in `webapp/WEB-INF/view` folder. You can download the jsp files [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/code/views.html).

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

6. Go to your PostgreSQL client and give the command **\dt**. It will list all the tables in the database so you can check that new tables have been created for the Entities.
```sql
postgres=# \dt
                 List of relations
 Schema |          Name          | Type  |  Owner
--------+------------------------+-------+----------
 public | attachment             | table | postgres
 public | expense                | table | postgres
 public | expense_report_expense | table | postgres
 public | expensereport          | table | postgres
 public | expensetype            | table | postgres
 public | role                   | table | postgres
 public | users                  | table | postgres
(7 rows)

```

## Complete Application Code
If you are getting any errors, download the Exercise2-Complete code and import into STS.

## Deploying to Cloud Foundry
* To learn how to deploy a Spring App using PostgreSQL to Cloud Foundry, please refer [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/springmvc-app-with-postgresql-deployment-to-cloudfoundry.html).

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/springmvc-template-project.html">Prev</a><a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/expensereport-app-with-spring-security.html">Next</a>

