---
title: Expense Reporting App using MongoDB
description: Create a Spring MVC App using MongoDB
tags:
    - Spring
    - STS
    - tomcat
    - maven
    - MongoDB
---

## Introduction
In this tutorial, you will use ExpenseReport app to modify for MongoDB and then deploy it on Cloud Foundry.
The Expense Reporting app will be used to create and manage expenses.

## Prerequisites
Before you begin this tutorial, you should:

1.  Be familiar with STS (Spring Tool Suite) or Eclipse.

Inorder to work with Spring data MongoDB add the following dependency in pom.xml.

+ mongo-java-driver
+ spring-data-mongodb

## **Step 1:** Modify Entities
Remove JPA based annotation from Entities and add `@Document` annotation at class level. The `@Document` annotation indicates that this class is a candidate for mapping to the database. By default, the class name will be choosen for collection name, if you want to store in different name then set collection property.
```java
@Document(collection = "Expense")
public class Expense implements Serializable{
    @Id
    private Long expenseId;
    private String description;
    private Double amount;
    private State state = State.NEW;
    private Date expenseDate;   
    private Attachment attachment;
}
```

The `@org.springframework.data.annotation.Id` annotation is applied to fields. MongoDB lets you store any type as the _id field in the database, including long and string. It is of course common to use ObjectId for this purpose. If the value on the @Id field is not null, it is stored into the database as-is. If it is null, then the converter will assume you want to store an ObjectId in the database.

You can get the Entity classes [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/entities.html)

## **Step 2:** Modify Config Class
Remove the beans from ComponentConfig class which are created to work with JPA and PostgreSQL like dataSource. Create a MongoTemplate bean, in order to work with data from a MongoDB database.
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
*Note:*
: Don't keep database properties such as username, password, database name, host name into separate file. While deploying to Cloud Foundry
    the auto-reconfiguration mechanism will not work.

## **Step 3:** Adding Services Implementation Using MongoDB
* Before that delete the package `com.springsouce.html5expense.servicesImpl`. Since this package implementation are specific to JPA and add the service implementation classes using MongoDB.

* Create a package `com.springsource.html5expense.mongodb.services` for service implementation and add the following classes
 `AttachmentRepository`, `ExpenseReportRepository`, `ExpenseRepository`, `ExpenseTypeRepository`,
`RoleRepository` and `UserRepository`. The MongoTemplate has been autowired. It handles transactions and is also responsible
 for persisting, removing and finding entities that are mapped to the database.

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
    
    public Expense getExpense(Long expenseId){
        List<Expense> expenseList = mongoTemplate.find(new Query(Criteria.where("id").is(expenseId)),Expense.class,EXPENSE_COLLECTION_NAME);
        return expenseList!=null && expenseList.size()>0?expenseList.get(0):null;
    }
    
    public Expense changeExpenseStatus(Long expenseId,String state){
        Update update = new Update();
        update.set("state", State.valueOf(state));
        mongoTemplate.updateFirst(new Query(Criteria.where("id").is(expenseId)), update,EXPENSE_COLLECTION_NAME);
        return getExpense(expenseId);
    }
        
    public void deleteExpense(Long expenseId){
        Expense expense = getExpense(expenseId);
        mongoTemplate.remove(expense);
    }
}
```
You can get the MongoDB service implementation classes [here](/frameworks/java/spring/tutorials/springmvc-mongodb/code/services-implementation.html)

Once you have added service implementation classes, ensure you aren't getting errors.


## **Step 4:** Initialize database with default values
* Create a new class called `InitService` in `com.springsource.html5expense.mogodb.services`, and add a init method with the `@PostConstruct` annotation
to initialize default values in database. The method annotated with `PostConstruct` annotation is used on a method that needs to be executed after dependency injection is done to perform initialization.

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

## **Step 5:** Modify Spring Security
Create a custom AuthenticationProvider by extending `AbstractUserDetailsAuthenticationProvider` and point it to spring-security.xml.

```java
@Component
public class LocalAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {
  
    @Autowired
    UserService userService;
    
    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication)
            throws AuthenticationException {
        
    }

    @Override
    public UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
            throws AuthenticationException {
        String password = (String) authentication.getCredentials();
        if (!StringUtils.hasText(password)) {
            throw new BadCredentialsException("Please enter password");
        }

        com.springsource.html5expense.model.User user = userService.getUserByUserName(username);
        if (user == null) {
            throw new BadCredentialsException("Invalid Username/Password");
        }
       
        final List<GrantedAuthority> auths;
        if (null!= user.getRole()) {
            auths = AuthorityUtils.commaSeparatedStringToAuthorityList(user.getRole().getRoleName());
        } else {
            auths = AuthorityUtils.NO_AUTHORITIES;
        }

        return new User(username, password, user.isEnabled(),true, true, 
                true, auths);
    }

}
```

Then modify AuthenticationProvider bean reference to this LocalAuthenticationProvider in `spring-security.xml`.
```xml
    <authentication-manager alias="authenticationManager">
        <authentication-provider ref="localAuthenticationProvider"/>
    </authentication-manager>
```

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

<a class="button-plain" style="height: 15px; width: 30px" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/springmvc-template-project.html">Prev</a> <span style="float: right;"><a class="button-plain" style="height: 15px; width: 30px" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/expensereport-app-with-spring-security.html">Next</a></span>

