---
title: View files for Expense Reporting App using MongoDB
description: View files for Expense Reporting App using MongoDB
---
Add the below jsp file as `expensereports.jsp`

```jsp
<%@ page session="false"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!doctype html>
<html>
<head>
<script src="http://code.jquery.com/jquery-latest.js"></script>
<script src="<c:url value="/resources/javascript/expensereport.js" />"></script>
<script
    src="<c:url value="resources/javascript/jquery.tablesorter.js"/>"></script>
<script src="<c:url value="resources/javascript/jquery.pagination.js"/>"></script>
<link rel="stylesheet" href="<c:url value="/resources/css/style.css" />" />
</head>
<body>
    <div id="contentWrapper">
        <div class="blue-b">
            <div class="inside-nav">
                <div class="logo">
                    <img src="<c:url value="/resources/images/logo.png" />" />
                </div>
            </div>
        </div>
        <div id="nav">
            <ul>
                <li><a id="myExpenseTab" class="tabLinks" href="#"
                    data-tab="myExpenseContainer">My Expenses</a></li>
                <li><a class="tabLinks" href="#" data-tab="newExpenseContainer">New
                        Expense</a></li>
                <c:if test="${user.role.roleName eq 'ROLE_MANAGER'}">
                    <li><a id="approvalTab" href="#" class="tabLinks"
                        data-tab="approvalsContainer">Approvals</a></li>
                </c:if>
            </ul>
        </div>
        <div class="masterContainer">
            <div id="newExpenseContainer" class="mc-a tabContainer">
                <br>
                <br>
                <br>
                <h2 class="header label-2 mbot20">Adding new expense</h2>
                <div class="action-a">
                    <label class="label-name" for="expense_description">Description</label>
                    <textarea class="textbox-a" placeholder="description"
                        required="required"></textarea>
                </div>
                <input type="hidden" id="expenseload" />
                <div class="action-a">
                    <label class="label-name" for="expense_amount">Amount</label> <input
                        class="textbox-a dd-b flt-left mleft0" id="amount" name="amount"
                        required="required" placeholder="Amount" size="30" type="text">
                </div>
                <div class="action-a">
                    <label class="label-name" for="expenseTypeId">Expense Type</label>
                    <div class="textbox-a dd-a mleft0">
                        <select id="expenseTypeId" name="expenseTypeId"
                            required="required">
                            <option value=""></option>
                            <c:forEach var="expenseType" items="${expenseTypeList}">
                                <option value="${expenseType.expenseTypeId}">${expenseType.name}</option>
                            </c:forEach>
                        </select>
                    </div>
                </div>
                <button class="btn-a add-expense-btn">ADD EXPENSE</button>
                <button class="btn-a edit-expense-btn">UPDATE EXPENSE</button>
            </div>
            <div id="myExpenseContainer" class="myexpense tabContainer active">
                <table id="expenseTable" class="table-a mtop80">
                    <thead>
                        <tr>
                            <td class="td-a">
                                <h5 class="label-3">Expense Id</h5>
                            </td>
                            <td class="td-b">
                                <h5 class="label-3">Date</h5>
                            </td>
                            <td class="td-c">
                                <h5 class="label-3">Expense Type</h5>
                            </td>
                            <td class="td-d">
                                <h5 class="label-3">Description</h5>
                            </td>
                            <td class="td-d">
                                <h5 class="label-3">Status</h5>
                            </td>
                        </tr>
                    </thead>
                    <tbody>
                    </tbody>
                </table>
                <div id="Pagination" class="pagination"></div>
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
    <input type="text" id="contextPath"
        value="${pageContext.request.contextPath}" />
</body>
</html>
```
