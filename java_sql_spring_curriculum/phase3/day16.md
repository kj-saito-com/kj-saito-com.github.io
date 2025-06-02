---
layout: default
title: "Day 16: Service層の実装"
---
# Day 16: Service層の実装

## 学習の目的と背景

TERASOLUNAフレームワークにおいて、Service層はビジネスロジックを担当する重要な層です。Controller層とRepository層の間に位置し、アプリケーションの中核となるビジネスルールやトランザクション管理を担当します。

本日の学習では、TERASOLUNAフレームワークにおけるService層の実装方法を学び、ToDoアプリケーションのService層を実装します。これにより、ビジネスロジックの適切な設計と実装、トランザクション管理、例外ハンドリングなどの重要な概念を理解することを目指します。

## 学習内容の解説

### 1. Service層の役割と責務

#### Service層とは

Service層は、アプリケーションのビジネスロジックを担当する層です。以下の特徴があります：

1. **ビジネスロジックの集約**
   - 業務ルールの実装
   - 複数のリポジトリの連携
   - データの加工・変換

2. **トランザクション境界**
   - トランザクションの開始・コミット・ロールバック
   - 複数の更新処理の一貫性確保

3. **認可（Authorization）**
   - ビジネスルールに基づくアクセス制御
   - データの所有者チェック

4. **例外ハンドリング**
   - ビジネス例外の発生
   - システム例外の変換

#### Service層の責務

TERASOLUNAフレームワークにおけるService層の主な責務は以下の通りです：

1. **ビジネスロジックの実装**
   - 業務ルールの実装
   - 入力値の業務的な妥当性検証
   - 計算・集計処理

2. **トランザクション管理**
   - トランザクション境界の設定
   - 複数の更新処理の一貫性確保
   - 楽観的ロック・悲観的ロックの制御

3. **リポジトリの呼び出し**
   - 必要なデータの取得
   - データの永続化
   - 複数のリポジトリの連携

4. **例外ハンドリング**
   - ビジネス例外の発生と処理
   - システム例外のラップと変換

5. **認可（Authorization）**
   - ビジネスルールに基づくアクセス制御
   - データの所有者チェック

### 2. Service層の設計指針

#### レイヤ化アーキテクチャにおけるService層

TERASOLUNAフレームワークでは、レイヤ化アーキテクチャを採用しており、Service層は以下の位置づけになります：

1. **Controller層からの呼び出し**
   - Controller層はService層のメソッドを呼び出す
   - Controller層はビジネスロジックを持たない

2. **Repository層の呼び出し**
   - Service層はRepository層のメソッドを呼び出す
   - Service層は直接データアクセスを行わない

3. **Domain層の利用**
   - Service層はDomain層のオブジェクトを操作する
   - Domain層のオブジェクトはビジネスルールを持つ

#### Service層の設計指針

TERASOLUNAフレームワークにおけるService層の設計指針は以下の通りです：

1. **責務の明確化**
   - 単一責任の原則（SRP）に基づく設計
   - 業務機能単位でのサービスの分割
   - メソッド単位での処理の分割

2. **インターフェースと実装の分離**
   - サービスインターフェースの定義
   - サービス実装クラスの作成
   - 依存性の注入（DI）による結合度の低減

3. **トランザクション境界の明確化**
   - トランザクション属性の適切な設定
   - 読み取り専用操作の最適化
   - 伝播属性の適切な選択

4. **例外ハンドリングの一貫性**
   - 例外の種類に応じた適切な処理
   - 例外の変換と伝播
   - 例外情報の適切な記録

5. **認可（Authorization）の実装**
   - メソッドレベルでの認可チェック
   - データレベルでの認可チェック
   - 認可失敗時の適切な例外発生

### 3. Service層の実装パターン

#### 基本的な実装パターン

TERASOLUNAフレームワークにおけるService層の基本的な実装パターンは以下の通りです：

**1. サービスインターフェース**

