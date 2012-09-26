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
