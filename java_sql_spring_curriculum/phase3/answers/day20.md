---
layout: default
title: "Day 20: View（JSP）層の実装と総合演習 - 解答例"
---
# Day 20: View（JSP）層の実装と総合演習 - 解答例

## 課題1: ログイン画面の実装

**login/loginForm.jsp**

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>

<h2><spring:message code="label.login.title" /></h2>

<c:if test="${param.error != null}">
    <div class="alert alert-danger">
        <spring:message code="label.login.error" />
    </div>
</c:if>
<c:if test="${param.logout != null}">
    <div class="alert alert-success">
        <spring:message code="label.logout.success" />
    </div>
</c:if>

<form:form action="${pageContext.request.contextPath}/login" method="post" modelAttribute="loginForm">
    <div>
        <label for="username"><spring:message code="label.user.username" />:</label>
        <form:input path="username" id="username" required="true" />
        <form:errors path="username" cssClass="error" />
    </div>
    <div>
        <label for="password"><spring:message code="label.user.password" />:</label>
        <form:password path="password" id="password" required="true" />
        <form:errors path="password" cssClass="error" />
    </div>
    <div>
        <form:checkbox path="rememberMe" id="remember-me" />
        <label for="remember-me"><spring:message code="label.login.rememberMe" /></label>
    </div>
    <div>
        <button type="submit"><spring:message code="label.common.login" /></button>
    </div>
</form:form>

<p><a href="<c:url value='/user/register'/>"><spring:message code="label.user.register.link" /></a></p>
```

## 課題2: ToDo一覧画面の実装

**todo/list.jsp**

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>

<h2><spring:message code="label.todo.list.title" /></h2>

<%-- 検索フォーム --%>
<form:form modelAttribute="todoSearchForm" method="get" action="${pageContext.request.contextPath}/todo/list">
    <form:input path="title" placeholder="Search by title..." />
    <form:select path="completed">
        <form:option value="" label="-- Status --" />
        <form:option value="false" label="Incomplete" />
        <form:option value="true" label="Completed" />
    </form:select>
    <button type="submit"><spring:message code="label.common.search" /></button>
</form:form>

<a href="<c:url value='/todo/create'/>"><spring:message code="label.todo.create.link" /></a>

<c:if test="${not empty page}">
    <table>
        <thead>
            <tr>
                <th><spring:message code="label.todo.title" /></th>
                <th><spring:message code="label.todo.description" /></th>
                <th><spring:message code="label.todo.completed" /></th>
                <th><spring:message code="label.todo.createdAt" /></th>
                <th><spring:message code="label.todo.updatedAt" /></th>
                <th><spring:message code="label.common.actions" /></th>
            </tr>
        </thead>
        <tbody>
            <c:forEach items="${page.content}" var="todo">
                <tr>
                    <td><c:out value="${todo.title}" /></td>
                    <td><c:out value="${todo.description}" /></td>
                    <td>
                        <form:form action="${pageContext.request.contextPath}/todo/toggle/${todo.id}" method="post">
                            <input type="checkbox" ${todo.completed ? 'checked' : ''} onchange="this.form.submit()">
                        </form:form>
                    </td>
                    <td><c:out value="${todo.createdAt}" /></td>
                    <td><c:out value="${todo.updatedAt}" /></td>
                    <td>
                        <a href="<c:url value='/todo/update/${todo.id}'/>"><spring:message code="label.common.edit" /></a>
                        <form:form action="${pageContext.request.contextPath}/todo/delete/${todo.id}" method="post" style="display:inline;">
                            <button type="submit" onclick="return confirm('Are you sure you want to delete this todo?');"><spring:message code="label.common.delete" /></button>
                        </form:form>
                    </td>
                </tr>
            </c:forEach>
        </tbody>
    </table>

    <%-- ページネーション --%>
    <nav>
        <ul class="pagination">
            <c:if test="${page.hasPrevious}">
                <li><a href="<c:url value='/todo/list?page=${page.number - 1}&size=${page.size}&title=${param.title}&completed=${param.completed}'/>">&laquo;</a></li>
            </c:if>
            <c:forEach begin="0" end="${page.totalPages - 1}" varStatus="loop">
                <li class="${page.number == loop.index ? 'active' : ''}">
                    <a href="<c:url value='/todo/list?page=${loop.index}&size=${page.size}&title=${param.title}&completed=${param.completed}'/>">${loop.index + 1}</a>
                </li>
            </c:forEach>
            <c:if test="${page.hasNext}">
                <li><a href="<c:url value='/todo/list?page=${page.number + 1}&size=${page.size}&title=${param.title}&completed=${param.completed}'/>">&raquo;</a></li>
            </c:if>
        </ul>
    </nav>
</c:if>

<c:if test="${empty page or empty page.content}">
    <p><spring:message code="label.todo.list.empty" /></p>
</c:if>
```

