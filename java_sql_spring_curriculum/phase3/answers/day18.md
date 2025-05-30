# Day 18: Domain層の実装 - 解答例

## 課題1: Todoエンティティの実装

**Todo.java**

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

## 課題2: TodoDomainServiceの実装

**TodoDomainService.java**

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

## 課題3: TodoDomainServiceのテスト

**TodoDomainServiceTest.java**

```java
package com.example.todo.domain.service;

import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

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

public class TodoDomainServiceTest {
    
    @InjectMocks
    private TodoDomainService todoDomainService;
    
    @Mock
    private TodoRepository todoRepository;
    
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }
    
    @Test
    public void testFindOne_Success() {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Test Todo");
        
        when(todoRepository.findById(1L)).thenReturn(Optional.of(todo));
        
        // 実行
        Todo result = todoDomainService.findOne(1L);
        
        // 検証
        assertNotNull(result);
        assertEquals(Long.valueOf(1L), result.getId());
        assertEquals("Test Todo", result.getTitle());
        
        verify(todoRepository).findById(1L);
    }
    
    @Test(expected = ResourceNotFoundException.class)
    public void testFindOne_NotFound() {
        // 準備
        when(todoRepository.findById(1L)).thenReturn(Optional.empty());
        
        // 実行
        todoDomainService.findOne(1L);
        
        // 検証
        verify(todoRepository).findById(1L);
    }
    
    @Test
    public void testExistsByUserIdAndTitle_True() {
        // 準備
        when(todoRepository.existsByUserIdAndTitle(1L, "Test Todo")).thenReturn(true);
        
        // 実行
        boolean result = todoDomainService.existsByUserIdAndTitle(1L, "Test Todo");
        
        // 検証
        assertTrue(result);
        
        verify(todoRepository).existsByUserIdAndTitle(1L, "Test Todo");
    }
    
    @Test
    public void testExistsByUserIdAndTitle_False() {
        // 準備
        when(todoRepository.existsByUserIdAndTitle(1L, "Test Todo")).thenReturn(false);
        
        // 実行
        boolean result = todoDomainService.existsByUserIdAndTitle(1L, "Test Todo");
        
        // 検証
        assertFalse(result);
        
        verify(todoRepository).existsByUserIdAndTitle(1L, "Test Todo");
    }
    
    @Test
    public void testExistsByUserIdAndTitleAndIdNot_True() {
        // 準備
        when(todoRepository.existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 2L)).thenReturn(true);
        
        // 実行
        boolean result = todoDomainService.existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 2L);
        
        // 検証
        assertTrue(result);
        
        verify(todoRepository).existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 2L);
    }
    
    @Test
    public void testExistsByUserIdAndTitleAndIdNot_False() {
        // 準備
        when(todoRepository.existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 2L)).thenReturn(false);
        
        // 実行
        boolean result = todoDomainService.existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 2L);
        
        // 検証
        assertFalse(result);
        
        verify(todoRepository).existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 2L);
    }
    
    @Test
    public void testIsOwnedBy_Success() {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Test Todo");
        todo.setUserId(1L);
        
        // 実行
        boolean result = todoDomainService.isOwnedBy(todo, 1L);
        
        // 検証
        assertTrue(result);
    }
    
    @Test(expected = BusinessException.class)
    public void testIsOwnedBy_Failure() {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Test Todo");
        todo.setUserId(1L);
        
        // 実行
        todoDomainService.isOwnedBy(todo, 2L);
    }
    
    @Test
    public void testCreate_Success() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("Test Todo");
        todo.setUserId(1L);
        
        when(todoRepository.existsByUserIdAndTitle(1L, "Test Todo")).thenReturn(false);
        when(todoRepository.save(todo)).thenReturn(todo);
        
        // 実行
        Todo result = todoDomainService.create(todo);
        
        // 検証
        assertNotNull(result);
        assertEquals("Test Todo", result.getTitle());
        
        verify(todoRepository).existsByUserIdAndTitle(1L, "Test Todo");
        verify(todoRepository).save(todo);
    }
    
    @Test(expected = BusinessException.class)
    public void testCreate_AlreadyExists() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("Test Todo");
        todo.setUserId(1L);
        
        when(todoRepository.existsByUserIdAndTitle(1L, "Test Todo")).thenReturn(true);
        
        // 実行
        todoDomainService.create(todo);
        
        // 検証
        verify(todoRepository).existsByUserIdAndTitle(1L, "Test Todo");
        verify(todoRepository, never()).save(any(Todo.class));
    }
    
    @Test
    public void testUpdate_Success() {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Test Todo");
        todo.setUserId(1L);
        
        when(todoRepository.existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 1L)).thenReturn(false);
        when(todoRepository.save(todo)).thenReturn(todo);
        
        // 実行
        Todo result = todoDomainService.update(todo);
        
        // 検証
        assertNotNull(result);
        assertEquals("Test Todo", result.getTitle());
        
        verify(todoRepository).existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 1L);
        verify(todoRepository).save(todo);
    }
    
    @Test(expected = BusinessException.class)
    public void testUpdate_AlreadyExists() {
        // 準備
        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle("Test Todo");
        todo.setUserId(1L);
        
        when(todoRepository.existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 1L)).thenReturn(true);
        
        // 実行
        todoDomainService.update(todo);
        
        // 検証
        verify(todoRepository).existsByUserIdAndTitleAndIdNot(1L, "Test Todo", 1L);
        verify(todoRepository, never()).save(any(Todo.class));
    }
    
    @Test
    public void testDelete() {
        // 実行
        todoDomainService.delete(1L);
        
        // 検証
        verify(todoRepository).deleteById(1L);
    }
}
```

## 課題4: ドメイン例外の実装

**BusinessException.java**

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

**ResourceNotFoundException.java**

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

### 解説

課題1～4の解答例では、TERASOLUNAフレームワークの推奨パターンに基づいて、Domain層の実装を行いました。

1. **Todoエンティティの実装**
   - シリアライズ可能なエンティティクラスの定義
   - ドメイン固有の振る舞いの実装（complete、incomplete、isOwnedBy）
   - Lombokを使用した冗長なコードの削減

2. **TodoDomainServiceの実装**
   - ドメイン固有のビジネスロジックの実装
   - トランザクション管理の適用
   - 例外ハンドリングの実装
   - リポジトリとの連携

3. **TodoDomainServiceのテスト**
   - Mockitoを使用したモックオブジェクトの作成
   - 正常系テストと異常系テストの実装
   - 境界条件のテスト
   - モックの検証

4. **ドメイン例外の実装**
   - 例外の階層化
   - シリアライズ可能な例外クラスの定義
   - 例外コードと例外メッセージの管理

これらの実装は、ドメイン駆動設計（DDD）の考え方に基づいており、ビジネスロジックを適切に表現し、アプリケーションの中核となる部分を構築しています。特に、ドメイン固有の振る舞いをエンティティに実装し、ドメインサービスでビジネスルールを実装することで、アプリケーションの本質的な機能を明確に表現しています。また、例外ハンドリングを適切に行うことで、エラー状態を適切に管理し、アプリケーションの堅牢性を高めています。
