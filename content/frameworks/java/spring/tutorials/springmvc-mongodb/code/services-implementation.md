---
title: MogoDB service implementation classes for Expense Reporting App using MongoDB
description: MongoDB service implementation classes for Expense Reporting App using MongoDB
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
        logger.warn("create new expense :" + expense.getExpenseId());
        return expense.getExpenseId();
    }

    @Override
    public List<Expense> getAllExpenses() {
        return mongoTemplate.findAll(Expense.class, EXPENSE_COLLECTION_NAME);
    }

    @Override
    public List<Expense> getPendingExpensesList() {
        List<State> stateList = new ArrayList<State>();
        stateList.add(State.NEW);
        stateList.add(State.OPEN);
        stateList.add(State.IN_REVIEW);
        return mongoTemplate.find(new Query(Criteria.where("state").in(stateList)),
                Expense.class, EXPENSE_COLLECTION_NAME);
    }

    @Override
    public Expense getExpense(Long expenseId) {
        List<Expense> expenseList = mongoTemplate.find(
                new Query(Criteria.where("expenseId").is(expenseId)), Expense.class);
        return expenseList != null && expenseList.size() > 0 ? expenseList.get(0) : null;
    }

    @Override
    public Expense changeExpenseStatus(Long expenseId, String state) {
        Update update = new Update();
        update.set("state", State.valueOf(state));
        Expense exp = getExpense(expenseId);
        logger.warn("Get Expense " + exp.getDescription());
        mongoTemplate.updateFirst(new Query(Criteria.where("_id").is(expenseId)), update,
                EXPENSE_COLLECTION_NAME);
        return getExpense(expenseId);
    }

    @Override
    public void deleteExpense(Long expenseId) {
        Expense expense = getExpense(expenseId);
        mongoTemplate.remove(expense);
    }

    @Override
    public void updateExpense(Long expenseId, Update update) {
        mongoTemplate.updateFirst(new Query(Criteria.where("_id").is(expenseId)), update,
                EXPENSE_COLLECTION_NAME);
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

    @Override
    public List<ExpenseType> getAllExpenseType() {
        logger.warn("getAllExpenseType");
        return mongoTemplate.findAll(ExpenseType.class, EXPENSE_TYPE_COLLECTION);
    }

    @Override
    public ExpenseType getExpenseTypeById(Long id) {
        List<ExpenseType> expenseTypeList = mongoTemplate.find(
                new Query(Criteria.where("expenseTypeId").is(id)), ExpenseType.class,
                EXPENSE_TYPE_COLLECTION);
        logger.warn("getExpenseTypeById");
        return expenseTypeList != null && expenseTypeList.size() > 0 ? expenseTypeList.get(0)
                : null;
    }

}
```
