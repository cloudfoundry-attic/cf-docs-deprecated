---
title: Spring
description: Spring Application Development with Cloud Foundry
tags:
    - spring
    - mongodb
    - redis
    - code-attached
---

This is a guide for Java developers using the Spring framework and PostgreSQL database to build and deploy their apps on Cloud Foundry. It shows you how to set up and successfully deploy Spring Java PostgreSQL applications to Cloud Foundry.


Before you get started, you need the following:

+  A [Cloud Foundry account](http://cloudfoundry.com/signup)

+  The [vmc](/tools/vmc/installing-vmc.html) Cloud Foundry command line tool

+  A [Spring Tool Suite™ (STS)](http://www.springsource.org/spring-tool-suite-download) installation

+  A Cloud Foundry plugin for STS.


## Introduction

In this tutorial, we take the following user case.

Create and Deploy an Expense Reporting App on Cloud Foundry.

## Subtopics

+ [Create Spring MVC Template Project](#create-spring-mvc-template-project)
+ [ExpenseReport App](#expensereport-app)
+ [Add Spring Security to ExpenseReport App](#add-spring-security-to-expensereport-app)
+ [Build and Run the App Locally](#build-and-run-the-app-locally)
+ [Deploy the App on Cloud Foundry Using VMC](#deploy-the-app-on-cloud-foundry-using-vmc)
+ [Deploy the App on Cloud Foundry Using STS](#deploy-the-app-on-cloud-foundry-using-sts)



## Create Spring MVC Template Project

This section provides details on how to get started with STS (SpringSource Tool Suite) and Spring MVC.Please open your STS and select dashboard by clicking,
    `Help -> Dashboard`

From the dashboard view select Spring Template Project.Window shown below opens up.
![Spring MVC Template Project](/images/spring_tutorial/spring_template_project_mvc.png)


then choose `Spring MVC Project`.
The first time you do it, STS might ask you to download some extra elements from the Internet. 
![spring_template_project_mvc_download.png](/images/spring_tutorial/spring_template_project_mvc_download.png)

You’ll need to give your project name and top-level package:


Click on `Finish` – and STS will create the project.

Before run the project.Select `Run As -> Maven clean`
![maven_clean.png](/images/spring_tutorial/maven_clean.png)

Once Maven clean completed select` Run As -> Maven install`.It will download dependencies from pom.xml.
![maven_install.png](/images/spring_tutorial/maven_install.png)

Select `Run As -> Maven build`
![maven_build.png](/images/spring_tutorial/maven_build.png)

And give Maven goal as , `tomcat:run`
![maven_run.png](/images/spring_tutorial/maven_run.png)


Once server starts completed,STS will open the home page for the project which displays Hello world message with the current server time.
![Hello World Output](/images/spring_tutorial/hello_world.png)


## ExpenseReport App

ExpenseReport is used to create and manage expenses.

In our expensereport application,we are using two roles.

1.   Normal User - can create expense,view status of expense,edit expense,delete expense.

2.   Admin User -  approve expense,reject expense which are created by normal user.

     Once Normal User submits his expense it will go to admin work items.Admin can approve or reject the expense.

![spring-expensereport-usecase.png](/images/spring_tutorial/usecase_diagram.png)

![spring-expensereport-usecase_admin.png](/images/spring_tutorial/usecase_diagram_admin.png)

These expenses are created by the users and the data
is stored in underlying PostgreSQL database.

The class diagram for our application is :

![spring-expensereport-class_diagram.png](/images/spring_tutorial/class_diagram.png)

Before create Entities,replace your pom.xml with the below one,
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.springsource</groupId>
	<artifactId>html5expense</artifactId>
	<name>SpringMVC-ExpenseReport</name>
	<packaging>war</packaging>
	<version>1.0.0-BUILD-SNAPSHOT</version>
	<properties>
		<java-version>1.6</java-version>
		<org.springframework-version>3.1.0.RELEASE</org.springframework-version>
		<org.aspectj-version>1.6.9</org.aspectj-version>
		<org.slf4j-version>1.5.10</org.slf4j-version>
		<org.spring-data-mongo-version>1.0.0.RELEASE</org.spring-data-mongo-version>
	</properties>
	<dependencies>
		<!-- Spring -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${org.springframework-version}</version>
			<exclusions>
				<!-- Exclude Commons Logging in favor of SLF4j -->
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				 </exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		
		 <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${org.springframework-version}</version>
        </dependency>
		
		 <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.8.1</version>
        </dependency>
		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		
		<!--Display Tag -->
		<dependency>
      		<groupId>displaytag</groupId>
      		<artifactId>displaytag</artifactId>
      		<version>1.1</version>
    	</dependency>
    	
    	<!-- Spring-Batch -->
    	<dependency>
            <groupId>org.springframework.batch</groupId>
            <artifactId>spring-batch-core</artifactId>
            <version>2.1.6.RELEASE</version>
        </dependency>
				
		<!-- AspectJ -->
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjrt</artifactId>
			<version>${org.aspectj-version}</version>
		</dependency>	
		
		<!-- Logging -->
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${org.slf4j-version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>jcl-over-slf4j</artifactId>
			<version>${org.slf4j-version}</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${org.slf4j-version}</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.15</version>
			<exclusions>
				<exclusion>
					<groupId>javax.mail</groupId>
					<artifactId>mail</artifactId>
				</exclusion>
				<exclusion>
					<groupId>javax.jms</groupId>
					<artifactId>jms</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jdmk</groupId>
					<artifactId>jmxtools</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jmx</groupId>
					<artifactId>jmxri</artifactId>
				</exclusion>
			</exclusions>
			<scope>runtime</scope>
		</dependency>

		<!-- @Inject -->
		<dependency>
			<groupId>javax.inject</groupId>
			<artifactId>javax.inject</artifactId>
			<version>1</version>
		</dependency>
				
		<!-- Servlet  -->
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.5</version>
			<scope>provided</scope>
		</dependency>
		<!-- <dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>javax.servlet-api</artifactId>
	<version>3.0.1</version>
</dependency>-->
            
		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>jsp-api</artifactId>
			<version>2.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
		
		
		<!-- CGLIB -->
		<dependency>
			<groupId>cglib</groupId>
			<artifactId>cglib</artifactId>
			<version>2.2.2</version>
		</dependency>
            
		
		<!-- POSTGRE SQL -->
		
		<dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>4.0.0.Final</version>
        </dependency>
		<dependency>
			<groupId>postgresql</groupId>
			<artifactId>postgresql</artifactId>
			<version>9.1-901.jdbc4</version>
		</dependency>
		
		<dependency>
      <groupId>commons-dbcp</groupId>
      <artifactId>commons-dbcp</artifactId>
      <version>20030825.184428</version>
    </dependency>
    <dependency>
      <groupId>commons-pool</groupId>
      <artifactId>commons-pool</artifactId>
      <version>1.5.4</version>
    </dependency>
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.2.1</version>
    </dependency>
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>1.3</version>
    </dependency>
	
		<!-- Test -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.7</version>
			<scope>test</scope>
		</dependency>        
		
				
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		
		<!-- Spring Security   -->
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-core</artifactId>
			<version>3.0.5.RELEASE</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-web</artifactId>
			<version>3.0.5.RELEASE</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-config</artifactId>
			<version>3.0.5.RELEASE</version>
		</dependency>
		
		<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-taglibs</artifactId>
	<version>3.0.5.RELEASE</version> 
</dependency>

<dependency>
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>1.1.2</version>
    </dependency>     
		
	</dependencies>
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-eclipse-plugin</artifactId>
                <version>2.9</version>
                <configuration>
                    <additionalProjectnatures>
                        <projectnature>org.springframework.ide.eclipse.core.springnature</projectnature>
                    </additionalProjectnatures>
                    <additionalBuildcommands>
                        <buildcommand>org.springframework.ide.eclipse.core.springbuilder</buildcommand>
                    </additionalBuildcommands>
                    <downloadSources>true</downloadSources>
                    <downloadJavadocs>true</downloadJavadocs>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <compilerArgument>-Xlint:all</compilerArgument>
                    <showWarnings>true</showWarnings>
                    <showDeprecation>true</showDeprecation>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.2.1</version>
                <configuration>
                    <mainClass>org.test.int1.Main</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

##Add Entities

   For our application we need to add Expense,User,Role,Attachment,ExpenseReport,State classes.These classes are POJO with getters and setters for its properties and annotate your business class with @Entity. By default JPA will take class name as table name.if you want to store in different table name then provide @Table annotation with name property. Create a package for entities as com.springsource.html5expense.model and add all entities.

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
	@Cascade(CascadeType.DELETE_ORPHAN)
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
The @Id annotation specifies the primary key of the entity.

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

##Create controller classes

   Controller is reponsible for mapping the request.Use RequestMapping annotation for mapping web requests onto specific handler methods. Create a package for controllers as com.springsource.html5expense.controller and add the following ExpenseController,FileAttachmentController classes.

```java
package com.springsource.html5expense.controller;

import java.security.Principal;
import java.sql.Blob;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

import org.hibernate.Hibernate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;

import com.springsource.html5expense.model.Attachment;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.ExpenseType;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.service.AttachmentService;
import com.springsource.html5expense.service.ExpenseService;
import com.springsource.html5expense.service.ExpenseTypeService;
import com.springsource.html5expense.service.RoleService;
import com.springsource.html5expense.service.UserService;

@Controller
public class ExpenseController {


	@Autowired
	ExpenseService expenseService;

	@Autowired
	AttachmentService attachmentService;

	@Autowired
	ExpenseTypeService expenseTypeService;

	@Autowired 
	UserService userService;

	@Autowired
	RoleService roleService;

	public ExpenseService getExpenseService() {
		return expenseService;
	}

	public void setExpenseService(ExpenseService expenseService) {
		this.expenseService = expenseService;
	}

	public AttachmentService getAttachmentService() {
		return attachmentService;
	}

	public void setAttachmentService(AttachmentService attachmentService) {
		this.attachmentService = attachmentService;
	}

	public ExpenseTypeService getExpenseTypeService() {
		return expenseTypeService;
	}

	public void setExpenseTypeService(ExpenseTypeService expenseTypeService) {
		this.expenseTypeService = expenseTypeService;
	}

	public UserService getUserService() {
		return userService;
	}

	public void setUserService(UserService userService) {
		this.userService = userService;
	}

	public RoleService getRoleService() {
		return roleService;
	}

	public void setRoleService(RoleService roleService) {
		this.roleService = roleService;
	}


	@RequestMapping(value="/loadApprovalExpenses" ,method = RequestMethod.GET)
	public String loadApprovedExpenses(ModelMap model){
		List<Expense> approvedExpenseList = getExpenseService().getPendingExpensesList();
		model.addAttribute("approvedExpenseList", approvedExpenseList);
		return "expenseapproval";

	}

	@RequestMapping(value="/deleteExpense",method=RequestMethod.GET)
	public String deleteExpense(HttpServletRequest request){
		String expenseId = request.getParameter("expenseId");
		Expense expense = getExpenseService().getExpense(new Long(expenseId));
		getExpenseService().deleteExpense(new Long(expenseId));
		return "redirect:/";
	}

	@RequestMapping(value="/editExpense",method=RequestMethod.GET)
	public String editExpense(HttpServletRequest request){
		String expenseId = request.getParameter("expenseId");
		Expense expense = getExpenseService().getExpense(new Long(expenseId));
		request.setAttribute("expense",expense);
		List<ExpenseType> expenseTypeList = expenseTypeService.getAllExpenseType();
		request.setAttribute("expenseTypeList", expenseTypeList);
		request.setAttribute("isEdit", "true");
		return "newexpense";
	}

	@RequestMapping(value="/changeState",method=RequestMethod.POST)
	public String changeState(HttpServletRequest request){
		String expenseId = request.getParameter("expenseId");
		String action = request.getParameter("action");
		getExpenseService().changeExpenseStatus(new Long(expenseId), action);
		List<Expense> approvedExpenseList = getExpenseService().getPendingExpensesList();
		request.setAttribute("approvedExpenseList", approvedExpenseList);
		return "expenseapproval";
	}


	@RequestMapping(value="/updateExpense",method=RequestMethod.POST)
	public String updateExpense(HttpServletRequest request){
		String expenseId = request.getParameter("expenseId");
		Expense expense = getExpenseService().getExpense(new Long(expenseId));
		String description = request.getParameter("description");
		String amount = request.getParameter("amount");
		String expenseTypeVal = request.getParameter("expenseTypeId");
		System.out.println("expense Id "+expenseTypeVal);
		ExpenseType expenseType = getExpenseTypeService().getExpenseTypeById(new Long(expenseTypeVal));
		expense.setExpenseType(expenseType);
		getExpenseService().updateExpense(new Long(expenseId), description, new Double(amount),expenseType);
		User user = (User)request.getSession().getAttribute("user");
		List<Expense> pendingExpenseList = getExpenseService().getExpensesByUser(user);
		request.setAttribute("pendingExpenseList",pendingExpenseList);
		return "myexpense";
	}

	@RequestMapping(value="/createNewExpenseReport",method = RequestMethod.POST)
	public String createNewExpenseReport(@RequestParam("file") MultipartFile file,HttpServletRequest request){
		String description = request.getParameter("description");
		String expenseTypeVal =request.getParameter("expenseTypeId");
		ExpenseType expenseType = expenseTypeService.getExpenseTypeById(new Long(expenseTypeVal));
		String amount = request.getParameter("amount");
		Date expenseDate = new Date();
		Double amountVal = new Double(amount);
		User user = (User)request.getSession().getAttribute("user");
		String fileName = "";
		String contentType = "";
		if(file!=null){
			try{
                            fileName =file.getOriginalFilename();
                            contentType = file.getContentType();
                            Attachment attachment = new Attachment(fileName, contentType,  file.getBytes());
                            attachmentService.save(attachment);
                            Long id = expenseService.createExpense(description,expenseType,expenseDate,
    				amountVal,user,attachment);
			}catch(Exception e){

			}
		}
		return "redirect:/";
	}

	@RequestMapping(value="/createNewExpense")
	public String createNewExpense(ModelMap model){

		List<ExpenseType> expenseTypeList = expenseTypeService.getAllExpenseType();
		model.addAttribute("expenseTypeList", expenseTypeList);
		return "newexpense";
	}

}
```
```java
package com.springsource.html5expense.controller;

import java.io.IOException;
import java.io.OutputStream;
import java.sql.Blob;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.hibernate.Hibernate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.util.FileCopyUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;

import com.springsource.html5expense.model.Attachment;
import com.springsource.html5expense.service.AttachmentService;
import com.springsource.html5expense.service.ExpenseService;



@Controller
public class FileAttachmentController {

	    @Autowired
	    private ExpenseService expenseService;

	    @Autowired
	    private AttachmentService attachmentService;

	    @RequestMapping(value = "/save", method = RequestMethod.POST)
	    public String save(@RequestParam("file") MultipartFile file) {
	        try {

	            String fileName =file.getOriginalFilename();
	            String contentType = file.getContentType();
	            Attachment attachment = new Attachment(fileName, contentType, file.getBytes());
	            attachmentService.save(attachment);
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	        return "redirect:/index.html";
	    }

	    @RequestMapping(value="/download")
	    public void download(HttpServletRequest request,HttpServletResponse response) throws Exception{
	    	String attachmentId = request.getParameter("attachmentId");
	    	         Attachment  attachment = this.attachmentService.getAttachment(new Long(attachmentId));
	    	         response.setContentType(attachment.getContentType());
	    	         response.setHeader("Content-Disposition","attachment; filename=\"" + attachment.getFileName() +"\"");
	    	         response.setHeader("cache-control", "must-revalidate");
	    	         OutputStream out = response.getOutputStream();
	    	         out.write(attachment.getContent());
	    	         out.flush();

	    }
}
```

## Add Service Interfaces

   Create a package for service interfaces as com.springsource.html5expense.service and add AttachmentService,ExpenseReportService,ExpenseService,ExpenseTypeService,UserService interfaces which define the business methods.

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

## Add Service Implementation

      Create a package com.springsource.html5expense.serviceImpl for service implementation and add the following JpaAttachmentServiceImpl,JpaExpenseReportServiceImpl,JpaExpenseServiceImpl,JpaExpenseTypeServiceImpl,JpaRoleServiceImpl,JpaUserServiceImpl classes. Here we autowired EntityManager bean which is responsible for persisting,removing,finding entities.@PersistenceContext which will use EntityManagerFactory to create EntityManager instance.
```java
package com.springsource.html5expense.serviceImpl;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.Query;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.springsource.html5expense.model.Attachment;
import com.springsource.html5expense.service.AttachmentService;

@Service
public class JpaAttachmentServiceImpl implements AttachmentService{

	private EntityManager entityManager;

	public EntityManager getEntityManager() {
		return entityManager;
	}

	@PersistenceContext
	public void setEntityManager(EntityManager entityManager) {
		this.entityManager = entityManager;
	}

	@Transactional
	public Attachment getAttachment(Long attachmentId){
		Attachment attachment = new Attachment();
		attachment.setId(attachmentId);
		attachment = entityManager.find(Attachment.class,attachmentId);
		return attachment;
	}

	@Transactional
	public void save(Attachment attachment){
		entityManager.persist(attachment);

	}

	@Transactional
	public List<Attachment> getAttachmentByExpenseId(Long expenseId){
		Query query = entityManager.createQuery("select a from Attachment a where a.expense.id =:expenseId");
		query.setParameter("expenseId", expenseId);

		List<Attachment> attachmentList = query.getResultList();
		return attachmentList;
	}

	@Transactional
	public void deleteAttachment(Long id){
		Attachment attchment = getAttachment(new Long(id));
		getEntityManager().remove(attchment);
	}


}
```

```java
package com.springsource.html5expense.serviceImpl;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.Query;

import org.springframework.transaction.annotation.Transactional;

import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.ExpenseReport;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.service.ExpenseReportService;

public class JpaExpenseReportServiceImpl implements ExpenseReportService {

private EntityManager entityManager;

	public EntityManager getEntityManager() {
		return entityManager;
	}

	@PersistenceContext
	public void setEntityManager(EntityManager entityManager) {
		this.entityManager = entityManager;
	}

	@Override
	@Transactional
	public Long createExpenseReport(Expense expense){
		ExpenseReport expenseReport = new ExpenseReport(expense);
		getEntityManager().persist(expenseReport);
		return expenseReport.getExpenseReportId();
	}

	@Override
	@Transactional
	public ExpenseReport getExpenseReportById(Long expenseReportId){
		ExpenseReport expenseReport = new ExpenseReport();
		expenseReport.setExpenseReportId(new Long(expenseReportId));
		expenseReport = getEntityManager().find(ExpenseReport.class, expenseReportId);
		return expenseReport;
	}

	@Override
	@Transactional
	public List getAllExpenseReports(){
		Query query = getEntityManager().createQuery("select e from expensereport e");
		return query.getResultList();
	}
}
```

```java
package com.springsource.html5expense.serviceImpl;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.Query;

import org.hibernate.criterion.DetachedCriteria;
import org.hibernate.criterion.Restrictions;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.springsource.html5expense.model.Attachment;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.ExpenseReport;
import com.springsource.html5expense.model.ExpenseType;
import com.springsource.html5expense.model.State;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.service.ExpenseService;
import com.springsource.html5expense.service.UserService;



@Service
public class JpaExpenseServiceImpl implements ExpenseService {


	private EntityManager entityManager;

	public EntityManager getEntityManager() {
		return entityManager;
	}

	@PersistenceContext
	public void setEntityManager(EntityManager entityManager) {
		this.entityManager = entityManager;
	}


	private UserService userService;


	public UserService getUserService() {
		return userService;
	}

	@Autowired
	public void setUserService(UserService userService) {
		this.userService = userService;
	}

	@Transactional
	public Long createExpense(String description,ExpenseType expenseType,Date expenseDate,
			Double amount,User user,Attachment attachment) {
		// TODO Auto-generated method stub
		Expense expense = new Expense(description,expenseType,expenseDate,amount,user,attachment);
		//ExpenseReport report = new ExpenseReport(expense);
		entityManager.persist(expense);
		//entityManager.persist(report);
		return expense.getId();
	}


	@Transactional
	public Expense getExpense(Long expenseId){
		return entityManager.find(Expense.class, expenseId);
	}


	@Transactional
    public void remove(Integer id) {
         
        
    }

	@Override
	@Transactional
	public List getAllExpenses(){
		Query query = getEntityManager().createQuery("select e from Expense e");
		return query.getResultList();
	}

	@Override
	@Transactional
	public List<Expense> getExpensesByUser(User user){
		Query query = getEntityManager().createQuery("select e from Expense e where e.user.userId =:userId ORDER BY e.expenseDate DESC",Expense.class);
		query.setParameter("userId", user.getUserId());
		return query.getResultList();
	}

	@Override
	@Transactional
	public List<Expense> getPendingExpensesList(){
		List<Enum> state = new ArrayList<Enum>();
		state.add(State.NEW);
		state.add(State.OPEN);
		state.add(State.IN_REVIEW);
		Query query = entityManager.createQuery("select e from Expense e where e.state IN (:state) ORDER BY e.expenseDate DESC",Expense.class);
		query.setParameter("state", state);
		return query.getResultList();
	}

	@Override
	@Transactional
	public List<Expense> getApprovedAndRejectedExpensesList(){
		List<Enum> state = new ArrayList<Enum>();
		state.add(State.REJECTED);
		state.add(State.APPROVED);
		Query query = entityManager.createQuery("select e from Expense e where e.state IN (:state) e.expenseDate DESC",Expense.class);
		query.setParameter("state", state);
		return query.getResultList();
	}

	@Override
	@Transactional
	public Expense changeExpenseStatus(Long expenseId,String state){
		Expense expense = getExpense(expenseId);
		expense.setState(State.valueOf(state));
		getEntityManager().merge(expense);
		return expense;


	}

	@Override
	@Transactional
	public void deleteExpense(Long expenseId){
		Expense expense = getExpense(expenseId);
		expense.setAttachment(null);
		getEntityManager().remove(expense);

	}

	@Override
	@Transactional
	public void updateExpense(Long expenseId,String description,Double amount,ExpenseType expenseType){
		Expense expense = getExpense(expenseId);
		expense.setDescription(description);
		expense.setAmount(amount);
		expense.setExpenseType(expenseType);
		getEntityManager().merge(expense);


	}


}
```

```java
package com.springsource.html5expense.serviceImpl;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.Query;

import org.springframework.stereotype.Service;

import com.springsource.html5expense.model.ExpenseType;
import com.springsource.html5expense.service.ExpenseTypeService;

@Service
public class JpaExpenseTypeServiceImpl implements ExpenseTypeService {

private EntityManager entityManager;

	public EntityManager getEntityManager() {
		return entityManager;
	}

	@PersistenceContext
	public void setEntityManager(EntityManager entityManager) {
		this.entityManager = entityManager;
	}

	public List<ExpenseType> getAllExpenseType(){
		return getEntityManager().createQuery("from ExpenseType",ExpenseType.class).getResultList();
	}

	public ExpenseType getExpenseTypeById(Long id){
		return getEntityManager().find(ExpenseType.class, id);
	}

}
```

```java
package com.springsource.html5expense.serviceImpl;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.Query;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.springsource.html5expense.model.Role;
import com.springsource.html5expense.service.RoleService;

@Service
public class JpaRoleServiceImpl implements RoleService{


	private EntityManager entityManager;

	public EntityManager getEntityManager() {
		return entityManager;
	}

	@PersistenceContext
	public void setEntityManager(EntityManager entityManager) {
		this.entityManager = entityManager;
	}

	@Transactional
	public Role getRole(Long id){
		return getEntityManager().find(Role.class, id);
	}

	@Transactional
	public Role getRoleByName(String name){
			Query query = getEntityManager().createQuery("select r from Role r where roleName =:roleName",Role.class);
			query.setParameter("roleName", name);
			return (query.getResultList()!=null && query.getResultList().size()>0)?(Role)query.getResultList().get(0):null;
	}
}
```

```java
package com.springsource.html5expense.serviceImpl;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.Query;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.springsource.html5expense.model.Role;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.service.RoleService;
import com.springsource.html5expense.service.UserService;

@Service
public class JpaUserServiceImpl implements UserService{


	private EntityManager entityManager;

	public EntityManager getEntityManager() {
		return entityManager;
	}


	@PersistenceContext
	public void setEntityManager(EntityManager entityManager) {
		this.entityManager = entityManager;
	}

	@Override
	public User getUserById(Long userId){
		return getEntityManager().find(User.class, userId);
	}

	@Override
	@Transactional
	public User getUserByUserName(String userName){
		Query query = getEntityManager().createQuery("from User u where u.userName =:username");
		query.setParameter("username",userName);
		return  (query.getResultList()!=null && query.getResultList().size()>0)?(User)(query.getResultList().get(0)):null;
		//return getEntityManager().find(User.class,userName);
	}

	@Override
	@Transactional
	public User createUser(String userName,String password,String mailId,Role role){
		User user = new User(userName,password,mailId);
		user.setEnabled(new Boolean(true));
		user.setRole(role);
		getEntityManager().persist(user);
		return user;

	}

}
```
   
##Configuring ExpenseReport application:

       To make our application ready to deploy,we have to configure our application.For that create a new package com.springsource.html5expense.config add the following ComponentConfig,WebConfig classes.We marked ComponentConfig class with @Configuration annotation to configure our beans.It is a alternative approach to bean definition instead of xml configuration.We can pass this class a argument to spring container as a source for bean creation. 
To declare a bean, simply annotate a method with the @Bean annotation in your config class. You use this method to register a bean definition within an ApplicationContext of the type specified as the method's return value. By default, the bean name will be the same as the method name.

```java
package com.springsource.html5expense.config;

import java.util.HashMap;
import java.util.Map;

import javax.sql.DataSource;

import org.apache.commons.collections.map.HashedMap;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;
import org.springframework.context.annotation.PropertySource;
import org.springframework.jdbc.datasource.SimpleDriverDataSource;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.JpaVendorAdapter;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.Database;
import org.springframework.orm.jpa.vendor.HibernateJpaDialect;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.web.multipart.MultipartResolver;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;

import com.springsource.html5expense.controller.FileAttachmentController;
import com.springsource.html5expense.controller.LoginController;
import com.springsource.html5expense.model.Attachment;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.service.ExpenseService;
import com.springsource.html5expense.serviceImpl.JpaExpenseServiceImpl;
import com.springsource.html5expense.serviceImpl.JpaRoleServiceImpl;

@Configuration
@EnableTransactionManagement
@ImportResource({ "classpath:spring-security.xml"})
public class ComponentConfig {


	@Bean
    public DataSource dataSource()  {
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
        dataSource.setUrl(String.format("jdbc:postgresql://%s:%s/%s", "localhost", "5432", "postgres"));
        dataSource.setDriverClass(org.postgresql.Driver.class);
        dataSource.setUsername("postgres");
        dataSource.setPassword("postgres");
        return dataSource;
    }

	@Bean
	public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
	    LocalContainerEntityManagerFactoryBean emfb = new LocalContainerEntityManagerFactoryBean();
	    emfb.setJpaVendorAdapter( jpaAdapter());
	    emfb.setDataSource(dataSource());
	    emfb.setJpaPropertyMap(createPropertyMap());
	    emfb.setJpaDialect(new HibernateJpaDialect());
	    emfb.setPackagesToScan(new String[]{Expense.class.getPackage().getName()});
	    return emfb;
	}

	public Map<String,String> createPropertyMap()
	{
		Map<String,String> map= new HashedMap();
		map.put(org.hibernate.cfg.Environment.HBM2DDL_AUTO, "create");
		map.put(org.hibernate.cfg.Environment.HBM2DDL_IMPORT_FILES, "import.sql");
		map.put("hibernate.c3p0.min_size", "5");
		map.put("hibernate.c3p0.max_size", "20");
		map.put("hibernate.c3p0.timeout", "360000");
		map.put("hibernate.dialect", "org.hibernate.dialect.PostgreSQLDialect");
		return map;

	}

	@Bean
	public JpaVendorAdapter jpaAdapter() {
	    HibernateJpaVendorAdapter hibernateJpaVendorAdapter = new HibernateJpaVendorAdapter();
	    hibernateJpaVendorAdapter.setShowSql(true);
	    hibernateJpaVendorAdapter.setDatabase(Database.POSTGRESQL);
	    hibernateJpaVendorAdapter.setShowSql(true);
	    hibernateJpaVendorAdapter.setGenerateDdl(true);
	    return hibernateJpaVendorAdapter;
	}


	@Bean
	public PlatformTransactionManager transactionManager() {
	    final JpaTransactionManager transactionManager =
	        new JpaTransactionManager();

	    transactionManager.setEntityManagerFactory(entityManagerFactory().getObject());
	    Map<String,String> jpaProperties = new HashMap<String,String>();
	    jpaProperties.put("transactionTimeout","43200");
	    transactionManager.setJpaPropertyMap(jpaProperties);

	    return transactionManager;
	}

	@Bean
	public MultipartResolver multipartResolver(){
		CommonsMultipartResolver multiPartResolver = new org.springframework.web.multipart.commons.CommonsMultipartResolver();
		multiPartResolver.setMaxUploadSize(10000000);
		return multiPartResolver;
	}

}
```
  @EnableTransactionManagement annotation is responsible for Spring's annotation-driven transaction management capability, similar to the support found in Spring's <tx:*> XML namespace. 

```java
package com.springsource.html5expense.config;

import java.util.List;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.HandlerMethodReturnValueHandler;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

import com.springsource.html5expense.controller.ExpenseController;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.serviceImpl.JpaExpenseServiceImpl;


@Configuration
@Import(ComponentConfig.class)
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter
 {
     @Override
     public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {

     }
    
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Bean
    public MessageSource messageSource() {
        String[] baseNames = "messages,errors".split(",");
        ResourceBundleMessageSource resourceBundleMessageSource = new ResourceBundleMessageSource();
        resourceBundleMessageSource.setBasenames(baseNames);
        return resourceBundleMessageSource;
    }

    @Bean
    public static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Override
    public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> returnValueHandlers) {

    }


}
```


Then modify your web.xml to load this ComponentConfig class as part of your contextConfiglocation.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

        <context-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </context-param>
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.springsource.html5expense.config</param-value>
        </context-param>
	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>  

	<!-- Processes application requests  -->
	<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml ,
			</param-value>
		</init-param>
		<load-on-startup>2</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping> 

	<servlet-mapping>
		<servlet-name>default</servlet-name>
		<url-pattern>/css/*</url-pattern>
	</servlet-mapping>
	<servlet-mapping>
		<servlet-name>default</servlet-name>
		<url-pattern>/images/*</url-pattern>
	</servlet-mapping>
</web-app>
```


##Add Spring Security to ExpenseReport App

This section provides how to get integrate ExpenseReport App with Spring Security.
These are the required jars that we already added into pom.xml,

  + spring-security-core.jar
  + spring-security-web.jar
  + spring-security-config.jar

## Enable Spring Security:

 To enable Spring security we need to add the following steps:

 1.      Add a DelegatingFilterProxy in the web.xml

 2.      Declare a custom XML configuration named spring-security.xml

 In the web.xml we declare an instance of a DelegatingFilterProxy.This will filter the requests based on the declared url-pattern.
```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

 In the spring-security.xml,define HTTP security configuration by adding http tag.And define which url pattern to intecept. Add "jdbc-user service" tag and define the query to get the data from database.

```xml
<beans:beans xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
	http://www.springframework.org/schema/security
	http://www.springframework.org/schema/security/spring-security-3.0.3.xsd">


	<http auto-config="true" use-expressions="true">
		<intercept-url pattern="/login" filters="none" />
		<intercept-url pattern="/logout" filters="none" />
		<intercept-url pattern="/signUp**" filters="none" />
		<intercept-url pattern="/resources/**" filters="none" />
		<intercept-url pattern="/loadApprovalExpenses" access="hasRole('ROLE_MANAGER')"/>
		<intercept-url pattern="/" access="hasAnyRole('ROLE_USER','ROLE_MANAGER')"/>
		<intercept-url pattern="/**" access="isAuthenticated()" />
		<form-login login-page="/login" default-target-url="/"
			authentication-failure-url="/loginfailed" />
		<logout logout-success-url="/logout" />
		<session-management invalid-session-url="/sessionTimeout" >
			<concurrency-control max-sessions="1" />
		</session-management>
	</http>
	<beans:bean id="sas" class="org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy" />

	<authentication-manager>
		<authentication-provider>
			<jdbc-user-service data-source-ref="dataSource"
 
		   users-by-username-query="
		      select username,password,enabled  
		      from users where username=?" 
 
		   authorities-by-username-query="
		      select u.username, r.rolename from users u, role r 
		      where u.role_roleid = r.roleid and u.username =?  " 
 
		/>
		</authentication-provider>
	</authentication-manager>

</beans:beans>

```

 To load this file as part of configuration we have already added @ImportResource({ "classpath:spring-security.xml"}) in ComponentConfig class.

 Create Logincontroller in com.springsource.html5expense.contoller package and add mapping for login,logout,loginfailed.

```java
package com.springsource.html5expense.controller;

import java.security.Principal;
import java.sql.Blob;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

import org.hibernate.Hibernate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;

import com.springsource.html5expense.model.Attachment;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.model.ExpenseType;
import com.springsource.html5expense.model.Role;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.service.AttachmentService;
import com.springsource.html5expense.service.ExpenseService;
import com.springsource.html5expense.service.ExpenseTypeService;
import com.springsource.html5expense.service.RoleService;
import com.springsource.html5expense.service.UserService;

@Controller
public class LoginController {

	@Autowired
	ExpenseService expenseService;

	@Autowired
	AttachmentService attachmentService;

	@Autowired
	ExpenseTypeService expenseTypeService;

	@Autowired 
	UserService userService;

	@Autowired
	RoleService roleService;

	@RequestMapping(value="/", method = RequestMethod.GET)
	public String printWelcome(ModelMap model, Principal principal,HttpServletRequest request ) {
 
		String name = principal.getName();
		model.addAttribute("username", name);
		User user = getUserService().getUserByUserName(name);
		List<Expense> pendingExpenseList = getExpenseService().getExpensesByUser(user);
		model.addAttribute("pendingExpenseList",pendingExpenseList);
		request.getSession().setAttribute("user", user);
		return "myexpense";
 
	}

	@RequestMapping(value="/signUp",method=RequestMethod.GET)
	public String signUp(){
		String name="signUp";
		return name;

	} 

	@RequestMapping(value="/signUpUser",method=RequestMethod.POST)
	public String signUp(ModelMap model,HttpServletRequest request){
		String userName = request.getParameter("userName");
		String password = request.getParameter("password");
		String mailId = request.getParameter("email");

		User user = getUserService().getUserByUserName(userName);
		if(user == null){
			Role role = getRoleService().getRoleByName("ROLE_USER");
			user = getUserService().createUser(userName, password, mailId,role);
			return "redirect:/login";

		}else{
			String errorMsg = "User registration fails already same user name exists please try " +
					"with different user name";
			request.setAttribute("errorMsg",errorMsg);
			return "redirect:/signUp?errorMsg="+errorMsg;
		}
	} 

	@RequestMapping(value="/login", method = RequestMethod.GET)
	public String login(ModelMap model) {
 
		return "login";
 
	}
 
	@RequestMapping(value="/loginfailed", method = RequestMethod.GET)
	public String loginerror(ModelMap model) {
 
		model.addAttribute("error", "true");
		return "login";
 
	}
 
	@RequestMapping(value="/logout", method = RequestMethod.GET)
	public String logout(ModelMap model) {
 
		return "login";
 
	}

	@RequestMapping(value="/sessionTimeout",method = RequestMethod.GET)
	public String sessionTimeout(ModelMap model){
		String errorMsg = "Session has expired .Please login";
		model.addAttribute("errorMsg",errorMsg);
		return "login";
	}


	public ExpenseService getExpenseService() {
		return expenseService;
	}

	public void setExpenseService(ExpenseService expenseService) {
		this.expenseService = expenseService;
	}

	public AttachmentService getAttachmentService() {
		return attachmentService;
	}

	public void setAttachmentService(AttachmentService attachmentService) {
		this.attachmentService = attachmentService;
	}

	public ExpenseTypeService getExpenseTypeService() {
		return expenseTypeService;
	}

	public void setExpenseTypeService(ExpenseTypeService expenseTypeService) {
		this.expenseTypeService = expenseTypeService;
	}

	public UserService getUserService() {
		return userService;
	}

	public void setUserService(UserService userService) {
		this.userService = userService;
	}

	public RoleService getRoleService() {
		return roleService;
	}

	public void setRoleService(RoleService roleService) {
		this.roleService = roleService;
	}
 
}
```

## Add View Files

         Add the following login.jsp,expenseapproval.jsp,myexpense.jsp,newexpense.jsp,usermain.jsp,signup.jsp files in your webapp/WEB-INF/view folders.The login.jsp has two input elements for username and password as j_username and j_password.These are spring's placeholder for the username and password respectively. when the form is submitted ,it will be sent to the following action URL:
j_spring_security_check.

add the below jsp file as `login.jsp`

```jsp
<!DOCTYPE HTML>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
	<head>
		<title>1</title>
		<link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
		<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
	</head>
	<body>
		<div class="blue-a">
			<div class="login-nav">
				
				<ul class="ul-a">
					<li>
						<i class="bg-logo-a"><img src="<c:url value="/resources/images/logo-a.png" /> "></i>
					</li>
					<li>
						<a class="label-1" href="#">Login</a>
						<i class="bg-arrow-right-a"><img class="arrow-right-a-img" src="<c:url value="/resources/images/arrow-right-a.png"/>" ></i>
					</li>
					<li>
						<a class="label-1" href="${pageContext.request.contextPath}/signUp">Register</a>
						
					</li>
				</ul>
			</div>
		</div>
		<div class="main-a">
			<c:if test="${not empty error}">
				<div class="errorblock">
					Your login attempt was not successful, try again.<br /> Caused :
					${sessionScope["SPRING_SECURITY_LAST_EXCEPTION"].message}
				</div>
			</c:if>
			${errorMsg}
			<form name='f' action="<c:url value='j_spring_security_check' />" method='POST'>
				<input class="textbox-a text-email" type="text" placeholder="Username" name="j_username" required="required">
				<input class="textbox-a" type="password" placeholder="Password" name="j_password" required="required">
				<button class="btn-a">Login</button>
			</form>
		</div>
		
		
	</body>
</html>
```
add the below jsp file as `expenseapproval.jsp`

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
 <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
 <%@ taglib uri="http://displaytag.sf.net" prefix="display" %>
 <%@ taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn" %>  
 
<!DOCTYPE HTML>
<html>
	<head>
		<title>4</title>
		<link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
		<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
		<script src="<c:url value="resources/javascript/jquery-1.8.1.js"/>" ></script>
		<script src="<c:url value="resources/javascript/jquery.tablesorter.js"/>" ></script>
		<script src="<c:url value="resources/javascript/jquery.tablesorter.min.js"/>" ></script>
		<script src="<c:url value="resources/javascript/jquery.tablesorter.pager.js"/>" ></script>
		<script type="text/javascript">
		  function changeState(contextPath,expenseId,rowCount){
			  var actionVal =  document.getElementById('action_'+rowCount).value;
			  var url = contextPath+"/changeState?action="+actionVal+"&expenseId="+expenseId;
			  if(actionVal == null || actionVal =="" ){
				  return false;
			  }
			  document.forms[0].action=url;
			  document.forms[0].submit();
			  
		  }
		</script>
	</head>
	<body>
		<div class="blue-b">
			<div class="inside-nav">
				<div class="topper">
					<img class="bg-logo-b" src="<c:url value="/resources/images/logo-b.png" />" />
					<ul class="nav-a">
						<li>
							<a href="#">Hi ${sessionScope.user.userName}</a> 
						</li>
						<li>
							|
						</li>
						<li>
							<a href="<c:url value="/j_spring_security_logout" />">Logout</a>
						</li>
					</ul>
					
				</div>
				<ul class="ul-b">
					<li>
						<a class="label-1" href="${pageContext.request.contextPath}/">My Expenses</a>
						<i class="bg-arrow-right-a"><img class="arrow-right-a-img" src="<c:url value="/resources/images/arrow-right-a.png" />"></i>
					</li>
					<li>
						<a class="label-1" href="${pageContext.request.contextPath}/createNewExpense">New Expenses</a>
						
					</li>
					<c:if test="${sessionScope.user.role.roleName eq 'ROLE_MANAGER'}">
						<li>
							<a class="label-1" href="#">Approvals</a>
						
						</li>
					</c:if>
				</ul>
			</div>
		</div>
		
		<div class="mc-a">
			<button class="btn-b add-expenses-btn"><img class="plus-a-img" src="<c:url value="/resources/images/plus-a.png"/> ">My Expenses
			</button>
			<h2 class="header label-2">
				Approvals Pending ${fn:length(approvedExpenseList)}
			</h2>
			
		<c:if test="${fn:length(approvedExpenseList) gt 0 }">
			<table class="table-a"  class="table-a">
				<thead>
					<tr>
						<td class="td-a">
							<h5 class="label-3">Date<i class="bg-arrow-down-a"><img src="<c:url value="/resources/images/arrow-down-a.png"/>"></i> </h5>
						</td>
						<td  class="td-b">
							<h5 class="label-3">Expense ID </h5>
						</td>
						<td  class="td-c">
							<h5 class="label-3">Description </h5>
						</td>
						<td  class="td-d">
							<h5 class="label-3">Status </h5>
						</td>
					</tr>
				</thead>
				 <tbody>
				<c:forEach var="expense" items="${approvedExpenseList}"  varStatus="rowCounter">
					<input type="hidden" name="rowcount" value="${rowCounter.count}"/>
					<tr class="tr-odd">
						<td>
							<h5 class="label-4">${expense.expenseDate}</h5>
						</td>
						<td>
							<h5 class="label-4">${expense.id}<img class="clip-b-img" src="<c:url value="/resources/images/clip-b.png" />" > </h5>
						</td>
						<td>
							<h5 class="label-4">${expense.description} </h5>
						</td>
						<td>
							<h5 class="label-4"> 
								<c:if test="${expense.state eq 'NEW' }"><form name="actionForm" id="actionForm" method="post">
								<%-- <i class="alt-a label-5 green">
									<img class="right-a-img" src="<c:url value="/resources/images/right-a.png" /> ">
									 Approved</i>
								<i class="alt-b label-6">
									<img class="right-b-img" src="<c:url value="/resources/images/right-b.png" />" > Approved
									<a href="${pageContext.request.contextPath}/deleteExpense?expenseId=${expense.id}" class="bg-cross-b"><img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />"> </a>
								</i> --%>
								
								<select id="action_${rowCounter.count}" name="${rowCounter.count}" style="font-size: 11px;" onchange="changeState('${pageContext.request.contextPath}','${expense.id}','${rowCounter.count}')">
									<option value="PENDING">     Pending    </option>
									<option value="APPROVED">    Approve    </option>
									<option value="REJECTED">    Reject     </option>
								</select>
									</form>
								</c:if>
								
								<c:if test="${expense.state eq 'REJECTED' }">
								 <i class="alt-a label-5 red"><img class="cross-a-img" src="<c:url value="/resources/images/cross-a.png" />" > Rejected</i> 
								<i class="alt-b label-6">
									<img class="right-b-img" src="<c:url value="/resources/images/cross-a.png" />" > Rejected
									<a href="${pageContext.request.contextPath}/deleteExpense?expenseId=${expense.id}" class="bg-cross-b"><img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />" > </a>
								</i>
								</c:if>
								
								
							</h5>
						</td>
					</tr>
					
					</c:forEach>
				</tbody>
				
			</table> 
	
	<div id="pager"  class="pagination pagination-1" >

		<form >
			<img src="<c:url value="/resources/images/first.png"/>"  class="first"/>

			<img src="<c:url value="/resources/images/prev.png"/>"  class="prev"/>

			<input type="text"  class="pagedisplay"/>

			<img src="<c:url value="/resources/images/next1.png" />"  class="next"/>

			<img src="<c:url value="/resources/images/last.png" />"  class="last"/>

			<select  class="pagesize">

				<option selected="selected"  value="10">10</option>

				<option value="20">20</option>

				<option value="30">30</option>

				<option  value="40">40</option>

			</select>

		</form>

	</div>
	
	</c:if>
			
		</div>
		
		
				<script>

    jQuery(document).ready(function() {
    	

    jQuery("#table-a")

    .tablesorter({widthFixed: true, widgets: ['zebra']})
    .tablesorterPager({container: $("#pager")}); 

    });

</script>
	</body>
	
</html>
```
add the 

```jsp
<!DOCTYPE HTML>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
 <%@ taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn" %>  
<html>
	<head>
		<title>2</title>
		<link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
		<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
		<script src="<c:url value="resources/javascript/jquery-1.8.1.js"/>" ></script>
		<script src="<c:url value="resources/javascript/jquery.tablesorter.js"/>" ></script>
		<script src="<c:url value="resources/javascript/jquery.tablesorter.min.js"/>" ></script>
		<script src="<c:url value="resources/javascript/jquery.tablesorter.pager.js"/>" ></script>

		
	</head>
	<body>
		<div class="blue-b">
			<div class="inside-nav">
				<div class="topper">
					<img class="bg-logo-b" src="<c:url value="/resources/images/logo-b.png" />" />
					<ul class="nav-a">
						<li>
							<a href="#">Hi ${sessionScope.user.userName} </a> 
						</li>
						<li>
							|
						</li>
						<li>
							<a href="<c:url value="/j_spring_security_logout" />">Logout</a> 
						</li>
					</ul>
					
				</div>
				<ul class="ul-b">
					<li>
						<a class="label-1" href="#">My Expenses</a>
						<i class="bg-arrow-right-a"><img class="arrow-right-a-img" src="<c:url value="/resources/images/arrow-right-a.png" />" /></i>
					</li>
					<li>
						<a class="label-1" href="${pageContext.request.contextPath}/createNewExpense">New Expenses</a>
						
					</li>
					<c:if test="${sessionScope.user.role.roleName eq 'ROLE_MANAGER'}">
						<li>
							<a class="label-1" href="${pageContext.request.contextPath}/loadApprovalExpenses">Approvals</a>
						
						</li>
					</c:if>
				</ul>
			</div>
		</div>
		
		<div class="mc-a">
			<h2 class="header label-2">
				Expenses
			</h2>
			
		<c:if test="${fn:length(pendingExpenseList) gt 0 }">
			<table  id="table-a" class="table-a">
				<thead>
					<tr>
					<td class="td-b">
						<h5 class="label-3">Attachment <img class="clip-b-img" src="<c:url value="/resources/images/clip-b.png" />"></h5>
					</td>
						<td class="td-b">
							<h5 class="label-3">Date </h5>
						</td>
						<td  class="td-b">
							<h5 class="label-3">Expense Type </h5>
						</td>
						<td  class="td-c">
							<h5 class="label-3">Description </h5>
						</td>
						<td  class="td-d">
							<h5 class="label-3">Status </h5>
						</td>
					</tr>
				</thead>
				<tbody>
				<c:forEach var="expense" items="${pendingExpenseList}">
					<tr class="tr-odd">
						<td>
							<c:if test="${not empty expense.attachment.fileName}">
							 <a href="${pageContext.request.contextPath}/download?attachmentId=${expense.attachment.id}"><img class="clip-b-img" src="<c:url value="/resources/images/clip-b.png" />"></a>
							</c:if>
						</td>
						<td>
							<h5 class="label-4">${expense.expenseDate} </h5>
						</td>
						<td>
							<h5 class="label-4">${expense.expenseType.name} </h5>
						</td>
						<td>
							<h5 class="label-4">${expense.description} </h5>
						</td>
						<td>
							<h5 class="label-4">
								<c:if test="${expense.state eq 'NEW' }">
								<i class="alt-a label-5 blue">
									<img class="right-a-img" src="<c:url value="/resources/images/question-a.png" /> ">
									 Pending</i>
								<i class="alt-b label-6">
									<img class="right-b-img" src="<c:url value="/resources/images/right-b.png" />" > Pending
									<a href="${pageContext.request.contextPath}/deleteExpense?expenseId=${expense.id}" class="bg-cross-b"><img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />"> </a>
									<a href="${pageContext.request.contextPath}/editExpense?expenseId=${expense.id}" class="bg-cross-c"><img class="cross-b-img" src="<c:url value="/resources/images/edit1.jpg" />"> </a>
								</i>
								
								</c:if>
								<c:if test="${expense.state eq 'APPROVED' }">
								<i class="alt-a label-5 green">
									<img class="right-a-img" src="<c:url value="/resources/images/right-a.png" /> ">
									 Approved</i>
								<i class="alt-b label-6">
									<img class="right-b-img" src="<c:url value="/resources/images/right-b.png" />" > Approved
									<a href="${pageContext.request.contextPath}/deleteExpense?expenseId=${expense.id}" class="bg-cross-b"><img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />"> </a>
									<a href="${pageContext.request.contextPath}/editExpense?expenseId=${expense.id}" class="bg-cross-c"><img class="cross-b-img" src="<c:url value="/resources/images/edit1.jpg" />"> </a>
								</i>
								</c:if>
								<c:if test="${expense.state eq 'REJECTED' }">
								 <i class="alt-a label-5 red"><img class="cross-a-img" src="<c:url value="/resources/images/cross-a.png" />" > Rejected</i> 
								<i class="alt-b label-6">
									<img class="right-b-img" src="<c:url value="/resources/images/cross-a.png" />" > Rejected
									<a href="${pageContext.request.contextPath}/deleteExpense?expenseId=${expense.id}" class="bg-cross-b"><img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />" > </a>
									<a href="${pageContext.request.contextPath}/editExpense?expenseId=${expense.id}" class="bg-cross-c"><img class="cross-b-img" src="<c:url value="/resources/images/edit1.jpg" />"> </a>
								</i>
								</c:if>
							</h5>
						</td>
					</tr>	
				</c:forEach>
				</tbody>
			</table>
 						
		<div id="pager"  class="pagination pagination-1" >

		<form>

			<img src="<c:url value="/resources/images/first.png"/>"  class="first"/>

			<img src="<c:url value="/resources/images/prev.png"/>"  class="prev"/>

			<input type="text"  class="pagedisplay"/>

			<img src="<c:url value="/resources/images/next1.png" />"  class="next"/>

			<img src="<c:url value="/resources/images/last.png" />"  class="last"/>

			<select  class="pagesize">

				<option selected="selected"  value="10">10</option>

				<option value="20">20</option>

				<option value="30">30</option>

				<option  value="40">40</option>

			</select>

		</form>

	</div>
		</c:if>
			
		</div>

		
				<script>

    jQuery(document).ready(function() {
    	

    jQuery("#table-a")

    .tablesorter({widthFixed: true, widgets: ['zebra']})
    .tablesorterPager({container: $("#pager")}); 

    });

</script>
	</body>
</html>
```

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
 <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
	<head>
		<title>3</title>
		<link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
		<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
	</head>
	<body>
		<div class="blue-b">
			<div class="inside-nav">
				<div class="topper">
					<img class="bg-logo-b" src="<c:url value="/resources/images/logo-b.png" />" />
					<ul class="nav-a">
						<li>
							<a href="#">Hi ${sessionScope.user.userName}</a> 
						</li>
						<li>
							|
						</li>
						<li>
							<a href="<c:url value="/j_spring_security_logout" />">Logout</a>
						</li>
					</ul>
				</div>
				<ul class="ul-b">
					<li>
						<a class="label-1" href="${pageContext.request.contextPath}/">My Expenses</a>
					</li>
					<li>
						<a class="label-1" href="${pageContext.request.contextPath}/createNewExpense">New Expense</a>
						<i class="bg-arrow-right-a"><img class="arrow-right-a-img" src="<c:url value="/resources/images/arrow-right-a.png" />"></i>
					</li>
					<c:if test="${sessionScope.user.role.roleName eq 'ROLE_MANAGER'}">
						<li>
							<a class="label-1" href="${pageContext.request.contextPath}/loadApprovalExpenses">Approvals</a>
						</li>
				   </c:if>
				</ul>
			</div>
		</div>
		<c:choose>
		<c:when test="${isEdit eq 'true'}">
			<div class="mc-a">
			<button class="btn-b my-expenses-btn"><img class="arrow-left-a-img" src="<c:url value="/resources/images/arrow-left-a.png" /> ">My Expenses
			</button>
			<h2 class="header label-2"> Editing an expense </h2>
			<form action="${pageContext.request.contextPath}/updateExpense" method="post" enctype="multipart/form-data">
			<textarea class="textbox-a" id="description" name="description" placeholder="description">${expense.description}</textarea><br>
			<input type="hidden" name="expenseId" id="expenseId" value="${expense.id}" />
			<input class="textbox-a" type="text" placeholder="Amount" name="amount" value="${expense.amount}"></input>
			<div class="action-a">
				<!-- <div class="textbox-a dd-a flt-left">
				Select Type <img class="arrow-down-c-img" src="<c:url value="/resources/images/arrow-down-c.png" />">  -->
				<select id="expenseTypeId" name="expenseTypeId">
					<c:forEach var="expenseType" items="${expenseTypeList}">
						
  					    <c:if test="${expense.expenseType.name eq expenseType.name }">
							<option value="${expenseType.id}" selected="selected">${expenseType.name}</option>
						</c:if>
						 <c:if test="${expense.expenseType.name ne expenseType.name }">
							<option value="${expenseType.id}">${expenseType.name}</option>
						</c:if>
					</c:forEach>
				</select>
				<!-- </div> -->
				<%-- <a class="label-7 flt-right" href="#"><img class="clip-a-img" src="<c:url value="/resources/images/clip-a.png" />"> --%>
				 <input type="file" name="file" id="file" class="file" /> <!-- Attach Receipt </a> -->
			</div>
			<button class="btn-a add-expense-btn">SAVE EXPENSE</button>
			</form>
		</div>
		</c:when>
		<c:otherwise>
		<div class="mc-a">
			<button class="btn-b my-expenses-btn"><img class="arrow-left-a-img" src="<c:url value="/resources/images/arrow-left-a.png" /> ">My Expenses
			</button>
			<h2 class="header label-2"> Adding new expense </h2>
			<form action="${pageContext.request.contextPath}/createNewExpenseReport" method="post" enctype="multipart/form-data">
			<textarea class="textbox-a" id="description" name="description" placeholder="description" required="required"  ></textarea><br>
			<input class="textbox-a" type="number" placeholder="Amount" name="amount" required="required"  max="100000" ></input>
			<div class="action-a">
			<!-- 	<div class="textbox-a dd-a flt-left"> -->
				<select id="expenseTypeId" name="expenseTypeId" style="	border: 1px solid #b8b8b8;
	            box-shadow: 0px 2px 0px 0px #b8b8b8;padding: 12px;font-size: 11px;color: #a7a5a6;width:100px;	margin:10px;">
					<option value="">                         </option>
					<c:forEach var="expenseType" items="${expenseTypeList}">
						<option value="${expenseType.id}">    ${expenseType.name}    </option>
					</c:forEach>
				</select>
				<!-- </div>
				 <a class="label-7 flt-right" href="#"><img class="clip-a-img" src="<c:url value="/resources/images/clip-a.png" />">  -->
				 <input type="file" name="file" id="file" title="Attachment"  class="file"/>
			    <!-- </a> -->
			</div>
			<button class="btn-a add-expense-btn">ADD EXPENSE</button>
			</form>
		</div>
		</c:otherwise>
		</c:choose>
	</body>
</html>
```

```jsp
<!DOCTYPE HTML>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
 <%@ taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn" %>  
<html>
	<head>
		<title>2</title>
		<link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
		<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
		<script src="<c:url value="resources/javascript/jquery-1.8.1.js"/>" ></script>
		<script src="<c:url value="resources/javascript/jquery.tablesorter.js"/>" ></script>
		<script src="<c:url value="resources/javascript/jquery.tablesorter.min.js"/>" ></script>
		<script src="<c:url value="resources/javascript/jquery.tablesorter.pager.js"/>" ></script>

		
	</head>
	<body>
		<div class="blue-b">
			<div class="inside-nav">
				<div class="topper">
					<img class="bg-logo-b" src="<c:url value="/resources/images/logo-b.png" />" />
					<ul class="nav-a">
						<li>
							<a href="#">Hi ${sessionScope.user.userName} </a> 
						</li>
						<li>
							|
						</li>
						<li>
							<a href="<c:url value="/j_spring_security_logout" />">Logout</a> 
						</li>
					</ul>
					
				</div>
				<ul class="ul-b">
					<li>
						<a class="label-1" href="#">My Expenses</a>
						<i class="bg-arrow-right-a"><img class="arrow-right-a-img" src="<c:url value="/resources/images/arrow-right-a.png" />" /></i>
					</li>
					<li>
						<a class="label-1" href="${pageContext.request.contextPath}/createNewExpense">New Expenses</a>
						
					</li>
					<c:if test="${sessionScope.user.role.roleName eq 'ROLE_MANAGER'}">
						<li>
							<a class="label-1" href="${pageContext.request.contextPath}/loadApprovalExpenses">Approvals</a>
						
						</li>
					</c:if>
				</ul>
			</div>
		</div>
		
		<div class="mc-a">
			<h2 class="header label-2">
				Expenses
			</h2>
			
		<c:if test="${fn:length(pendingExpenseList) gt 0 }">
			<table  id="table-a" class="table-a">
				<thead>
					<tr>
					<td class="td-b">
						<h5 class="label-3">Attachment <img class="clip-b-img" src="<c:url value="/resources/images/clip-b.png" />"></h5>
					</td>
						<td class="td-b">
							<h5 class="label-3">Date </h5>
						</td>
						<td  class="td-b">
							<h5 class="label-3">Expense Type </h5>
						</td>
						<td  class="td-c">
							<h5 class="label-3">Description </h5>
						</td>
						<td  class="td-d">
							<h5 class="label-3">Status </h5>
						</td>
					</tr>
				</thead>
				<tbody>
				<c:forEach var="expense" items="${pendingExpenseList}">
					<tr class="tr-odd">
						<td>
							<c:if test="${not empty expense.attachment.fileName}">
							 <a href="${pageContext.request.contextPath}/download?attachmentId=${expense.attachment.id}"><img class="clip-b-img" src="<c:url value="/resources/images/clip-b.png" />"></a>
							</c:if>
						</td>
						<td>
							<h5 class="label-4">${expense.expenseDate} </h5>
						</td>
						<td>
							<h5 class="label-4">${expense.expenseType.name} </h5>
						</td>
						<td>
							<h5 class="label-4">${expense.description} </h5>
						</td>
						<td>
							<h5 class="label-4">
								<c:if test="${expense.state eq 'NEW' }">
								<i class="alt-a label-5 blue">
									<img class="right-a-img" src="<c:url value="/resources/images/question-a.png" /> ">
									 Pending</i>
								<i class="alt-b label-6">
									<img class="right-b-img" src="<c:url value="/resources/images/right-b.png" />" > Pending
									<a href="${pageContext.request.contextPath}/deleteExpense?expenseId=${expense.id}" class="bg-cross-b"><img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />"> </a>
									<a href="${pageContext.request.contextPath}/editExpense?expenseId=${expense.id}" class="bg-cross-c"><img class="cross-b-img" src="<c:url value="/resources/images/edit1.jpg" />"> </a>
								</i>
								
								</c:if>
								<c:if test="${expense.state eq 'APPROVED' }">
								<i class="alt-a label-5 green">
									<img class="right-a-img" src="<c:url value="/resources/images/right-a.png" /> ">
									 Approved</i>
								<i class="alt-b label-6">
									<img class="right-b-img" src="<c:url value="/resources/images/right-b.png" />" > Approved
									<a href="${pageContext.request.contextPath}/deleteExpense?expenseId=${expense.id}" class="bg-cross-b"><img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />"> </a>
									<a href="${pageContext.request.contextPath}/editExpense?expenseId=${expense.id}" class="bg-cross-c"><img class="cross-b-img" src="<c:url value="/resources/images/edit1.jpg" />"> </a>
								</i>
								</c:if>
								<c:if test="${expense.state eq 'REJECTED' }">
								 <i class="alt-a label-5 red"><img class="cross-a-img" src="<c:url value="/resources/images/cross-a.png" />" > Rejected</i> 
								<i class="alt-b label-6">
									<img class="right-b-img" src="<c:url value="/resources/images/cross-a.png" />" > Rejected
									<a href="${pageContext.request.contextPath}/deleteExpense?expenseId=${expense.id}" class="bg-cross-b"><img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />" > </a>
									<a href="${pageContext.request.contextPath}/editExpense?expenseId=${expense.id}" class="bg-cross-c"><img class="cross-b-img" src="<c:url value="/resources/images/edit1.jpg" />"> </a>
								</i>
								</c:if>
							</h5>
						</td>
					</tr>	
				</c:forEach>
				</tbody>
			</table>
   <!-- 
			<div class="pagination pagination-1">
						<ul id="tata">
							<li class="disabled">
								<a href="#">< &nbsp;&nbsp; previous</a>
							</li>
							<li class="active">
								<a href="#">1</a>
							</li>
							<li>
								<a href="#">2</a>
							</li>
							<li>
								<a href="#">3</a>
							</li>
							<li>
								<a href="#">4</a>
							</li>
							<li>
								<a href="#">next &nbsp;&nbsp;></a>
							</li>
						</ul>
					</div> -->
					
		
		
		
	
						
		<div id="pager"  class="pagination pagination-1" >

		<form>

			<img src="<c:url value="/resources/images/first.png"/>"  class="first"/>

			<img src="<c:url value="/resources/images/prev.png"/>"  class="prev"/>

			<input type="text"  class="pagedisplay"/>

			<img src="<c:url value="/resources/images/next1.png" />"  class="next"/>

			<img src="<c:url value="/resources/images/last.png" />"  class="last"/>

			<select  class="pagesize">

				<option selected="selected"  value="10">10</option>

				<option value="20">20</option>

				<option value="30">30</option>

				<option  value="40">40</option>

			</select>

		</form>

	</div>
		</c:if>
			
		</div>

		
				<script>

    jQuery(document).ready(function() {
    	

    jQuery("#table-a")

    .tablesorter({widthFixed: true, widgets: ['zebra']})
    .tablesorterPager({container: $("#pager")}); 

    });

</script>
	</body>
</html>
```

```jsp
<!DOCTYPE HTML>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
	<head>
		<title>1</title>
		<link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
		<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
		<script type="text/javascript">
		
		function ValidateEmail() 
		{
	     var email = document.getElementById("email");
	     var errorblock = document.getElementById("errorblock");
	     if(email!=null && email.value!=""){
		 if (/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(email.value))
		  {
			 alert("email1"+email.value);
		    return true;
		  }
		 	errorblock.innerHTML = "Please Enter valid Email Address";
		    return false;
	     }else{
	    	 alert("please enter email address!");
	    	 return false;
	     }
		}
		</script>
	</head>
	<body>
		<div class="blue-a">
			<div class="login-nav">
				
				<ul class="ul-a">
					<li>
						<i class="bg-logo-a"><img src="<c:url value="/resources/images/logo-a.png" /> "></i>
					</li>
					<li>
						<a class="label-1" href="${pageContext.request.contextPath}/login">Login</a>
					</li>
					<li>
						<i class="bg-arrow-right-a"><img class="arrow-right-a-img" src="<c:url value="/resources/images/arrow-right-a.png" /> "></i>
						<a class="label-1" href="#">Register</a>
						
					</li>
				</ul>
			</div>
		</div>
		
		<div class="main-a">
			<c:if test="${not empty param.errorMsg}">
			
				<div class="errorblock">
					${param.errorMsg}
				</div>
			</c:if>
		<br><br><br>
			 <form action="${pageContext.request.contextPath}/signUpUser" method="post">
					<input class="textbox-a text-email" type="email" placeholder="Email" id="email" name="email" required="required" >
					<input class="textbox-a" type="text" placeholder="UserName" id="userName" name="userName" required="required">
					<input class="textbox-a" type="password" placeholder="Password" id="password" name="password" required="required">
					<button class="btn-a">Submit</button>
			</form>
		</div>
		
		
	</body>
</html>
```



##Build and Run the App Locally
 
      Now we have successfully created expensereport application and make sure you have added these dependencies into your pom.xml.



Now we can run the application in locally to test it.
Select `Run As -> Maven clean`
![maven_clean.png](/images/spring_tutorial/maven_clean.png)

Once Maven clean completed select` Run As -> Maven install`.It will download dependencies from pom.xml.
![maven_install.png](/images/spring_tutorial/maven_install.png)

Select `Run As -> Maven build`
![maven_build.png](/images/spring_tutorial/maven_build.png)

And give Maven goal as , `tomcat:run`
![maven_run.png](/images/spring_tutorial/maven_run.png)

![STS Fabric Server.png](/images/spring_tutorial/localhost_login.png)


Once server starts completed,STS will open the home page for the project which dispalys login page.

![spring-expensereport-login.png](/images/spring_tutorial/localhost_login.png)

![spring-expensereport-create_expense.png](/images/spring_tutorial/create_new_expense.png)

![spring-expensereport-user_expense.png](/images/spring_tutorial/my_expense.png)


##Deploy the App on Cloud Foundry Using VMC
Deployment to Cloud Foundry can be done in two simple steps:

+  Create war file
+  Push the war file using `vmc push`

Commands below show the exact commands and output when the app is deployed:

Go to your project work space and issue the following commands.

``` bash
$ mvn war:war
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building SpringMVC-ExpenseReport 1.0.0-BUILD-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-war-plugin:2.1.1:war (default-cli) @ html5expense ---
[INFO] Packaging webapp
[INFO] Assembling webapp [html5expense] in [/home/senthils/Code/From Home/SpringMVC-ExpenseReport/target/html5expense-1.0.0-BUILD-SNAPSHOT]
[INFO] Processing war project
[INFO] Copying webapp resources [/home/senthils/Code/From Home/SpringMVC-ExpenseReport/src/main/webapp]
[INFO] Webapp assembled in [380 msecs]
[INFO] Building war: /home/senthils/Code/From Home/SpringMVC-ExpenseReport/target/html5expense-1.0.0-BUILD-SNAPSHOT.war
[INFO] WEB-INF/web.xml already added, skipping
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.443s
[INFO] Finished at: Mon Sep 17 15:57:50 IST 2012
[INFO] Final Memory: 6M/124M
[INFO] ------------------------------------------------------------------------

$ vmc push
Would you like to deploy from the current directory? [Yn]: /home/senthils/.rvm/gems/ruby-1.9.2-head/gems/interact-0.4.8/lib/interact/interactive.rb:569: warning: Insecure world writable dir /home/senthils/Downloads/springsource in PATH, mode 040777

Application Name: html5expense
Detected a Java SpringSource Spring Application, is this correct? [Yn]: Y
Application Deployed URL [html5expense.cloudfoundry.com]: 
Memory reservation (128M, 256M, 512M, 1G, 2G) [512M]: 128M
How many instances? [1]: 
Bind existing services to 'html5expense'? [yN]: 
Create services to bind to 'html5expense'? [yN]: Y
1: mongodb
2: mysql
3: postgresql
4: rabbitmq
5: redis
What kind of service?: 3
Specify the name of the service [postgresql-94876]: 
Create another? [yN]: 
Would you like to save this configuration? [yN]: 
Creating Application: OK
Creating Service [postgresql-94876]: OK
Binding Service [postgresql-94876]: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (10K): OK   
Push Status: OK
Staging Application 'html5expense': OK                                          
Starting Application 'html5expense': OK
```

##Deploy the App on Cloud Foundry Using STS

1.     Go to servers view,right click add New Server.Now you can see Cloud Foundry under VMware.Select Cloud Foundry,then click next.It will ask your account information and Choose VMware Cloud Foundry - http://api.cloudfoundry.com from the URL.
![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry.png)

2.     Now give your Cloud Foundry account information.Click validate and make sure you have entered  valid credentials.
![cloud foundry account information.png](/images/spring_tutorial/cloud_foundry_account.png)

3.     Click next it will show you add and remove view.Now you can select our spring application.Once you added your application it will open new window with application name and application type.You can enter your application name.
![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry_project_deploy.png) 
![spring-expensereport-project-deploy.png](/images/spring_tutorial/project_deploy_step2.png)


4.     Click next you can see deployed URL(your application name+cloudfoundry.com) and memory reservation.by default memory reservation selected to 512M.You can increase memory if required..
![spring-expensereport-project-deploy-step-2.png](/images/spring_tutorial/project_deploy_step3.png)

5.     Click next now it will ask for services selection.Since we are using postgresql in our application we need to add postgresql service.Click add services to server it will open service configuration window.select type as Postgre SQL database service(VFabric) and give name to this service then click 
![service_selection.png](/images/spring_tutorial/service_selection.png)
![create_new_service.png](/images/spring_tutorial/create_new_service.png)

6.     Now you can see your services in services selection window.Select postgresql service and click finish.Now server starts to deploy our application into Cloud Foundry.
![spring-expensereport-login.png](/images/spring_tutorial/service_selection_1.png)



Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`

![deployed_application_in_cloud_foundry.png](/images/spring_tutorial/deployed_application_in_cloud_foundry.png)



