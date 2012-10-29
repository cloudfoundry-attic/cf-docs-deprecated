---
title: Expense Reporting App using MongoDB
description: Create a Spring MVC App using MongoDB and Deploying on Cloud Foundry
tags:
    - Spring
    - STS
    - tomcat
    - maven
    - MongoDB
---

## Introduction
In this tutorial, you will use Spring MVC and MongoDB to create an Expense Reporting app and then deploy it on Cloud Foundry.
The Expense Reporting app will be used to create and manage expenses.

## Prerequisites
Before you begin this tutorial, you should:

1.  Be familiar with STS (Spring Tool Suite) or Eclipse.

### Setup:Exercise1-Starter
Import the source code from the downloaded folder Exercise1-Starter as follows. Open STS, Select **File > Import > Maven > Existing Maven Projects** and select Exercise1-Starter folder.
![maven-import-step1](/images/spring_tutorial/import-maven-project-step1.png)
![maven-import-step2](/images/spring_tutorial/import-maven-project-exercise1-starter.png)

In the Expense Reporting App, there are two roles:

1.   Normal User - Can create an expense, view its status, edit or delete it.

2.   Manager User -  Can Approve or Reject expenses created by a Normal user.

Once a Normal User submits an expense, it goes to the Manager's work items. The Manager can then Approve or Reject the expense.

![normal-user-usecase.png](/images/spring_tutorial/normal-user-usecase.png)

![manager-user-usecase_admin.png](/images/spring_tutorial/manager-user-use-case.png)

These expenses are created by the users and the data
is stored in MongoDB document database.

## Application Flow:
The below figure shows the runtime interaction of the application.

<img src="/images/spring_tutorial/sequence-flow-expense-report-app-using-mongo.png" width="70%" alt="sequence-flow">


1.  User logs in to the Expense Reporting system.

2.  User creates or deletes or updates an expense.

3.  The expense goes to the ExpenseSevice where MongoTemplate will persist or delete or update its state in the MongoDB document database.

*  The class diagram of the application is :

	![spring-expensereport-class_diagram.png](/images/spring_tutorial/class-diagram.png)

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

## **Step 1:** Adding Documents
MongoDB is a document oriented database, the java classes will be stored as BSON documents. The domain classes are marked with @Document annotation at class level to indicate that this class is a candidate to map the database. By default, the class name is choosen as a collection name, if you want to store it under a different name, set under collection property.
```java
@Document(collection = "Expense")
public class Expense implements Serializable{

 @Id
 private Long expenseId;
 private String description;
 private Double amount;
 private State state = State.NEW;
 private Date expenseDate;
 private User user;
 private ExpenseType expenseType;
 private Attachment attachment;
 public Expense(){
  
 }
 public Expense(String description,ExpenseType expenseType,Date expenseDate,
   Double amount,User user,Attachment attachment){
  this.description = description;
  this.expenseType = expenseType;
  this.expenseDate = expenseDate;
  this.amount = amount;
  this.user = user;
  this.attachment = attachment;
 }
 
 public String getDescription() {
  return description;
 }
 public void setDescription(String description) {
  this.description = description;
 }
 public State getState() {
  return state;
 }
 public void setState(State state) {
  this.state = state;
 }
 public Long getExpenseId() {
  return expenseId;
 }
```
The `@org.springframework.data.annotation.Id` annotation is applied to fields. The field which annotated with @Id will be stored as `_id` in MongoDB. This field is used to uniquely identify the document in a collection. The collection is similar to a table in RDBMS and the document is similar to a row in RDBMS.

Create a package for model class as `com.springsource.html5expense.model` and add the following classes `Attachment`, `Expense`, `ExpenseType`,
`Role`, `User`.

You can find the Entity classes [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/documents.html)

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

* Create a package for service interfaces as `com.springsource.html5expense.service` and add `AttachmentService`, `ExpenseService`, `ExpenseTypeService`, `UserService` interfaces which define the business methods.
You can get the service interfaces [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/services.html)

* Create a package `com.springsource.html5expense.mongodb.services` for service implementation and add the following classes
 `AttachmentRepository`, `ExpenseRepository`, `ExpenseTypeRepository`,
`RoleRepository` and `UserRepository`. The MongoTemplate has been autowired. It handles transactions and is also responsible
 for persisting, removing and finding domain objects that are mapped to the database as documents.

