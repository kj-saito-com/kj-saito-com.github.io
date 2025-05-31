---
layout: default
title: Day 14: ToDoアプリケーションの要件定義と設計
---
# Day 14: ToDoアプリケーションの要件定義と設計

## 学習の目的と背景

ソフトウェア開発において、要件定義と設計は成功の鍵となる重要なフェーズです。特に、TERASOLUNAフレームワークを使用した開発では、アーキテクチャの理解と適切な設計が、保守性の高いアプリケーション構築に不可欠です。

本日の学習では、ToDoアプリケーションの要件を明確に定義し、TERASOLUNAフレームワークに準拠した設計を行います。これにより、実装フェーズでの混乱を防ぎ、品質の高いアプリケーション開発の基盤を築くことを目指します。

## 学習内容の解説

### 1. 要件定義の基本

#### 要件定義とは

要件定義とは、システムが満たすべき機能的・非機能的な要求を明確にするプロセスです。要件定義の目的は以下の通りです：

1. **ステークホルダーの期待を明確化する**
   - 顧客、ユーザー、開発者など全ての関係者が同じ認識を持つ
   - 誤解や認識のずれを防ぐ

2. **開発の範囲と制約を明確にする**
   - 何を作るのか、何を作らないのかを明確にする
   - 時間、コスト、技術的制約を考慮する

3. **テストの基準を提供する**
   - 要件は検証可能であるべき
   - 受け入れ基準の基礎となる

#### 要件の種類

要件は大きく分けて以下の2種類があります：

1. **機能要件**
   - システムが「何をするか」を定義
   - ユーザーストーリー、ユースケース、機能仕様書などで表現

2. **非機能要件**
   - システムが「どのように動作するか」を定義
   - パフォーマンス、セキュリティ、可用性、保守性など

#### 要件定義のプロセス

要件定義は以下のようなプロセスで行われます：

1. **要件の収集**
   - インタビュー、アンケート、ワークショップなど
   - 既存システムの分析
   - 市場調査

2. **要件の分析**
   - 収集した要件の整理と優先順位付け
   - 矛盾や曖昧さの解消
   - 実現可能性の検討

3. **要件の文書化**
   - 要件仕様書の作成
   - ユースケース図、画面遷移図などの作成
   - プロトタイプの作成

4. **要件の検証**
   - レビューとフィードバック
   - 要件の妥当性確認
   - ステークホルダーの承認

### 2. ToDoアプリケーションの要件定義

#### 機能要件

ToDoアプリケーションの主な機能要件は以下の通りです：

1. **ユーザー管理機能**
   - ユーザー登録
   - ログイン・ログアウト
   - パスワード変更

2. **ToDo管理機能**
   - ToDoの一覧表示
   - ToDoの詳細表示
   - ToDoの新規作成
   - ToDoの編集
   - ToDoの削除
   - ToDoの完了・未完了の切り替え

3. **検索・フィルタリング機能**
   - タイトルによる検索
   - 完了状態によるフィルタリング
   - 作成日時によるソート

#### 非機能要件

ToDoアプリケーションの主な非機能要件は以下の通りです：

1. **パフォーマンス**
   - ページロード時間は3秒以内
   - 100件のToDoを表示しても応答時間が劣化しない

2. **セキュリティ**
   - パスワードはハッシュ化して保存
   - クロスサイトスクリプティング（XSS）対策
   - クロスサイトリクエストフォージェリ（CSRF）対策
   - SQLインジェクション対策

3. **可用性**
   - 24時間365日の稼働
   - 計画的なメンテナンス時間を除き、99.9%の可用性

4. **保守性**
   - コードの可読性と一貫性
   - 適切なドキュメント
   - テストの自動化

5. **ユーザビリティ**
   - 直感的なUI
   - レスポンシブデザイン
   - アクセシビリティへの配慮

#### ユースケース

ToDoアプリケーションの主なユースケースは以下の通りです：

