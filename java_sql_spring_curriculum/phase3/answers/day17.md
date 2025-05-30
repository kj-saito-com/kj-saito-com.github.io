# Day 17: Repository層の実装 - 解答例

## 課題1: TodoRepositoryの実装

**TodoRepository.java**

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

**TodoRepositoryImpl.java**

```java
package com.example.todo.domain.repository.todo;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Repository;

import com.example.todo.domain.model.Todo;

@Repository
public class TodoRepositoryImpl implements TodoRepository {
    
    @Autowired
    private TodoMapper todoMapper;
    
    @Override
    public List<Todo> findAll() {
        return todoMapper.findAll();
    }
    
    @Override
    public List<Todo> findByUserId(Long userId) {
        return todoMapper.findByUserId(userId);
    }
    
    @Override
    public List<Todo> findByUserIdAndCompleted(Long userId, boolean completed) {
        return todoMapper.findByUserIdAndCompleted(userId, completed);
    }
    
    @Override
    public Optional<Todo> findById(Long id) {
        Todo todo = todoMapper.findById(id);
        return Optional.ofNullable(todo);
    }
    
    @Override
    public boolean existsByUserIdAndTitle(Long userId, String title) {
        return todoMapper.countByUserIdAndTitle(userId, title) > 0;
    }
    
    @Override
    public boolean existsByUserIdAndTitleAndIdNot(Long userId, String title, Long id) {
        return todoMapper.countByUserIdAndTitleAndIdNot(userId, title, id) > 0;
    }
    
    @Override
    public Todo save(Todo todo) {
        if (todo.getId() == null) {
            // 新規作成
            todo.setCreatedAt(LocalDateTime.now());
            todo.setUpdatedAt(LocalDateTime.now());
            todoMapper.insert(todo);
        } else {
            // 更新
            todo.setUpdatedAt(LocalDateTime.now());
            todoMapper.update(todo);
        }
        return todo;
    }
    
    @Override
    public void deleteById(Long id) {
        todoMapper.delete(id);
    }
    
    @Override
    public Page<Todo> findPage(TodoSearchCriteria criteria, Pageable pageable) {
        List<Todo> content = todoMapper.findPage(criteria, pageable);
        long total = count(criteria);
        return new PageImpl<>(content, pageable, total);
    }
    
    @Override
    public long count(TodoSearchCriteria criteria) {
        return todoMapper.count(criteria);
    }
}
```

**TodoMapper.java**

```java
package com.example.todo.domain.repository.todo;

import java.util.List;

import org.apache.ibatis.annotations.Param;
import org.springframework.data.domain.Pageable;

import com.example.todo.domain.model.Todo;

public interface TodoMapper {
    
    List<Todo> findAll();
    
    List<Todo> findByUserId(Long userId);
    
    List<Todo> findByUserIdAndCompleted(@Param("userId") Long userId, @Param("completed") boolean completed);
    
    Todo findById(Long id);
    
    long countByUserIdAndTitle(@Param("userId") Long userId, @Param("title") String title);
    
    long countByUserIdAndTitleAndIdNot(@Param("userId") Long userId, @Param("title") String title, @Param("id") Long id);
    
    void insert(Todo todo);
    
    void update(Todo todo);
    
    void delete(Long id);
    
    List<Todo> findPage(@Param("criteria") TodoSearchCriteria criteria, @Param("pageable") Pageable pageable);
    
    long count(TodoSearchCriteria criteria);
}
```

**TodoMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.todo.domain.repository.todo.TodoMapper">

    <resultMap id="todoResultMap" type="com.example.todo.domain.model.Todo">
        <id property="id" column="id" />
        <result property="title" column="title" />
        <result property="description" column="description" />
        <result property="completed" column="completed" />
        <result property="createdAt" column="created_at" />
        <result property="updatedAt" column="updated_at" />
        <result property="completedAt" column="completed_at" />
        <result property="userId" column="user_id" />
        <result property="version" column="version" />
    </resultMap>

    <select id="findAll" resultMap="todoResultMap">
        SELECT
            id,
            title,
            description,
            completed,
            created_at,
            updated_at,
            completed_at,
            user_id,
            version
        FROM
            todo
        ORDER BY
            id
    </select>

    <select id="findByUserId" parameterType="long" resultMap="todoResultMap">
        SELECT
            id,
            title,
            description,
            completed,
            created_at,
            updated_at,
            completed_at,
            user_id,
            version
        FROM
            todo
        WHERE
            user_id = #{userId}
        ORDER BY
            id
    </select>

    <select id="findByUserIdAndCompleted" resultMap="todoResultMap">
        SELECT
            id,
            title,
            description,
            completed,
            created_at,
            updated_at,
            completed_at,
            user_id,
            version
        FROM
            todo
        WHERE
            user_id = #{userId}
            AND completed = #{completed}
        ORDER BY
            id
    </select>

    <select id="findById" parameterType="long" resultMap="todoResultMap">
        SELECT
            id,
            title,
            description,
            completed,
            created_at,
            updated_at,
            completed_at,
            user_id,
            version
        FROM
            todo
        WHERE
            id = #{id}
    </select>

    <select id="countByUserIdAndTitle" resultType="long">
        SELECT
            COUNT(*)
        FROM
            todo
        WHERE
            user_id = #{userId}
            AND title = #{title}
    </select>

    <select id="countByUserIdAndTitleAndIdNot" resultType="long">
        SELECT
            COUNT(*)
        FROM
            todo
        WHERE
            user_id = #{userId}
            AND title = #{title}
            AND id != #{id}
    </select>

    <insert id="insert" parameterType="com.example.todo.domain.model.Todo" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO todo (
            title,
            description,
            completed,
            created_at,
            updated_at,
            completed_at,
            user_id,
            version
        ) VALUES (
            #{title},
            #{description},
            #{completed},
            #{createdAt},
            #{updatedAt},
            #{completedAt},
            #{userId},
            1
        )
    </insert>

    <update id="update" parameterType="com.example.todo.domain.model.Todo">
        UPDATE
            todo
        SET
            title = #{title},
            description = #{description},
            completed = #{completed},
            updated_at = #{updatedAt},
            completed_at = #{completedAt},
            version = version + 1
        WHERE
            id = #{id}
            AND version = #{version}
    </update>

    <delete id="delete" parameterType="long">
        DELETE FROM
            todo
        WHERE
            id = #{id}
    </delete>

    <select id="findPage" resultMap="todoResultMap">
        SELECT
            id,
            title,
            description,
            completed,
            created_at,
            updated_at,
            completed_at,
            user_id,
            version
        FROM
            todo
        <where>
            <if test="criteria.userId != null">
                user_id = #{criteria.userId}
            </if>
            <if test="criteria.completed != null">
                AND completed = #{criteria.completed}
            </if>
            <if test="criteria.title != null and criteria.title != ''">
                AND title LIKE '%' || #{criteria.title} || '%'
            </if>
        </where>
        ORDER BY
            <choose>
                <when test="criteria.orderBy != null and criteria.orderBy == 'title'">
                    title
                </when>
                <when test="criteria.orderBy != null and criteria.orderBy == 'createdAt'">
                    created_at
                </when>
                <otherwise>
                    id
                </otherwise>
            </choose>
            <if test="criteria.ascending != null and criteria.ascending == true">
                ASC
            </if>
            <if test="criteria.ascending == null or criteria.ascending == false">
                DESC
            </if>
        LIMIT #{pageable.pageSize} OFFSET #{pageable.offset}
    </select>

    <select id="count" parameterType="com.example.todo.domain.repository.todo.TodoSearchCriteria" resultType="long">
        SELECT
            COUNT(*)
        FROM
            todo
        <where>
            <if test="userId != null">
                user_id = #{userId}
            </if>
            <if test="completed != null">
                AND completed = #{completed}
            </if>
            <if test="title != null and title != ''">
                AND title LIKE '%' || #{title} || '%'
            </if>
        </where>
    </select>
