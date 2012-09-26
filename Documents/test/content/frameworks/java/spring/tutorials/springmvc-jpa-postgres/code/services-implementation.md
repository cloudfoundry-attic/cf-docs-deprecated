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
		Query query = entityManager.createQuery("select a from Attachment a where 
				a.expense.id =:expenseId");
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
		entityManager.persist(expense);
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
		Query query = getEntityManager().createQuery("select e from Expense e where 
				e.user.userId =:userId ORDER BY e.expenseDate DESC",Expense.class);
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
		Query query = entityManager.createQuery("select e from Expense e where 
				e.state IN (:state) ORDER BY e.expenseDate DESC",Expense.class);
				query.setParameter("state", state);
				return query.getResultList();
	}

	@Override
	@Transactional
	public List<Expense> getApprovedAndRejectedExpensesList(){
		List<Enum> state = new ArrayList<Enum>();
		state.add(State.REJECTED);
		state.add(State.APPROVED);
		Query query = entityManager.createQuery("select e from Expense e where 
				e.state IN (:state) e.expenseDate DESC",Expense.class);
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
	public void updateExpense(Long expenseId,String description,Double amount,
			ExpenseType expenseType){
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
		Query query = getEntityManager().createQuery("select r from Role r where 
				roleName =:roleName",Role.class);
				query.setParameter("roleName", name);
				return (query.getResultList()!=null && query.getResultList().size()>0)?
						(Role)query.getResultList().get(0):null;
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
		Query query = getEntityManager().createQuery("from User u where 
				u.userName =:username");
				query.setParameter("username",userName);
				return  (query.getResultList()!=null && query.getResultList().size()>0)?
						(User)(query.getResultList().get(0)):null;
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
