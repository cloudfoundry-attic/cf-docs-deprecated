---
title: ExpenseReport Application Development using Spring MVC
description: ExpenseReport Application
tags:
    - Spring
    - STS
    - tomcat
    - maven
---

## Introduction
In this tutorial, we take up the following use case, creating and deploying an expense reporting app on local Tomcat server.
The ExpenseReport will be used to create and manage expenses.

## Prerequisites
Before you begin this tutorial, you should be familiar with:

1.  Working with STS or Eclipse IDE.

2.  Your previous exercise is complete.

In expensereport application,we are using two roles.

1.   Normal User - Can create an expense,view its status of expense,edit or delete it.

2.   Manager User -  Can Approve or Reject expenses created by Normal user.

     Once a Normal User submits his expense it goes to the Admin's work items. He can Approve or Reject the expense.

![normal-user-usecase.png](/images/spring_tutorial/normal-user-usecase.png)

![manager-user-usecase_admin.png](/images/spring_tutorial/manager-user-use-case.png)

These expenses are created by the users and the data
is stored in PostgreSQL database.

The ER diagram for our application is :

![spring-expensereport-class_diagram.png](/images/spring_tutorial/expensereport-er-diagram.png)

Before creating Entities,delete `HomeController.java` from your `com.springsource.html5config` package and replace your pom.xml with this [pom.xml](/code/springmvc-expense-report/pom.xml).

*Notes:*
: Before going on to build the application make sure you add Entities,Services,Controllers,
    Application Configuration classes and Views.

: You can get the expense report application code [here](#application-code).

## Adding Entities
An entity is a lightweight persistence domain object. Typically, an entity represents a table in a relational database, and each entity instance corresponds to a row in that table.The persistent state of an entity is represented through either persistent fields or persistent properties. These fields or properties use object/relational mapping annotations to map the entities and entity relationships to the relational data in the underlying data store.
The application requires the following entities to be added.  `Expense`,`User`,`Role`,`Attachment`,`ExpenseReport`and,`State` classes.These classes are POJO with getters and setters for its properties and annotated with `@Entity`. By default JPA takes class name as table name. If you want to store in under a different table name then provide `@Table` annotation with name property. Create a package for entities as `com.springsource.html5expense.model` and add all entities to it.
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

The `@Id` annotation specifies the primary key of the entity `@GeneratedValue` annotations which tell JPA that the value in that field should map to the primary key and that the primary key should use an auto-incrementing value strategy. `@OneToOne` annotation defines a single-valued association 
to another entity that has one-to-one multiplicity.  

## Adding Services
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
    public void updateExpense(Long expenseId,String description,Double amount,ExpenseType expenseType);
}
```
These are the business methods to,

+ creating,updating,deleting expenses.

+ getting all expenses and pending expenses based on user.

+ updating expense status.

* Create a package for service interfaces as `com.springsource.html5expense.service` and add `AttachmentService`,`ExpenseReportService`,`ExpenseService`,`ExpenseTypeService`,`UserService` interfaces which define the business methods.

* Create a package `com.springsource.html5expense.serviceImpl` for service implementation and add the following classes 
`JpaAttachmentServiceImpl`,`JpaExpenseReportServiceImpl`,`JpaExpenseServiceImpl`,`JpaExpenseTypeServiceImpl`,
`JpaRoleServiceImpl` and ,`JpaUserServiceImpl`. The EntityManager has been autowired. It handles transaction and is also responsible 
for persisting,removing, and finding entities that are mapped by JPA to database. @PersistenceContext which will use EntityManagerFactory to create EntityManager instance.
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
public List<Expense> getExpensesByUser(User user)
{
  Query query = getEntityManager().createQuery("select e from Expense e where e.user.userId =:userId ORDER BY e.expenseDate DESC",Expense.class);
  query.setParameter("userId", user.getUserId());
  return query.getResultList();
}
```

One you have added service implementation classes,ensure you aren't getting errors.

## Adding controller
The Controller is responsible for mapping requests. The `@RequestMapping` annotation is used to map requests onto specific handler methods. Create a package for Controllers as `com.springsource.html5expense.controller` and add the following classes `ExpenseController`, `FileAttachmentController`.
To support uploading attachments of expenses we have added Save function in FileAttachmentController this calls for a file upload mechanism. To automatically detect the presence of the fileupload simply add a `@RequestParam` argument of type MultipartFile.
```java
        @RequestMapping(value = "/save", method = RequestMethod.POST)
        public String save(@RequestParam("file") MultipartFile file) {
            try {
                String fileName =file.getOriginalFilename();
                String contentType = file.getContentType();
                Attachment attachment = new Attachment(fileName, contentType, file.getBytes());
                attachmentService.save(attachment);
            } catch (IOException e) {
                e.printStackTrace();
            }
            return "redirect:/";
        }
```
The method below we have used `ModelMap`. It is a place where you can store key/value pairs to be used by the view in rendering a response. It is through this context that you communicate request specific information in the response. 
In our handler, we have gathered some data about the pending expenses and made them available in the ModelMap using the injected ExpenseService reference. We can also use HTTP request to store key/values pair of data, to use HttpServletRequest simply add another argument of type HttpServletRequest.

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


## Configuring ExpenseReport application
The application needs to be configured in order to make it ready to deploy. Create a new package `com.springsource.html5expense.config` and add the following classes ComponentConfig,WebConfig. The ComponentConfig class has been marked with @Configuration annotation to configure beans. This is an alternative to XML configuration for bean definition.We can pass this class as an argument to Spring container as a source for bean creation. 
To declare a bean, simply annotate a method with the @Bean annotation in your config class.This method is used to register a bean definition within an ApplicationContext of the type specified as the method's return value. By default, the bean name will be the same as the method name.