</mapper>
```

**TodoSearchCriteria.java**

```java
package com.example.todo.domain.repository.todo;

import java.io.Serializable;

import lombok.Data;

@Data
public class TodoSearchCriteria implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private Long userId;
    
    private Boolean completed;
    
    private String title;
    
    private String orderBy;
    
    private Boolean ascending;
}
```

## 課題2: UserRepositoryの実装

**UserRepository.java**

```java
package com.example.todo.domain.repository.user;

import java.util.List;
import java.util.Optional;

import com.example.todo.domain.model.User;

public interface UserRepository {
    
    List<User> findAll();
    
    Optional<User> findById(Long id);
    
    Optional<User> findByUsername(String username);
    
    boolean existsByUsername(String username);
    
    boolean existsByUsernameAndIdNot(String username, Long id);
    
    User save(User user);
    
    void deleteById(Long id);
}
```

**UserRepositoryImpl.java**

```java
package com.example.todo.domain.repository.user;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import com.example.todo.domain.model.User;

@Repository
public class UserRepositoryImpl implements UserRepository {
    
    @Autowired
    private UserMapper userMapper;
    
    @Override
    public List<User> findAll() {
        return userMapper.findAll();
    }
    
    @Override
    public Optional<User> findById(Long id) {
        User user = userMapper.findById(id);
        return Optional.ofNullable(user);
    }
    
    @Override
    public Optional<User> findByUsername(String username) {
        User user = userMapper.findByUsername(username);
        return Optional.ofNullable(user);
    }
    
    @Override
    public boolean existsByUsername(String username) {
        return userMapper.countByUsername(username) > 0;
    }
    
    @Override
    public boolean existsByUsernameAndIdNot(String username, Long id) {
        return userMapper.countByUsernameAndIdNot(username, id) > 0;
    }
    
    @Override
    public User save(User user) {
        if (user.getId() == null) {
            // 新規作成
            user.setCreatedAt(LocalDateTime.now());
            user.setUpdatedAt(LocalDateTime.now());
            userMapper.insert(user);
        } else {
            // 更新
            user.setUpdatedAt(LocalDateTime.now());
            userMapper.update(user);
        }
        return user;
    }
    
    @Override
    public void deleteById(Long id) {
        userMapper.delete(id);
    }
}
```

**UserMapper.java**

```java
package com.example.todo.domain.repository.user;

import java.util.List;

import org.apache.ibatis.annotations.Param;

import com.example.todo.domain.model.User;

public interface UserMapper {
    
    List<User> findAll();
    
    User findById(Long id);
    
    User findByUsername(String username);
    
    long countByUsername(String username);
    
    long countByUsernameAndIdNot(@Param("username") String username, @Param("id") Long id);
    
    void insert(User user);
    
    void update(User user);
    
