---
title: Expense Reporting App using Redis
description: Create a Spring MVC App using Redis
tags:
    - Spring
    - STS
    - tomcat
    - maven
    - Redis
---

## Introduction
In this tutorial, you will use ExpenseReport app to modify for Redis and then deploy it on Cloud Foundry.
The Expense Reporting app will be used to create and manage expenses.

## Prerequisites
Before you begin this tutorial, you should:

1.  Be familiar with STS (Spring Tool Suite) or Eclipse.

Inorder to work with Spring data Redis add the following dependency in pom.xml.

+ spring-data-redis
+ redis.clients-jedis

## **Step 1:** Modify Config Class
Remove the beans from ComponentConfig class which are created to work with JPA and PostgreSQL like dataSource. Create a JedisConnectionFactory bean, in order to work with data from a Redis database.
```java
@Configuration
@EnableTransactionManagement
@PropertySource("/config.properties")
@ImportResource("/WEB-INF/spring-security.xml")
@ComponentScan(basePackageClasses = {ExpenseRepository.class,ExpenseController.class,
Expense.class})
public class ComponentConfig {

 @Bean
 public JedisConnectionFactory jedisConnectionFactory(){
   JedisConnectionFactory jedisConenctionFactory = new JedisConnectionFactory();
   jedisConenctionFactory.setHostName("localhost");
   jedisConenctionFactory.setPort(6379);
   return jedisConenctionFactory;
 }

 @Bean
 public RedisTemplate<String,Cacheable> redisTemplate(){
   RedisTemplate<String,Cacheable> redisTemplate = new RedisTemplate<String,Cacheable>();
   redisTemplate.setConnectionFactory(jedisConnectionFactory());
   return redisTemplate;
 }
}

```
*Note:*
: Don't keep database properties such as username, password, database name, host name into separate file. While deploying to Cloud Foundry
    the auto-reconfiguration mechanism will not work.

## **Step 2:** Adding Services Implementation Using Redis
* Before that delete the package `com.springsouce.html5expense.servicesImpl'. Since this package implementation are specific to JPA and add the service implementation classes using Redis.

* Create a package `com.springsource.html5expense.redis.services` for service implementation and add the following classes
 `AttachmentRepository`, `ExpenseReportRepository`, `ExpenseRepository`, `ExpenseTypeRepository`,
`RoleRepository` and `UserRepository`. The RedisTemplate has been autowired. It handles transactions and is also responsible
 for persisting, removing and finding entities that are mapped to the database.

```java
@Service
@Service
public class ExpenseRepository implements ExpenseService{
    
    @Autowired
    RedisTemplate<String, Cacheable> redisTemplate;
    
    private final Logger logger = Logger.getLogger(ExpenseRepository.class);
    
    private static final String EXPENSE = "EXPENSE";
    
    public Long createExpense(String description,ExpenseType expenseType,Date expenseDate,
            Double amount,User user,Attachment attachment){
        Expense expense = new Expense(description,expenseType,expenseDate,amount,
                user,attachment);
        long id = 1;
        List<Expense> expenseList = new ArrayList<Expense>();
        expenseList = getAllExpenses();
        id = expenseList.size();
        expense.setId(id+1);
        redisTemplate.opsForHash().put(EXPENSE, id, expense);
        logger.info("create expense expense id "+(id+1));
        return expense.getId();
        
    }
    
    public Expense getExpense(Long expenseId){
        return (Expense)redisTemplate.opsForHash().get(EXPENSE, expenseId);
    }
    
    public List<Expense> getAllExpenses(){
        Set<Object> keys = redisTemplate.opsForHash().keys(EXPENSE);
        List<Expense> expenseList = new ArrayList<Expense>();
        for(Object key:keys){
            expenseList.add((Expense)redisTemplate.opsForHash().get(EXPENSE, key));
            logger.info("get all expense expense key "+key);
        }
        return expenseList;
    }
    
    public List<Expense> getExpensesByUser(User user){
        Set<Object> keys = redisTemplate.opsForHash().keys(EXPENSE);
        List<Expense> expenseList = new ArrayList<Expense>();
        for(Object key:keys){
            Expense exp = (Expense)redisTemplate.opsForHash().get(EXPENSE, key);
            if(exp!=null && exp.getUser().getUserId().equals(user.getUserId())){
                expenseList.add((Expense)redisTemplate.opsForHash().get(EXPENSE, key));
                logger.info("get expense by user expense key "+key);
            }
        }return expenseList;
    }
    
