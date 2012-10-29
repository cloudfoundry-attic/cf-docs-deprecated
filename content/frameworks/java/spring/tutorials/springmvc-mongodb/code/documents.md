```java
package com.springsource.html5expense.model;

import java.io.Serializable;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "Attachment")
public class Attachment implements Serializable{

    @Id
    private Long attachmentId;
    
    private String fileName;
    
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
    
    public Long getAttachmentId() {
        return attachmentId;
    }

    public void setAttachmentId(Long id) {
        this.attachmentId = id;
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

import java.io.Serializable;
import java.util.Date;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "Expense")
public class Expense implements Serializable{

    @Id
    private Long expenseId;

    private String description;
    
    private Double amount;
    
    private State state = State.NEW;
    
    private Date expenseDate;
    
    private User user;

    private ExpenseType expenseType;
    
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

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "ExpenseType")
public class ExpenseType implements Serializable{

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
```java
package com.springsource.html5expense.model;

import java.io.Serializable;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "Role")
public class Role implements Serializable {

    @Id
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

import java.io.Serializable;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "User")
public class User implements Serializable{

    @Id
    private Long userId;

    private String userName;
    
    private String password;
    
    private String emailId;
    
    private boolean enabled;
    
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
