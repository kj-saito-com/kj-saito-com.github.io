---
layout: default
title: "Day 18: Domain層の実装"
---
# Day 18: Domain層の実装

## 学習の目的と背景

TERASOLUNAフレームワークにおいて、Domain層はビジネスロジックを担当する中核的な層です。アプリケーションのビジネスルールやドメインモデルを表現し、アプリケーションの本質的な機能を提供します。

本日の学習では、TERASOLUNAフレームワークにおけるDomain層の実装方法を学び、ToDoアプリケーションのDomain層を実装します。これにより、ドメイン駆動設計（DDD）の考え方や、ビジネスロジックの適切な設計と実装方法を理解することを目指します。

## 学習内容の解説

### 1. Domain層の役割と責務

#### Domain層とは

Domain層は、アプリケーションのビジネスロジックを担当する層です。以下の特徴があります：

1. **ビジネスルールの表現**
   - ドメインモデルの定義
   - ビジネスルールの実装
   - ドメイン固有の振る舞いの実装

2. **ドメインオブジェクトの提供**
   - エンティティ
   - 値オブジェクト
   - ドメインサービス
   - リポジトリインターフェース

3. **ビジネスロジックの集約**
   - ビジネスルールの一元管理
   - ドメイン知識の表現
   - ビジネスプロセスの実装

4. **技術的な詳細からの独立**
   - フレームワークに依存しない設計
   - データアクセス技術に依存しない設計
   - UI技術に依存しない設計

#### Domain層の責務

TERASOLUNAフレームワークにおけるDomain層の主な責務は以下の通りです：

1. **ドメインモデルの定義**
   - エンティティの定義
   - 値オブジェクトの定義
   - ドメインサービスの定義
   - リポジトリインターフェースの定義

2. **ビジネスルールの実装**
   - ドメイン固有のロジックの実装
   - バリデーションルールの実装
   - ビジネス制約の実装

3. **ドメイン間の関連の管理**
   - エンティティ間の関連の定義
   - 集約の定義
   - 集約間の関連の定義

4. **ドメインイベントの管理**
   - ドメインイベントの定義
   - ドメインイベントの発行
   - ドメインイベントのハンドリング

5. **ドメイン固有の例外の定義**
   - ビジネスルール違反の例外
   - ドメイン固有のエラー状態の表現
   - 例外の階層化

### 2. ドメイン駆動設計（DDD）の基本概念

#### ドメイン駆動設計とは

ドメイン駆動設計（Domain-Driven Design, DDD）は、複雑なビジネスドメインを扱うソフトウェア開発のためのアプローチです。以下の特徴があります：

1. **ドメインモデルの重視**
   - ビジネスドメインの理解を深める
   - ドメインの言語（ユビキタス言語）を使用する
   - ドメインの概念をコードに反映する

2. **戦略的設計と戦術的設計**
   - 戦略的設計：境界づけられたコンテキスト、コンテキストマップ
   - 戦術的設計：エンティティ、値オブジェクト、集約、リポジトリなど

3. **ドメインエキスパートとの協業**
   - ドメインエキスパートの知識を活用
   - ユビキタス言語の構築
   - ドメインモデルの共同開発

4. **モデル駆動設計**
   - モデルとコードの一致
   - モデルの継続的な改善
   - モデルの表現力の向上

#### DDDの主要な概念

DDDの主要な概念は以下の通りです：

1. **エンティティ（Entity）**
   - 同一性（ID）を持つオブジェクト
   - ライフサイクルを持つ
   - 状態が変化しても同じエンティティとして識別される

2. **値オブジェクト（Value Object）**
   - 属性のみで構成されるオブジェクト
   - 同一性を持たない
   - 不変（イミュータブル）である
   - 等価性で比較される

3. **集約（Aggregate）**
   - 関連するエンティティと値オブジェクトのクラスタ
   - 集約ルート（Aggregate Root）を通じてのみアクセスされる
   - トランザクションの一貫性の境界を定義する

4. **リポジトリ（Repository）**
   - 集約の永続化と取得を担当
   - コレクションのようなインターフェースを提供
   - 永続化の詳細を隠蔽する

5. **ドメインサービス（Domain Service）**
   - 特定のエンティティに属さないドメインロジック
   - 複数のエンティティにまたがる操作
   - ステートレスな操作

6. **ドメインイベント（Domain Event）**
   - ドメイン内で発生した重要な出来事
   - 過去形で表現される
   - イベント駆動アーキテクチャの基盤

