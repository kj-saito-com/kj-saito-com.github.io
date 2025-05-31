---
layout: default
title: Day 19: Form/DTOの実装
---
# Day 19: Form/DTOの実装

## 学習の目的と背景

TERASOLUNAフレームワークにおいて、Form/DTOはユーザーインターフェース層とドメイン層の間でデータを受け渡すための重要な役割を担います。これらのクラスは、ユーザー入力の検証やデータの変換、画面表示用のデータ構造の提供などの責務を持ちます。

本日の学習では、TERASOLUNAフレームワークにおけるForm/DTOの実装方法を学び、ToDoアプリケーションのForm/DTOを実装します。これにより、ユーザーインターフェース層とドメイン層の適切な分離、入力検証の実装方法、データ変換の手法などを理解することを目指します。

## 学習内容の解説

### 1. Form/DTOの役割と責務

#### Form/DTOとは

Form/DTOは、ユーザーインターフェース層とドメイン層の間でデータを受け渡すためのオブジェクトです。以下の特徴があります：

1. **Formの役割**
   - ユーザー入力の受け取り
   - 入力値の検証
   - ドメインオブジェクトへの変換
   - 画面表示用のデータ保持

2. **DTOの役割**
   - レイヤー間のデータ転送
   - 複数のドメインオブジェクトの集約
   - 画面表示用のデータ構造の提供
   - ドメインオブジェクトの詳細の隠蔽

3. **Form/DTOの共通の特徴**
   - シンプルなデータ構造
   - 基本的にはロジックを持たない
   - シリアライズ可能
   - 画面の要件に合わせた設計

#### Form/DTOの責務

TERASOLUNAフレームワークにおけるForm/DTOの主な責務は以下の通りです：

1. **ユーザー入力の受け取り**
   - リクエストパラメータのバインド
   - ファイルアップロードの処理
   - 複雑なフォーム構造の処理

2. **入力値の検証**
   - 単項目チェック
   - 相関チェック
   - ビジネスルールに基づく検証

3. **データ変換**
   - 文字列からドメインオブジェクトへの変換
   - 日付や数値の型変換
   - コード値の変換

4. **画面表示用のデータ保持**
   - 選択リストの項目
   - エラーメッセージ
   - 画面表示用の整形データ

5. **レイヤー間のデータ転送**
   - Controllerからサービス層へのデータ転送
   - サービス層からControllerへのデータ転送
   - 複数のドメインオブジェクトの集約

### 2. Bean Validationによる入力検証

#### Bean Validationとは

Bean Validationは、Javaのオブジェクトに対する検証を宣言的に行うためのAPI仕様です。以下の特徴があります：

1. **アノテーションベースの検証**
   - クラスのフィールドにアノテーションを付与
   - メソッドの引数や戻り値に対する検証
   - クラスレベルの検証

2. **標準的な検証アノテーション**
   - `@NotNull`: nullでないことを検証
   - `@Size`: 文字列や配列のサイズを検証
   - `@Min`/`@Max`: 数値の最小値/最大値を検証
   - `@Pattern`: 正規表現によるパターン検証
   - `@Email`: メールアドレス形式の検証

3. **カスタム検証の作成**
   - カスタムアノテーションの定義
   - 検証ロジックの実装
   - メッセージの国際化

4. **グループ検証**
   - 検証グループの定義
   - 状況に応じた検証の適用
   - 検証の順序制御

#### 基本的な検証アノテーション

Bean Validationの基本的な検証アノテーションは以下の通りです：

1. **`@NotNull`**
   - nullでないことを検証
   - 例: `@NotNull private String title;`

2. **`@NotEmpty`**
   - 空でないことを検証（文字列、コレクション、配列、マップ）
   - 例: `@NotEmpty private String title;`

3. **`@NotBlank`**
   - 空白でないことを検証（文字列）
   - 例: `@NotBlank private String title;`

4. **`@Size`**
   - 文字列の長さ、コレクションのサイズを検証
   - 例: `@Size(min = 1, max = 100) private String title;`

5. **`@Min`/`@Max`**
   - 数値の最小値/最大値を検証
   - 例: `@Min(0) @Max(100) private Integer age;`

6. **`@Pattern`**
   - 正規表現によるパターン検証
   - 例: `@Pattern(regexp = "^[A-Za-z0-9]+$") private String username;`