1. **ユーザー登録**
   - アクター：未登録ユーザー
   - 前提条件：なし
   - 基本フロー：
     1. ユーザーが登録ページにアクセスする
     2. ユーザー名、パスワードを入力する
     3. 登録ボタンをクリックする
     4. システムがユーザー情報を検証する
     5. システムがユーザー情報を保存する
     6. システムがログインページにリダイレクトする
   - 代替フロー：
     - 4a. ユーザー名が既に使用されている場合
       1. システムがエラーメッセージを表示する
       2. ユーザーが別のユーザー名を入力する
     - 4b. パスワードが要件を満たさない場合
       1. システムがエラーメッセージを表示する
       2. ユーザーが別のパスワードを入力する

2. **ログイン**
   - アクター：登録済みユーザー
   - 前提条件：ユーザーが登録済みであること
   - 基本フロー：
     1. ユーザーがログインページにアクセスする
     2. ユーザー名、パスワードを入力する
     3. ログインボタンをクリックする
     4. システムが認証情報を検証する
     5. システムがToDoリストページにリダイレクトする
   - 代替フロー：
     - 4a. 認証情報が不正な場合
       1. システムがエラーメッセージを表示する
       2. ユーザーが認証情報を再入力する

3. **ToDoの作成**
   - アクター：ログイン済みユーザー
   - 前提条件：ユーザーがログイン済みであること
   - 基本フロー：
     1. ユーザーがToDoリストページで「新規作成」ボタンをクリックする
     2. システムが新規作成フォームを表示する
     3. ユーザーがタイトル、説明を入力する
     4. ユーザーが保存ボタンをクリックする
     5. システムがToDoを保存する
     6. システムがToDoリストページを更新する
   - 代替フロー：
     - 4a. 必須項目が未入力の場合
       1. システムがエラーメッセージを表示する
       2. ユーザーが必須項目を入力する

4. **ToDoの編集**
   - アクター：ログイン済みユーザー
   - 前提条件：ユーザーがログイン済みであり、ToDoが存在すること
   - 基本フロー：
     1. ユーザーがToDoリストページで編集したいToDoの「編集」ボタンをクリックする
     2. システムが編集フォームを表示する
     3. ユーザーがタイトル、説明を編集する
     4. ユーザーが保存ボタンをクリックする
     5. システムがToDoを更新する
     6. システムがToDoリストページを更新する
   - 代替フロー：
     - 4a. 必須項目が未入力の場合
       1. システムがエラーメッセージを表示する
       2. ユーザーが必須項目を入力する

5. **ToDoの削除**
   - アクター：ログイン済みユーザー
   - 前提条件：ユーザーがログイン済みであり、ToDoが存在すること
   - 基本フロー：
     1. ユーザーがToDoリストページで削除したいToDoの「削除」ボタンをクリックする
     2. システムが確認ダイアログを表示する
     3. ユーザーが「はい」をクリックする
     4. システムがToDoを削除する
     5. システムがToDoリストページを更新する
   - 代替フロー：
     - 3a. ユーザーが「いいえ」をクリックした場合
       1. システムが確認ダイアログを閉じる
       2. 操作を中止する

6. **ToDoの完了・未完了の切り替え**
   - アクター：ログイン済みユーザー
   - 前提条件：ユーザーがログイン済みであり、ToDoが存在すること
   - 基本フロー：
     1. ユーザーがToDoリストページでToDoの完了チェックボックスをクリックする
     2. システムがToDoの完了状態を切り替える
     3. システムがToDoリストページを更新する

### 3. アプリケーション設計の基本

#### アーキテクチャ設計

アーキテクチャ設計は、システムの構造と構成要素、およびそれらの関係を定義するプロセスです。TERASOLUNAフレームワークでは、以下のような多層アーキテクチャを採用しています：

1. **プレゼンテーション層**
   - ユーザーインターフェース
   - コントローラ
   - フォーム

2. **アプリケーション層**
   - サービス
   - DTO（Data Transfer Object）

3. **ドメイン層**
   - ドメインオブジェクト
   - ドメインサービス
   - リポジトリインターフェース

4. **インフラストラクチャ層**
   - リポジトリ実装
   - データアクセス
   - 外部システム連携

#### データベース設計

