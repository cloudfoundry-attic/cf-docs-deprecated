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
