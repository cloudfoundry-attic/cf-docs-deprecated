---
title: Spring MVC ExpenseReport App ServiceImplementation Code
description: Spring MVC ExpenseReport App ServiceImplementation Code
tags:
    - spring
    - code
    - java
    - services
---

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
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.State;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.services.ExpenseService;

@Service
public class ExpenseRepository implements ExpenseService {
    
    @Autowired
    MongoTemplate mongoTemplate;
    
    private final Logger logger = LoggerFactory.getLogger(getClass());
    
    public static String EXPENSE_COLLECTION_NAME = "Expense";

    @Override
    public Long save(Expense expense) {
        List<Expense> expenseList = new ArrayList<Expense>();
        long val = 1;
        expenseList = getAllExpenses();
        val = expenseList.size();
        expense.setExpenseId(val + 1);
        mongoTemplate.save(expense, EXPENSE_COLLECTION_NAME);
        logger.warn("create new expense :"+expense.getExpenseId());
        return expense.getExpenseId();
    }

    @Override
    public List<Expense> getAllExpenses() {
        return mongoTemplate.findAll(Expense.class, EXPENSE_COLLECTION_NAME);
    }

    @Override
    public List<Expense> getExpensesByUser(User user) {
        return mongoTemplate.find(new Query(Criteria.where("user").is(user)),
                Expense.class,EXPENSE_COLLECTION_NAME);
    }

    @Override
    public List<Expense> getPendingExpensesList() {
        List<State> stateList = new ArrayList<State>();
        stateList.add(State.NEW);
        stateList.add(State.OPEN);
        stateList.add(State.IN_REVIEW);
        return mongoTemplate.find(new Query(Criteria.where("state").in(
                stateList)), Expense.class, EXPENSE_COLLECTION_NAME);
    }

    @Override
    public Expense getExpense(Long expenseId) {
        List<Expense> expenseList = mongoTemplate.find(new Query(
                Criteria.where("expenseId").is(expenseId)),Expense.class);
        return expenseList != null && expenseList.size() > 0
                ? expenseList.get(0) : null;
    }
    
    public Expense changeExpenseStatus(Long expenseId,String state) {
        Update update = new Update();
        update.set("state", State.valueOf(state));
        Expense exp = getExpense(expenseId);
        logger.warn("Get Expense "+exp.getDescription());
        mongoTemplate.updateFirst(new Query(Criteria.where("_id").
                is(expenseId)), update, EXPENSE_COLLECTION_NAME);
        return getExpense(expenseId);
    }

    public void deleteExpense(Long expenseId) {
        Expense expense = getExpense(expenseId);
        mongoTemplate.remove(expense);
    }

    public void updateExpense(Long expenseId, Update update) {
        mongoTemplate.updateFirst(new Query(Criteria.where(
                "_id").is(expenseId)), update, EXPENSE_COLLECTION_NAME);
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
import com.springsource.html5expense.services.ExpenseTypeService;

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

import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;
import com.springsource.html5expense.model.Role;
import com.springsource.html5expense.services.RoleService;

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
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.services.UserService;

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

    public User save(User user){
        List<User> usersList = new ArrayList<User>();
        long val = 1;
        usersList = getAllUsers();
        val = usersList.size();
        logger.warn("size of values "+val);
        user.setUserId(val+1);
        mongoTemplate.save(user, USER_COLLECTION);
        return user;
    }
    
    public List<User> getAllUsers(){
        return mongoTemplate.findAll(User.class, USER_COLLECTION);
    }
}
```