    void delete(Long id);
}
```

**UserMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.todo.domain.repository.user.UserMapper">

    <resultMap id="userResultMap" type="com.example.todo.domain.model.User">
        <id property="id" column="id" />
        <result property="username" column="username" />
        <result property="password" column="password" />
        <result property="email" column="email" />
        <result property="enabled" column="enabled" />
        <result property="createdAt" column="created_at" />
        <result property="updatedAt" column="updated_at" />
        <result property="version" column="version" />
    </resultMap>

    <select id="findAll" resultMap="userResultMap">
        SELECT
            id,
            username,
            password,
            email,
            enabled,
            created_at,
            updated_at,
            version
        FROM
            app_user
        ORDER BY
            id
    </select>

    <select id="findById" parameterType="long" resultMap="userResultMap">
        SELECT
            id,
            username,
            password,
            email,
            enabled,
            created_at,
            updated_at,
            version
        FROM
            app_user
        WHERE
            id = #{id}
    </select>

    <select id="findByUsername" parameterType="string" resultMap="userResultMap">
        SELECT
            id,
            username,
            password,
            email,
            enabled,
            created_at,
            updated_at,
            version
        FROM
            app_user
        WHERE
            username = #{username}
    </select>

    <select id="countByUsername" parameterType="string" resultType="long">
        SELECT
            COUNT(*)
        FROM
            app_user
        WHERE
            username = #{username}
    </select>

    <select id="countByUsernameAndIdNot" resultType="long">
        SELECT
            COUNT(*)
        FROM
            app_user
        WHERE
            username = #{username}
            AND id != #{id}
    </select>

    <insert id="insert" parameterType="com.example.todo.domain.model.User" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO app_user (
            username,
            password,
            email,
            enabled,
            created_at,
            updated_at,
            version
        ) VALUES (
            #{username},
            #{password},
            #{email},
            #{enabled},
            #{createdAt},
            #{updatedAt},
            1
        )
    </insert>

    <update id="update" parameterType="com.example.todo.domain.model.User">
        UPDATE
            app_user
        SET
            username = #{username},
            password = #{password},
            email = #{email},
            enabled = #{enabled},
            updated_at = #{updatedAt},
            version = version + 1
        WHERE
            id = #{id}
            AND version = #{version}
    </update>

    <delete id="delete" parameterType="long">
        DELETE FROM
            app_user
        WHERE
            id = #{id}
    </delete>
</mapper>
```

## 課題3: Repository層のテスト

**TodoRepositoryTest.java**

