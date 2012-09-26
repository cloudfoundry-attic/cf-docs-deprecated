```java
package com.springsource.html5expense.service;
import java.util.List;
import com.springsource.html5expense.model.Attachment;
public interface AttachmentService {

public void save(Attachment attachment);
public Attachment getAttachment(Long attachmentId);
public List<Attachment> getAttachmentByExpenseId(Long expenseId);
public void deleteAttachment(Long id);
}
```
```java
package com.springsource.html5expense.service;
import java.util.List;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.ExpenseReport;
public interface ExpenseReportService {

public Long createExpenseReport(Expense expense);
public ExpenseReport getExpenseReportById(Long expensReportId);
public List getAllExpenseReports();
}
```
```java
package com.springsource.html5expense.service;

import java.util.Date;
import java.util.List;
import com.springsource.html5expense.model.Attachment;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.ExpenseType;
import com.springsource.html5expense.model.User;

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
```java
package com.springsource.html5expense.service;
import java.util.List;
import com.springsource.html5expense.model.ExpenseType;

public interface ExpenseTypeService {
public List<ExpenseType> getAllExpenseType();
public ExpenseType getExpenseTypeById(Long id);
}
```
```java
package com.springsource.html5expense.service;
import com.springsource.html5expense.model.Role;

public interface RoleService {
public Role getRole(Long id);
public Role getRoleByName(String name);
}
```
```java
package com.springsource.html5expense.service;
import com.springsource.html5expense.model.Role;
import com.springsource.html5expense.model.User;

public interface UserService {
public User getUserById(Long id);
public User getUserByUserName(String userName);
public User createUser(String userName,String password,String mailId,Role role);

}
```
