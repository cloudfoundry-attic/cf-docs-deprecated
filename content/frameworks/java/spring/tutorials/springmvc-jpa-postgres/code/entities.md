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

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class ExpenseType {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
package com.springsource.html5expense.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Role {

    @Id
    @GeneratedValue
    private Long roleId;

    private String roleName;

    public Long getRoleId() {
        return roleId;
    }

    public void setRoleId(Long roleId) {
        this.roleId = roleId;
    }

    public String getRoleName() {
        return roleName;
    }

    public void setRoleName(String roleName) {
        this.roleName = roleName;
    }
}

```

```java
package com.springsource.html5expense.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.Table;

@Entity
@Table(name = "USERS")
public class User {

    @Id
    @GeneratedValue
    private Long userId;

    private String userName;

    private String password;

    private String emailId;

    private boolean enabled;

    @OneToOne
    private Role role;

    public Role getRole() {
         return role;
    }

    public void setRole(Role role) {
        this.role = role;
    }

    public boolean isEnabled() {
        return enabled;
    }

    public void setEnabled(boolean enabled) {
       this.enabled = enabled;
    }

    public Long getUserId() {
        return userId;
    }

    public void setUserId(Long userId) {
        this.userId = userId;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getEmailId() {
        return emailId;
    }

    public void setEmailId(String emailId) {
        this.emailId = emailId;
    }

    public User() {

    }

    public User(String userName, String password, String mailId) {
        this.userName = userName;
        this.password = password;
        this.emailId = mailId;
    }
}
```

```java
package com.springsource.html5expense.model;

import java.util.Date;
import javax.persistence.Entity;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.Table;

@Entity
@Table(name = "EXPENSE")
public class Expense {

    @Id
    @GeneratedValue
    private Long id;

    private String description;

    private Double amount;

    @Enumerated(EnumType.STRING)
    private State state = State.NEW;

    private Date expenseDate;
    
    @OneToOne
    private User user;

    @OneToOne
    private ExpenseType expenseType;

    public Expense() {

    }

    public Expense(String description, ExpenseType expenseType, Date expenseDate,
      Double amount, User user) {
        this.description = description;
        this.expenseType = expenseType;
        this.expenseDate = expenseDate;
        this.amount = amount;
        this.user = user;
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

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
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

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```