```java
package com.example.todo.domain.repository.todo;

import static org.junit.Assert.*;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.dao.OptimisticLockingFailureException;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import com.example.todo.domain.model.Todo;

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class TodoRepositoryTest {
    
    @Autowired
    private TodoRepository todoRepository;
    
    @Test
    public void testFindAll() {
        // 準備
        Todo todo1 = new Todo();
        todo1.setTitle("Todo 1");
        todo1.setDescription("Description 1");
        todo1.setCompleted(false);
        todo1.setUserId(1L);
        
        Todo todo2 = new Todo();
        todo2.setTitle("Todo 2");
        todo2.setDescription("Description 2");
        todo2.setCompleted(false);
        todo2.setUserId(1L);
        
        todoRepository.save(todo1);
        todoRepository.save(todo2);
        
        // 実行
        List<Todo> todos = todoRepository.findAll();
        
        // 検証
        assertNotNull(todos);
        assertTrue(todos.size() >= 2);
    }
    
    @Test
    public void testFindByUserId() {
        // 準備
        Todo todo1 = new Todo();
        todo1.setTitle("Todo 1");
        todo1.setDescription("Description 1");
        todo1.setCompleted(false);
        todo1.setUserId(1L);
        
        Todo todo2 = new Todo();
        todo2.setTitle("Todo 2");
        todo2.setDescription("Description 2");
        todo2.setCompleted(false);
        todo2.setUserId(2L);
        
        todoRepository.save(todo1);
        todoRepository.save(todo2);
        
        // 実行
        List<Todo> todos = todoRepository.findByUserId(1L);
        
        // 検証
        assertNotNull(todos);
        assertTrue(todos.size() >= 1);
        for (Todo todo : todos) {
            assertEquals(Long.valueOf(1L), todo.getUserId());
        }
    }
    
    @Test
    public void testFindByUserIdAndCompleted() {
        // 準備
        Todo todo1 = new Todo();
        todo1.setTitle("Todo 1");
        todo1.setDescription("Description 1");
        todo1.setCompleted(true);
        todo1.setUserId(1L);
        
        Todo todo2 = new Todo();
        todo2.setTitle("Todo 2");
        todo2.setDescription("Description 2");
        todo2.setCompleted(false);
        todo2.setUserId(1L);
        
        todoRepository.save(todo1);
        todoRepository.save(todo2);
        
        // 実行
        List<Todo> completedTodos = todoRepository.findByUserIdAndCompleted(1L, true);
        List<Todo> incompleteTodos = todoRepository.findByUserIdAndCompleted(1L, false);
        
        // 検証
        assertNotNull(completedTodos);
        assertNotNull(incompleteTodos);
        
        for (Todo todo : completedTodos) {
            assertEquals(Long.valueOf(1L), todo.getUserId());
            assertTrue(todo.isCompleted());
        }
        
        for (Todo todo : incompleteTodos) {
            assertEquals(Long.valueOf(1L), todo.getUserId());
            assertFalse(todo.isCompleted());
        }
    }
    
    @Test
    public void testFindById() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("Todo 1");
        todo.setDescription("Description 1");
        todo.setCompleted(false);
        todo.setUserId(1L);
        
        todo = todoRepository.save(todo);
        
        // 実行
        Optional<Todo> found = todoRepository.findById(todo.getId());
        
        // 検証
        assertTrue(found.isPresent());
        assertEquals(todo.getId(), found.get().getId());
        assertEquals(todo.getTitle(), found.get().getTitle());
    }
    
    @Test
    public void testExistsByUserIdAndTitle() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("Unique Title");
        todo.setDescription("Description");
        todo.setCompleted(false);
        todo.setUserId(1L);
        
        todoRepository.save(todo);
        
        // 実行と検証
        assertTrue(todoRepository.existsByUserIdAndTitle(1L, "Unique Title"));
        assertFalse(todoRepository.existsByUserIdAndTitle(1L, "Non-existent Title"));
        assertFalse(todoRepository.existsByUserIdAndTitle(2L, "Unique Title"));
    }
    
    @Test
    public void testExistsByUserIdAndTitleAndIdNot() {
        // 準備
        Todo todo1 = new Todo();
        todo1.setTitle("Unique Title");
        todo1.setDescription("Description 1");
        todo1.setCompleted(false);
        todo1.setUserId(1L);
        
        Todo todo2 = new Todo();
        todo2.setTitle("Another Title");
        todo2.setDescription("Description 2");
        todo2.setCompleted(false);
        todo2.setUserId(1L);
        
        todo1 = todoRepository.save(todo1);
        todo2 = todoRepository.save(todo2);
        
        // 実行と検証
        assertFalse(todoRepository.existsByUserIdAndTitleAndIdNot(1L, "Unique Title", todo1.getId()));
        assertTrue(todoRepository.existsByUserIdAndTitleAndIdNot(1L, "Unique Title", todo2.getId()));
        assertFalse(todoRepository.existsByUserIdAndTitleAndIdNot(1L, "Non-existent Title", todo1.getId()));
    }
    
    @Test
    public void testSave() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("New Todo");
        todo.setDescription("Description");
        todo.setCompleted(false);
        todo.setUserId(1L);
        
        // 実行
        Todo saved = todoRepository.save(todo);
        
        // 検証
        assertNotNull(saved.getId());
        assertEquals("New Todo", saved.getTitle());
        assertEquals("Description", saved.getDescription());
        assertFalse(saved.isCompleted());
        assertEquals(Long.valueOf(1L), saved.getUserId());
        assertNotNull(saved.getCreatedAt());
        assertNotNull(saved.getUpdatedAt());
    }
    
    @Test
    public void testUpdate() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("Original Title");
        todo.setDescription("Original Description");
        todo.setCompleted(false);
        todo.setUserId(1L);
        
        todo = todoRepository.save(todo);
        
        // 実行
        todo.setTitle("Updated Title");
        todo.setDescription("Updated Description");
        todo.setCompleted(true);
        
        Todo updated = todoRepository.save(todo);
        
        // 検証
        assertEquals(todo.getId(), updated.getId());
        assertEquals("Updated Title", updated.getTitle());
        assertEquals("Updated Description", updated.getDescription());
        assertTrue(updated.isCompleted());
    }
    
    @Test
    public void testDelete() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("Todo to Delete");
        todo.setDescription("Description");
        todo.setCompleted(false);
        todo.setUserId(1L);
        
        todo = todoRepository.save(todo);
        
        // 実行
        todoRepository.deleteById(todo.getId());
        
        // 検証
        Optional<Todo> found = todoRepository.findById(todo.getId());
        assertFalse(found.isPresent());
    }
    
    @Test
    public void testFindPage() {
        // 準備
        for (int i = 0; i < 20; i++) {
            Todo todo = new Todo();
            todo.setTitle("Todo " + i);
            todo.setDescription("Description " + i);
            todo.setCompleted(i % 2 == 0);
            todo.setUserId(1L);
            
            todoRepository.save(todo);
        }
        
        TodoSearchCriteria criteria = new TodoSearchCriteria();
        criteria.setUserId(1L);
        criteria.setCompleted(true);
        
        Pageable pageable = PageRequest.of(0, 5);
        
        // 実行
        Page<Todo> page = todoRepository.findPage(criteria, pageable);
        
        // 検証
        assertNotNull(page);
        assertEquals(5, page.getContent().size());
        
        for (Todo todo : page.getContent()) {
            assertEquals(Long.valueOf(1L), todo.getUserId());
            assertTrue(todo.isCompleted());
        }
    }
    
    @Test
    public void testCount() {
        // 準備
        for (int i = 0; i < 20; i++) {
            Todo todo = new Todo();
            todo.setTitle("Todo " + i);
            todo.setDescription("Description " + i);
            todo.setCompleted(i % 2 == 0);
            todo.setUserId(1L);
            
            todoRepository.save(todo);
        }
        
        TodoSearchCriteria criteria = new TodoSearchCriteria();
        criteria.setUserId(1L);
        criteria.setCompleted(true);
        
        // 実行
        long count = todoRepository.count(criteria);
        
        // 検証
        assertEquals(10, count);
    }
    
    @Test(expected = OptimisticLockingFailureException.class)
    public void testOptimisticLock() {
        // 準備
        Todo todo = new Todo();
        todo.setTitle("Original Title");
        todo.setDescription("Original Description");
        todo.setCompleted(false);
        todo.setUserId(1L);
        
        todo = todoRepository.save(todo);
        
        // 別のトランザクションで更新
        Todo sameTodo = todoRepository.findById(todo.getId()).get();
        sameTodo.setTitle("Updated by Another Transaction");
        todoRepository.save(sameTodo);
        
        // 実行
        todo.setTitle("Updated Title");
        todoRepository.save(todo);
        
        // 例外が発生することを期待
    }
}
```

