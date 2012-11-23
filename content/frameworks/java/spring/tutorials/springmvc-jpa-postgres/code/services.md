```java
package com.springsource.html5expense.services;

import java.util.Date;
import java.util.List;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.ExpenseType;
import com.springsource.html5expense.model.User;
public interface ExpenseService {
    Long createExpense(String description, ExpenseType expenseType, Date expenseDate,
               Double amount, User user);

    Expense getExpense(Long expenseId);

    List<Expense> getAllExpenses();

    List<Expense> getExpensesByUser(User user);

    List<Expense> getPendingExpensesList();

    List<Expense> getApprovedAndRejectedExpensesList();

    Expense changeExpenseStatus(Long expenseId, String state);

    void deleteExpense(Long expenseId);

    void updateExpense(Expense expense);
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

    User createUser(User user);
}
```