## 課題3: ToDo登録/更新画面の実装

**todo/form.jsp**

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>

<c:choose>
    <c:when test="${todoForm.id == null}">
        <h2><spring:message code="label.todo.create.title" /></h2>
        <c:set var="formAction" value="${pageContext.request.contextPath}/todo/create" />
    </c:when>
    <c:otherwise>
        <h2><spring:message code="label.todo.update.title" /></h2>
        <c:set var="formAction" value="${pageContext.request.contextPath}/todo/update/${todoForm.id}" />
    </c:otherwise>
</c:choose>

<form:form modelAttribute="todoForm" method="post" action="${formAction}">
    <%-- 更新の場合、IDをhiddenで保持 --%>
    <c:if test="${todoForm.id != null}">
        <form:hidden path="id" />
    </c:if>

    <%-- グローバルエラー表示 --%>
    <form:errors path="*" cssClass="error-global" element="div" />

    <div>
        <label for="title"><spring:message code="label.todo.title" />:</label>
        <form:input path="title" id="title" required="true" />
        <form:errors path="title" cssClass="error" />
    </div>
    <div>
        <label for="description"><spring:message code="label.todo.description" />:</label>
        <form:textarea path="description" id="description" rows="5" />
        <form:errors path="description" cssClass="error" />
    </div>
    <div>
        <form:checkbox path="completed" id="completed" />
        <label for="completed"><spring:message code="label.todo.completed" /></label>
        <form:errors path="completed" cssClass="error" />
    </div>
    <div>
        <button type="submit">
            <c:choose>
                <c:when test="${todoForm.id == null}"><spring:message code="label.common.create" /></c:when>
                <c:otherwise><spring:message code="label.common.update" /></c:otherwise>
            </c:choose>
        </button>
        <a href="<c:url value='/todo/list'/>"><spring:message code="label.common.cancel" /></a>
    </div>
</form:form>
```

## 課題4: ユーザー登録画面の実装

**user/registerForm.jsp**

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>

<h2><spring:message code="label.user.register.title" /></h2>

<form:form modelAttribute="userForm" method="post" action="${pageContext.request.contextPath}/user/register">
    <form:errors path="*" cssClass="error-global" element="div" />

    <div>
        <label for="username"><spring:message code="label.user.username" />:</label>
        <form:input path="username" id="username" required="true" />
        <form:errors path="username" cssClass="error" />
    </div>
    <div>
        <label for="password"><spring:message code="label.user.password" />:</label>
        <form:password path="password" id="password" required="true" />
        <form:errors path="password" cssClass="error" />
    </div>
    <div>
        <label for="confirmPassword"><spring:message code="label.user.confirmPassword" />:</label>
        <form:password path="confirmPassword" id="confirmPassword" required="true" />
        <form:errors path="confirmPassword" cssClass="error" />
    </div>
    <div>
        <label for="email"><spring:message code="label.user.email" />:</label>
        <form:input path="email" id="email" type="email" />
        <form:errors path="email" cssClass="error" />
    </div>
    <div>
        <button type="submit"><spring:message code="label.common.register" /></button>
        <a href="<c:url value='/login'/>"><spring:message code="label.common.cancel" /></a>
    </div>
</form:form>
```

## 課題5: 国際化対応

**messages.properties (英語)**

```properties
# Common
label.common.login=Login
label.common.logout=Logout
label.common.register=Register
label.common.create=Create
label.common.update=Update
label.common.delete=Delete
label.common.edit=Edit
label.common.cancel=Cancel
label.common.search=Search
label.common.actions=Actions

# Application
label.todo.app.title=ToDo Application

# Login
label.login.title=Login
label.login.error=Invalid username or password.
label.login.rememberMe=Remember me
label.logout.success=You have been logged out successfully.

# User
label.user.username=Username
label.user.password=Password
label.user.confirmPassword=Confirm Password
label.user.email=Email
label.user.register.title=Register
label.user.register.link=Register a new account

# Todo
label.todo.title=Title
label.todo.description=Description
label.todo.completed=Completed
label.todo.createdAt=Created At
label.todo.updatedAt=Updated At
label.todo.completedAt=Completed At
label.todo.list.title=ToDo List
label.todo.list.empty=No todos found.
label.todo.create.title=Create ToDo
label.todo.create.link=Create New ToDo
label.todo.update.title=Update ToDo

# Validation
NotBlank.todoForm.title=Title is required.
Size.todoForm.title=Title must be less than or equal to {1} characters.
Size.todoForm.description=Description must be less than or equal to {1} characters.

NotBlank.userForm.username=Username is required.
Size.userForm.username=Username must be between {2} and {1} characters.
Pattern.userForm.username=Username must contain only letters, numbers, and underscores.
NotBlank.userForm.password=Password is required.
Size.userForm.password=Password must be between {2} and {1} characters.
Email.userForm.email=Email must be a valid email address.
```