```java
package com.example.todo.domain.service.todo;

import java.util.List;
import java.util.Optional;

import com.example.todo.domain.model.Todo;

public interface TodoService {
    
    List<Todo> findAll();
    
    List<Todo> findByUserId(Long userId);
    
    List<Todo> findByUserIdAndCompleted(Long userId, boolean completed);
    
    Optional<Todo> findById(Long id);
    
    Todo create(Todo todo);
    
    Todo update(Todo todo);
    
    void delete(Long id);
}
```

**2. サービス実装クラス**

```java
package com.example.todo.domain.service.todo;

import java.util.List;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.todo.domain.model.Todo;
import com.example.todo.domain.repository.todo.TodoRepository;

@Service
@Transactional
public class TodoServiceImpl implements TodoService {
    
    @Autowired
    private TodoRepository todoRepository;
    
    @Override
    @Transactional(readOnly = true)
    public List<Todo> findAll() {
        return todoRepository.findAll();
    }
    
    @Override
    @Transactional(readOnly = true)
    public List<Todo> findByUserId(Long userId) {
        return todoRepository.findByUserId(userId);
    }
    
    @Override
    @Transactional(readOnly = true)
    public List<Todo> findByUserIdAndCompleted(Long userId, boolean completed) {
        return todoRepository.findByUserIdAndCompleted(userId, completed);
    }
    
    @Override
    @Transactional(readOnly = true)
    public Optional<Todo> findById(Long id) {
        return todoRepository.findById(id);
    }
    
    @Override
    public Todo create(Todo todo) {
        return todoRepository.save(todo);
    }
    
    @Override
    public Todo update(Todo todo) {
        return todoRepository.save(todo);
    }
    
    @Override
    public void delete(Long id) {
        todoRepository.deleteById(id);
    }
}
```

#### トランザクション管理

トランザクション管理は、`@Transactional`アノテーションを使用して行います。

```java
@Service
@Transactional
public class TodoServiceImpl implements TodoService {
    
    // クラスレベルで@Transactionalを指定すると、
    // すべてのメソッドがトランザクション管理の対象になる
    
    @Transactional(readOnly = true)
    public List<Todo> findAll() {
        // 読み取り専用のトランザクション
        // パフォーマンス最適化のため
        return todoRepository.findAll();
    }
    
    @Transactional(timeout = 10)
    public Todo create(Todo todo) {
        // タイムアウトを10秒に設定
        return todoRepository.save(todo);
    }
    
    @Transactional(rollbackFor = Exception.class)
    public Todo update(Todo todo) {
        // すべての例外でロールバック
        return todoRepository.save(todo);
    }
    
    @Transactional(noRollbackFor = NotFoundException.class)
    public void delete(Long id) {
        // NotFoundExceptionが発生してもロールバックしない
        todoRepository.deleteById(id);
    }
}
```

#### 例外ハンドリング

例外ハンドリングは、ビジネス例外とシステム例外に分けて行います。

```java
@Service
@Transactional
public class TodoServiceImpl implements TodoService {
    
    @Autowired
    private TodoRepository todoRepository;
    
    @Override
    @Transactional(readOnly = true)
    public Todo findOne(Long id) {
        return todoRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo not found"));
    }
    
    @Override
    public Todo create(Todo todo) {
        try {
            return todoRepository.save(todo);
        } catch (DataAccessException e) {
            throw new SystemException("Failed to create todo", e);
        }
    }
    
    @Override
    public Todo update(Todo todo) {
        // 楽観的ロックの例外ハンドリング
        try {
            return todoRepository.save(todo);
        } catch (OptimisticLockingFailureException e) {
            throw new BusinessException("Todo has been updated by another user", e);
        } catch (DataAccessException e) {
            throw new SystemException("Failed to update todo", e);
        }
    }
}
```

#### 認可（Authorization）

認可は、メソッドレベルとデータレベルで行います。

```java
@Service
@Transactional
public class TodoServiceImpl implements TodoService {
    
    @Autowired
    private TodoRepository todoRepository;
    
    @Override
    @Transactional(readOnly = true)
    public Todo findOne(Long id) {
        Todo todo = todoRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo not found"));
        
        // データレベルの認可チェック
        if (!todo.getUserId().equals(currentUser.getUserId())) {
            throw new AccessDeniedException("Access denied");
        }
        
        return todo;
    }
    
    @Override
    @PreAuthorize("hasRole('ADMIN')")
    public List<Todo> findAll() {
        // メソッドレベルの認可チェック
        // ADMIN権限を持つユーザーのみアクセス可能
        return todoRepository.findAll();
    }
}
```

