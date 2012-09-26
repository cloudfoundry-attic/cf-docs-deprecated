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
add the below jsp file as `myexpense.jsp`

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
add the below jsp file as `newexpense.jsp`
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
            <!--     <div class="textbox-a dd-a flt-left"> -->
                <select id="expenseTypeId" name="expenseTypeId" style="    border: 1px solid #b8b8b8;
                box-shadow: 0px 2px 0px 0px #b8b8b8;padding: 12px;font-size: 11px;color: #a7a5a6;width:100px;    margin:10px;">
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
add the below jsp file as `usermain.jsp`
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
add the below jsp file as `signUp.jsp`
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
