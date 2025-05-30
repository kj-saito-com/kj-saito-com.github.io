# Day 15: Controller層の実装

## 学習の目的と背景

TERASOLUNAフレームワークにおいて、Controller層はWebアプリケーションの入り口として重要な役割を担います。ユーザーからのリクエストを受け付け、適切な処理を行い、結果をビューに渡す責務があります。

本日の学習では、TERASOLUNAフレームワークにおけるController層の実装方法を学び、ToDoアプリケーションのController層を実装します。これにより、Spring MVCの基本的な使い方と、TERASOLUNAフレームワークにおけるController層の設計・実装パターンを理解することを目指します。

## 学習内容の解説

### 1. Spring MVCの基本

#### Spring MVCとは

Spring MVCは、Spring Frameworkの一部として提供されるWebアプリケーションフレームワークです。MVCアーキテクチャ（Model-View-Controller）に基づいており、以下の特徴があります：

1. **リクエスト駆動型**
   - HTTPリクエストを受け取り、適切なコントローラメソッドにディスパッチ
   - アノテーションベースの設定

2. **柔軟な設定**
   - XMLベースの設定
   - Javaベースの設定
   - アノテーションベースの設定

3. **強力なバインディング機能**
   - リクエストパラメータからJavaオブジェクトへの自動バインディング
   - 型変換とバリデーション

4. **ビューテクノロジーとの統合**
   - JSP、Thymeleaf、FreeMarkerなど様々なビューテクノロジーをサポート
   - ビューリゾルバによる柔軟なビュー解決

#### Spring MVCのアーキテクチャ

Spring MVCのアーキテクチャは以下のコンポーネントで構成されています：

1. **DispatcherServlet**
   - フロントコントローラとして機能
   - リクエストを受け取り、適切なコントローラにディスパッチ
   - レスポンスの生成を調整

2. **HandlerMapping**
   - リクエストとハンドラ（コントローラメソッド）のマッピングを管理
   - URLパターン、HTTPメソッド、リクエストパラメータなどに基づいてマッピング

3. **Controller**
   - リクエストを処理し、モデルを更新
   - ビュー名を返す

4. **ModelAndView**
   - モデル（データ）とビュー（表示）の情報を保持
   - コントローラからビューへの橋渡し

5. **ViewResolver**
   - ビュー名からビューオブジェクトを解決
   - 様々なビューテクノロジーをサポート

6. **View**
   - モデルデータを使用してレスポンスを生成
   - HTML、JSON、XMLなど様々な形式をサポート

#### リクエスト処理の流れ

Spring MVCにおけるリクエスト処理の流れは以下の通りです：

1. クライアントがリクエストを送信
2. DispatcherServletがリクエストを受け取る
3. HandlerMappingがリクエストを処理するコントローラメソッドを特定
4. DispatcherServletがコントローラメソッドを呼び出す
5. コントローラメソッドがリクエストを処理し、ModelAndViewを返す
6. DispatcherServletがViewResolverを使用してビューを解決
7. ビューがモデルデータを使用してレスポンスを生成
8. DispatcherServletがレスポンスをクライアントに返す

### 2. TERASOLUNAフレームワークにおけるController層

#### Controller層の責務

TERASOLUNAフレームワークにおけるController層の主な責務は以下の通りです：

1. **リクエストの受け付け**
   - URLマッピング
   - HTTPメソッドの指定
   - リクエストパラメータの取得

2. **入力値の検証**
   - バリデーションの実行
   - エラーハンドリング

3. **ビジネスロジックの呼び出し**
   - Serviceの呼び出し
   - トランザクション境界の設定

4. **ビューの選択**
   - 遷移先の決定
   - リダイレクト・フォワードの選択

5. **レスポンスの生成**
   - モデル属性の設定
   - ビュー名の返却

#### Controller層の設計指針

TERASOLUNAフレームワークにおけるController層の設計指針は以下の通りです：

1. **リソース指向の設計**
   - URLはリソースを表現
   - HTTPメソッドはリソースに対する操作を表現

2. **責務の分離**
   - コントローラはリクエスト処理に集中
   - ビジネスロジックはServiceに委譲
   - 表示ロジックはViewに委譲

3. **シンプルな実装**
   - 複雑なロジックを避ける
   - 条件分岐を最小限に抑える
   - 共通処理はAOPで実現

4. **適切な粒度**
   - 機能単位でコントローラを分割
   - メソッド単位で処理を分割

#### Controller層のアノテーション

TERASOLUNAフレームワークにおけるController層の主なアノテーションは以下の通りです：

1. **@Controller**
   - クラスをコントローラとして指定
   - コンポーネントスキャンの対象となる

