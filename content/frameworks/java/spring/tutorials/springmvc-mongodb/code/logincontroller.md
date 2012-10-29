```java
package com.springsource.html5expense.controllers;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import com.springsource.html5expense.model.Role;
import com.springsource.html5expense.model.User;
import com.springsource.html5expense.services.RoleService;
import com.springsource.html5expense.services.UserService;

@Controller
public class LoginController {

    @Autowired
    private UserService userService;

    @Autowired
    private RoleService roleService;

    @RequestMapping(value = "/signup", method = RequestMethod.GET)
    public String signUp() {
        return "signup";
    }

    @RequestMapping(value = "/signup", method = RequestMethod.POST)
    public String signUp(ModelMap model, @RequestParam("userName")String userName, @RequestParam("password")String password,
    		@RequestParam("email")String mailId) {
        User user = userService.getUserByUserName(userName);
        if (user == null) {
            Role role = roleService.getRoleByName("ROLE_USER");
            user = new User(userName, password, mailId);
            user.setRole(role);
            user.setEnabled(true);
            user = userService.save(user);
            String succMsg = "User registration succeeds ";
            return "redirect:/login?succMsg=" + succMsg;
        } else {
            String errorMsg = "User registration fails already same user name exists please try "
                 + "with different user name";
            model.addAttribute("errorMsg", errorMsg);
            return "redirect:/signup?errorMsg=" + errorMsg;
        }
    }

    @RequestMapping(value = "/login", method = RequestMethod.GET)
    public String login(ModelMap model) {
        return "login";
    }

    @RequestMapping(value = "/loginfailed", method = RequestMethod.GET)
    public String loginerror(ModelMap model) {
        model.addAttribute("error", "true");
        return "login";
    }

    @RequestMapping(value = "/logout", method = RequestMethod.GET)
    public String logout(ModelMap model) {
        return "login";
    }

    @RequestMapping(value = "/sessiontimeout", method = RequestMethod.GET)
    public String sessionTimeout(ModelMap model) {
        String errorMsg = "Session has expired. Please login";
        model.addAttribute("errorMsg", errorMsg);
        return "login";
    }
}
```