## 課題4: 複雑なクエリの実装

**TodoMapper.java（追加メソッド）**

```java
package com.example.todo.domain.repository.todo;

import java.time.LocalDateTime;
import java.util.List;

import org.apache.ibatis.annotations.Param;

import com.example.todo.domain.model.Todo;
import com.example.todo.domain.model.TodoWithUser;

public interface TodoMapper {
    
    // 既存のメソッド
    
    TodoWithUser findWithUserInfo(Long id);
    
    List<Todo> findExpired(@Param("now") LocalDateTime now);
    
    List<Todo> findByCreatedAtBetween(@Param("start") LocalDateTime start, @Param("end") LocalDateTime end);
    
    double calculateCompletionRate(Long userId);
    
    List<Todo> findIncompleteByUserIdOrderByDueDate(Long userId);
}
```

**TodoMapper.xml（追加クエリ）**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.todo.domain.repository.todo.TodoMapper">

    <!-- 既存のマッピングとクエリ -->
    
    <resultMap id="todoWithUserResultMap" type="com.example.todo.domain.model.TodoWithUser">
        <id property="id" column="id" />
        <result property="title" column="title" />
        <result property="description" column="description" />
        <result property="completed" column="completed" />
        <result property="createdAt" column="created_at" />
        <result property="updatedAt" column="updated_at" />
        <result property="completedAt" column="completed_at" />
        <result property="userId" column="user_id" />
        <result property="version" column="version" />
        <association property="user" javaType="com.example.todo.domain.model.User">
            <id property="id" column="user_id" />
            <result property="username" column="username" />
            <result property="email" column="email" />
        </association>
    </resultMap>
    
    <select id="findWithUserInfo" parameterType="long" resultMap="todoWithUserResultMap">
        SELECT
            t.id,
            t.title,
            t.description,
            t.completed,
            t.created_at,
            t.updated_at,
            t.completed_at,
            t.user_id,
            t.version,
            u.username,
            u.email
        FROM
            todo t
        INNER JOIN
            app_user u
        ON
            t.user_id = u.id
        WHERE
            t.id = #{id}
    </select>
    
    <select id="findExpired" resultMap="todoResultMap">
        SELECT
            id,
            title,
            description,
            completed,
            created_at,
            updated_at,
            completed_at,
            user_id,
            version
        FROM
            todo
        WHERE
            completed = false
            AND due_date < #{now}
        ORDER BY
            due_date
    </select>
    
    <select id="findByCreatedAtBetween" resultMap="todoResultMap">
        SELECT
            id,
            title,
            description,
            completed,
            created_at,
            updated_at,
            completed_at,
            user_id,
            version
        FROM
            todo
        WHERE
            created_at BETWEEN #{start} AND #{end}
        ORDER BY
            created_at
    </select>
    
    <select id="calculateCompletionRate" parameterType="long" resultType="double">
        SELECT
            CASE
                WHEN COUNT(*) = 0 THEN 0
                ELSE CAST(SUM(CASE WHEN completed = true THEN 1 ELSE 0 END) AS DOUBLE) / COUNT(*) * 100
            END
        FROM
            todo
        WHERE
            user_id = #{userId}
    </select>
    
    <select id="findIncompleteByUserIdOrderByDueDate" parameterType="long" resultMap="todoResultMap">
        SELECT
            id,
            title,
            description,
            completed,
            created_at,
            updated_at,
            completed_at,
            user_id,
            version,
            due_date
        FROM
            todo
        WHERE
            user_id = #{userId}
            AND completed = false
            AND due_date IS NOT NULL
        ORDER BY
            due_date
    </select>