2. **@RequestMapping**
   - リクエストとハンドラメソッドのマッピングを指定
   - クラスレベルとメソッドレベルで指定可能
   - value属性でURLパターンを指定
   - method属性でHTTPメソッドを指定
   - params属性でリクエストパラメータの条件を指定
   - consumes属性でContent-Typeの条件を指定
   - produces属性でAcceptの条件を指定

3. **@GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping**
   - 特定のHTTPメソッドに対するマッピングを簡潔に指定
   - @RequestMappingのショートカット

4. **@PathVariable**
   - URLパスの一部をメソッド引数にバインド
   - RESTfulなURLの実現に有用

5. **@RequestParam**
   - リクエストパラメータをメソッド引数にバインド
   - required属性でパラメータの必須/任意を指定
   - defaultValue属性でデフォルト値を指定

6. **@ModelAttribute**
   - リクエストパラメータをJavaオブジェクトにバインド
   - フォームデータの受け取りに使用

7. **@Valid**
   - バインドされたオブジェクトのバリデーションを実行
   - JSR-303/JSR-349のバリデーションアノテーションと連携

8. **@InitBinder**
   - カスタムバインディングの設定
   - バリデータの登録
   - プロパティエディタの登録

9. **@ExceptionHandler**
   - コントローラ内で発生した例外のハンドリング
   - 特定の例外クラスに対する処理を指定

10. **@ControllerAdvice**
    - 複数のコントローラに共通する設定を提供
    - グローバルな例外ハンドリング
    - グローバルなモデル属性の追加
    - グローバルなバインディング設定

### 3. Controller層の実装パターン

#### 基本的な実装パターン

TERASOLUNAフレームワークにおけるController層の基本的な実装パターンは以下の通りです：

**1. 一覧表示**

```java
@Controller
@RequestMapping("/todos")
public class TodoController {
    
    @Autowired
    private TodoService todoService;
    
    @GetMapping
    public String list(Model model) {
        List<Todo> todos = todoService.findAll();
        model.addAttribute("todos", todos);
        return "todo/list";
    }
}
```

**2. 詳細表示**

```java
@GetMapping("/{id}")
public String show(@PathVariable Long id, Model model) {
    Todo todo = todoService.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Todo not found"));
    model.addAttribute("todo", todo);
    return "todo/show";
}
```

**3. 新規作成（フォーム表示）**

```java
@GetMapping("/create")
public String createForm(@ModelAttribute TodoForm todoForm) {
    return "todo/createForm";
}
```

**4. 新規作成（フォーム送信）**

```java
@PostMapping("/create")
public String create(@Validated TodoForm todoForm,
                    BindingResult bindingResult,
                    Model model) {
    if (bindingResult.hasErrors()) {
        return "todo/createForm";
    }
    
    Todo todo = beanMapper.map(todoForm, Todo.class);
    todoService.create(todo);
    
    return "redirect:/todos";
}
```

**5. 編集（フォーム表示）**

```java
@GetMapping("/{id}/edit")
public String editForm(@PathVariable Long id, Model model) {
    Todo todo = todoService.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Todo not found"));
    
    TodoForm todoForm = beanMapper.map(todo, TodoForm.class);
    model.addAttribute("todoForm", todoForm);
    model.addAttribute("todoId", id);
    
    return "todo/editForm";
}
```

**6. 編集（フォーム送信）**

```java
@PostMapping("/{id}/edit")
public String edit(@PathVariable Long id,
                  @Validated TodoForm todoForm,
                  BindingResult bindingResult,
                  Model model) {
    if (bindingResult.hasErrors()) {
        model.addAttribute("todoId", id);
        return "todo/editForm";
    }
    
    Todo todo = todoService.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Todo not found"));
    
    beanMapper.map(todoForm, todo);
    todoService.update(todo);
    
    return "redirect:/todos";
}
```

**7. 削除**

```java
@PostMapping("/{id}/delete")
public String delete(@PathVariable Long id) {
    todoService.delete(id);
    return "redirect:/todos";
}
```

#### PRG（Post-Redirect-Get）パターン

PRGパターンは、フォーム送信後にリダイレクトを行うことで、ブラウザの更新ボタンによる二重送信を防ぐパターンです。

```java
@PostMapping("/create")
public String create(@Validated TodoForm todoForm,
                    BindingResult bindingResult,
                    Model model,
                    RedirectAttributes redirectAttributes) {
    if (bindingResult.hasErrors()) {
        return "todo/createForm";
    }
    
    Todo todo = beanMapper.map(todoForm, Todo.class);
    todoService.create(todo);
    
    redirectAttributes.addFlashAttribute("successMessage", "Todo created successfully");
    
    return "redirect:/todos";
}
```

#### フラッシュスコープの活用

フラッシュスコープは、リダイレクト後も値を保持するためのスコープです。