データベース設計は、データの構造と関係を定義するプロセスです。以下のような手順で行われます：

1. **概念設計**
   - エンティティの識別
   - エンティティ間の関係の定義
   - ER図の作成

2. **論理設計**
   - テーブルの定義
   - 正規化
   - インデックスの設計

3. **物理設計**
   - データ型の選択
   - パフォーマンスの最適化
   - ストレージの考慮

#### 画面設計

画面設計は、ユーザーインターフェースの構造と外観を定義するプロセスです。以下のような手順で行われます：

1. **画面遷移の設計**
   - 画面の一覧
   - 画面間の遷移
   - 画面遷移図の作成

2. **レイアウトの設計**
   - 画面の構成要素
   - 配置と構造
   - ワイヤーフレームの作成

3. **デザインの設計**
   - 色彩
   - フォント
   - アイコン
   - モックアップの作成

### 4. TERASOLUNAフレームワークに準拠した設計

#### TERASOLUNAのアーキテクチャ

TERASOLUNAフレームワークは、Spring Frameworkをベースにした多層アーキテクチャを採用しています。以下の層で構成されています：

1. **Webレイヤ**
   - Controller
   - Form
   - View（JSP）

2. **Serviceレイヤ**
   - Service
   - DTO

3. **Domainレイヤ**
   - Domain Object
   - Repository Interface
   - Domain Service

4. **Infrastructureレイヤ**
   - Repository Implementation
   - Datasource
   - Integration

#### TERASOLUNAの命名規約

TERASOLUNAフレームワークでは、以下のような命名規約を採用しています：

1. **パッケージ**
   - `com.example.todo.app.xxx`：Webレイヤ
   - `com.example.todo.domain.service`：Serviceレイヤ
   - `com.example.todo.domain.model`：Domainレイヤ（モデル）
   - `com.example.todo.domain.repository`：Domainレイヤ（リポジトリインターフェース）
   - `com.example.todo.infra.xxx`：Infrastructureレイヤ

2. **クラス**
   - `XxxController`：コントローラ
   - `XxxForm`：フォーム
   - `XxxDto`：DTO
   - `XxxService`：サービスインターフェース
   - `XxxServiceImpl`：サービス実装
   - `Xxx`：ドメインオブジェクト
   - `XxxRepository`：リポジトリインターフェース
   - `XxxRepositoryImpl`：リポジトリ実装

#### TERASOLUNAの依存関係

TERASOLUNAフレームワークでは、以下のような依存関係を持ちます：

1. **Webレイヤ → Serviceレイヤ**
   - ControllerはServiceを呼び出す

2. **Serviceレイヤ → Domainレイヤ**
   - ServiceはRepositoryを呼び出す
   - ServiceはDomain Objectを操作する

3. **Domainレイヤ → Infrastructureレイヤ**
   - Repository InterfaceはRepository Implementationによって実装される

4. **循環依存の禁止**
   - 上位レイヤから下位レイヤへの依存のみ許可
   - 下位レイヤから上位レイヤへの依存は禁止

### 5. ToDoアプリケーションの設計

#### パッケージ構成

ToDoアプリケーションのパッケージ構成は以下の通りです：

```
com.example.todo
├── app
│   ├── todo
│   │   ├── TodoController.java
│   │   ├── TodoForm.java
│   │   └── TodoValidator.java
│   └── user
│       ├── UserController.java
│       ├── UserForm.java
│       └── UserValidator.java
├── domain
│   ├── model
│   │   ├── Todo.java
│   │   └── User.java
│   ├── repository
│   │   ├── TodoRepository.java
│   │   └── UserRepository.java
│   └── service
│       ├── todo
│       │   ├── TodoService.java
│       │   └── TodoServiceImpl.java
│       └── user
│           ├── UserService.java
│           └── UserServiceImpl.java
└── infra
    ├── mybatis
    │   ├── repository
    │   │   ├── TodoRepositoryImpl.java
    │   │   └── UserRepositoryImpl.java
    │   └── typehandler
    │       └── LocalDateTimeTypeHandler.java
    └── config
        ├── app
        │   └── ApplicationContextConfig.java
        ├── web
        │   └── WebMvcConfig.java
        └── infra
            └── InfrastructureConfig.java
```