    public List<Expense> getPendingExpensesList(){
        Set<Object> keys = redisTemplate.opsForHash().keys(EXPENSE);
        List<Expense> expenseList = new ArrayList<Expense>();
        for(Object key:keys){
            Expense exp = (Expense)redisTemplate.opsForHash().get(EXPENSE, key);
            if(exp!=null && !exp.getState().equals(State.APPROVED)
                    && !exp.getState().equals(State.REJECTED)){
                expenseList.add((Expense)redisTemplate.opsForHash().get(EXPENSE, key));
            }
        }return expenseList;
    }
    
    public List<Expense> getApprovedAndRejectedExpensesList(){
        Set<Object> keys = redisTemplate.opsForHash().keys(EXPENSE);
        List<Expense> expenseList = new ArrayList<Expense>();
        for(Object key:keys){
            Expense exp = (Expense)redisTemplate.opsForHash().get(EXPENSE, key);
            if(exp!=null && (exp.getState().equals(State.APPROVED)
                    || exp.getState().equals(State.REJECTED))){
                expenseList.add((Expense)redisTemplate.opsForHash().get(EXPENSE, key));
            }
        }return expenseList;
    }
    
    public Expense changeExpenseStatus(Long expenseId,String state){
        Expense expense = getExpense(expenseId);
        expense.setState(State.valueOf(state));
        redisTemplate.opsForHash().put(EXPENSE, expenseId, expense);
        return expense;
        
    }
    
    public void deleteExpense(Long expenseId){
        Expense expense = getExpense(expenseId);
        redisTemplate.opsForHash().delete(EXPENSE, expense);
    }
    
    public void updateExpense(Long expenseId,String description,Double amount,ExpenseType expenseType){
        Expense expense = getExpense(expenseId);
        expense.setAmount(amount);
        expense.setDescription(description);
        expense.setExpenseType(expenseType);
        redisTemplate.opsForHash().put(EXPENSE, expenseId, expense);
    }
}
```
You can get the Redis service implementation classes [here](/frameworks/java/spring/tutorials/springmvc-redis/code/services-implementation.html)

Once you have added service implementation classes, ensure you aren't getting errors.


## **Step 3:** Initialize database with default values
* Create a new class called `InitService` in `com.springsource.html5expense.redis.services`, and add a init method with the `@PostConstruct` annotation
to initialize default values in database. The method annotated with `PostConstruct` annotation is used on a method that needs to be executed after dependency injection is done to perform initialization.

```java
@Service
@Transactional
public class InitService {

    @Autowired
    RoleService roleService;
    
    @Autowired
    UserService userService;
    
    @Autowired
    ExpenseTypeService expenseTypeService;
    
    @PostConstruct
    public void init(){
        
        Role role = new Role();
        role.setRoleName("ROLE_USER");
        role.setRoleId(new Long(1));
        roleService.save(role);
        
        Role adminRole = new Role();
        adminRole.setRoleName("ROLE_MANAGER");
        adminRole.setRoleId(new Long(2));
        roleService.save(adminRole);
        
        User user = new User();
        user.setUserId(new Long(1));
        user.setEmailId("admin@expense.com");
        user.setEnabled(true);
        user.setRole(adminRole);
        user.setUserName("admin");
        user.setPassword("admin");
        userService.save(user);

        ExpenseType expenseType = new ExpenseType();
        expenseType.setId(new Long(1));
        expenseType.setName("GYM");
        expenseTypeService.save(expenseType);
    }
}
```

## **Step 4:** Modify Spring Security
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

6. Go to your PostgreSQL client and give the command **keys * **. It will list all the keys in the database so you can check that new keys have been created for the Entities.
```bash
redis 127.0.0.1:6379$ keys *
1) "\xac\xed\x00\x05t\x00\nATTACHMENT"
2) "\xac\xed\x00\x05t\x00\x04USER"
3) "mykey"
4) "\xac\xed\x00\x05t\x00\x0bEXPENSETYPE"
5) "username"
6) "\xac\xed\x00\x05t\x00\x04ROLE"
7) "\xac\xed\x00\x05t\x00\aEXPENSE"
```

## Deploying to Cloud Foundry
* To learn how to deploy a Spring App using PostgreSQL to Cloud Foundry, please refer [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/springmvc-app-with-postgresql-deployment-to-cloudfoundry.html).

<a class="button-plain" style="height: 15px; width: 30px" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/springmvc-template-project.html">Prev</a> <span style="float: right;"><a class="button-plain" style="height: 15px; width: 30px" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/expensereport-app-with-spring-security.html">Next</a></span>