**messages_ja.properties (日本語)**

```properties
# Common
label.common.login=ログイン
label.common.logout=ログアウト
label.common.register=登録
label.common.create=作成
label.common.update=更新
label.common.delete=削除
label.common.edit=編集
label.common.cancel=キャンセル
label.common.search=検索
label.common.actions=操作

# Application
label.todo.app.title=ToDoアプリケーション

# Login
label.login.title=ログイン
label.login.error=ユーザー名またはパスワードが無効です。
label.login.rememberMe=ログイン状態を保持する
label.logout.success=ログアウトしました。

# User
label.user.username=ユーザー名
label.user.password=パスワード
label.user.confirmPassword=パスワード（確認）
label.user.email=メールアドレス
label.user.register.title=ユーザー登録
label.user.register.link=新規アカウント登録

# Todo
label.todo.title=タイトル
label.todo.description=説明
label.todo.completed=完了
label.todo.createdAt=作成日時
label.todo.updatedAt=更新日時
label.todo.completedAt=完了日時
label.todo.list.title=ToDo一覧
label.todo.list.empty=ToDoが見つかりません。
label.todo.create.title=ToDo作成
label.todo.create.link=新規ToDo作成
label.todo.update.title=ToDo更新

# Validation
NotBlank.todoForm.title=タイトルは必須です。
Size.todoForm.title=タイトルは{1}文字以内で入力してください。
Size.todoForm.description=説明は{1}文字以内で入力してください。

NotBlank.userForm.username=ユーザー名は必須です。
Size.userForm.username=ユーザー名は{2}文字以上{1}文字以内で入力してください。
Pattern.userForm.username=ユーザー名は英数字とアンダースコアのみ使用できます。
NotBlank.userForm.password=パスワードは必須です。
Size.userForm.password=パスワードは{2}文字以上{1}文字以内で入力してください。
Email.userForm.email=有効なメールアドレスを入力してください。
```

### 解説

課題1～5の解答例では、TERASOLUNAフレームワークの推奨パターンに基づいて、View（JSP）層の実装を行いました。

1. **ログイン画面の実装**
   - Spring Securityと連携したログインフォーム
   - エラーメッセージの表示
   - Remember Me機能の実装

2. **ToDo一覧画面の実装**
   - 検索フォームの実装
   - ToDoリストの表示
   - 完了状態の切り替え
   - 編集・削除機能の実装
   - ページネーションの実装

3. **ToDo登録/更新画面の実装**
   - 登録と更新で共通のフォームを使用
   - 画面タイトルとボタンラベルの切り替え
   - 入力検証エラーの表示
   - キャンセル機能の実装

4. **ユーザー登録画面の実装**
   - ユーザー登録フォームの実装
   - パスワード確認フィールドの実装
   - 入力検証エラーの表示
   - キャンセル機能の実装

5. **国際化対応**
   - 英語と日本語のメッセージファイルの作成
   - 画面表示メッセージの国際化
   - 入力検証エラーメッセージの国際化

これらの実装により、ユーザーフレンドリーなWebアプリケーションを構築することができます。特に、Spring Form Tagsを活用することで、Formオブジェクトとの連携や入力検証エラーの表示が容易になります。また、JSTLを活用することで、条件分岐やループ処理などの動的な表示制御を行うことができます。国際化対応により、多言語対応のアプリケーションを構築することができます。

JSPの実装では、HTMLとJavaコードの分離を意識し、スクリプトレットの使用を避け、ELやJSTL、Spring Tagsを活用することで、可読性と保守性の高いコードを実現しています。また、共通レイアウトを活用することで、コードの重複を避け、一貫性のあるユーザーインターフェースを提供しています。