#### データベース設計

ToDoアプリケーションのデータベース設計は以下の通りです：

**todosテーブル**

| カラム名 | データ型 | 制約 | 説明 |
|---------|---------|------|------|
| id | SERIAL | PRIMARY KEY | ToDoのID |
| title | VARCHAR(256) | NOT NULL | タイトル |
| description | TEXT | | 説明 |
| completed | BOOLEAN | NOT NULL DEFAULT FALSE | 完了状態 |
| created_at | TIMESTAMP | NOT NULL | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL | 更新日時 |

**usersテーブル**

| カラム名 | データ型 | 制約 | 説明 |
|---------|---------|------|------|
| id | SERIAL | PRIMARY KEY | ユーザーID |
| username | VARCHAR(100) | NOT NULL UNIQUE | ユーザー名 |
| password | VARCHAR(100) | NOT NULL | パスワード（ハッシュ化） |
| enabled | BOOLEAN | NOT NULL DEFAULT TRUE | 有効状態 |
| created_at | TIMESTAMP | NOT NULL | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL | 更新日時 |

**user_todosテーブル**

| カラム名 | データ型 | 制約 | 説明 |
|---------|---------|------|------|
| user_id | INTEGER | NOT NULL REFERENCES users(id) | ユーザーID |
| todo_id | INTEGER | NOT NULL REFERENCES todos(id) | ToDoID |
| | | PRIMARY KEY (user_id, todo_id) | 複合主キー |

#### 画面設計

ToDoアプリケーションの画面設計は以下の通りです：

**画面一覧**

1. ログイン画面
2. ユーザー登録画面
3. ToDoリスト画面
4. ToDo詳細画面
5. ToDo作成画面
6. ToDo編集画面

**画面遷移図**

```
[ログイン画面] -----> [ToDoリスト画面] -----> [ToDo詳細画面]
    |                      |                      |
    |                      |                      |
    v                      v                      v
[ユーザー登録画面]    [ToDo作成画面]        [ToDo編集画面]
```

**画面レイアウト**

1. **ログイン画面**
   - ユーザー名入力フィールド
   - パスワード入力フィールド
   - ログインボタン
   - ユーザー登録リンク

2. **ユーザー登録画面**
   - ユーザー名入力フィールド
   - パスワード入力フィールド
   - パスワード確認入力フィールド
   - 登録ボタン
   - ログイン画面へ戻るリンク

3. **ToDoリスト画面**
   - ヘッダー（ユーザー名、ログアウトリンク）
   - 新規作成ボタン
   - 検索フォーム
   - フィルタリングオプション
   - ToDoリスト
     - タイトル
     - 完了チェックボックス
     - 編集ボタン
     - 削除ボタン
   - ページネーション

4. **ToDo詳細画面**
   - ヘッダー（ユーザー名、ログアウトリンク）
   - タイトル
   - 説明
   - 完了状態
   - 作成日時
   - 更新日時
   - 編集ボタン
   - 削除ボタン
   - 戻るボタン

5. **ToDo作成画面**
   - ヘッダー（ユーザー名、ログアウトリンク）
   - タイトル入力フィールド
   - 説明入力フィールド
   - 保存ボタン
   - キャンセルボタン

6. **ToDo編集画面**
   - ヘッダー（ユーザー名、ログアウトリンク）
   - タイトル入力フィールド
   - 説明入力フィールド
   - 完了チェックボックス
   - 保存ボタン
   - キャンセルボタン

#### ドメインモデル設計

ToDoアプリケーションのドメインモデル設計は以下の通りです：

**Todo.java**

```java
package com.example.todo.domain.model;

import java.time.LocalDateTime;

import lombok.Data;

@Data
public class Todo {
    private Long id;
    private String title;
    private String description;
    private boolean completed;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

**User.java**

```java
package com.example.todo.domain.model;

import java.time.LocalDateTime;
import java.util.List;

import lombok.Data;

