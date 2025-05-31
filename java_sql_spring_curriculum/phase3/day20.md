---
layout: default
title: "Day 20: View（JSP）層の実装と総合演習"
---
# Day 20: View（JSP）層の実装と総合演習

## 学習の目的と背景

Webアプリケーション開発において、View層はユーザーインターフェースを担当し、ユーザーとの対話を実現する重要な要素です。TERASOLUNAフレームワークでは、JSP（JavaServer Pages）を主要なビュー技術として利用し、JSTL（JSP Standard Tag Library）やSpring Frameworkが提供するカスタムタグライブラリを組み合わせて、動的で保守性の高い画面を構築します。

本日の学習では、これまでのフェーズで実装してきたToDoアプリケーションのView層をJSPを用いて実装します。ログイン画面、ToDo一覧画面、ToDo登録・更新画面などを実際に作成することで、JSPの基本構文、JSTLやSpringタグライブラリの効果的な使い方、Formオブジェクトとの連携、エラーメッセージの表示方法などを習得します。最終日として、アプリケーション全体の動作確認と、これまでの学習内容の総括を行います。

## 学習内容の解説

### 1. JSP（JavaServer Pages）の基本

#### JSPとは
JSPは、HTML内にJavaコードを埋め込むことで、動的なWebページを生成するための技術です。サーバーサイドで実行され、最終的にHTMLとしてクライアントに返されます。

#### JSPの構成要素
- **ディレクティブ (`<%@ ... %>`)**: ページ全体の設定を行います。（例: `page`, `include`, `taglib`）
- **スクリプトレット (`<% ... %>`)**: Javaコードを記述します。（現在は非推奨）
- **宣言 (`<%! ... %>`)**: メソッドや変数を宣言します。（現在は非推奨）
- **式 (`<%= ... %>`)**: Javaの式の結果を出力します。
- **コメント (`<%-- ... --%>`)**: JSPコメント。クライアントには送信されません。
- **アクションタグ (`<jsp:...>`)**: JSPが提供する標準的な機能を利用します。（例: `<jsp:include>`, `<jsp:forward>`, `<jsp:useBean>`）

#### EL（Expression Language）
ELは、JSPページ内でJavaBeansのプロパティやリクエストパラメータなどに簡単にアクセスするための式言語です。`${...}` の形式で記述します。
例: `${todo.title}` は、`todo`オブジェクトの`getTitle()`メソッドを呼び出して結果を出力します。

### 2. JSTL（JSP Standard Tag Library）

JSTLは、JSP開発でよく使われる機能をカスタムタグとして提供するライブラリです。スクリプトレットの使用を減らし、コードの可読性と保守性を向上させます。

#### JSTLの主要なライブラリ
- **Core (`c`)**: 変数操作、フロー制御（if, forEach）、URL操作など。
  - `<c:set>`, `<c:if>`, `<c:forEach>`, `<c:url>`, `<c:out>`
- **Formatting (`fmt`)**: 数値、日付、時刻のフォーマット、国際化対応。
  - `<fmt:formatNumber>`, `<fmt:formatDate>`, `<fmt:message>`
- **Functions (`fn`)**: 文字列操作などの便利な関数。
  - `fn:length()`, `fn:substring()`, `fn:escapeXml()`

#### JSTLの利用準備
JSPページの先頭で `taglib` ディレクティブを使ってライブラリを宣言します。
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
```

### 3. Spring Frameworkのカスタムタグ

Spring Frameworkも、フォーム処理やメッセージ表示などを容易にするためのカスタムタグライブラリを提供しています。

#### Spring Form Tags (`form`)
フォームの生成、データバインディング、エラー表示などを簡潔に記述できます。
- `<form:form>`: HTMLの `<form>` タグを生成し、モデルオブジェクト（Formオブジェクト）と関連付けます。
- `<form:input>`, `<form:password>`, `<form:textarea>`, `<form:checkbox>`, `<form:radiobutton>`, `<form:select>`: 各種フォーム部品を生成します。`path`属性でFormオブジェクトのプロパティと紐付けます。
- `<form:errors>`: 特定のフィールドまたはフォーム全体のエラーメッセージを表示します。

#### Spring Tags (`spring`)
メッセージの表示やURLの生成などをサポートします。
- `<spring:message>`: 国際化メッセージを取得して表示します。
- `<spring:url>`: URLを生成します。

#### Spring Security Tags (`sec`)
認証情報へのアクセスや認可制御（表示要素の制御）を行います。
- `<sec:authentication>`: 認証済みユーザーの情報を取得します。
- `<sec:authorize>`: ユーザーの権限に基づいてコンテンツの表示を制御します。

#### 利用準備
JSPページの先頭で `taglib` ディレクティブを使ってライブラリを宣言します。
```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
```

### 4. ToDoアプリケーションのView（JSP）実装

TERASOLUNAの標準的な構成に従い、`src/main/webapp/WEB-INF/views/` 以下にJSPファイルを配置します。

#### 共通レイアウト (`layout/template.jsp`)
多くのページで共通となるヘッダー、フッター、ナビゲーションなどを定義します。Apache Tilesなどのレイアウトフレームワークを利用することも一般的ですが、ここでは簡単な共通インクルードで示します。

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title><spring:message code="label.todo.app.title" /></title>
    <link rel="stylesheet" href="<c:url value='/resources/css/style.css'/>">
    <%-- 他のCSSやJavaScriptの読み込み --%>
</head>
<body>
    <header>
        <h1><spring:message code="label.todo.app.title" /></h1>
        <nav>
            <sec:authorize access="isAuthenticated()">
                <span>Welcome, <sec:authentication property="principal.user.username"/></span>
                <form:form action="@{/logout}" method="post">
                    <button type="submit"><spring:message code="label.common.logout" /></button>
                </form:form>
            </sec:authorize>
            <sec:authorize access="!isAuthenticated()">
                <a href="<c:url value='/login'/>"><spring:message code="label.common.login" /></a>
                <a href="<c:url value='/user/register'/>"><spring:message code="label.user.register.title" /></a>
            </sec:authorize>
        </nav>
    </header>

    <main>
        <%-- 各画面固有のコンテンツがここに挿入される --%>
        <jsp:include page="${content}.jsp" />
    </main>

    <footer>
        <p>&copy; 2025 ToDo Application</p>
    </footer>
</body>
</html>
```