7. **境界づけられたコンテキスト（Bounded Context）**
   - 特定のドメインモデルが適用される範囲
   - 異なるコンテキスト間の関係を明確にする
   - モデルの整合性を保つ

### 3. TERASOLUNAフレームワークにおけるDomain層の実装

#### エンティティの実装

TERASOLUNAフレームワークにおけるエンティティの実装例を示します：

```java
package com.example.todo.domain.model;

import java.io.Serializable;
import java.time.LocalDateTime;

import lombok.Data;

@Data
public class Todo implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private Long id;
    
    private String title;
    
    private String description;
    
    private boolean completed;
    
    private LocalDateTime createdAt;
    
    private LocalDateTime updatedAt;
    
    private LocalDateTime completedAt;
    
    private Long userId;
    
    private Integer version;
    
    /**
     * ToDoを完了状態に変更します。
     */
    public void complete() {
        this.completed = true;
        this.completedAt = LocalDateTime.now();
    }
    
    /**
     * ToDoを未完了状態に変更します。
     */
    public void incomplete() {
        this.completed = false;
        this.completedAt = null;
    }
    
    /**
     * ToDoが指定されたユーザーに所有されているかどうかを判定します。
     * 
     * @param userId ユーザーID
     * @return 指定されたユーザーに所有されている場合はtrue、そうでない場合はfalse
     */
    public boolean isOwnedBy(Long userId) {
        return this.userId != null && this.userId.equals(userId);
    }
}
```

#### 値オブジェクトの実装

TERASOLUNAフレームワークにおける値オブジェクトの実装例を示します：

```java
package com.example.todo.domain.model;

import java.io.Serializable;
import java.util.Objects;

import lombok.Value;

@Value
public class UserId implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private final Long value;
    
    public UserId(Long value) {
        if (value == null) {
            throw new IllegalArgumentException("value must not be null");
        }
        this.value = value;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        UserId userId = (UserId) o;
        return Objects.equals(value, userId.value);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
    
    @Override
    public String toString() {
        return value.toString();
    }
}
```

#### ドメインサービスの実装

TERASOLUNAフレームワークにおけるドメインサービスの実装例を示します：

```java
package com.example.todo.domain.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.todo.domain.model.Todo;
import com.example.todo.domain.repository.todo.TodoRepository;

@Service
@Transactional
public class TodoDomainService {
    
    @Autowired
    private TodoRepository todoRepository;
    
    /**
     * 指定されたユーザーIDとタイトルのToDoが既に存在するかどうかを判定します。
     * 
     * @param userId ユーザーID
     * @param title タイトル
     * @return 既に存在する場合はtrue、そうでない場合はfalse
     */
    public boolean existsByUserIdAndTitle(Long userId, String title) {
        return todoRepository.existsByUserIdAndTitle(userId, title);
    }
    
    /**
     * 指定されたユーザーIDとタイトルのToDoが、指定されたID以外で既に存在するかどうかを判定します。
     * 
     * @param userId ユーザーID
     * @param title タイトル
     * @param id ID
     * @return 既に存在する場合はtrue、そうでない場合はfalse
     */
    public boolean existsByUserIdAndTitleAndIdNot(Long userId, String title, Long id) {
        return todoRepository.existsByUserIdAndTitleAndIdNot(userId, title, id);
    }
    
    /**
     * 指定されたToDoが指定されたユーザーに所有されているかどうかを判定します。
     * 
     * @param todo ToDo
     * @param userId ユーザーID
     * @return 指定されたユーザーに所有されている場合はtrue、そうでない場合はfalse
     */
    public boolean isOwnedBy(Todo todo, Long userId) {
        return todo.isOwnedBy(userId);
    }
}
```

#### リポジトリインターフェースの定義

TERASOLUNAフレームワークにおけるリポジトリインターフェースの定義例を示します：

```java
package com.example.todo.domain.repository.todo;

import java.util.List;
import java.util.Optional;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import com.example.todo.domain.model.Todo;

public interface TodoRepository {
    
    List<Todo> findAll();
    
    List<Todo> findByUserId(Long userId);
    
    List<Todo> findByUserIdAndCompleted(Long userId, boolean completed);
    
    Optional<Todo> findById(Long id);
    
    boolean existsByUserIdAndTitle(Long userId, String title);
    
    boolean existsByUserIdAndTitleAndIdNot(Long userId, String title, Long id);
    
    Todo save(Todo todo);
    
    void deleteById(Long id);
    
    Page<Todo> findPage(TodoSearchCriteria criteria, Pageable pageable);
    
    long count(TodoSearchCriteria criteria);
}
```

#### ドメイン例外の定義