@Data
public class User {
    private Long id;
    private String username;
    private String password;
    private boolean enabled;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private List<Todo> todos;
}
```

#### リポジトリインターフェース設計

ToDoアプリケーションのリポジトリインターフェース設計は以下の通りです：

**TodoRepository.java**

```java
package com.example.todo.domain.repository;

import java.util.List;
import java.util.Optional;

import com.example.todo.domain.model.Todo;

public interface TodoRepository {
    List<Todo> findAll();
    Optional<Todo> findById(Long id);
    List<Todo> findByUserId(Long userId);
    List<Todo> findByUserIdAndCompleted(Long userId, boolean completed);
    void create(Todo todo);
    void update(Todo todo);
    void delete(Long id);
    void deleteByUserId(Long userId);
}
```

**UserRepository.java**

```java
package com.example.todo.domain.repository;

import java.util.List;
import java.util.Optional;

import com.example.todo.domain.model.User;

public interface UserRepository {
    List<User> findAll();
    Optional<User> findById(Long id);
    Optional<User> findByUsername(String username);
    void create(User user);
    void update(User user);
    void delete(Long id);
    boolean exists(String username);
}
```

#### サービスインターフェース設計

ToDoアプリケーションのサービスインターフェース設計は以下の通りです：

**TodoService.java**

```java
package com.example.todo.domain.service.todo;

import java.util.List;
import java.util.Optional;

import com.example.todo.domain.model.Todo;

public interface TodoService {
    List<Todo> findAll();
    Optional<Todo> findById(Long id);
    List<Todo> findByUserId(Long userId);
    List<Todo> findByUserIdAndCompleted(Long userId, boolean completed);
    void create(Todo todo, Long userId);
    void update(Todo todo);
    void delete(Long id);
    void deleteByUserId(Long userId);
}
```

**UserService.java**

```java
package com.example.todo.domain.service.user;

import java.util.List;
import java.util.Optional;

import com.example.todo.domain.model.User;

public interface UserService {
    List<User> findAll();
    Optional<User> findById(Long id);
    Optional<User> findByUsername(String username);
    void create(User user);
    void update(User user);
    void delete(Long id);
    boolean exists(String username);
    boolean authenticate(String username, String password);
}
```

#### コントローラ設計

ToDoアプリケーションのコントローラ設計は以下の通りです：

**TodoController.java**

```java
package com.example.todo.app.todo;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import com.example.todo.domain.model.Todo;
import com.example.todo.domain.service.todo.TodoService;
import com.example.todo.domain.service.user.UserDetails;

@Controller
@RequestMapping("/todos")
public class TodoController {
    
    @Autowired
    private TodoService todoService;
    
    @GetMapping
    public String list(@AuthenticationPrincipal UserDetails userDetails,
                      @RequestParam(required = false) Boolean completed,
                      Model model) {
        List<Todo> todos;
        if (completed != null) {
            todos = todoService.findByUserIdAndCompleted(userDetails.getUserId(), completed);
        } else {
            todos = todoService.findByUserId(userDetails.getUserId());
        }
        model.addAttribute("todos", todos);
        return "todo/list";
    }
    
    @GetMapping("/{id}")
    public String show(@AuthenticationPrincipal UserDetails userDetails,
                      @PathVariable Long id,
                      Model model) {
        Todo todo = todoService.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo not found"));
        model.addAttribute("todo", todo);
        return "todo/show";
    }
    
    @GetMapping("/create")
    public String createForm(@ModelAttribute TodoForm todoForm) {
        return "todo/createForm";
    }
    
    @PostMapping("/create")
    public String create(@AuthenticationPrincipal UserDetails userDetails,
                        @Validated TodoForm todoForm,
                        BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return "todo/createForm";
        }
        
        Todo todo = new Todo();
        todo.setTitle(todoForm.getTitle());
        todo.setDescription(todoForm.getDescription());
        todo.setCompleted(false);
        
        todoService.create(todo, userDetails.getUserId());
        
