---
title: Entity classes for Expense Reporting App using MongoDB
description: Entity classes for Expense Reporting App using MongoDB
---
```java
package com.springsource.html5expense.model;

public enum State {

    NEW,

    OPEN,

    IN_REVIEW,

    REJECTED,

    APPROVED
}

```

```java
package com.springsource.html5expense.model;

import java.io.Serializable;
import java.util.Date;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "Expense")
public class Expense implements Serializable {

    @Id
    private Long expenseId;

    private String description;

    private Double amount;

    private State state = State.NEW;

    private Date expenseDate;

    private ExpenseType expenseType;

    public Expense() {

    }

    public Expense(String description, ExpenseType expenseType, Date expenseDate, Double amount) {
        this.description = description;
        this.expenseType = expenseType;
        this.expenseDate = expenseDate;
        this.amount = amount;
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

    public void setExpenseId(Long id) {
        this.expenseId = id;
    }

    public ExpenseType getExpenseType() {
        return expenseType;
    }

    public void setExpenseType(ExpenseType expenseType) {
        this.expenseType = expenseType;
    }

    public Double getAmount() {
        return amount;
    }

    public void setAmount(Double amount) {
        this.amount = amount;
    }

    public Date getExpenseDate() {
        return expenseDate;
    }

    public void setExpenseDate(Date expenseDate) {
        this.expenseDate = expenseDate;
    }

}

```

```java
package com.springsource.html5expense.model;

import java.io.Serializable;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "ExpenseType")
public class ExpenseType implements Serializable {

    @Id
    private Long expenseTypeId;

    private String name;

    public Long getExpenseTypeId() {
        return expenseTypeId;
    }

    public void setExpenseTypeId(Long id) {
        this.expenseTypeId = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

}
```