7. **`@Email`**
   - メールアドレス形式の検証
   - 例: `@Email private String email;`

8. **`@Past`/`@Future`**
   - 日付が過去/未来であることを検証
   - 例: `@Past private LocalDate birthDate;`

9. **`@Valid`**
   - ネストしたオブジェクトの検証
   - 例: `@Valid private Address address;`

#### カスタム検証の作成

カスタム検証を作成する手順は以下の通りです：

**1. カスタムアノテーションの定義**

```java
package com.example.todo.app.common.validation;

import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
import static java.lang.annotation.ElementType.CONSTRUCTOR;
import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.TYPE_USE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import javax.validation.Constraint;
import javax.validation.Payload;

@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = { UniqueUsernameValidator.class })
public @interface UniqueUsername {
    
    String message() default "{com.example.todo.app.common.validation.UniqueUsername.message}";
    
    Class<?>[] groups() default {};
    
    Class<? extends Payload>[] payload() default {};
    
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
    @Retention(RUNTIME)
    @Documented
    @interface List {
        UniqueUsername[] value();
    }
}
```

**2. 検証ロジックの実装**

```java
package com.example.todo.app.common.validation;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

import org.springframework.beans.factory.annotation.Autowired;

import com.example.todo.domain.service.UserDomainService;

public class UniqueUsernameValidator implements ConstraintValidator<UniqueUsername, String> {
    
    @Autowired
    private UserDomainService userDomainService;
    
    @Override
    public void initialize(UniqueUsername constraintAnnotation) {
    }
    
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }
        
        return !userDomainService.existsByUsername(value);
    }
}
```

**3. メッセージの定義**

```properties
# src/main/resources/ValidationMessages.properties
com.example.todo.app.common.validation.UniqueUsername.message=Username is already used.
```

#### グループ検証

グループ検証を使用すると、状況に応じて異なる検証を適用することができます。

**1. グループの定義**

```java
package com.example.todo.app.common.validation;

public interface Create {
}

public interface Update {
}
```

**2. グループの適用**

```java
package com.example.todo.app.todo;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;

import com.example.todo.app.common.validation.Create;
import com.example.todo.app.common.validation.Update;

import lombok.Data;

@Data
public class TodoForm {
    
    @NotBlank(groups = { Create.class, Update.class })
    @Size(max = 100, groups = { Create.class, Update.class })
    private String title;
    
    @Size(max = 1000, groups = { Create.class, Update.class })
    private String description;
    
    @NotNull(groups = { Update.class })
    private Long id;
}
```

**3. グループの指定**

```java
@RequestMapping(value = "create", method = RequestMethod.POST)
public String create(@Validated({ Create.class }) TodoForm todoForm, BindingResult bindingResult, Model model) {
    // ...
}

@RequestMapping(value = "update", method = RequestMethod.POST)
public String update(@Validated({ Update.class }) TodoForm todoForm, BindingResult bindingResult, Model model) {
    // ...
}
```

### 3. TERASOLUNAフレームワークにおけるForm/DTOの実装パターン

#### 基本的な実装パターン

TERASOLUNAフレームワークにおけるForm/DTOの基本的な実装パターンは以下の通りです：

**1. 入力フォーム**

```java
package com.example.todo.app.todo;

import java.io.Serializable;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;

import lombok.Data;

@Data
public class TodoForm implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    @NotBlank
    @Size(max = 100)
    private String title;
    
    @Size(max = 1000)
    private String description;
    
    private boolean completed;
}
```

**2. 検索条件フォーム**

```java
package com.example.todo.app.todo;

import java.io.Serializable;

import lombok.Data;

@Data
public class TodoSearchForm implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private String title;
    
    private Boolean completed;
    
    private String orderBy;
    
    private Boolean ascending;
}
```

**3. DTO**

```java
package com.example.todo.app.todo;

import java.io.Serializable;
import java.time.LocalDateTime;

import lombok.Data;

@Data
public class TodoDto implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private Long id;
    
    private String title;
    
    private String description;
    
    private boolean completed;
    
    private LocalDateTime createdAt;
    
    private LocalDateTime updatedAt;
    
    private LocalDateTime completedAt;
    
    private Long userId;
}
```

#### 複雑なフォーム構造の実装