        return "redirect:/todos";
    }
    
    @GetMapping("/{id}/edit")
    public String editForm(@AuthenticationPrincipal UserDetails userDetails,
                          @PathVariable Long id,
                          Model model) {
        Todo todo = todoService.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo not found"));
        
        TodoForm todoForm = new TodoForm();
        todoForm.setTitle(todo.getTitle());
        todoForm.setDescription(todo.getDescription());
        todoForm.setCompleted(todo.isCompleted());
        
        model.addAttribute("todoForm", todoForm);
        model.addAttribute("todoId", id);
        
        return "todo/editForm";
    }
    
    @PostMapping("/{id}/edit")
    public String edit(@AuthenticationPrincipal UserDetails userDetails,
                      @PathVariable Long id,
                      @Validated TodoForm todoForm,
                      BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return "todo/editForm";
        }
        
        Todo todo = todoService.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo not found"));
        
        todo.setTitle(todoForm.getTitle());
        todo.setDescription(todoForm.getDescription());
        todo.setCompleted(todoForm.isCompleted());
        
        todoService.update(todo);
        
        return "redirect:/todos";
    }
    
    @PostMapping("/{id}/delete")
    public String delete(@AuthenticationPrincipal UserDetails userDetails,
                        @PathVariable Long id) {
        todoService.delete(id);
        return "redirect:/todos";
    }
}
```

**UserController.java**

```java
package com.example.todo.app.user;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

import com.example.todo.domain.model.User;
import com.example.todo.domain.service.user.UserService;

@Controller
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/login")
    public String loginForm() {
        return "user/loginForm";
    }
    
    @GetMapping("/register")
    public String registerForm(@ModelAttribute UserForm userForm) {
        return "user/registerForm";
    }
    
    @PostMapping("/register")
    public String register(@Validated UserForm userForm,
                          BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return "user/registerForm";
        }
        
        if (!userForm.getPassword().equals(userForm.getConfirmPassword())) {
            bindingResult.rejectValue("confirmPassword", "error.userForm.confirmPassword", "パスワードが一致しません");
            return "user/registerForm";
        }
        
        if (userService.exists(userForm.getUsername())) {
            bindingResult.rejectValue("username", "error.userForm.username", "このユーザー名は既に使用されています");
            return "user/registerForm";
        }
        
        User user = new User();
        user.setUsername(userForm.getUsername());
        user.setPassword(userForm.getPassword());
        user.setEnabled(true);
        
        userService.create(user);
        
        return "redirect:/login";
    }
}
```

**JSPファイル**

**user/loginForm.jsp**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ログイン</title>
</head>
<body>
    <h1>ログイン</h1>
    
    <c:if test="${param.error}">
        <div>
            ユーザー名またはパスワードが正しくありません。
        </div>
    </c:if>
    
    <form action="${pageContext.request.contextPath}/login" method="post">
        <div>
            <label for="username">ユーザー名</label>
            <input type="text" id="username" name="username">
        </div>
        <div>
            <label for="password">パスワード</label>
            <input type="password" id="password" name="password">
        </div>
        <div>
            <input type="submit" value="ログイン">
        </div>
        <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
    </form>
    
    <div>
        <a href="${pageContext.request.contextPath}/register">ユーザー登録</a>
    </div>
</body>
</html>
```

**user/registerForm.jsp**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ユーザー登録</title>
</head>
<body>
    <h1>ユーザー登録</h1>
    
    <form:form modelAttribute="userForm" action="${pageContext.request.contextPath}/register" method="post">
        <div>
            <form:label path="username">ユーザー名</form:label>
            <form:input path="username" />
            <form:errors path="username" />
        </div>
        <div>
            <form:label path="password">パスワード</form:label>
            <form:password path="password" />
            <form:errors path="password" />
        </div>
        <div>
            <form:label path="confirmPassword">パスワード（確認）</form:label>
            <form:password path="confirmPassword" />
            <form:errors path="confirmPassword" />
        </div>
        <div>
            <input type="submit" value="登録">
        </div>
    </form:form>
    
    <div>
        <a href="${pageContext.request.contextPath}/login">ログイン画面へ戻る</a>
    </div>