#### ログイン画面 (`login/loginForm.jsp`)

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

<form:form action="@{/login}" method="post" modelAttribute="loginForm">
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

#### ToDo一覧画面 (`todo/list.jsp`)

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>

<h2><spring:message code="label.todo.list.title" /></h2>

<%-- 検索フォーム --%>
<form:form modelAttribute="todoSearchForm" method="get" action="@{/todo/list}">
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
                        <form:form action="@{/todo/toggle/{id}(id=${todo.id})}" method="post">
                            <input type="checkbox" ${todo.completed ? 'checked' : ''} onchange="this.form.submit()">
                        </form:form>
                    </td>
                    <td><c:out value="${todo.createdAt}" /></td>
                    <td><c:out value="${todo.updatedAt}" /></td>
                    <td>
                        <a href="<c:url value='/todo/update/${todo.id}'/>"><spring:message code="label.common.edit" /></a>
                        <form:form action="@{/todo/delete/{id}(id=${todo.id})}" method="post" style="display:inline;">
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
            <c:if test="${page.hasPrevious()}">
                <li><a href="<c:url value='/todo/list?page=${page.number - 1}&size=${page.size}&title=${param.title}&completed=${param.completed}'/>">&laquo;</a></li>
            </c:if>
            <c:forEach begin="0" end="${page.totalPages - 1}" varStatus="loop">
                <li class="${page.number == loop.index ? 'active' : ''}">
                    <a href="<c:url value='/todo/list?page=${loop.index}&size=${page.size}&title=${param.title}&completed=${param.completed}'/>">${loop.index + 1}</a>
                </li>
            </c:forEach>
            <c:if test="${page.hasNext()}">
                <li><a href="<c:url value='/todo/list?page=${page.number + 1}&size=${page.size}&title=${param.title}&completed=${param.completed}'/>">&raquo;</a></li>
            </c:if>
        </ul>
    </nav>
</c:if>

<c:if test="${empty page or empty page.content}">
    <p><spring:message code="label.todo.list.empty" /></p>
