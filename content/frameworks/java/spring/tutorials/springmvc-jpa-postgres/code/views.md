add the below jsp file as `login.jsp`

```jsp
<!DOCTYPE HTML>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
    <head>
    <title>Login</title>
    <link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
    </head>
    <body>
    <div id="contentWrapper">
    <div class="blue-b">
    <div class="inside-nav">
        <div class="logo">
            <img src="<c:url value="/resources/images/logo.png" />" />
        </div>
        <ul class="nav-a">
            <li>
            <a class="selected" href="#">Login</a>
            </li>
            <li>|</li>
            <li>
            <a class="" href="${pageContext.request.contextPath}/signup">Register</a>
            </li>
        </ul>
    </div>
    </div>
    <div class="mc-a">
    <div class="signing-form">
    <c:if test="${not empty error}">
        <div class="errorblock">
            Your login attempt was not successful, try again.<br /> Caused :
            ${sessionScope["SPRING_SECURITY_LAST_EXCEPTION"].message}
        </div>
    </c:if>
    <c:if test="${not empty param.succMsg }">
        <div class="successblock">
         ${param.succMsg}
        </div>
    
    </c:if>
    ${errorMsg}
    <form action="<c:url value='j_spring_security_check' />" method='POST'>
        <div>
            <label>Username</label>
              <input class="textbox-a" id="username" name="j_username" placeholder="UserName" type="text" required="required">
          </div>
          <div>
              <label>Password</label>
              <input class="textbox-a" id="password" name="j_password" placeholder="Password" type="password" required="required">
          </div>
          <div>
              <input class="btn-a" name="commit" type="submit" value="Log in">
          </div>
    </form>
    </div>
    
    </div>
    </div>
    </body>
</html>
```
add the below jsp file as `expensereports.jsp`

```jsp
<%@ page session="false" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!doctype html>
<html>
<head>
    <!-- <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script> -->
    <script src="http://code.jquery.com/jquery-latest.js"></script>
    <script type="text/javascript" src="http://jzaefferer.github.com/jquery-validation/jquery.validate.js"></script>
    <script src="<c:url value="/resources/javascript/expensereport.js" />"></script>
    <script src="<c:url value="resources/javascript/jquery.tablesorter.js"/>" ></script>
    <script src="<c:url value="resources/javascript/jquery.pagination.js"/>" ></script>
    <link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
</head>
<body>
<div id="contentWrapper">
   <div class="blue-b">
     <div class="inside-nav">
        <div class="logo">
            <img src="<c:url value="/resources/images/logo.png" />" />
        </div>
        <ul class="nav-a">
            <li><a href="#">Hi ${user.userName}</a></li>
            <li>|</li>
            <li><a href="<c:url value="/j_spring_security_logout" />">Logout</a></li>
         </ul>
     </div>
  </div>
  <div id="nav">
    <ul>
     <li><a id="myExpenseTab" class="tabLinks" href="#" data-tab="myExpenseContainer">My Expenses</a></li>
     <li><a  class="tabLinks" href="#" data-tab="newExpenseContainer">New Expense</a></li>
     <c:if test="${user.role.roleName eq 'ROLE_MANAGER'}">
      <li><a id="approvalTab" href="#" class="tabLinks" data-tab="approvalsContainer">Approvals</a></li>
     </c:if>
    </ul>
  </div>
      <div class = "masterContainer">
        <div id="newExpenseContainer" class="mc-a tabContainer">
            <br><br><br>
            <h2 class="header label-2 mbot20"> Adding new expense </h2>
            <div class="action-a">
                 <label class="label-name" for="expense_description">Description</label>
                 <textarea class="textbox-a" placeholder="description" required="required"></textarea>
            </div>
            <input type="hidden" id="expenseload" />
            <div class="action-a">
                 <label class="label-name" for="expense_amount">Amount</label>
                  <input class="textbox-a dd-b flt-left mleft0" id="amount" name="amount" required="required" placeholder="Amount" size="30" type="text">
            </div>
            <div class="action-a">
            <label class="label-name" for="expenseTypeId">Expense Type</label>
                <div class="textbox-a dd-a mleft0">
                    <select id="expenseTypeId" name="expenseTypeId"  required="required"  >
                      <option value=""></option>
                       <c:forEach var="expenseType" items="${expenseTypeList}">
                         <option value="${expenseType.id}">${expenseType.name}</option>
                       </c:forEach>
                    </select>
                </div>
             </div>
            <button class="btn-a add-expense-btn">ADD EXPENSE</button>
            <button class="btn-a edit-expense-btn">UPDATE EXPENSE</button>
        </div>
        <div id="myExpenseContainer" class="myexpense tabContainer active">
            <table  id="expenseTable" class="table-a mtop80">
                <thead>
                    <tr>
                    <td class="td-a">
                        <h5 class="label-3">Expense Id </h5>
                    </td>
                        <td class="td-b">
                            <h5 class="label-3">Date </h5>
                        </td>
                        <td  class="td-c">
                            <h5 class="label-3">Expense Type </h5>
                        </td>
                        <td  class="td-d">
                            <h5 class="label-3">Description </h5>
                        </td>
                        <td  class="td-d">
                            <h5 class="label-3">Status </h5>
                        </td>
                    </tr>
                </thead>
                <tbody> </tbody>
            </table>
            <div id="Pagination" class="pagination"> </div>
          </div>
          <div id="approvalsContainer" class="approvalExpense tabContainer">
             <table id="approvalTable" class="table-a mtop80">
                    <thead>
                        <tr>
                            <td class="td-a">
                                <h5 class="label-3">
                                    Date<i class="bg-arrow-down-a"><img
                                        src="<c:url value="/resources/images/arrow-down-a.png"/>"></i>
                                </h5>
                            </td>
                            <td class="td-b">
                                <h5 class="label-3">Expense ID</h5>
                            </td>
                            <td class="td-c">
                                <h5 class="label-3">Description</h5>
                            </td>
                            <td class="td-d">
                                <h5 class="label-3">Status</h5>
                            </td>
                        </tr>
                    </thead>
                    <tbody></tbody>
                </table>
           </div>
        </div>
 </div>
<input type="hidden" id="contextPath" value="${pageContext.request.contextPath}"/>
</body>
</html>
```

