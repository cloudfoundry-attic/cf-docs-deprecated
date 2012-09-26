```java
package com.springsource.html5expense.model;

import java.io.Serializable;
import java.util.Date;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.Table;
import org.hibernate.annotations.Cascade;
import org.hibernate.annotations.CascadeType;

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
    private User user;

    @OneToOne
    private ExpenseType expenseType;

    @OneToOne
    private Attachment attachment;

    public Expense(){

    }

    public Expense(String description,ExpenseType expenseType,Date expenseDate,
            Double amount,User user,Attachment attachment){
        this.description = description;
        this.expenseType = expenseType;
        this.expenseDate = expenseDate;
        this.amount = amount;
        this.user = user;
        this.attachment = attachment;
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

    public Attachment getAttachment() {
        return attachment;
    }

    public void setAttachment(Attachment attachment) {
        this.attachment = attachment;
    }
}

```

```java
package com.springsource.html5expense.model;

import java.io.Serializable;
import java.sql.Blob;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Lob;

import org.hibernate.annotations.Type;

@Entity
public class Attachment implements Serializable{
    


    @Id
    @GeneratedValue
    private Long id;
    
    private String fileName;
    
    @Type(type="org.hibernate.type.BinaryType") 
    private byte[] content;
    
    private String contentType;
    
    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName;
    }

    public String getContentType() {
        return contentType;
    }

    public void setContentType(String contentType) {
        this.contentType = contentType;
    }
    
    
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getFileNmae() {
        return fileName;
    }

    public void setFileNmae(String fileNmae) {
        this.fileName = fileNmae;
    }

    public byte[] getContent() {
        return content;
    }

    public void setContent(byte[] content) {
        this.content = content;
    }

    public Attachment(){
        
    }
    public Attachment(String fileName,String contentType,byte[] content){
        this.content = content;
        this.contentType = contentType;
        this.fileName = fileName;
    }

}

```
```java
package com.springsource.html5expense.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToOne;

@Entity
public class ExpenseReport {
    
    @Id
    @GeneratedValue
    private Long expenseReportId;
    
    
    @OneToOne
    private Expense expense;
    
    public  ExpenseReport(){
        
    }
    
    public Long getExpenseReportId() {
        return expenseReportId;
    }

    public void setExpenseReportId(Long expenseReportId) {
        this.expenseReportId = expenseReportId;
    }


    public Expense getExpense() {
        return expense;
    }

    public void setExpense(Expense expense) {
        this.expense = expense;
    }

    public  ExpenseReport(Expense expense){
        this.expense = expense;
    }

}

```

```java
package com.springsource.html5expense.model;

import java.io.Serializable;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class ExpenseType implements Serializable {

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

import java.io.Serializable;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Role implements Serializable {

    @Id
    @GeneratedValue
    private Long roleId;
    
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

    private String roleName;
}


```
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

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.Table;

@Entity
@Table(name="USERS")
public class User implements Serializable{

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

    public User(){

    }
    public User(String userName,String password,String mailId){
        this.userName = userName;
        this.password = password;
        this.emailId = mailId;
    }
}
```