```java
@PostMapping("/create")
public String create(@Validated TodoForm todoForm,
                    BindingResult bindingResult,
                    Model model,
                    RedirectAttributes redirectAttributes) {
    if (bindingResult.hasErrors()) {
        return "todo/createForm";
    }
    
    Todo todo = beanMapper.map(todoForm, Todo.class);
    todoService.create(todo);
    
    redirectAttributes.addFlashAttribute("successMessage", "Todo created successfully");
    redirectAttributes.addFlashAttribute("todoId", todo.getId());
    
    return "redirect:/todos";
}

@GetMapping
public String list(Model model) {
    List<Todo> todos = todoService.findAll();
    model.addAttribute("todos", todos);
    
    // フラッシュスコープの値は自動的にモデルに追加される
    // successMessage, todoIdがモデルに含まれる
    
    return "todo/list";
}
```

#### 例外ハンドリング

コントローラ内での例外ハンドリングは、@ExceptionHandlerアノテーションを使用します。

```java
@ExceptionHandler(ResourceNotFoundException.class)
public String handleResourceNotFoundException(ResourceNotFoundException e, Model model) {
    model.addAttribute("errorMessage", e.getMessage());
    return "error/resourceNotFound";
}
```

グローバルな例外ハンドリングは、@ControllerAdviceアノテーションを使用します。

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public String handleResourceNotFoundException(ResourceNotFoundException e, Model model) {
        model.addAttribute("errorMessage", e.getMessage());
        return "error/resourceNotFound";
    }
    
    @ExceptionHandler(Exception.class)
    public String handleException(Exception e, Model model) {
        model.addAttribute("errorMessage", "An error occurred");
        return "error/systemError";
    }
}
```

#### バリデーション

バリデーションは、@Validatedアノテーションと、BindingResultを使用します。

```java
@PostMapping("/create")
public String create(@Validated TodoForm todoForm,
                    BindingResult bindingResult,
                    Model model) {
    if (bindingResult.hasErrors()) {
        return "todo/createForm";
    }
    
    // バリデーションエラーがない場合の処理
    
    return "redirect:/todos";
}
```

カスタムバリデーションは、@InitBinderアノテーションを使用します。

```java
@InitBinder("todoForm")
public void initBinder(WebDataBinder binder) {
    binder.addValidators(todoValidator);
}
```

#### ページネーション

ページネーションは、Pageableインターフェースを使用します。

```java
@GetMapping
public String list(@PageableDefault(size = 10, sort = "id") Pageable pageable, Model model) {
    Page<Todo> todoPage = todoService.findAll(pageable);
    model.addAttribute("todoPage", todoPage);
    return "todo/list";
}
```

### 4. ToDoアプリケーションのController層の実装

#### TodoController

ToDoアプリケーションのController層の実装例を示します。

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
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import com.example.todo.domain.model.Todo;
import com.example.todo.domain.service.todo.TodoService;
import com.example.todo.domain.service.user.UserDetails;

@Controller
@RequestMapping("/todos")
public class TodoController {
    
    @Autowired
    private TodoService todoService;
    
    @Autowired
    private TodoValidator todoValidator;
    
    @Autowired
    private BeanMapper beanMapper;
    
    @ModelAttribute
    public TodoForm setUpForm() {
        return new TodoForm();
    }
    
    @InitBinder("todoForm")
    public void initBinder(WebDataBinder binder) {
        binder.addValidators(todoValidator);
    }
    
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
        Todo todo = todoService.findOne(id);
        if (todo == null || !todo.getUserId().equals(userDetails.getUserId())) {
            throw new ResourceNotFoundException("Todo not found");
        }
        model.addAttribute("todo", todo);
        return "todo/show";
    }
    
    @GetMapping("/create")
    public String createForm() {
        return "todo/createForm";
    }
    
    @PostMapping("/create")
    public String create(@AuthenticationPrincipal UserDetails userDetails,
                        @Validated TodoForm todoForm,
                        BindingResult bindingResult,
                        Model model,
                        RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            return "todo/createForm";
        }
        
        Todo todo = beanMapper.map(todoForm, Todo.class);
        todo.setUserId(userDetails.getUserId());
        todo.setCompleted(false);
        
        todoService.create(todo);
        
        redirectAttributes.addFlashAttribute("successMessage", "Todo created successfully");
        
        return "redirect:/todos";
    }
    
    @GetMapping("/{id}/edit")
    public String editForm(@AuthenticationPrincipal UserDetails userDetails,
                          @PathVariable Long id,
                          Model model) {
        Todo todo = todoService.findOne(id);
        if (todo == null || !todo.getUserId().equals(userDetails.getUserId())) {
            throw new ResourceNotFoundException("Todo not found");
        }
        
        TodoForm todoForm = beanMapper.map(todo, TodoForm.class);
        model.addAttribute("todoForm", todoForm);
        model.addAttribute("todoId", id);
        
        return "todo/editForm";
    }
    
    @PostMapping("/{id}/edit")
    public String edit(@AuthenticationPrincipal UserDetails userDetails,
                      @PathVariable Long id,
                      @Validated TodoForm todoForm,
                      BindingResult bindingResult,
                      Model model,
                      RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            model.addAttribute("todoId", id);
            return "todo/editForm";
        }
        
        Todo todo = todoService.findOne(id);
        if (todo == null || !todo.getUserId().equals(userDetails.getUserId())) {
            throw new ResourceNotFoundException("Todo not found");
        }
        
        beanMapper.map(todoForm, todo);
        
        todoService.update(todo);
        
        redirectAttributes.addFlashAttribute("successMessage", "Todo updated successfully");
        
        return "redirect:/todos";
    }
    
    @PostMapping("/{id}/delete")
    public String delete(@AuthenticationPrincipal UserDetails userDetails,
                        @PathVariable Long id,
                        RedirectAttributes redirectAttributes) {
        Todo todo = todoService.findOne(id);
        if (todo == null || !todo.getUserId().equals(userDetails.getUserId())) {
            throw new ResourceNotFoundException("Todo not found");
        }
        
        todoService.delete(id);
        
        redirectAttributes.addFlashAttribute("successMessage", "Todo deleted successfully");
        
        return "redirect:/todos";
    }
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public String handleResourceNotFoundException(ResourceNotFoundException e, Model model) {
        model.addAttribute("errorMessage", e.getMessage());
        return "error/resourceNotFound";
    }
}
```