#### ビジネスルールの実装

ビジネスルールは、Service層で実装します。

```java
@Service
@Transactional
public class TodoServiceImpl implements TodoService {
    
    @Autowired
    private TodoRepository todoRepository;
    
    @Override
    public Todo create(Todo todo) {
        // ビジネスルール: 同じタイトルのToDoは作成できない
        if (todoRepository.existsByUserIdAndTitle(todo.getUserId(), todo.getTitle())) {
            throw new BusinessException("Todo with the same title already exists");
        }
        
        // ビジネスルール: ToDoの期限は現在日時より後でなければならない
        if (todo.getDueDate() != null && todo.getDueDate().isBefore(LocalDateTime.now())) {
            throw new BusinessException("Due date must be in the future");
        }
        
        return todoRepository.save(todo);
    }
    
    @Override
    public Todo complete(Long id) {
        Todo todo = findOne(id);
        
        // ビジネスルール: 既に完了しているToDoは再度完了できない
        if (todo.isCompleted()) {
            throw new BusinessException("Todo is already completed");
        }
        
        todo.setCompleted(true);
        todo.setCompletedAt(LocalDateTime.now());
        
        return todoRepository.save(todo);
    }
}
```

### 4. ToDoアプリケーションのService層の実装

#### TodoService

ToDoアプリケーションのService層の実装例を示します。

**TodoService.java**

```java
package com.example.todo.domain.service.todo;

import java.util.List;
import java.util.Optional;

import com.example.todo.domain.model.Todo;

public interface TodoService {
    
    List<Todo> findAll();
    
    List<Todo> findByUserId(Long userId);
    
    List<Todo> findByUserIdAndCompleted(Long userId, boolean completed);
    
    Optional<Todo> findById(Long id);
    
    Todo findOne(Long id);
    
    Todo create(Todo todo);
    
    Todo update(Todo todo);
    
    void delete(Long id);
    
    Todo complete(Long id);
}
```

**TodoServiceImpl.java**

