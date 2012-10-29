```java
package com.springsource.html5expense.mongodb.services;

import java.util.ArrayList;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;
import com.springsource.html5expense.model.Attachment;
import com.springsource.html5expense.service.AttachmentService;

@Service
public class AttachmentRepository implements AttachmentService{

       @Autowired
       MongoTemplate mongoTemplate;
       
       public static String ATTACHMENT_COLLECTION = "Attachment";

       public void save(Attachment attachment){
              List<Attachment> attachmentList = new ArrayList<Attachment>();
              attachmentList = getAllAttachment();
              long val = 1;
              val = attachmentList.size();
              attachment.setAttachmentId(val+1);
              mongoTemplate.save(attachment,ATTACHMENT_COLLECTION);
       }

       public Attachment getAttachment(Long attachmentId){
              List<Attachment> attachmentList = mongoTemplate.find(new Query(Criteria.where("attachmentId").
                            is(attachmentId)), Attachment.class,ATTACHMENT_COLLECTION);
              return attachmentList!=null && attachmentList.size()>0?attachmentList.get(0):null;
       }


       public void deleteAttachment(Long id){
              Attachment attachment = getAttachment(id);
              mongoTemplate.remove(attachment,ATTACHMENT_COLLECTION);
       }
       
       public List<Attachment> getAttachmentByExpenseId(Long expenseId){
              List<Attachment> attachmentList = mongoTemplate.find(new Query(Criteria.where("expenseId").
                            is(expenseId)), Attachment.class,ATTACHMENT_COLLECTION);
              return attachmentList;
       }
       
       public List<Attachment> getAllAttachment(){
              return mongoTemplate.findAll(Attachment.class, ATTACHMENT_COLLECTION);
       }
}
```
```java
package com.springsource.html5expense.mongodb.services;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;

import com.springsource.html5expense.model.Attachment;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.ExpenseType;
import com.springsource.html5expense.model.State;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.service.ExpenseService;

@Service
public class ExpenseRepository implements ExpenseService {
       
       @Autowired
       MongoTemplate mongoTemplate;
       
       private final Logger logger = LoggerFactory.getLogger(getClass());
       
       public static String EXPENSE_COLLECTION_NAME = "Expense";
       
       public Long createExpense(String description,ExpenseType expenseType,Date expenseDate,
                     Double amount,User user,Attachment attachment) {
              Expense expense = new Expense(description,expenseType,expenseDate,amount,user,attachment);
              List<Expense> expenseList = new ArrayList<Expense>();
              long val = 1;
              expenseList = getAllExpenses();
              val = expenseList.size();
              expense.setExpenseId(val+1);
              mongoTemplate.save(expense,EXPENSE_COLLECTION_NAME);
              logger.warn("create new expense :"+expense.getExpenseId());
              return expense.getExpenseId();
       }
       
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
              List<Expense> expenseList = mongoTemplate.find(new Query(Criteria.where("expenseId").is(expenseId)),Expense.class,EXPENSE_COLLECTION_NAME);
              return expenseList!=null && expenseList.size()>0?expenseList.get(0):null;
       }
       
       public Expense changeExpenseStatus(Long expenseId,String state){
              Update update = new Update();
              update.set("state", State.valueOf(state));
              Expense exp = getExpense(expenseId);
              logger.warn("Get Expense "+exp.getDescription());
              mongoTemplate.updateFirst(new Query(Criteria.where("_id").is(expenseId)), update,EXPENSE_COLLECTION_NAME);
              return getExpense(expenseId);
       }
       
       
       
       public void deleteExpense(Long expenseId){
              Expense expense = getExpense(expenseId);
              mongoTemplate.remove(expense);
       }
       
       public void updateExpense(Long expenseId,String description,Double amount,ExpenseType expenseType){
              Update update = new Update();
              update.set("description", description);
              update.set("amount",amount);
              update.set("expenseType",expenseType);
              mongoTemplate.updateFirst(new Query(Criteria.where("_id").is(expenseId)), update,EXPENSE_COLLECTION_NAME);
              
       }

}

```
```java
package com.springsource.html5expense.mongodb.services;

import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;

import com.springsource.html5expense.model.ExpenseType;
import com.springsource.html5expense.service.ExpenseTypeService;

@Service
public class ExpenseTypeRepository implements ExpenseTypeService {


       @Autowired
       MongoTemplate mongoTemplate;
       
       private final Logger logger = LoggerFactory.getLogger(getClass());
       
       public static final String EXPENSE_TYPE_COLLECTION = "ExpenseType";

       public List<ExpenseType> getAllExpenseType(){
              return mongoTemplate.findAll(ExpenseType.class,EXPENSE_TYPE_COLLECTION);
       }

       public ExpenseType getExpenseTypeById(Long id){
              List<ExpenseType> expenseTypeList = mongoTemplate.find(
                            new Query(Criteria.where("expenseTypeId").is(id)), 
                            ExpenseType.class,EXPENSE_TYPE_COLLECTION);
              return expenseTypeList!=null && expenseTypeList.size()>0?expenseTypeList.get(0):null;
       }

}

```
```java
package com.springsource.html5expense.mongodb.services;

import javax.annotation.PostConstruct;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Service;

import com.mongodb.BasicDBObject;
import com.mongodb.DBCollection;

@Service
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
```java
package com.springsource.html5expense.mongodb.services;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;

import com.springsource.html5expense.model.Role;
import com.springsource.html5expense.service.RoleService;

@Service
public class RoleRepository implements RoleService {

       @Autowired
       MongoTemplate mongoTemplate;

       public static final String ROLE_COLLECTION = "Role";

       public Role getRole(Long id){
              return mongoTemplate.findById(id, Role.class,ROLE_COLLECTION);
       }

       public Role getRoleByName(String name){
              List<Role> roleList = mongoTemplate.find(new Query(Criteria.where("roleName").
                            is(name)), Role.class,ROLE_COLLECTION);
              return roleList!=null && roleList.size()>0?roleList.get(0):null;
       }

}

```
```java
package com.springsource.html5expense.mongodb.services;

import java.util.ArrayList;
import java.util.List;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;
import com.springsource.html5expense.model.Role;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.service.UserService;

@Service
public class UserRepository implements UserService {

       @Autowired
       MongoTemplate mongoTemplate;
       
       private final Logger logger = LoggerFactory.getLogger(getClass());
       
       public static final String USER_COLLECTION = "User";

       public User getUserById(Long id){
              return mongoTemplate.findById(id, User.class);
       }

       public User getUserByUserName(String userName){
              List<User> roleList = mongoTemplate.find(new Query(Criteria.where("userName").
                            is(userName)), User.class,USER_COLLECTION);
              return roleList!=null && roleList.size()>0?roleList.get(0):null;
       }

       public User createUser(String userName,String password,String mailId,Role role){
              User user = new User(userName,password,mailId);
              user.setRole(role);
              user.setEnabled(true);
              List<User> usersList = new ArrayList<User>();
              long val = 1;
              usersList = getAllUsers();
              val = usersList.size();
              user.setUserId(val+1);
              mongoTemplate.save(user,USER_COLLECTION);
              return user;
       }
       
       public List<User> getAllUsers(){
              return mongoTemplate.findAll(User.class, USER_COLLECTION);
       }
}

```