複雑なフォーム構造を実装する場合は、ネストしたクラスを使用します。

```java
package com.example.todo.app.todo;

import java.io.Serializable;
import java.util.List;

import javax.validation.Valid;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

import lombok.Data;

@Data
public class TodoListForm implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    @Valid
    @NotNull
    @Size(min = 1, max = 10)
    private List<TodoForm> todos;
    
    @Data
    public static class TodoForm implements Serializable {
        
        private static final long serialVersionUID = 1L;
        
        @NotBlank
        @Size(max = 100)
        private String title;
        
        @Size(max = 1000)
        private String description;
        
        private boolean completed;
    }
}
```

#### ドメインオブジェクトとの変換

ドメインオブジェクトとの変換を行うためのメソッドを実装します。

```java
package com.example.todo.app.todo;

import java.io.Serializable;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;

import com.example.todo.domain.model.Todo;

import lombok.Data;

@Data
public class TodoForm implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    @NotBlank
    @Size(max = 100)
    private String title;
    
    @Size(max = 1000)
    private String description;
    
    private boolean completed;
    
    /**
     * TodoFormからTodoエンティティを作成します。
     * 
     * @param userId ユーザーID
     * @return Todoエンティティ
     */
    public Todo toEntity(Long userId) {
        Todo todo = new Todo();
        todo.setTitle(title);
        todo.setDescription(description);
        todo.setCompleted(completed);
        todo.setUserId(userId);
        return todo;
    }
    
    /**
     * Todoエンティティの値をTodoFormに設定します。
     * 
     * @param todo Todoエンティティ
     * @return TodoForm
     */
    public static TodoForm fromEntity(Todo todo) {
        TodoForm form = new TodoForm();
        form.setTitle(todo.getTitle());
        form.setDescription(todo.getDescription());
        form.setCompleted(todo.isCompleted());
        return form;
    }
}
```

#### ページネーションの実装

ページネーションを実装する場合は、ページ情報を保持するクラスを使用します。

```java
package com.example.todo.app.common.pagination;

import java.io.Serializable;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import lombok.Data;

@Data
public class PageInfo implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private int page;
    
    private int size;
    
    private int totalPages;
    
    private long totalElements;
    
    private boolean first;
    
    private boolean last;
    
    private boolean hasPrevious;
    
    private boolean hasNext;
    
    /**
     * PageオブジェクトからPageInfoを作成します。
     * 
     * @param page Pageオブジェクト
     * @return PageInfo
     */
    public static <T> PageInfo fromPage(Page<T> page) {
        PageInfo pageInfo = new PageInfo();
        pageInfo.setPage(page.getNumber());
        pageInfo.setSize(page.getSize());
        pageInfo.setTotalPages(page.getTotalPages());
        pageInfo.setTotalElements(page.getTotalElements());
        pageInfo.setFirst(page.isFirst());
        pageInfo.setLast(page.isLast());
        pageInfo.setHasPrevious(page.hasPrevious());
        pageInfo.setHasNext(page.hasNext());
        return pageInfo;
    }
    
    /**
     * 前のページのページ番号を取得します。
     * 
     * @return 前のページのページ番号
     */
    public int getPreviousPage() {
        return hasPrevious ? page - 1 : page;
    }
    
    /**
     * 次のページのページ番号を取得します。
     * 
     * @return 次のページのページ番号
     */
    public int getNextPage() {
        return hasNext ? page + 1 : page;
    }
}
```

### 4. ToDoアプリケーションのForm/DTOの実装

#### TodoForm

ToDoアプリケーションのTodoFormの実装例を示します：