TERASOLUNAフレームワークにおけるドメイン例外の定義例を示します：

```java
package com.example.todo.domain.exception;

public class BusinessException extends RuntimeException {
    
    private static final long serialVersionUID = 1L;
    
    private final String code;
    
    public BusinessException(String code) {
        super();
        this.code = code;
    }
    
    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }
    
    public BusinessException(String code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }
    
    public String getCode() {
        return code;
    }
}
```

```java
package com.example.todo.domain.exception;

public class ResourceNotFoundException extends BusinessException {
    
    private static final long serialVersionUID = 1L;
    
    public ResourceNotFoundException(String code) {
        super(code);
    }
    
    public ResourceNotFoundException(String code, String message) {
        super(code, message);
    }
    
    public ResourceNotFoundException(String code, String message, Throwable cause) {
        super(code, message, cause);
    }
}
```

### 4. ToDoアプリケーションのDomain層の実装

#### Todoエンティティ

ToDoアプリケーションのTodoエンティティの実装例を示します：

```java
package com.example.todo.domain.model;

import java.io.Serializable;
import java.time.LocalDateTime;

import lombok.Data;

@Data
public class Todo implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private Long id;
    
    private String title;
    
    private String description;
    
    private boolean completed;
    
    private LocalDateTime createdAt;
    
    private LocalDateTime updatedAt;
    
    private LocalDateTime completedAt;
    
    private Long userId;
    
    private Integer version;
    
    /**
     * ToDoを完了状態に変更します。
     */
    public void complete() {
        this.completed = true;
        this.completedAt = LocalDateTime.now();
    }
    
    /**
     * ToDoを未完了状態に変更します。
     */
    public void incomplete() {
        this.completed = false;
        this.completedAt = null;
    }
    
    /**
     * ToDoが指定されたユーザーに所有されているかどうかを判定します。
     * 
     * @param userId ユーザーID
     * @return 指定されたユーザーに所有されている場合はtrue、そうでない場合はfalse
     */
    public boolean isOwnedBy(Long userId) {
        return this.userId != null && this.userId.equals(userId);
    }
}
```

#### Userエンティティ

ToDoアプリケーションのUserエンティティの実装例を示します：

```java
package com.example.todo.domain.model;

import java.io.Serializable;
import java.time.LocalDateTime;

import lombok.Data;

@Data
public class User implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private Long id;
    
    private String username;
    
    private String password;
    
    private String email;
    
    private boolean enabled;
    
    private LocalDateTime createdAt;
    
    private LocalDateTime updatedAt;
    
    private Integer version;
}
```

#### TodoDomainService

ToDoアプリケーションのTodoDomainServiceの実装例を示します：

```java
package com.example.todo.domain.service;

import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.todo.domain.exception.BusinessException;
import com.example.todo.domain.exception.ResourceNotFoundException;
import com.example.todo.domain.model.Todo;
import com.example.todo.domain.repository.todo.TodoRepository;

@Service
@Transactional
public class TodoDomainService {
    
    @Autowired
    private TodoRepository todoRepository;
    
    /**
     * 指定されたIDのToDoを取得します。
     * 
     * @param id ID
     * @return ToDo
     * @throws ResourceNotFoundException 指定されたIDのToDoが存在しない場合
     */
    public Todo findOne(Long id) {
        Optional<Todo> todo = todoRepository.findById(id);
        return todo.orElseThrow(() -> new ResourceNotFoundException("E404", "Todo is not found. [id=" + id + "]"));
    }
    
    /**
     * 指定されたユーザーIDとタイトルのToDoが既に存在するかどうかを判定します。
     * 
     * @param userId ユーザーID
     * @param title タイトル
     * @return 既に存在する場合はtrue、そうでない場合はfalse
     */
    public boolean existsByUserIdAndTitle(Long userId, String title) {
        return todoRepository.existsByUserIdAndTitle(userId, title);
    }
    
    /**
     * 指定されたユーザーIDとタイトルのToDoが、指定されたID以外で既に存在するかどうかを判定します。
     * 
     * @param userId ユーザーID
     * @param title タイトル
     * @param id ID
     * @return 既に存在する場合はtrue、そうでない場合はfalse
     */
    public boolean existsByUserIdAndTitleAndIdNot(Long userId, String title, Long id) {
        return todoRepository.existsByUserIdAndTitleAndIdNot(userId, title, id);
    }
    
    /**
     * 指定されたToDoが指定されたユーザーに所有されているかどうかを判定します。
     * 
     * @param todo ToDo
     * @param userId ユーザーID
     * @return 指定されたユーザーに所有されている場合はtrue、そうでない場合はfalse
     * @throws BusinessException 指定されたユーザーに所有されていない場合
     */
    public boolean isOwnedBy(Todo todo, Long userId) {
        if (!todo.isOwnedBy(userId)) {
            throw new BusinessException("E403", "Todo is not owned by the user. [id=" + todo.getId() + ", userId=" + userId + "]");
        }
        return true;
    }
    
    /**
     * 指定されたToDoを作成します。
     * 
     * @param todo ToDo
     * @return 作成されたToDo
     * @throws BusinessException 同じユーザーIDとタイトルのToDoが既に存在する場合
     */
    public Todo create(Todo todo) {
        if (existsByUserIdAndTitle(todo.getUserId(), todo.getTitle())) {
            throw new BusinessException("E400", "Todo with the same title already exists. [title=" + todo.getTitle() + "]");
        }
        return todoRepository.save(todo);
    }
    
    /**
     * 指定されたToDoを更新します。
     * 
     * @param todo ToDo
     * @return 更新されたToDo
     * @throws BusinessException 同じユーザーIDとタイトルのToDoが、指定されたID以外で既に存在する場合
     */
    public Todo update(Todo todo) {
        if (existsByUserIdAndTitleAndIdNot(todo.getUserId(), todo.getTitle(), todo.getId())) {
            throw new BusinessException("E400", "Todo with the same title already exists. [title=" + todo.getTitle() + "]");
        }
        return todoRepository.save(todo);
    }
    
    /**
     * 指定されたIDのToDoを削除します。
     * 
     * @param id ID
     */
    public void delete(Long id) {
        todoRepository.deleteById(id);
    }
}
```