```java
package com.example.todo.domain.service.todo;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.dao.OptimisticLockingFailureException;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.todo.domain.exception.BusinessException;
import com.example.todo.domain.exception.ResourceNotFoundException;
import com.example.todo.domain.exception.SystemException;
import com.example.todo.domain.model.Todo;
import com.example.todo.domain.repository.todo.TodoRepository;

@Service
@Transactional
public class TodoServiceImpl implements TodoService {
    
    @Autowired
    private TodoRepository todoRepository;
    
    @Override
    @Transactional(readOnly = true)
    public List<Todo> findAll() {
        return todoRepository.findAll();
    }
    
    @Override
    @Transactional(readOnly = true)
    public List<Todo> findByUserId(Long userId) {
        return todoRepository.findByUserId(userId);
    }
    
    @Override
    @Transactional(readOnly = true)
    public List<Todo> findByUserIdAndCompleted(Long userId, boolean completed) {
        return todoRepository.findByUserIdAndCompleted(userId, completed);
    }
    
    @Override
    @Transactional(readOnly = true)
    public Optional<Todo> findById(Long id) {
        return todoRepository.findById(id);
    }
    
    @Override
    @Transactional(readOnly = true)
    public Todo findOne(Long id) {
        return todoRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo not found"));
    }
    
    @Override
    public Todo create(Todo todo) {
        // ビジネスルール: 同じタイトルのToDoは作成できない
        if (todoRepository.existsByUserIdAndTitle(todo.getUserId(), todo.getTitle())) {
            throw new BusinessException("Todo with the same title already exists");
        }
        
        // ビジネスルール: ToDoの期限は現在日時より後でなければならない
        if (todo.getDueDate() != null && todo.getDueDate().isBefore(LocalDateTime.now())) {
            throw new BusinessException("Due date must be in the future");
        }
        
        try {
            return todoRepository.save(todo);
        } catch (DataAccessException e) {
            throw new SystemException("Failed to create todo", e);
        }
    }
    
    @Override
    public Todo update(Todo todo) {
        // 存在チェック
        Todo currentTodo = findOne(todo.getId());
        
        // 所有者チェック
        if (!currentTodo.getUserId().equals(todo.getUserId())) {
            throw new AccessDeniedException("Access denied");
        }
        
        // ビジネスルール: 同じタイトルのToDoは作成できない（自分自身は除く）
        if (todoRepository.existsByUserIdAndTitleAndIdNot(
                todo.getUserId(), todo.getTitle(), todo.getId())) {
            throw new BusinessException("Todo with the same title already exists");
        }
        
        // ビジネスルール: ToDoの期限は現在日時より後でなければならない
        if (todo.getDueDate() != null && todo.getDueDate().isBefore(LocalDateTime.now())) {
            throw new BusinessException("Due date must be in the future");
        }
        
        try {
            return todoRepository.save(todo);
        } catch (OptimisticLockingFailureException e) {
            throw new BusinessException("Todo has been updated by another user", e);
        } catch (DataAccessException e) {
            throw new SystemException("Failed to update todo", e);
        }
    }
    
    @Override
    public void delete(Long id) {
        // 存在チェック
        Todo todo = findOne(id);
        
        try {
            todoRepository.deleteById(id);
        } catch (DataAccessException e) {
            throw new SystemException("Failed to delete todo", e);
        }
    }
    
    @Override
    public Todo complete(Long id) {
        // 存在チェック
        Todo todo = findOne(id);
        
        // ビジネスルール: 既に完了しているToDoは再度完了できない
        if (todo.isCompleted()) {
            throw new BusinessException("Todo is already completed");
        }
        
        todo.setCompleted(true);
        todo.setCompletedAt(LocalDateTime.now());
        
        try {
            return todoRepository.save(todo);
        } catch (OptimisticLockingFailureException e) {
            throw new BusinessException("Todo has been updated by another user", e);
        } catch (DataAccessException e) {
            throw new SystemException("Failed to complete todo", e);
        }
    }
}
```

#### UserService

ユーザー管理のService層の実装例を示します。

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
    
    User findOne(Long id);
    
    boolean exists(String username);
    
    User create(User user);
    
    User update(User user);
    
    void delete(Long id);
}
```

**UserServiceImpl.java**

```java
package com.example.todo.domain.service.user;

import java.util.List;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.dao.OptimisticLockingFailureException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.todo.domain.exception.BusinessException;
import com.example.todo.domain.exception.ResourceNotFoundException;
import com.example.todo.domain.exception.SystemException;
import com.example.todo.domain.model.User;
import com.example.todo.domain.repository.user.UserRepository;

@Service
@Transactional
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Override
    @Transactional(readOnly = true)
    public List<User> findAll() {
        return userRepository.findAll();
    }
    
    @Override
    @Transactional(readOnly = true)
    public Optional<User> findById(Long id) {
        return userRepository.findById(id);
    }
    
    @Override
    @Transactional(readOnly = true)
    public Optional<User> findByUsername(String username) {
        return userRepository.findByUsername(username);
    }
    
    @Override
    @Transactional(readOnly = true)
    public User findOne(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found"));
    }
    
    @Override
    @Transactional(readOnly = true)
    public boolean exists(String username) {
        return userRepository.existsByUsername(username);
    }
    
    @Override
    public User create(User user) {
        // ビジネスルール: ユーザー名は一意でなければならない
        if (exists(user.getUsername())) {
            throw new BusinessException("Username already exists");
        }
        
        // パスワードのハッシュ化
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        
        try {
            return userRepository.save(user);
        } catch (DataAccessException e) {
            throw new SystemException("Failed to create user", e);
        }
    }
    
    @Override
    public User update(User user) {
        // 存在チェック
        User currentUser = findOne(user.getId());
        
        // ビジネスルール: ユーザー名は一意でなければならない（自分自身は除く）
        if (userRepository.existsByUsernameAndIdNot(
                user.getUsername(), user.getId())) {
            throw new BusinessException("Username already exists");
        }
        
        // パスワードが変更されている場合のみハッシュ化
        if (!user.getPassword().equals(currentUser.getPassword())) {
            user.setPassword(passwordEncoder.encode(user.getPassword()));
        }
        
        try {
            return userRepository.save(user);
        } catch (OptimisticLockingFailureException e) {
            throw new BusinessException("User has been updated by another user", e);
        } catch (DataAccessException e) {
            throw new SystemException("Failed to update user", e);
        }
    }
    
    @Override
    public void delete(Long id) {
        // 存在チェック
        User user = findOne(id);
        
        try {
            userRepository.deleteById(id);
        } catch (DataAccessException e) {
            throw new SystemException("Failed to delete user", e);
        }
    }
}
```

### 5. Service層のテスト

Service層のテストは、単体テストと統合テストに分けて行います。

#### 単体テスト

単体テストは、依存オブジェクトをモック化して行います。

```java
package com.example.todo.domain.service.todo;