```java
package com.example.todo.app.todo;

import java.io.Serializable;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;

import com.example.todo.domain.model.Todo;

import lombok.Data;

@Data
public class TodoForm implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    @NotBlank(message = "Title is required.")
    @Size(max = 100, message = "Title must be less than or equal to {max} characters.")
    private String title;
    
    @Size(max = 1000, message = "Description must be less than or equal to {max} characters.")
    private String description;
    
    private boolean completed;
    
    /**
     * TodoFormからTodoエンティティを作成します。
     * 
     * @param userId ユーザーID
     * @return Todoエンティティ
     */
    public Todo toEntity(Long userId) {
        Todo todo = new Todo();
        todo.setTitle(title);
        todo.setDescription(description);
        todo.setCompleted(completed);
        todo.setUserId(userId);
        return todo;
    }
    
    /**
     * TodoFormからTodoエンティティを更新します。
     * 
     * @param todo 更新対象のTodoエンティティ
     * @return 更新されたTodoエンティティ
     */
    public Todo updateEntity(Todo todo) {
        todo.setTitle(title);
        todo.setDescription(description);
        if (completed && !todo.isCompleted()) {
            todo.complete();
        } else if (!completed && todo.isCompleted()) {
            todo.incomplete();
        }
        return todo;
    }
    
    /**
     * Todoエンティティの値をTodoFormに設定します。
     * 
     * @param todo Todoエンティティ
     * @return TodoForm
     */
    public static TodoForm fromEntity(Todo todo) {
        TodoForm form = new TodoForm();
        form.setTitle(todo.getTitle());
        form.setDescription(todo.getDescription());
        form.setCompleted(todo.isCompleted());
        return form;
    }
}
```

#### TodoSearchForm

ToDoアプリケーションのTodoSearchFormの実装例を示します：

```java
package com.example.todo.app.todo;

import java.io.Serializable;

import com.example.todo.domain.repository.todo.TodoSearchCriteria;

import lombok.Data;

@Data
public class TodoSearchForm implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private String title;
    
    private Boolean completed;
    
    private String orderBy;
    
    private Boolean ascending;
    
    /**
     * TodoSearchFormからTodoSearchCriteriaを作成します。
     * 
     * @param userId ユーザーID
     * @return TodoSearchCriteria
     */
    public TodoSearchCriteria toSearchCriteria(Long userId) {
        TodoSearchCriteria criteria = new TodoSearchCriteria();
        criteria.setUserId(userId);
        criteria.setTitle(title);
        criteria.setCompleted(completed);
        criteria.setOrderBy(orderBy);
        criteria.setAscending(ascending);
        return criteria;
    }
}
```

#### TodoDto

ToDoアプリケーションのTodoDtoの実装例を示します：

```java
package com.example.todo.app.todo;

import java.io.Serializable;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

import com.example.todo.domain.model.Todo;

import lombok.Data;

@Data
public class TodoDto implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private Long id;
    
    private String title;
    
    private String description;
    
    private boolean completed;
    
    private String createdAt;
    
    private String updatedAt;
    
    private String completedAt;
    
    /**
     * Todoエンティティの値をTodoDtoに設定します。
     * 
     * @param todo Todoエンティティ
     * @return TodoDto
     */
    public static TodoDto fromEntity(Todo todo) {
        TodoDto dto = new TodoDto();
        dto.setId(todo.getId());
        dto.setTitle(todo.getTitle());
        dto.setDescription(todo.getDescription());
        dto.setCompleted(todo.isCompleted());
        
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        
        if (todo.getCreatedAt() != null) {
            dto.setCreatedAt(todo.getCreatedAt().format(formatter));
        }
        
        if (todo.getUpdatedAt() != null) {
            dto.setUpdatedAt(todo.getUpdatedAt().format(formatter));
        }
        
        if (todo.getCompletedAt() != null) {
            dto.setCompletedAt(todo.getCompletedAt().format(formatter));
        }
        
        return dto;
    }
}
```

#### UserForm

ToDoアプリケーションのUserFormの実装例を示します：

```java
package com.example.todo.app.user;

import java.io.Serializable;

import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;
import javax.validation.constraints.Size;

import com.example.todo.app.common.validation.UniqueUsername;
import com.example.todo.domain.model.User;

import lombok.Data;

@Data
public class UserForm implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    @NotBlank(message = "Username is required.")
    @Size(min = 3, max = 20, message = "Username must be between {min} and {max} characters.")
    @Pattern(regexp = "^[A-Za-z0-9_]+$", message = "Username must contain only letters, numbers, and underscores.")
    @UniqueUsername(message = "Username is already used.")
    private String username;
    
    @NotBlank(message = "Password is required.")
    @Size(min = 8, max = 100, message = "Password must be between {min} and {max} characters.")
    private String password;
    
    @NotBlank(message = "Email is required.")
    @Email(message = "Email must be a valid email address.")
    private String email;
    
    /**
     * UserFormからUserエンティティを作成します。
     * 
     * @return Userエンティティ
     */
    public User toEntity() {
        User user = new User();
        user.setUsername(username);
        user.setPassword(password);
        user.setEmail(email);
        user.setEnabled(true);
        return user;
    }
    
    /**
     * UserFormからUserエンティティを更新します。
     * 
     * @param user 更新対象のUserエンティティ
     * @return 更新されたUserエンティティ
     */
    public User updateEntity(User user) {
        user.setUsername(username);
        user.setPassword(password);
        user.setEmail(email);
        return user;
    }
    
    /**
     * Userエンティティの値をUserFormに設定します。
     * 
     * @param user Userエンティティ
     * @return UserForm
     */
    public static UserForm fromEntity(User user) {
        UserForm form = new UserForm();
        form.setUsername(user.getUsername());
        form.setPassword(user.getPassword());
        form.setEmail(user.getEmail());
        return form;
    }
}
```