</mapper>
```

**TodoWithUser.java**

```java
package com.example.todo.domain.model;

import java.time.LocalDateTime;

import lombok.Data;
import lombok.EqualsAndHashCode;

@Data
@EqualsAndHashCode(callSuper = true)
public class TodoWithUser extends Todo {
    
    private static final long serialVersionUID = 1L;
    
    private User user;
}
```

### 解説

課題1～4の解答例では、TERASOLUNAフレームワークの推奨パターンに基づいて、Repository層の実装を行いました。

1. **TodoRepositoryの実装**
   - インターフェースと実装クラスの分離
   - MyBatisを使用したデータアクセス
   - 楽観的ロックの実装（version列の利用）
   - 動的クエリの構築（検索条件に応じたSQL生成）
   - ページネーションの実装（LIMIT/OFFSET句の利用）

2. **UserRepositoryの実装**
   - インターフェースと実装クラスの分離
   - MyBatisを使用したデータアクセス
   - 楽観的ロックの実装
   - ユーザー名の重複チェック機能

3. **Repository層のテスト**
   - JUnitとSpring Testを使用したテスト
   - トランザクション管理（@Transactional）
   - 正常系テスト（findAll、findById、save、update、delete）
   - 異常系テスト（楽観的ロックの競合）
   - ページネーションのテスト
   - 検索条件のテスト

4. **複雑なクエリの実装**
   - JOINを使用した関連データの取得
   - 条件に基づくフィルタリング
   - 集計関数の使用
   - ソート順の指定
   - 日時範囲の指定

これらの実装は、実務でのWebアプリケーション開発において、保守性と拡張性の高いRepository層を構築するための基礎となります。特に、MyBatisを使用したSQLマッピング、動的クエリの構築、ページネーションの実装は、データアクセスの効率化と柔軟性の向上に寄与します。また、テストの実装により、コードの品質と信頼性を確保することができます。
