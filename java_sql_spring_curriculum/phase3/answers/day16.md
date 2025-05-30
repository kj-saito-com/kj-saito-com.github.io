# Day 16: Service層の実装 - 解答例

## 課題1: TodoServiceの実装

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

## 課題2: UserServiceの実装

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

## 課題3: UserDetailsServiceの実装

**TodoUserDetailsService.java**

```java
package com.example.todo.domain.service.user;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.todo.domain.model.User;
import com.example.todo.domain.repository.user.UserRepository;

@Service
public class TodoUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    @Transactional(readOnly = true)
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        
        List<GrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
        
        return new com.example.todo.domain.service.user.UserDetails(
                user.getId(),
                user.getUsername(),
                user.getPassword(),
                user.isEnabled(),
                true,
                true,
                true,
                authorities);
    }
}
```

**UserDetails.java**

```java
package com.example.todo.domain.service.user;

import java.util.Collection;

import org.springframework.security.core.GrantedAuthority;

public class UserDetails extends org.springframework.security.core.userdetails.User {
    
    private static final long serialVersionUID = 1L;
    
    private Long userId;
    
    public UserDetails(Long userId, String username, String password,
            boolean enabled, boolean accountNonExpired,
            boolean credentialsNonExpired, boolean accountNonLocked,
            Collection<? extends GrantedAuthority> authorities) {
        super(username, password, enabled, accountNonExpired,
                credentialsNonExpired, accountNonLocked, authorities);
        this.userId = userId;
    }
    
    public Long getUserId() {
        return userId;
    }
}
```

## 課題4: Service層のテスト

**TodoServiceImplTest.java**

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

### 解説

課題1～4の解答例では、TERASOLUNAフレームワークの推奨パターンに基づいて、Service層の実装を行いました。

1. **TodoServiceの実装**
   - インターフェースと実装クラスの分離
   - トランザクション管理（@Transactional）
   - 読み取り専用操作の最適化（readOnly = true）
   - ビジネスルールの実装（タイトルの重複チェック、期限の妥当性チェック）
   - 例外ハンドリング（BusinessException、ResourceNotFoundException、SystemException）
   - データレベルの認可チェック（所有者チェック）

2. **UserServiceの実装**
   - インターフェースと実装クラスの分離
   - トランザクション管理
   - パスワードのハッシュ化（PasswordEncoder）
   - ビジネスルールの実装（ユーザー名の重複チェック）
   - 例外ハンドリング

3. **UserDetailsServiceの実装**
   - Spring Securityとの連携
   - ユーザー認証情報の提供
   - 権限の設定（ROLE_USER）
   - カスタムUserDetailsの実装（ユーザーIDの保持）

4. **Service層のテスト**
   - Mockitoを使用したモックオブジェクトの作成
   - 正常系テスト（findAll、findOne、create、complete）
   - 異常系テスト（findOneNotFound、createDuplicateTitle、createPastDueDate、completeAlreadyCompleted）
   - 例外のテスト（@Test(expected = ...)）

これらの実装は、実務でのWebアプリケーション開発において、保守性と拡張性の高いService層を構築するための基礎となります。特に、ビジネスルールの実装、トランザクション管理、例外ハンドリングは、アプリケーションの品質と信頼性を確保する上で重要です。また、テストの実装により、コードの品質と信頼性を確保することができます。