## 実践課題（演習）

### 課題1: TodoFormの実装

以下の要件に基づいて、TodoFormを実装してください。

**要件:**
1. タイトル（必須、100文字以内）
2. 説明（任意、1000文字以内）
3. 完了フラグ
4. Todoエンティティへの変換メソッド
5. TodoエンティティからTodoFormへの変換メソッド
6. 入力検証のためのアノテーション

### 課題2: TodoSearchFormの実装

以下の要件に基づいて、TodoSearchFormを実装してください。

**要件:**
1. タイトル（部分一致検索用）
2. 完了フラグ（完了/未完了のフィルタリング用）
3. 並び順（タイトル、作成日時など）
4. 昇順/降順
5. TodoSearchCriteriaへの変換メソッド

### 課題3: TodoDtoの実装

以下の要件に基づいて、TodoDtoを実装してください。

**要件:**
1. ID
2. タイトル
3. 説明
4. 完了フラグ
5. 作成日時（文字列形式）
6. 更新日時（文字列形式）
7. 完了日時（文字列形式）
8. TodoエンティティからTodoDtoへの変換メソッド

### 課題4: カスタム検証の実装

以下の要件に基づいて、カスタム検証を実装してください。

**要件:**
1. タイトルの重複チェック用のカスタムアノテーション
2. 検証ロジックの実装
3. エラーメッセージの定義

## 実務での活用シーン

### 1. 業務アプリケーション開発

Form/DTOの実装は、業務アプリケーション開発において重要です。以下のような場面で活用されます：

- **ユーザー入力の受け取りと検証**
  - 入力フォームの実装
  - 入力値の検証
  - エラーメッセージの表示

- **データの変換と転送**
  - ユーザーインターフェース層とドメイン層の間のデータ変換
  - レイヤー間のデータ転送
  - 複数のドメインオブジェクトの集約

- **画面表示用のデータ構造の提供**
  - 画面表示用のデータ構造の定義
  - データの整形
  - 表示用の変換

### 2. RESTful API開発

Form/DTOの実装は、RESTful API開発においても重要です：

- **リクエストボディのバインド**
  - JSONリクエストのバインド
  - リクエストパラメータのバインド
  - 入力値の検証

- **レスポンスの構造化**
  - JSONレスポンスの構造の定義
  - データの整形
  - エラーレスポンスの定義

- **APIバージョニング**
  - APIバージョンごとのDTOの定義
  - 後方互換性の確保
  - 新機能の追加

### 3. バッチ処理

Form/DTOの実装は、バッチ処理においても活用されます：

- **データの入力と検証**
  - CSVファイルからのデータ読み込み
  - データの検証
  - エラーデータの処理

- **データの変換と転送**
  - 入力データからドメインオブジェクトへの変換
  - バッチ処理間のデータ転送
  - 処理結果の集約

- **レポート生成**
  - レポートデータの構造の定義
  - データの整形
  - レポートの出力

### 4. テスト駆動開発（TDD）

Form/DTOの実装は、テスト駆動開発においても重要です：

- **ユーザーインターフェース層のテスト**
  - 入力検証のテスト
  - データ変換のテスト
  - エラーハンドリングのテスト

- **モックオブジェクトの作成**
  - サービス層のモック化
  - リポジトリ層のモック化
  - 外部システムのモック化

- **テストデータの作成**
  - テストデータの構造の定義
  - テストデータの生成
  - テスト結果の検証