import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

import java.time.LocalDateTime;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import org.junit.Before;
import org.junit.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import com.example.todo.domain.exception.BusinessException;
import com.example.todo.domain.exception.ResourceNotFoundException;
import com.example.todo.domain.model.Todo;
import com.example.todo.domain.repository.todo.TodoRepository;

public class TodoServiceImplTest {
    
    @Mock
    private TodoRepository todoRepository;
    
    @InjectMocks
    private TodoServiceImpl todoService;
    
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }
    
    @Test
    public void testFindAll() {
        // 準備
        Todo todo1 = new Todo();
        todo1.setId(1L);
        todo1.setTitle("Todo 1");
        
        Todo todo2 = new Todo();
        todo2.setId(2L);
        todo2.setTitle("Todo 2");
        
        List<Todo> todos = Arrays.asList(todo1, todo2);
        
        when(todoRepository.findAll()).thenReturn(todos);
        
        // 実行
        List<Todo> result = todoService.findAll();
        
        // 検証
        assertEquals(2, result.size());
        assertEquals("Todo 1", result.get(0).getTitle());
        assertEquals("Todo 2", result.get(1).getTitle());
    }
    
    @Test
    public void testFindOne() {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Todo 1");
        
        when(todoRepository.findById(1L)).thenReturn(Optional.of(todo));
        
        // 実行
        Todo result = todoService.findOne(1L);
        
        // 検証
        assertEquals("Todo 1", result.getTitle());
    }
    
    @Test(expected = ResourceNotFoundException.class)
    public void testFindOneNotFound() {
        // 準備
        when(todoRepository.findById(1L)).thenReturn(Optional.empty());
        
        // 実行
        todoService.findOne(1L);
        
        // 例外が発生することを期待
    }
    
    @Test
    public void testCreate() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("New Todo");
        todo.setUserId(1L);
        todo.setDueDate(LocalDateTime.now().plusDays(1));
        
        when(todoRepository.existsByUserIdAndTitle(1L, "New Todo")).thenReturn(false);
        when(todoRepository.save(todo)).thenReturn(todo);
        
        // 実行
        Todo result = todoService.create(todo);
        
        // 検証
        assertEquals("New Todo", result.getTitle());
    }
    
    @Test(expected = BusinessException.class)
    public void testCreateDuplicateTitle() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("Duplicate Todo");
        todo.setUserId(1L);
        todo.setDueDate(LocalDateTime.now().plusDays(1));
        
        when(todoRepository.existsByUserIdAndTitle(1L, "Duplicate Todo")).thenReturn(true);
        
        // 実行
        todoService.create(todo);
        
        // 例外が発生することを期待
    }
    
    @Test(expected = BusinessException.class)
    public void testCreatePastDueDate() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("Past Due Todo");
        todo.setUserId(1L);
        todo.setDueDate(LocalDateTime.now().minusDays(1));
        
        when(todoRepository.existsByUserIdAndTitle(1L, "Past Due Todo")).thenReturn(false);
        
        // 実行
        todoService.create(todo);
        
        // 例外が発生することを期待
    }
    
    @Test
    public void testComplete() {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Todo 1");
        todo.setCompleted(false);
        
        when(todoRepository.findById(1L)).thenReturn(Optional.of(todo));
        when(todoRepository.save(todo)).thenReturn(todo);
        
        // 実行
        Todo result = todoService.complete(1L);
        
        // 検証
        assertTrue(result.isCompleted());
        assertNotNull(result.getCompletedAt());
    }
    
    @Test(expected = BusinessException.class)
    public void testCompleteAlreadyCompleted() {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Todo 1");
        todo.setCompleted(true);
        
        when(todoRepository.findById(1L)).thenReturn(Optional.of(todo));
        
        // 実行
        todoService.complete(1L);
        
        // 例外が発生することを期待
    }
}
```

## 実践課題（演習）

### 課題1: TodoServiceの実装

以下の要件に基づいて、TodoServiceを実装してください。

**要件:**
1. ToDoの一覧取得機能
2. ToDoの詳細取得機能
3. ToDoの新規作成機能
4. ToDoの更新機能
5. ToDoの削除機能
6. ToDoの完了機能
7. ユーザーIDによるフィルタリング機能
8. 完了状態によるフィルタリング機能
9. ビジネスルールの実装
10. 例外ハンドリング

### 課題2: UserServiceの実装

以下の要件に基づいて、UserServiceを実装してください。

**要件:**
1. ユーザーの一覧取得機能
2. ユーザーの詳細取得機能
3. ユーザーの新規作成機能
4. ユーザーの更新機能
5. ユーザーの削除機能
6. ユーザー名による検索機能
7. ユーザー名の重複チェック機能
8. パスワードのハッシュ化
9. ビジネスルールの実装
10. 例外ハンドリング

### 課題3: UserDetailsServiceの実装

以下の要件に基づいて、UserDetailsServiceを実装してください。

**要件:**
1. ユーザー名によるユーザー検索機能
2. Spring Securityの認証に必要な情報の提供
3. ユーザーの権限の設定
4. 例外ハンドリング

### 課題4: Service層のテスト

以下の要件に基づいて、TodoServiceのテストを実装してください。

**要件:**
1. 一覧取得機能のテスト
2. 詳細取得機能のテスト
3. 新規作成機能のテスト
4. 更新機能のテスト
5. 削除機能のテスト
6. 完了機能のテスト
7. ビジネスルールのテスト
8. 例外ハンドリングのテスト

## 実務での活用シーン

### 1. 業務アプリケーション開発

Service層の実装は、業務アプリケーション開発の中核となります。以下のような場面で活用されます：

- **企業の基幹システム開発**
  - 複雑なビジネスルールの実装
  - 複数のデータソースの連携
  - トランザクション管理

- **金融システム開発**
  - 厳格なビジネスルールの実装
  - 高い整合性の確保
  - 監査証跡の記録

- **ECサイトの開発**
  - 注文処理のビジネスロジック
  - 在庫管理のビジネスロジック
  - 決済処理のビジネスロジック

### 2. マイクロサービスアーキテクチャ

Service層の実装は、マイクロサービスアーキテクチャにおいても重要です：

- **サービス境界の定義**
  - ビジネスドメインに基づくサービスの分割
  - サービス間の依存関係の管理
  - APIの設計

- **分散トランザクション**
  - サービス間のトランザクション管理
  - 補償トランザクションの実装
  - 結果整合性の確保

- **認証・認可**
  - サービス間の認証
  - リソースへのアクセス制御
  - トークンベースの認証

### 3. バッチ処理

Service層の実装は、バッチ処理においても重要です：

- **大量データ処理**
  - 効率的なデータ処理
  - トランザクション管理
  - エラーハンドリング

- **定期的な処理**
  - スケジュール管理
  - ジョブの実行制御
  - 実行結果の管理

- **データ連携**
  - 外部システムとのデータ連携
  - データ変換
  - エラーリカバリ

### 4. API開発

Service層の実装は、API開発においても重要です：

- **RESTful API**
  - リソース指向の設計
  - HTTPメソッドとの対応
  - ステータスコードの管理

- **GraphQL API**
  - リゾルバの実装
  - データローダの実装
  - 認証・認可の実装

- **WebSocket API**
  - リアルタイム通信の実装
  - メッセージの処理
  - セッション管理

## 解答例

<a href="answers/day16">解答例はこちら</a>