In order to work with data from a PostgreSQL database, we need to obtain a connection to the database. For that configure Spring DataSource bean.
```java
@Configuration
@EnableTransactionManagement
@PropertySource("/config.properties")
@ImportResource({ "classpath:spring-security.xml"})
@ComponentScan(basePackageClasses = {JpaExpenseServiceImpl.class,ExpenseController.class,Expense.class})

public class ComponentConfig {

 @Autowired
 Environment env;
   
 @Bean
 public DataSource dataSource()  {
  SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
  dataSource.setUrl(String.format("jdbc:postgresql://%s:%s/%s", env.getProperty("database.host"), env.getProperty("database.port"),
                env.getProperty("database.name")));
  dataSource.setDriverClass(org.postgresql.Driver.class);
  dataSource.setUsername(env.getProperty("database.username"));
  dataSource.setPassword(env.getProperty("database.password"));
  return dataSource;
 }

@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
 LocalContainerEntityManagerFactoryBean emfb = new LocalContainerEntityManagerFactoryBean();
 emfb.setJpaVendorAdapter( jpaAdapter());
 emfb.setDataSource(dataSource());
 emfb.setJpaPropertyMap(createPropertyMap());
 emfb.setJpaDialect(new HibernateJpaDialect());
 emfb.setPersistenceUnitName("sample");
 emfb.setPackagesToScan(new String[]{Expense.class.getPackage().getName()});
 return emfb;
}

public Map<String,String> createPropertyMap()
{
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

To identify multipart requests we need to configure multipartResolver bean.
```java
    @Bean
    public MultipartResolver multipartResolver(){
        CommonsMultipartResolver multipartResolver = new org.springframework.web.multipart.commons.CommonsMultipartResolver();
        multipartResolver.setMaxUploadSize(10000000);
        return multipartResolver;
    }
```

Next, configure the components using @ComponentScan that will Configure component scanning directives for use with @Configuration classes. It provides  parallel support with Spring XML's <context:component-scan> element and pass one of basePackageClasses(), basePackages() value.

  @EnableTransactionManagement annotation is responsible for Spring's annotation-driven transaction management capability, similar to the support found in Spring's <tx:*> XML namespace. 

* Create a new file `import.sql` in `src/main/resources` and add sql statements.

	    insert into role(roleid,rolename) values(nextval( 'hibernate_sequence'),'ROLE_USER');
	    insert into role(roleid,rolename) values(nextval( 'hibernate_sequence'),'ROLE_MANAGER');
	    insert into users(userid,emailid,enabled,password,username,role_roleid) values(nextval('hibernate_sequence'),'admin@expense.com',true,'admin','admin',2);
	    insert into expenseType(id,name) values(nextval( 'hibernate_sequence'),'MEDICAL');
	    insert into expenseType(id,name) values(nextval( 'hibernate_sequence'),'TRAVEL');
	    insert into expenseType(id,name) values(nextval( 'hibernate_sequence'),'TELEPHONE');
	    insert into expenseType(id,name) values(nextval( 'hibernate_sequence'),'GYM');
	    insert into expenseType(id,name) values(nextval( 'hibernate_sequence'),'EDUCATION');
    
To execute these statements automatically in the database while the app starts we have set 
these two JPA properties in the entityManagerFactory bean.  
```java
map.put(org.hibernate.cfg.Environment.HBM2DDL_AUTO, "create");
map.put(org.hibernate.cfg.Environment.HBM2DDL_IMPORT_FILES, "import.sql");
```

* To load `ComponentConfig` class as part of contextConfiglocation, add contextClass as AnnotationConfigWebApplicationContext and contextConfigLocation as
`com.springsource.html5expense.config`.
```xml
<context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
</context-param>
<context-param>
   <param-name>contextConfigLocation</param-name>
   <param-value>com.springsource.html5expense.config</param-value>
 </context-param>
```

## Adding View Files
The ExpenseReport app service is ready. To get users to interact with it,it needs a user interface. To resolve the Controllers return `view` value we have defined InternalViewResolver

```java
    @Bean
    public InternalResourceViewResolver internalResourceViewResolver() {
        InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
        internalResourceViewResolver.setPrefix("/WEB-INF/views/");
        internalResourceViewResolver.setSuffix(".jsp");
        return internalResourceViewResolver;
}
```

The application requires the following files to be added,`login.jsp`,`expenseapproval.jsp`,`myexpense.jsp`,`newexpense.jsp`,`usermain.jsp`,`signUp.jsp` in `webapp/WEB-INF/view` folder.

## Application Code:
[Click here](/code/springmvc-expense-report.zip) to get the application code :

## Check Point
Let us build the app and test it locally.
Select **Run As -> Maven clean**

![maven_clean.png](/images/spring_tutorial/maven_clean.png)

Once Maven clean completes select **Run As -> Maven install**. Dependencies will be downloaded from pom.xml.

![maven_install.png](/images/spring_tutorial/maven_install.png)

Select **Run As -> Maven build**

![maven_build.png](/images/spring_tutorial/maven_build.png)

And give Maven goal as , **tomcat:run**

![maven_run.png](/images/spring_tutorial/maven_run.png)

Once server start completes - open your browser and enter the application url : `http://localhost:8080/html5expense/createNewExpense`. A form to create new expense will open.

![STS Fabric Server.png](/images/spring_tutorial/create_new_expense.png)

[<==Prev](/frameworks/java/spring/springmvc-template-project.html)                                                         [Next==>](/frameworks/java/spring/expensereport-app-with-spring-security.html)