#### UserDomainService

ToDoアプリケーションのUserDomainServiceの実装例を示します：

```java
package com.example.todo.domain.service;

import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.todo.domain.exception.BusinessException;
import com.example.todo.domain.exception.ResourceNotFoundException;
import com.example.todo.domain.model.User;
import com.example.todo.domain.repository.user.UserRepository;

@Service
@Transactional
public class UserDomainService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    /**
     * 指定されたIDのユーザーを取得します。
     * 
     * @param id ID
     * @return ユーザー
     * @throws ResourceNotFoundException 指定されたIDのユーザーが存在しない場合
     */
    public User findOne(Long id) {
        Optional<User> user = userRepository.findById(id);
        return user.orElseThrow(() -> new ResourceNotFoundException("E404", "User is not found. [id=" + id + "]"));
    }
    
    /**
     * 指定されたユーザー名のユーザーを取得します。
     * 
     * @param username ユーザー名
     * @return ユーザー
     * @throws ResourceNotFoundException 指定されたユーザー名のユーザーが存在しない場合
     */
    public User findByUsername(String username) {
        Optional<User> user = userRepository.findByUsername(username);
        return user.orElseThrow(() -> new ResourceNotFoundException("E404", "User is not found. [username=" + username + "]"));
    }
    
    /**
     * 指定されたユーザー名のユーザーが既に存在するかどうかを判定します。
     * 
     * @param username ユーザー名
     * @return 既に存在する場合はtrue、そうでない場合はfalse
     */
    public boolean existsByUsername(String username) {
        return userRepository.existsByUsername(username);
    }
    
    /**
     * 指定されたユーザー名のユーザーが、指定されたID以外で既に存在するかどうかを判定します。
     * 
     * @param username ユーザー名
     * @param id ID
     * @return 既に存在する場合はtrue、そうでない場合はfalse
     */
    public boolean existsByUsernameAndIdNot(String username, Long id) {
        return userRepository.existsByUsernameAndIdNot(username, id);
    }
    
    /**
     * 指定されたユーザーを作成します。
     * 
     * @param user ユーザー
     * @return 作成されたユーザー
     * @throws BusinessException 同じユーザー名のユーザーが既に存在する場合
     */
    public User create(User user) {
        if (existsByUsername(user.getUsername())) {
            throw new BusinessException("E400", "User with the same username already exists. [username=" + user.getUsername() + "]");
        }
        
        // パスワードをハッシュ化
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        
        return userRepository.save(user);
    }
    
    /**
     * 指定されたユーザーを更新します。
     * 
     * @param user ユーザー
     * @return 更新されたユーザー
     * @throws BusinessException 同じユーザー名のユーザーが、指定されたID以外で既に存在する場合
     */
    public User update(User user) {
        if (existsByUsernameAndIdNot(user.getUsername(), user.getId())) {
            throw new BusinessException("E400", "User with the same username already exists. [username=" + user.getUsername() + "]");
        }
        
        // 現在のユーザー情報を取得
        User currentUser = findOne(user.getId());
        
        // パスワードが変更されている場合のみハッシュ化
        if (!currentUser.getPassword().equals(user.getPassword())) {
            user.setPassword(passwordEncoder.encode(user.getPassword()));
        }
        
        return userRepository.save(user);
    }
    
    /**
     * 指定されたIDのユーザーを削除します。
     * 
     * @param id ID
     */
    public void delete(Long id) {
        userRepository.deleteById(id);
    }
}
```

