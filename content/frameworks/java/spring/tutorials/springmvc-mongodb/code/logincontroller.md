```java
package com.springsource.html5expense.controller;

import javax.servlet.http.HttpServletRequest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import com.springsource.html5expense.model.Role;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.service.RoleService;
import com.springsource.html5expense.service.UserService;

@Controller
public class LoginController {

    @Autowired
    UserService userService;

    @Autowired
    RoleService roleService;

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

    @RequestMapping(value="/signup",method=RequestMethod.GET)
    public String signUp(){
       String name="signup";
       return name;
    }

    @RequestMapping(value="/signup",method=RequestMethod.POST)
    public String signUp(ModelMap model,HttpServletRequest request){
       String userName = request.getParameter("userName");
       String password = request.getParameter("password");
       String mailId = request.getParameter("email");
       User user = getUserService().getUserByUserName(userName);
       if(user == null){
          Role role = getRoleService().getRoleByName("ROLE_USER");
          user = getUserService().createUser(userName, password, mailId,role);
          String succMsg = "User registration succeeds ";
          return "redirect:/login?succMsg="+succMsg;
      }else{
           String errorMsg = "User registration fails already same user name exists please try " +
            "with different user name";
           request.setAttribute("errorMsg",errorMsg);
           return "redirect:/signup?errorMsg="+errorMsg;
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

    @RequestMapping(value="/sessiontimeout",method = RequestMethod.GET)
    public String sessionTimeout(ModelMap model){
       String errorMsg = "Session has expired. Please login";
       model.addAttribute("errorMsg",errorMsg);
       return "login";
    }
}
```
