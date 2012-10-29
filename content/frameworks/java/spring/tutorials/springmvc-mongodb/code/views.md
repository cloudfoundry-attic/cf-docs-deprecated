---
title: Spring MVC ExpenseReport App Views Code
description: Spring MVC ExpenseReport App Views Code
tags:
    - Spring
    - code
    - jsp
    - view
---

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
add the below jsp file as `expenseapproval.jsp`

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn"%>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>


<!DOCTYPE HTML>
<html>
<head>
<title>Expense Approval</title>
    <link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
    <script src="<c:url value="resources/javascript/jquery-1.8.1.js"/>" ></script>
    <script src="<c:url value="resources/javascript/jquery.tablesorter.js"/>" ></script>
    <script src="<c:url value="resources/javascript/jquery.tablesorter.min.js"/>" ></script>
    <script src="<c:url value="resources/javascript/jquery.tablesorter.pager.js"/>" ></script>
<script type="text/javascript">
    function changeState(contextPath, expenseId, rowCount) {
    var actionVal = document.getElementById('action_' + rowCount).value;
    var url = contextPath + "/expensereports/changestate/"+expenseId+"/"+actionVal;
        //+ "&expenseId=" + expenseId;
    if (actionVal == null || actionVal == "") {
    return false;
    }
    document.forms[0].action = url;
    document.forms[0].submit();
    }
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
        <li><a href="#">Hi ${sessionScope.user.userName}</a></li>
        <li>|</li>
        <li><a href="<c:url value="/j_spring_security_logout" />">Logout</a>
        </li>
        </ul>
    </div>
    </div>
    <div id="nav">
    <ul>
        <li><a href="${pageContext.request.contextPath}/expensereports">My
        Expenses</a></li>
        <li><a
        href="${pageContext.request.contextPath}/expensereports/new">New
        Expense</a></li>
        <c:if test="${sessionScope.user.role.roleName eq 'ROLE_MANAGER'}">
        <li><a
        href="${pageContext.request.contextPath}/expensereports/approvals">Approvals</a>
        </li>
        </c:if>
    </ul>
    </div>
    <div class="mc-a">
    <h2 class="header label-2">Approvals Pending
        ${fn:length(approvedExpenseList)}</h2>
    <c:if test="${fn:length(approvedExpenseList) gt 0 }">
        <table id="table-a" class="table-a mtop80">
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
        <tbody>
        <c:forEach var="expense" items="${approvedExpenseList}"
            varStatus="rowCounter">
            <input type="hidden" name="rowcount" value="${rowCounter.count}" />
            <tr class="tr-odd">
            <td>
            <h5 class="label-4">${expense.expenseDate}</h5>
            </td>
            <td>
            <h5 class="label-4">
                ${expense.expenseId}<img class="clip-b-img"
                src="<c:url value="/resources/images/clip-b.png" />">
            </h5>
            </td>
            <td>
            <h5 class="label-4">${expense.description}</h5>
            </td>
            <td>
            <h5 class="label-4">
                <c:if test="${expense.state eq 'NEW' }">
                <form:form name="actionForm" id="actionForm" method="put">
                <select id="action_${rowCounter.count}"
                    name="${rowCounter.count}" style="font-size: 11px;"
                    onchange="changeState('${pageContext.request.contextPath}','${expense.expenseId}','${rowCounter.count}')">
                    <option value="PENDING">Pending</option>
                    <option value="APPROVED">Approve</option>
                    <option value="REJECTED">Reject</option>
                </select>
                </form:form>
                </c:if>
                <c:if test="${expense.state eq 'REJECTED' }">
                <i class="alt-a label-5 red"><img class="cross-a-img"
                src="<c:url value="/resources/images/cross-a.png" />">
                Rejected</i>
                <i class="alt-b label-6"> <img class="right-b-img"
                src="<c:url value="/resources/images/cross-a.png" />">
                Rejected <a
                href="${pageContext.request.contextPath}/expensereports/${expense.expenseId}"
                class="bg-cross-b"><img class="cross-b-img"
                    src="<c:url value="/resources/images/cross-b.png" />">
                </a>
                </i>
                </c:if>
            </h5>
            </td>
            </tr>
        </c:forEach>
        </tbody>
        </table>
        <div id="pager" class="pagination">
        <form>
        <img src="<c:url value="/resources/images/first.png"/>"
            class="first" /> <img
            src="<c:url value="/resources/images/prev.png"/>" class="prev" />
        <input type="text" class="pagedisplay" /> <img
            src="<c:url value="/resources/images/next1.png" />" class="next" />
        <img src="<c:url value="/resources/images/last.png" />"
            class="last" /> <select class="pagesize">
            <option selected="selected" value="10">10</option>
            <option value="20">20</option>
            <option value="30">30</option>
            <option value="40">40</option>
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
</div>
</body>
</html>
```
add the below jsp file as `myexpense.jsp`

```jsp
<!DOCTYPE HTML>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
 <%@ taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn" %>
 <%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<html>
    <head>
    <title>My Expense</title>
    <link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
    <script src="<c:url value="resources/javascript/jquery-1.8.1.js"/>" ></script>
    <script src="<c:url value="resources/javascript/jquery.tablesorter.js"/>" ></script>
    <script src="<c:url value="resources/javascript/jquery.tablesorter.min.js"/>" ></script>
    <script src="<c:url value="resources/javascript/jquery.tablesorter.pager.js"/>" ></script>
    <script type="text/javascript">
      function deleteexpense(contextPath, formid, expenseId) {
       var url = contextPath + "/expensereports/"+expenseId;
       var formno = parseInt(formid) - 1;
       document.forms[formno].action = url;
       document.forms[formno].submit();
      }
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
          </div>
          <div id="nav">
        <ul>
        <li>
        <a  href="${pageContext.request.contextPath}/expensereports">My Expenses</a>
        </li>
        <li>
        <a  href="${pageContext.request.contextPath}/expensereports/new">New Expense</a>
        </li>
        <c:if test="${sessionScope.user.role.roleName eq 'ROLE_MANAGER'}">
        <li>
            <a href="${pageContext.request.contextPath}/expensereports/approvals">Approvals</a>
        </li>
           </c:if>
        </ul>
        </div>
    <div class="mc-a">
    <h2 class="header label-2">
        Expenses
    </h2>
    <c:if test="${fn:length(pendingExpenseList) gt 0 }">
    <table  id="table-a" class="table-a mtop80">
        <thead>
        <tr>
        <td class="td-a">
        <h5 class="label-3">Attachment <img class="clip-b-img" src="<c:url value="/resources/images/clip-b.png" />"></h5>
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
        <tbody>
        <c:forEach var="expense" items="${pendingExpenseList}">
        <tr class="tr-odd">
        <td>
            <c:if test="${not empty expense.attachment.fileName}">
             <a href="${pageContext.request.contextPath}/attachments/${expense.attachment.attachmentId}"><img class="clip-b-img" src="<c:url value="/resources/images/clip-b.png" />"></a>
            </c:if>
        </td>
        <td>
            <h5 class="label-4">${expense.expenseDate}</h5>
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
            <img class="question-a-img" src="<c:url value="/resources/images/question-a.png" />" > Pending
            <a href="${pageContext.request.contextPath}/expensereports/${expense.expenseId}" class="bg-edit-b">
                 <img class="clip-b-img" src="<c:url value="/resources/images/edit-a.png" />"> </a>
             <form:form id="deleteform1" name="deleteform1" method="delete" class="bg-cross-b">
               <a onclick="deleteexpense('${pageContext.request.contextPath}',1,'${expense.expenseId}')">
                 <img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />"> </a>
              </form:form>
            </i>
            </c:if>
            <c:if test="${expense.state eq 'APPROVED' }">
            <i class="alt-a label-5 green">
            <img class="right-a-img" src="<c:url value="/resources/images/right-a.png" /> ">
             Approved</i>
            <i class="alt-b label-6">
            <img class="right-b-img" src="<c:url value="/resources/images/right-b.png" />" > Approved
            <a href="${pageContext.request.contextPath}/expensereports/${expense.expenseId}" class="bg-edit-b">
              <img class="clip-b-img" src="<c:url value="/resources/images/edit-a.png" />"> </a>
            <form:form method="delete" name="deleteform2" id="deleteform2" class="bg-cross-b">
            <a  onclick="deleteexpense('${pageContext.request.contextPath}',2,'${expense.expenseId}')">
              <img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />"> </a>
            </form:form>
            </i>
            </c:if>
            <c:if test="${expense.state eq 'REJECTED' }">
             <i class="alt-a label-5 red"><img class="cross-a-img" src="<c:url value="/resources/images/cross-a.png" />" > Rejected</i>
            <i class="alt-b label-6">
            <img class="right-b-img" src="<c:url value="/resources/images/cross-a.png" />" > Rejected
            <a href="${pageContext.request.contextPath}/expensereports/${expense.expenseId}" class="bg-edit-b">
               <img class="cclip-b-img" src="<c:url value="/resources/images/edit-a.png" />"> </a>
             <form:form id="deleteform3" name="deleteform3" method="delete" class="bg-cross-b">
              <a  onclick="deleteexpense('${pageContext.request.contextPath}',3,'${expense.expenseId}')">
               <img class="cross-b-img" src="<c:url value="/resources/images/cross-b.png" />" > </a>
             </form:form>
            </i>
            </c:if>
            </h5>
        </td>
        </tr>
        </c:forEach>
        </tbody>
    </table>
    <div id="pager" class="pagination">
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
</div>
</body>
</html>
```
add the below jsp file as `newexpense.jsp`
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>


<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<title>New Expense</title>
<link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
<script
    src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
<script src="http://code.jquery.com/jquery-latest.js"></script>
<script type="text/javascript"
    src="http://jzaefferer.github.com/jquery-validation/jquery.validate.js"></script>
<style type="text/css">
* {
    font-family: Verdana;
    font-size: 96%;
}

label {
    width: 10em;
    float: left;
}

label.error {
    float: none;
    color: red;
    vertical-align: bottom;
    padding: 100px 0px 0px 0px;
    display: inline
}

p {
    clear: both;
}

.submit {
    margin-left: 12em;
}

em {
    font-weight: bold;
    padding-right: 1em;
    vertical-align: top;
}
</style>
<script>
    $(document).ready(function() {
    $("#newexpenseForm").validate({
    rules : {
        description : "required",
        amount : "required digits",
        expenseTypeId : "required",
    },
    messages : {
        description : "Please enter description",
        amount : "Please enter amount",
        expenseTypeId : "Please select expense type",
    }
    });
    $("#editExpenseForm").validate({
    rules : {
        description : "required",
        amount : "required digits",
        expenseTypeId : "required",
    },
    messages : {
        description : "Please enter description",
        amount : "Please enter amount",
        expenseTypeId : "Please select expense type",
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
        <li><a href="#">Hi ${sessionScope.user.userName}</a></li>
        <li>|</li>
        <li><a href="<c:url value="/j_spring_security_logout" />">Logout</a>
        </li>
        </ul>
    </div>
    </div>
    <div id="nav">
    <ul>
        <li><a href="${pageContext.request.contextPath}/expensereports">My
        Expenses</a></li>
        <li><a
        href="${pageContext.request.contextPath}/expensereports/new">New
        Expense</a></li>
        <c:if test="${sessionScope.user.role.roleName eq 'ROLE_MANAGER'}">
        <li><a
        href="${pageContext.request.contextPath}/expensereports/approvals">Approvals</a>
        </li>
        </c:if>
    </ul>
    </div>
    <c:choose>
    <c:when test="${isEdit eq 'true'}">
        <div class="mc-a">
        <form:form action="${pageContext.request.contextPath}/expensereports"
        class="mtop80" method="put" enctype="multipart/form-data"
        id="editExpenseForm">
        <h2 class="header label-2 mbot20">Editing an expense</h2>
        <div class="action-a">
            <label class="label-name" for="expense_description">Description</label>
            <textarea class="textbox-a" cols="40" id="description"
            name="description" placeholder="description" rows="20"
            required="required">${expense.description}</textarea>
        </div>
        <input type="hidden" name="expenseId" id="expenseId"
            value="${expense.expenseId}" />
        <div class="action-a">
            <label class="label-name" for="expense_amount">Amount</label> <input
            class="textbox-a dd-b flt-left mleft0" id="amount" name="amount"
            placeholder="Amount" size="30" type="text"
            required="required digits" name="amount" max="100000"
            value="${expense.amount}"> <a class="label-7 flt-right"
            href="#" id="attachment-holder"><img class="clip-a-img"
            src="<c:url value="/resources/images/clip-a.png" />"><span
            id="attachment-name">Attach Receipt</span> </a> <input id="file"
            name="file" value="${expense.attachment}" style="display: none;"
            type="file">
        </div>
        <div class="action-a">
            <label class="label-name" for="expenseTypeId">Expense
            type</label> <select id="expenseTypeId" name="expenseTypeId"
            required="required" class="textbox-a dd-a mleft0">
            <c:forEach var="expenseType" items="${expenseTypeList}">
            <c:if test="${expense.expenseType.name eq expenseType.name }">
                <option value="${expenseType.expenseTypeId}" selected="selected">${expenseType.name}</option>
            </c:if>
            <c:if test="${expense.expenseType.name ne expenseType.name }">
                <option value="${expenseType.expenseTypeId}">${expenseType.name}</option>
            </c:if>
            </c:forEach>
            </select><br>
        </div>
        <button class="btn-a add-expense-btn">SAVE EXPENSE</button>
        </form:form>
        </div>
    </c:when>
    <c:otherwise>
        <div class="mc-a">
        <form
        action="${pageContext.request.contextPath}/expensereports"
        class="mtop80" method="post" enctype="multipart/form-data"
        id="newexpenseForm">
        <h2 class="header label-2 mbot20">Adding new expense</h2>
        <div class="action-a">
            <label class="label-name" for="expense_description">Description</label>
            <textarea class="textbox-a" cols="40" id="description"
            name="description" placeholder="Description" rows="20"
            required="required"></textarea>
        </div>
        <div class="action-a">
            <label class="label-name" for="expense_amount">Amount</label> <input
            class="textbox-a dd-b flt-left mleft0" id="amount" name="amount"
            placeholder="Amount" size="30" type="text"
            required="required digits" name="amount" max="100000">

            <a class="label-7 flt-right" href="#" id="attachment-holder"><img
            class="clip-a-img"
            src="<c:url value="/resources/images/clip-a.png" />"><span
            id="attachment-name">Attach Receipt</span> </a> <input id="file"
            name="file" style="display: none;" type="file">
        </div>
        <div class="action-a">
            <label class="label-name" for="expenseTypeId">Expense
            type</label> <select id="expenseTypeId" name="expenseTypeId"
            class="textbox-a dd-a mleft0">
            <option value=""></option>
            <c:forEach var="expenseType" items="${expenseTypeList}">
            <option value="${expenseType.expenseTypeId}">${expenseType.name}
            </option>
            </c:forEach>
            </select><br>
        </div>
        <button class="btn-a add-expense-btn">ADD EXPENSE</button>
        </form>
        <script type="text/javascript">
        $(document)
            .ready(
                function() {
                $('#attachment-holder').live(
                    'click',
                    function() {
                    $('#file').trigger(
                        'click');
                    });

                $("input[type='file']")
                    .change(
                    function() {
                        if ($(this)
                        .val() == "")
                        $(
                            '#attachment-name')
                            .html(
                            "Attach Receipt");
                        else
                        $(
                            '#attachment-name')
                            .html(
                            $(
                                this)
                                .val()
                                .replace(
                                    /^.*\\/g,
                                    ''));
                    });
                })
        </script>
        </div>
    </c:otherwise>
    </c:choose>
    </div>
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
</html>
```