```java
@Service
public class ExpenseRepository implements ExpenseService {
    @Autowired
    MongoTemplate mongoTemplate;
    public static String EXPENSE_COLLECTION_NAME = "Expense";

    public List<Expense> getAllExpenses(){
        return mongoTemplate.findAll(Expense.class,EXPENSE_COLLECTION_NAME);
    }
    public List<Expense> getExpensesByUser(User user){
        return mongoTemplate.find(new Query(Criteria.where("user").is(user)) , Expense.class,EXPENSE_COLLECTION_NAME);
    }
    public List<Expense> getPendingExpensesList(){
        List<State> stateList = new ArrayList<State>();
        stateList.add(State.NEW);
        stateList.add(State.OPEN);
        stateList.add(State.IN_REVIEW);
        return mongoTemplate.find(new Query(Criteria.where("state").in(stateList)), Expense.class, EXPENSE_COLLECTION_NAME);
    }
    public List<Expense> getApprovedAndRejectedExpensesList(){
        List<State> stateList = new ArrayList<State>();
        stateList.add(State.APPROVED);
        stateList.add(State.REJECTED);
        return mongoTemplate.find(new Query(Criteria.where("state").in(stateList)), Expense.class,EXPENSE_COLLECTION_NAME);
    }
}
```
You can find the MongoDB service implementation classes [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/services-implementation.html)

Once you have added service implementation classes, ensure you aren't getting errors.

## **Step 3:** Adding the controller
The Controller is responsible for mapping requests. The `@RequestMapping` annotation is used to map requests onto specific handler methods. Create a package for Controllers as `com.springsource.html5expense.controller` and add the following class: `ExpenseController`.
To support uploading attachments of expenses, we have added a createNewExpenseReport function in ExpenseController which calls for a file upload mechanism. To automatically detect the presence of the fileupload simply add a `@RequestParam` argument of type MultipartFile.
```java
@RequestMapping(value="/createNewExpenseReport",method = RequestMethod.POST)
public String createNewExpenseReport(@RequestParam("file") MultipartFile file,
     HttpServletRequest request){
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
    return "redirect:/";
}
```
In the method below we have used `ModelMap`. It is a place where you can store key/value pairs to be used by the view in rendering a response.  It is through this context that you request specific information in the response.
 In our controller, we have gathered some data about the pending expenses and made them available in the ModelMap using the autowired ExpenseService reference. We can also use HTTP request to store key/values pair of data. To use HttpServletRequest, simply add another argument of type HttpServletRequest.

```java
@RequestMapping(value="/", method = RequestMethod.GET)
public String printWelcome(ModelMap model, Principal principal,HttpServletRequest request ) {
    String name = principal.getName();
    model.addAttribute("username", name);
    User user = getUserService().getUserByUserName(name);
    List<Expense> pendingExpenseList = getExpenseService().getExpensesByUser(user);
    model.addAttribute("pendingExpenseList",pendingExpenseList);
    request.getSession().setAttribute("user", user);
    return "myexpense";
}
```
You can get the controller classes code [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/controllers.html)

## **Step 4:** Configuring the ExpenseReport application
The application needs to be configured in order to make it ready to deploy. Create a new package `com.springsource.html5expense.config` and add the following classes: `ComponentConfig`, `WebConfig`. The ComponentConfig class has been marked with `@Configuration` annotation to configure beans. This is an alternative to XML configuration for bean definition. We can pass this class as an argument to Spring container as a source for bean creation.
 To declare a bean, simply annotate a method with the `@Bean` annotation in your config class. This method is used to register a bean definition within an ApplicationContext of the type specified as the method's return value. By default, the bean name will be the same as the method name.

Create a `MongoDbFactory` bean to establish a connection to MongoDB and create a `MongoTemplate` bean to work with data from a MongoDB database.

*Note:*
: While deploying to Cloud Foundry, do not keep database properties such as username, password, database name, host name in a separate file. Doing so will cause the auto-reconfiguration mechanism will not work.

