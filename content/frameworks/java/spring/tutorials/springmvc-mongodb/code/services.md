---
title: Spring MVC ExpenseReport App Services Code
description: Spring MVC ExpenseReport App Services Code
tags:
    - spring
    - code
    - java
    - services
---

```java
package com.springsource.html5expense.services;

import java.util.List;
import org.springframework.data.mongodb.core.query.Update;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.User;

public interface ExpenseService {
    Long save(Expense expense);

    Expense getExpense(Long expenseId);

    List<Expense> getAllExpenses();

    List<Expense> getExpensesByUser(User user);

    List<Expense> getPendingExpensesList();

    Expense changeExpenseStatus(Long expenseId, String state);

    void deleteExpense(Long expenseId);

    void updateExpense(Long expenseId, Update update);
}
```
```java
package com.springsource.html5expense.services;

import java.util.List;
import com.springsource.html5expense.model.ExpenseType;

public interface ExpenseTypeService {
    List<ExpenseType> getAllExpenseType();

    ExpenseType getExpenseTypeById(Long id);
}
```
```java
package com.springsource.html5expense.services;

import com.springsource.html5expense.model.Role;
public interface RoleService {
    Role getRole(Long id);

    Role getRoleByName(String name);
}
```
```java
package com.springsource.html5expense.services;

import com.springsource.html5expense.model.User;

public interface UserService {
    User getUserById(Long id);

    User getUserByUserName(String userName);

    User save(User user);
}
```