add the below jsp file as `signup.jsp`
```jsp
<!DOCTYPE HTML>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
    <head>
    <title>Sign Up</title>
    <link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
    <script src="http://code.jquery.com/jquery-latest.js"></script>
      <script type="text/javascript" src="http://jzaefferer.github.com/jquery-validation/jquery.validate.js"></script>
    <style type="text/css">
     * { font-family: Verdana; font-size: 96%; }
    label { width: 10em; float: left; }
    label.error { float: none; color: red; vertical-align: bottom;
            padding:20px 0px 10px 0px; display:inline}
    p { clear: both; }
    .submit { margin-left: 12em; }
    em { font-weight: bold; padding-right: 1em; vertical-align: top; } 
    </style>
      <script>
      $(document).ready(function(){
        $("#signupForm").validate({
           rules: {
                 userName: "required",
                 password: "required",
                 email: {
                   required: true,
                   email: true
                 }
               },
               messages: {
                 userName: "Please enter your user name",
                 password: "Please enter your password",
                 email: {
                   required: "Please enter your email",
                   email: "Your email address must be in the format of name@domain.com"
                 }
               }
            });
      });
      </script>
    
    </head>
    <body>
    <div id="contentWrapper">
    <div class="blue-b">
    <div class="inside-nav">
        <div class="logo">
            <img src="<c:url value="/resources/images/logo.png" />" />
        </div>
        
        <ul class="nav-a">
            <li>
            <a href="${pageContext.request.contextPath}/login" class="">Login</a>
            </li>
                 <li>|</li>
            <li>
            <a class="selected" href="#">Register</a>
            
            </li>
        </ul>
    </div>
    </div>
    
    <div class="mc-a">
    <c:if test="${not empty param.errorMsg}">
    
        <div class="errorblock">
            ${param.errorMsg}
        </div>
    </c:if>
    <div class="signing-form">
     <form action="${pageContext.request.contextPath}/signup" class="new_user" method="post" id="signupForm">
         <div>
              <label>Username</label>
              <input class="textbox-a" id="userName" name="userName" required="required"  placeholder="UserName" size="30" type="text">
        </div>
        <div>
              <label>Password</label>
              <input class="textbox-a" id="password" name="password" placeholder="Password" size="30" type="password" required="required">
        </div>
        <div>
              <label>Email</label>
              <input class="textbox-a" id="email" name="email" placeholder="email" size="30" type="email" required="required">
        </div>
        <div>
              <input class="btn-a" name="commit" type="submit" value="Sign Up">
        </div>
    </form>
    </div>
    </div>
    </div>
    </body>
</html>>
```