## 実践課題（演習）

### 課題1: TodoControllerの実装

以下の要件に基づいて、TodoControllerを実装してください。

**要件:**
1. ToDoの一覧表示
2. ToDoの詳細表示
3. ToDoの新規作成
4. ToDoの編集
5. ToDoの削除
6. 完了状態によるフィルタリング

### 課題2: UserControllerの実装

以下の要件に基づいて、UserControllerを実装してください。

**要件:**
1. ユーザー登録
2. ユーザー名の重複チェック
3. パスワードの一致チェック
4. 登録成功時のメッセージ表示

### 課題3: 例外ハンドリングの実装

以下の要件に基づいて、例外ハンドリングを実装してください。

**要件:**
1. リソースが見つからない場合の例外ハンドリング
2. システムエラーの例外ハンドリング
3. バリデーションエラーの例外ハンドリング

### 課題4: コントローラのテスト実装

以下の要件に基づいて、コントローラのテストを実装してください。

**要件:**
1. 一覧表示のテスト
2. 詳細表示のテスト
3. 新規作成のテスト
4. 編集のテスト
5. 削除のテスト
6. 例外ハンドリングのテスト

## 実務での活用シーン

### 1. Webアプリケーション開発

Controller層の実装は、Webアプリケーション開発の中核となります。以下のような場面で活用されます：

- **企業の業務システム開発**
  - 社内システムの開発
  - 顧客管理システムの開発
  - 在庫管理システムの開発

- **ECサイトの開発**
  - 商品一覧・詳細表示
  - 注文処理
  - ユーザー管理

- **情報ポータルサイトの開発**
  - 記事の一覧・詳細表示
  - 検索機能
  - ユーザーコメント機能

### 2. RESTful APIの開発

Controller層の実装は、RESTful APIの開発にも活用されます：

- **モバイルアプリのバックエンド**
  - データの取得・更新API
  - 認証・認可API
  - プッシュ通知API

- **マイクロサービスアーキテクチャ**
  - サービス間通信のAPI
  - ゲートウェイAPI
  - 外部連携API

- **SPA（Single Page Application）のバックエンド**
  - データの取得・更新API
  - 認証・認可API
  - ファイルアップロードAPI

### 3. バッチ処理のWeb管理画面

バッチ処理のWeb管理画面の開発にも活用されます：

- **バッチジョブの管理**
  - ジョブの一覧・詳細表示
  - ジョブの実行・停止
  - ジョブのスケジュール設定

- **バッチ処理の監視**
  - 実行状況の表示
  - エラー通知
  - パフォーマンス監視

- **バッチ処理のパラメータ設定**
  - パラメータの一覧・詳細表示
  - パラメータの編集
  - パラメータのインポート・エクスポート

### 4. 管理画面の開発

管理画面の開発にも活用されます：

- **コンテンツ管理システム（CMS）**
  - コンテンツの一覧・詳細表示
  - コンテンツの作成・編集・削除
  - コンテンツの公開・非公開

- **ユーザー管理**
  - ユーザーの一覧・詳細表示
  - ユーザーの作成・編集・削除
  - ユーザーの権限設定

- **システム設定**
  - 設定の一覧・詳細表示
  - 設定の編集
  - 設定のインポート・エクスポート