</body>
</html>
```

**todo/list.jsp**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ToDoリスト</title>
</head>
<body>
    <h1>ToDoリスト</h1>
    
    <div>
        <form action="${pageContext.request.contextPath}/logout" method="post">
            <input type="submit" value="ログアウト">
            <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
        </form>
    </div>
    
    <div>
        <a href="${pageContext.request.contextPath}/todos/create">新規作成</a>
    </div>
    
    <div>
        <a href="${pageContext.request.contextPath}/todos">全て</a>
        <a href="${pageContext.request.contextPath}/todos?completed=false">未完了</a>
        <a href="${pageContext.request.contextPath}/todos?completed=true">完了</a>
    </div>
    
    <table>
        <thead>
            <tr>
                <th>タイトル</th>
                <th>完了</th>
                <th>操作</th>
            </tr>
        </thead>
        <tbody>
            <c:forEach items="${todos}" var="todo">
                <tr>
                    <td><a href="${pageContext.request.contextPath}/todos/${todo.id}">${todo.title}</a></td>
                    <td>
                        <c:if test="${todo.completed}">完了</c:if>
                        <c:if test="${!todo.completed}">未完了</c:if>
                    </td>
                    <td>
                        <a href="${pageContext.request.contextPath}/todos/${todo.id}/edit">編集</a>
                        <form action="${pageContext.request.contextPath}/todos/${todo.id}/delete" method="post" style="display: inline;">
                            <input type="submit" value="削除">
                            <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
                        </form>
                    </td>
                </tr>
            </c:forEach>
        </tbody>
    </table>
</body>
</html>
```

**todo/show.jsp**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ToDo詳細</title>
</head>
<body>
    <h1>ToDo詳細</h1>
    
    <div>
        <form action="${pageContext.request.contextPath}/logout" method="post">
            <input type="submit" value="ログアウト">
            <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
        </form>
    </div>
    
    <div>
        <h2>${todo.title}</h2>
        <p>
            <c:if test="${todo.completed}">完了</c:if>
            <c:if test="${!todo.completed}">未完了</c:if>
        </p>
        <p>${todo.description}</p>
        <p>作成日時: <fmt:formatDate value="${todo.createdAt}" pattern="yyyy/MM/dd HH:mm:ss" /></p>
        <p>更新日時: <fmt:formatDate value="${todo.updatedAt}" pattern="yyyy/MM/dd HH:mm:ss" /></p>
    </div>
    
    <div>
        <a href="${pageContext.request.contextPath}/todos/${todo.id}/edit">編集</a>
        <form action="${pageContext.request.contextPath}/todos/${todo.id}/delete" method="post" style="display: inline;">
            <input type="submit" value="削除">
            <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
        </form>
        <a href="${pageContext.request.contextPath}/todos">戻る</a>
    </div>
</body>
</html>
```

**todo/createForm.jsp**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ToDo作成</title>
</head>
<body>
    <h1>ToDo作成</h1>
    
    <div>
        <form action="${pageContext.request.contextPath}/logout" method="post">
            <input type="submit" value="ログアウト">
            <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
        </form>
    </div>
    
    <form:form modelAttribute="todoForm" action="${pageContext.request.contextPath}/todos/create" method="post">
        <div>
            <form:label path="title">タイトル</form:label>
            <form:input path="title" />
            <form:errors path="title" />
        </div>
        <div>
            <form:label path="description">説明</form:label>
            <form:textarea path="description" />
            <form:errors path="description" />
        </div>
        <div>
            <input type="submit" value="保存">
        </div>
    </form:form>
    
    <div>
        <a href="${pageContext.request.contextPath}/todos">キャンセル</a>
    </div>
</body>
</html>
```