```java
@Configuration
@EnableTransactionManagement
@PropertySource("/config.properties")
@ImportResource("/WEB-INF/spring-security.xml")
@ComponentScan(basePackageClasses = {JpaExpenseServiceImpl.class,ExpenseController.class,
Expense.class})
public class ComponentConfig {

  @Bean
  public MongoDbFactory mongoDbFactory() throws Exception {
    return new SimpleMongoDbFactory(new Mongo("localhost",27017), "test");
   }

  @Bean
  public MongoTemplate mongoTemplate() throws Exception {
      MongoTemplate mongoTemplate = new MongoTemplate(mongoDbFactory());
      return mongoTemplate;
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

  To load `ComponentConfig` class as part of contextConfiglocation, add contextClass as `AnnotationConfigWebApplicationContext` and `contextConfigLocation` as
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

You can get the Config classes [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/config.html).

## **Step 5:** Initialize database with default values
* Create a new class called `InitService` in `com.springsource.html5expense.mogodb.services`, and add a init method with the `@PostConstruct` annotation
to initialize default values in database. The method annotated with `PostConstruct` annotation needs to be executed after dependency injection is done to perform initialization.

```java
@Service
@Transactional
public class InitService {

    @Autowired
    MongoTemplate mongoTemplate;

    @PostConstruct
    private void init() {

        mongoTemplate.getCollection("ExpenseType").drop();
        mongoTemplate.getCollection("User").drop();
        mongoTemplate.getCollection("ExpenseType").drop();
        mongoTemplate.getCollection("Expense").drop();
        mongoTemplate.getCollection("Role").drop();
        DBCollection userCollection = mongoTemplate.getCollection("User");
        DBCollection roleCollection = mongoTemplate.getCollection("Role");
        DBCollection expenseTypeCollection = mongoTemplate.
                getCollection("ExpenseType");

        BasicDBObject roleDoc = new BasicDBObject();
        roleDoc.put("_id", new Long(1));
        roleDoc.put("roleId", new Long(1));
        roleDoc.put("roleName", "ROLE_USER");
        roleCollection.insert(roleDoc);

        roleDoc.clear();
        roleDoc.put("_id", new Long(1));
        roleDoc.put("roleId", new Long(1));
        roleDoc.put("roleName", "ROLE_MANAGER");
        roleCollection.insert(roleDoc);

        BasicDBObject userDoc = new BasicDBObject();
        userDoc.put("_id", new Long(1));
        userDoc.put("userId", new Long(1));
        userDoc.put("emailId", "admin@expensereport.com");
        userDoc.put("enabled", new Boolean(true));
        userDoc.put("userName","admin");
        userDoc.put("password", "admin");
        userDoc.put("role",roleDoc);

        userCollection.insert(userDoc);
        BasicDBObject expenseTypeDoc = new BasicDBObject();
        expenseTypeDoc.put("_id", new Long(1));
        expenseTypeDoc.put("expenseTypeId", new Long(1));
        expenseTypeDoc.put("name","GYM");
        expenseTypeCollection.insert(expenseTypeDoc);

        expenseTypeDoc.clear();
        expenseTypeDoc.put("_id", new Long(2));
        expenseTypeDoc.put("expenseTypeId", new Long(2));
        expenseTypeDoc.put("name","TELEPHONE");
        expenseTypeCollection.insert(expenseTypeDoc);
        expenseTypeDoc.clear();
        expenseTypeDoc.put("_id", new Long(3));
        expenseTypeDoc.put("expenseTypeId", new Long(3));;
        expenseTypeDoc.put("name","MEDICARE");
        expenseTypeCollection.insert(expenseTypeDoc);
        expenseTypeDoc.clear();
        expenseTypeDoc.put("_id", new Long(4));
        expenseTypeDoc.put("expenseTypeId", new Long(4));
        expenseTypeDoc.put("name","TRAVEL");
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

The application requires the following files to be added, `login.jsp`, `expenseapproval.jsp`, `myexpense.jsp`, `newexpense.jsp`, `signUp.jsp` in `webapp/WEB-INF/view` folder. You can download the jsp files [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/views.html).

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

5. Once the server starts, open your browser and enter the application url : `http://localhost:8080/html5expense/createNewExpense`. A form to create a new expense will open.

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

<a class="button-plain" style="height: 15px; width: 30px" href="/frameworks/java/spring/tutorials/springmvc-mongodb/spring-getting-started-with-STS.html">Prev</a> <span style="float: right;"><a class="button-plain" style="height: 15px; width: 30px" href="/frameworks/java/spring/tutorials/springmvc-mongodb/expensereport-app-with-spring-security.html">Next</a></span>