## 実践課題（演習）

### 課題1: Todoエンティティの実装

以下の要件に基づいて、Todoエンティティを実装してください。

**要件:**
1. ID、タイトル、説明、完了フラグ、作成日時、更新日時、完了日時、ユーザーID、バージョンの属性を持つ
2. 完了状態に変更するメソッドを持つ
3. 未完了状態に変更するメソッドを持つ
4. 指定されたユーザーに所有されているかどうかを判定するメソッドを持つ
5. シリアライズ可能であること

### 課題2: TodoDomainServiceの実装

以下の要件に基づいて、TodoDomainServiceを実装してください。

**要件:**
1. TodoRepositoryを使用してToDoの永続化と取得を行う
2. 指定されたIDのToDoを取得するメソッドを持つ
3. 指定されたユーザーIDとタイトルのToDoが既に存在するかどうかを判定するメソッドを持つ
4. 指定されたユーザーIDとタイトルのToDoが、指定されたID以外で既に存在するかどうかを判定するメソッドを持つ
5. 指定されたToDoが指定されたユーザーに所有されているかどうかを判定するメソッドを持つ
6. 指定されたToDoを作成するメソッドを持つ
7. 指定されたToDoを更新するメソッドを持つ
8. 指定されたIDのToDoを削除するメソッドを持つ
9. トランザクション管理を行う

### 課題3: TodoDomainServiceのテスト

以下の要件に基づいて、TodoDomainServiceのテストを実装してください。

**要件:**
1. findOneメソッドのテスト（成功ケースと失敗ケース）
2. existsByUserIdAndTitleメソッドのテスト
3. existsByUserIdAndTitleAndIdNotメソッドのテスト
4. isOwnedByメソッドのテスト
5. createメソッドのテスト（成功ケースと失敗ケース）
6. updateメソッドのテスト（成功ケースと失敗ケース）
7. deleteメソッドのテスト
8. Mockitoを使用してTodoRepositoryをモック化する

### 課題4: ドメイン例外の実装

以下の要件に基づいて、ドメイン例外を実装してください。

**要件:**
1. BusinessExceptionクラスの実装
2. ResourceNotFoundExceptionクラスの実装（BusinessExceptionを継承）
3. 例外コードと例外メッセージを持つ
4. シリアライズ可能であること

## 実務での活用シーン

### 1. 業務アプリケーション開発

Domain層の実装は、業務アプリケーション開発の中核となります。以下のような場面で活用されます：

- **企業の基幹システム開発**
  - 複雑なビジネスルールの実装
  - ドメインモデルの設計
  - ビジネスプロセスの実装

- **金融システム開発**
  - 取引ルールの実装
  - リスク管理ロジックの実装
  - 規制対応の実装

- **ECサイトの開発**
  - 商品管理ロジックの実装
  - 注文処理ロジックの実装
  - 在庫管理ロジックの実装

### 2. マイクロサービスアーキテクチャ

Domain層の実装は、マイクロサービスアーキテクチャにおいても重要です：

- **サービス境界の定義**
  - 境界づけられたコンテキストの設計
  - サービス間の関係の定義
  - ドメインモデルの分割

- **サービス間の連携**
  - ドメインイベントの発行と購読
  - サービス間のデータ整合性の確保
  - 分散トランザクションの管理

- **サービスの自律性の確保**
  - ビジネスロジックの独立性
  - 外部依存の最小化
  - 変更の局所化

### 3. レガシーシステムのリファクタリング

Domain層の実装は、レガシーシステムのリファクタリングにおいても重要です：

- **ビジネスロジックの抽出**
  - 散在したビジネスロジックの集約
  - ドメインモデルの再構築
  - ビジネスルールの明確化

- **アーキテクチャの改善**
  - レイヤ化アーキテクチャの導入
  - 関心の分離
  - テスト容易性の向上

- **段階的なリファクタリング**
  - 境界づけられたコンテキストの特定
  - ドメインモデルの段階的な導入
  - 既存システムとの共存

## 解答例

<a href="answers/day18">解答例はこちら</a>