</c:if>
```

#### ToDo登録/更新画面 (`todo/form.jsp`)
登録と更新で共通のフォームを使用します。

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>

<c:choose>
    <c:when test="${todoForm.id == null}">
        <h2><spring:message code="label.todo.create.title" /></h2>
        <c:set var="formAction" value="@{/todo/create}" />
    </c:when>
    <c:otherwise>
        <h2><spring:message code="label.todo.update.title" /></h2>
        <c:set var="formAction" value="@{/todo/update/{id}(id=${todoForm.id})}" />
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

#### ユーザー登録画面 (`user/registerForm.jsp`)

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>

<h2><spring:message code="label.user.register.title" /></h2>

<form:form modelAttribute="userForm" method="post" action="@{/user/register}">
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

### 5. エラーメッセージの表示と国際化

- **フィールドエラー**: `<form:errors path="fieldName" />` で表示。
- **グローバルエラー**: `<form:errors path="*" />` で表示。
- **メッセージ**: `ValidationMessages.properties` や `messages.properties` に定義し、`<spring:message code="..." />` やBean Validationのアノテーションの `message` 属性で参照します。

### 6. 環境構築と総合確認

Day 11-13の内容に基づき、開発環境（JDK, Maven/Gradle, PostgreSQL, IDE）が正しくセットアップされていることを確認します。
TERASOLUNAのブランクプロジェクトから始め、これまで作成したController, Service, Repository, Domain, Form, DTO, JSPを組み込み、アプリケーション全体をビルド・デプロイ（Tomcat等のアプリケーションサーバーを使用）して動作確認を行います。

- ユーザー登録ができるか？
- ログイン/ログアウトができるか？
- ToDoの登録、一覧表示、更新、削除ができるか？
- 完了/未完了の切り替えができるか？
- 検索、ページネーションが機能するか？
- 入力エラー時に適切なメッセージが表示されるか？
- 保守性を意識したコードになっているか（TERASOLUNAガイドライン準拠）？

## 実践課題（演習）

### 課題1: ログイン画面の実装
`login/loginForm.jsp` を作成し、ユーザー名、パスワード、Remember Me機能を持つログインフォームを実装してください。Spring Securityと連携し、ログインエラー時にはエラーメッセージが表示されるようにしてください。

### 課題2: ToDo一覧画面の実装
`todo/list.jsp` を作成し、ログインユーザーのToDoを一覧表示する機能を実装してください。以下の要素を含めてください。
- 検索フォーム（タイトル、完了状態でフィルタリング）
- 新規登録画面へのリンク
- ToDoリスト（タイトル、説明、完了状態、作成日時、更新日時）
- 完了/未完了を切り替えるチェックボックス（またはボタン）
- 編集画面へのリンク
- 削除ボタン（確認ダイアログ付き）
- ページネーション

### 課題3: ToDo登録/更新画面の実装
`todo/form.jsp` を作成し、ToDoの登録と更新を行う共通フォームを実装してください。以下の要素を含めてください。
- 登録/更新で画面タイトルを切り替え
- タイトル入力フィールド（必須）
- 説明入力フィールド（テキストエリア）
- 完了状態チェックボックス
- 送信ボタン（登録/更新で表示を切り替え）
- キャンセルボタン（一覧画面へ戻る）
- 入力エラー時のメッセージ表示

### 課題4: ユーザー登録画面の実装
`user/registerForm.jsp` を作成し、新規ユーザー登録フォームを実装してください。以下の要素を含めてください。
- ユーザー名入力フィールド（必須）
- パスワード入力フィールド（必須）
- パスワード確認入力フィールド（必須）
- メールアドレス入力フィールド
- 登録ボタン
- キャンセルボタン（ログイン画面へ戻る）
- 入力エラー時のメッセージ表示

### 課題5: 国際化対応
`messages.properties`（英語）と `messages_ja.properties`（日本語）を作成し、アプリケーション全体の表示メッセージを国際化対応してください。以下の要素を含めてください。
- 画面タイトル
- フィールドラベル
- ボタンラベル
- エラーメッセージ
- 成功メッセージ

## 実務での活用シーン

### 1. 企業向けWebアプリケーション開発

企業向けWebアプリケーション開発では、TERASOLUNAフレームワークのような標準化されたアーキテクチャとJSPを用いたView層の実装が広く活用されています。

- **業務システムの開発**
  - 社内向け業務システム（販売管理、在庫管理、人事管理など）
  - 顧客向けポータルサイト
  - 管理者向け運用ツール

- **多層アーキテクチャの実践**
  - プレゼンテーション層、アプリケーション層、ドメイン層、インフラストラクチャ層の明確な分離
  - 各層の責務の明確化
  - 保守性と拡張性の向上

- **大規模チーム開発**
  - 標準化されたコーディング規約の適用
  - 共通コンポーネントの活用
  - チーム間の連携とコミュニケーション

### 2. レガシーシステムのリニューアル

多くの企業では、古いJavaベースのWebアプリケーションをモダンなアーキテクチャへリニューアルする需要があります。

- **段階的なリニューアル**
  - 既存システムの分析と要件整理
  - アーキテクチャの再設計
  - 段階的な移行計画の策定と実行

- **最新技術の導入**
  - Spring Frameworkの最新バージョンへの移行
  - セキュリティ対策の強化
  - レスポンシブデザインの導入

- **保守性の向上**
  - コードの可読性向上
  - テスト容易性の向上
  - ドキュメントの整備

### 3. マイクロサービスアーキテクチャへの移行

モノリシックなアプリケーションからマイクロサービスアーキテクチャへの移行においても、TERASOLUNAの知識は活用できます。

- **サービスの分割**
  - ドメイン駆動設計に基づくサービス境界の特定
  - 適切な粒度でのサービス分割
  - サービス間の連携設計

- **フロントエンドとバックエンドの分離**
  - APIファーストアプローチの採用
  - RESTful APIの設計と実装
  - フロントエンドのSPA化

- **DevOpsの実践**
  - CI/CDパイプラインの構築
  - コンテナ化とオーケストレーション
  - 監視と運用の自動化

### 4. チーム開発とプロジェクト管理

実務では、技術的なスキルだけでなく、チーム開発とプロジェクト管理のスキルも重要です。

- **アジャイル開発の実践**
  - スクラムやカンバンの導入
  - イテレーション開発とフィードバックの活用
  - 継続的な改善

- **コードレビューとペアプログラミング**
  - コードの品質向上
  - 知識の共有
  - チームメンバーのスキルアップ

- **ドキュメンテーション**
  - 設計ドキュメントの作成
  - APIドキュメントの整備
  - ユーザーマニュアルの作成