**todo/editForm.jsp**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ToDo編集</title>
</head>
<body>
    <h1>ToDo編集</h1>
    
    <div>
        <form action="${pageContext.request.contextPath}/logout" method="post">
            <input type="submit" value="ログアウト">
            <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
        </form>
    </div>
    
    <form:form modelAttribute="todoForm" action="${pageContext.request.contextPath}/todos/${todoId}/edit" method="post">
        <div>
            <form:label path="title">タイトル</form:label>
            <form:input path="title" />
            <form:errors path="title" />
        </div>
        <div>
            <form:label path="description">説明</form:label>
            <form:textarea path="description" />
            <form:errors path="description" />
        </div>
        <div>
            <form:label path="completed">完了</form:label>
            <form:checkbox path="completed" />
            <form:errors path="completed" />
        </div>
        <div>
            <input type="submit" value="保存">
        </div>
    </form:form>
    
    <div>
        <a href="${pageContext.request.contextPath}/todos">キャンセル</a>
    </div>
</body>
</html>
```

## 実践課題（演習）

### 課題1: 要件定義書の作成

以下の要件に基づいて、ToDoアプリケーションの要件定義書を作成してください。

**要件:**
1. 機能要件と非機能要件を明確に区別して記述
2. ユースケース図を作成
3. 画面遷移図を作成
4. 各機能の詳細な説明を記述

### 課題2: データベース設計

以下の要件に基づいて、ToDoアプリケーションのデータベース設計を行ってください。

**要件:**
1. ER図を作成
2. テーブル定義書を作成
3. インデックス設計を行う
4. 制約の設定を行う

### 課題3: アプリケーション設計

以下の要件に基づいて、ToDoアプリケーションのアプリケーション設計を行ってください。

**要件:**
1. パッケージ構成を設計
2. クラス図を作成
3. シーケンス図を作成
4. 各レイヤの責務を明確に定義

### 課題4: 画面設計

以下の要件に基づいて、ToDoアプリケーションの画面設計を行ってください。

**要件:**
1. 各画面のワイヤーフレームを作成
2. 画面遷移図を作成
3. 各画面の入力項目と検証ルールを定義
4. エラーメッセージの定義

## 実務での活用シーン

### 課題1（要件定義）の活用シーン

1. **新規プロジェクトの立ち上げ**
   - 顧客との要件のすり合わせ
   - プロジェクトスコープの定義
   - 開発チームへの要件の共有

2. **既存システムの機能追加**
   - 追加機能の要件定義
   - 既存機能との整合性確認
   - 影響範囲の分析

3. **システムリプレイス**
   - 現行システムの機能分析
   - 新システムへの移行要件
   - ギャップ分析

4. **RFP（提案依頼書）の作成**
   - ベンダー選定のための要件提示
   - 見積もり依頼の基礎資料
   - 評価基準の明確化

### 課題2（データベース設計）の活用シーン

1. **新規データベースの構築**
   - テーブル設計
   - インデックス設計
   - 制約の設定

2. **データベースのパフォーマンスチューニング**
   - インデックスの最適化
   - クエリの最適化
   - テーブル構造の見直し

3. **データ移行**
   - 移行元と移行先のマッピング
   - データ変換ルールの定義
   - 整合性チェック

4. **データベースの統合**
   - 複数データベースの統合設計
   - マスターデータの一元化
   - 重複データの排除

### 課題3（アプリケーション設計）の活用シーン

1. **新規アプリケーションの開発**
   - アーキテクチャ設計
   - コンポーネント設計
   - インターフェース設計

2. **レガシーシステムのリファクタリング**
   - 責務の明確化
   - 依存関係の整理
   - テスト容易性の向上

3. **マイクロサービスへの移行**
   - サービス境界の定義
   - APIの設計
   - データの分割

4. **チーム開発の効率化**
   - 開発ガイドラインの策定
   - コーディング規約の策定
   - レビュー基準の明確化

### 課題4（画面設計）の活用シーン

1. **ユーザーインターフェースの設計**
   - 画面レイアウトの設計
   - 操作性の向上
   - 視覚的一貫性の確保

2. **ユーザビリティテスト**
   - プロトタイプの作成
   - ユーザーフィードバックの収集
   - 改善点の特定

3. **レスポンシブデザインの実装**
   - デバイス別のレイアウト設計
   - ブレークポイントの定義
   - コンテンツの優先順位付け

4. **アクセシビリティの向上**
   - 色のコントラスト確保
   - キーボード操作のサポート
   - スクリーンリーダー対応
