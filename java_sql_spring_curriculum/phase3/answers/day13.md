---
layout: default
title: "Day 13: MyBatisとPostgreSQLの連携 - 解答例"
---
# Day 13: MyBatisとPostgreSQLの連携 - 解答例

## コア演習1: PostgreSQLのテーブル作成

```sql
-- データベースの作成
CREATE DATABASE todo_app;

-- データベースに接続
\c todo_app

-- usersテーブルの作成
CREATE TABLE users (
    username VARCHAR(50) PRIMARY KEY,
    password VARCHAR(100) NOT NULL,
    role VARCHAR(20) NOT NULL,
    enabled BOOLEAN NOT NULL
);

-- todoテーブルの作成
CREATE TABLE todo (
    todo_id VARCHAR(36) PRIMARY KEY,
    todo_title VARCHAR(100) NOT NULL,
    finished BOOLEAN NOT NULL,
    created_at TIMESTAMP NOT NULL,
    created_by VARCHAR(50) NOT NULL,
    updated_at TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES users(username)
);

-- インデックスの作成
CREATE INDEX idx_todo_created_by ON todo(created_by);
CREATE INDEX idx_todo_finished ON todo(finished);
```

### 解説
この演習では、TODOアプリケーションで使用するデータベースとテーブルを作成しています。

1. `todo_app`データベースを作成します。
2. `users`テーブルを作成し、ユーザー情報を管理します。
   - `username`：プライマリキーとして使用するユーザー名
   - `password`：ハッシュ化されたパスワード
   - `role`：ユーザーのロール（権限）
   - `enabled`：アカウントの有効/無効フラグ
3. `todo`テーブルを作成し、TODOアイテムを管理します。
   - `todo_id`：TODOのID（UUID形式）
   - `todo_title`：TODOのタイトル
   - `finished`：完了フラグ
   - `created_at`：作成日時
   - `created_by`：作成者（usersテーブルの外部キー）
   - `updated_at`：更新日時
4. パフォーマンス向上のためのインデックスを作成します。
   - `created_by`列のインデックス：特定ユーザーのTODO検索を高速化
   - `finished`列のインデックス：完了/未完了でのフィルタリングを高速化

## コア演習2: MyBatisのマッパーインターフェースとマッピングXMLの作成

### TodoRepositoryMapper.java
```java
package com.example.todo.domain.repository.mybatis;

import com.example.todo.domain.model.Todo;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;

@Mapper
public interface TodoRepositoryMapper {
    List<Todo> findAll(@Param("createdBy") String createdBy);
    Todo findOne(@Param("todoId") String todoId);
    void create(Todo todo);
    void update(Todo todo);
    void delete(@Param("todoId") String todoId);
    long countByCreatedBy(@Param("createdBy") String createdBy);
}
```

### TodoRepositoryMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.todo.domain.repository.mybatis.TodoRepositoryMapper">

    <resultMap id="todoResultMap" type="Todo">
        <id property="todoId" column="todo_id" />
        <result property="todoTitle" column="todo_title" />
        <result property="finished" column="finished" />
        <result property="createdAt" column="created_at" />
        <result property="createdBy" column="created_by" />
        <result property="updatedAt" column="updated_at" />
    </resultMap>

    <select id="findAll" resultMap="todoResultMap">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            created_by,
            updated_at
        FROM
            todo
        WHERE
            created_by = #{createdBy}
        ORDER BY
            created_at DESC
    </select>

    <select id="findOne" resultMap="todoResultMap">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            created_by,
            updated_at
        FROM
            todo
        WHERE
            todo_id = #{todoId}
    </select>

    <insert id="create" parameterType="Todo">
        INSERT INTO todo
            (todo_id, todo_title, finished, created_at, created_by)
        VALUES
            (#{todoId}, #{todoTitle}, #{finished}, #{createdAt}, #{createdBy})
    </insert>

    <update id="update" parameterType="Todo">
        UPDATE todo
        SET
            todo_title = #{todoTitle},
            finished = #{finished},
            updated_at = #{updatedAt}
        WHERE
            todo_id = #{todoId}
    </update>

    <delete id="delete">
        DELETE FROM todo
        WHERE
            todo_id = #{todoId}
    </delete>

    <select id="countByCreatedBy" resultType="long">
        SELECT
            COUNT(*)
        FROM
            todo
        WHERE
            created_by = #{createdBy}
    </select>
</mapper>
```

### UserRepositoryMapper.java
```java
package com.example.todo.domain.repository.mybatis;

import com.example.todo.domain.model.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

@Mapper
public interface UserRepositoryMapper {
    User findOne(@Param("username") String username);
    void create(User user);
    int exists(@Param("username") String username);
}
```

### UserRepositoryMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.todo.domain.repository.mybatis.UserRepositoryMapper">

    <resultMap id="userResultMap" type="User">
        <id property="username" column="username" />
        <result property="password" column="password" />
        <result property="role" column="role" />
        <result property="enabled" column="enabled" />
    </resultMap>

    <select id="findOne" resultMap="userResultMap">
        SELECT
            username,
            password,
            role,
            enabled
        FROM
            users
        WHERE
            username = #{username}
    </select>

    <insert id="create" parameterType="User">
        INSERT INTO users
            (username, password, role, enabled)
        VALUES
            (#{username}, #{password}, #{role}, #{enabled})
    </insert>

    <select id="exists" resultType="int">
        SELECT
            COUNT(*)
        FROM
            users
        WHERE
            username = #{username}
    </select>
</mapper>
```

### 解説
この演習では、MyBatisのマッパーインターフェースとマッピングXMLを作成しています。

1. **TodoRepositoryMapper**：
   - TODOに関するデータアクセス操作を定義するインターフェース
   - 主なメソッド：findAll、findOne、create、update、delete、countByCreatedBy

2. **TodoRepositoryMapper.xml**：
   - SQLとJavaオブジェクトのマッピングを定義するXML
   - resultMapでカラム名とプロパティ名のマッピングを定義
   - 各メソッドに対応するSQLを定義

3. **UserRepositoryMapper**：
   - ユーザーに関するデータアクセス操作を定義するインターフェース
   - 主なメソッド：findOne、create、exists

4. **UserRepositoryMapper.xml**：
   - ユーザー関連のSQLとJavaオブジェクトのマッピングを定義するXML
   - ユーザーの検索、登録、存在確認のSQLを定義

## コア演習3: Repositoryインターフェースと実装の作成

### TodoRepository.java
```java
package com.example.todo.domain.repository;

import com.example.todo.domain.model.Todo;

import java.util.List;
import java.util.Optional;

public interface TodoRepository {
    List<Todo> findAll(String createdBy);
    Optional<Todo> findOne(String todoId);
    Todo create(Todo todo);
    Todo update(Todo todo);
    void delete(String todoId);
    long countByCreatedBy(String createdBy);
}
```

### TodoRepositoryImpl.java
```java
package com.example.todo.domain.repository;

import com.example.todo.domain.model.Todo;
import com.example.todo.domain.repository.mybatis.TodoRepositoryMapper;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public class TodoRepositoryImpl implements TodoRepository {

    private final TodoRepositoryMapper todoRepositoryMapper;

    public TodoRepositoryImpl(TodoRepositoryMapper todoRepositoryMapper) {
        this.todoRepositoryMapper = todoRepositoryMapper;
    }

    @Override
    public List<Todo> findAll(String createdBy) {
        return todoRepositoryMapper.findAll(createdBy);
    }

    @Override
    public Optional<Todo> findOne(String todoId) {
        return Optional.ofNullable(todoRepositoryMapper.findOne(todoId));
    }

    @Override
    public Todo create(Todo todo) {
        todoRepositoryMapper.create(todo);
        return todo;
    }

    @Override
    public Todo update(Todo todo) {
        todoRepositoryMapper.update(todo);
        return todo;
    }

    @Override
    public void delete(String todoId) {
        todoRepositoryMapper.delete(todoId);
    }

    @Override
    public long countByCreatedBy(String createdBy) {
        return todoRepositoryMapper.countByCreatedBy(createdBy);
    }
}
```

### UserRepository.java
```java
package com.example.todo.domain.repository;

import com.example.todo.domain.model.User;

import java.util.Optional;

public interface UserRepository {
    Optional<User> findOne(String username);
    User create(User user);
    boolean exists(String username);
}
```

### UserRepositoryImpl.java
```java
package com.example.todo.domain.repository;

import com.example.todo.domain.model.User;
import com.example.todo.domain.repository.mybatis.UserRepositoryMapper;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public class UserRepositoryImpl implements UserRepository {

    private final UserRepositoryMapper userRepositoryMapper;

    public UserRepositoryImpl(UserRepositoryMapper userRepositoryMapper) {
        this.userRepositoryMapper = userRepositoryMapper;
    }

    @Override
    public Optional<User> findOne(String username) {
        return Optional.ofNullable(userRepositoryMapper.findOne(username));
    }

    @Override
    public User create(User user) {
        userRepositoryMapper.create(user);
        return user;
    }

    @Override
    public boolean exists(String username) {
        return userRepositoryMapper.exists(username) > 0;
    }
}
```

### 解説
この演習では、リポジトリパターンに基づいて、Repositoryインターフェースとその実装クラスを作成しています。

1. **TodoRepository**：
   - TODOに関するビジネスロジックから見たデータアクセスインターフェース
   - 戻り値の型としてOptionalを使用し、nullの扱いを明示的に

2. **TodoRepositoryImpl**：
   - TodoRepositoryの実装クラス
   - TodoRepositoryMapperを使用してデータアクセスを実装
   - @Repositoryアノテーションでリポジトリとして登録

3. **UserRepository**：
   - ユーザーに関するデータアクセスインターフェース
   - ユーザーの検索、登録、存在確認の操作を定義

4. **UserRepositoryImpl**：
   - UserRepositoryの実装クラス
   - UserRepositoryMapperを使用してデータアクセスを実装

このリポジトリパターンにより、以下のメリットが得られます：
- ビジネスロジックとデータアクセスの分離
- テスタビリティの向上
- データアクセス技術の変更に対する柔軟性

## 発展演習: 動的SQLを使用した検索機能の実装

### TodoSearchCriteria.java
```java
package com.example.todo.domain.model;

import java.io.Serializable;

public class TodoSearchCriteria implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private String createdBy;
    private Boolean finished;
    private String todoTitle;
    
    public String getCreatedBy() {
        return createdBy;
    }
    
    public void setCreatedBy(String createdBy) {
        this.createdBy = createdBy;
    }
    
    public Boolean getFinished() {
        return finished;
    }
    
    public void setFinished(Boolean finished) {
        this.finished = finished;
    }
    
    public String getTodoTitle() {
        return todoTitle;
    }
    
    public void setTodoTitle(String todoTitle) {
        this.todoTitle = todoTitle;
    }
}
```

### TodoRepositoryMapper.java（追加）
```java
// 既存のTodoRepositoryMapperインターフェースに以下のメソッドを追加
List<Todo> findByCondition(@Param("criteria") TodoSearchCriteria criteria);
```

### TodoRepositoryMapper.xml（追加）
```xml
<!-- 既存のTodoRepositoryMapper.xmlに以下のSQLマッピングを追加 -->
<select id="findByCondition" resultMap="todoResultMap">
    SELECT
        todo_id,
        todo_title,
        finished,
        created_at,
        created_by,
        updated_at
    FROM
        todo
    <where>
        <if test="criteria.createdBy != null">
            created_by = #{criteria.createdBy}
        </if>
        <if test="criteria.finished != null">
            AND finished = #{criteria.finished}
        </if>
        <if test="criteria.todoTitle != null and criteria.todoTitle != ''">
            AND todo_title LIKE '%' || #{criteria.todoTitle} || '%'
        </if>
    </where>
    ORDER BY
        created_at DESC
</select>
```

### TodoRepository.java（追加）
```java
// 既存のTodoRepositoryインターフェースに以下のメソッドを追加
List<Todo> findByCondition(TodoSearchCriteria criteria);
```

### TodoRepositoryImpl.java（追加）
```java
// 既存のTodoRepositoryImplクラスに以下のメソッドを追加
@Override
public List<Todo> findByCondition(TodoSearchCriteria criteria) {
    return todoRepositoryMapper.findByCondition(criteria);
}
```

### 使用例
```java
// 検索条件の作成
TodoSearchCriteria criteria = new TodoSearchCriteria();
criteria.setCreatedBy("user1");
criteria.setFinished(false);
criteria.setTodoTitle("買い物");

// 検索の実行
List<Todo> todos = todoRepository.findByCondition(criteria);
```

### 解説
この発展演習では、MyBatisの動的SQLを使用して、条件に応じたTODOの検索機能を実装しています。

1. **TodoSearchCriteria**：
   - 検索条件を表すクラス
   - 作成者、完了状態、タイトルによる検索条件を保持

2. **動的SQL**：
   - `<where>`要素：WHERE句を動的に生成
   - `<if>`要素：条件が存在する場合のみSQL条件を追加
   - LIKE検索：タイトルのあいまい検索を実装

3. **リポジトリの拡張**：
   - TodoRepositoryインターフェースに検索メソッドを追加
   - TodoRepositoryImplに実装を追加

この実装により、以下のような柔軟な検索が可能になります：
- 特定ユーザーのTODOのみ検索
- 完了/未完了のTODOのみ検索
- タイトルに特定の文字列を含むTODOの検索
- これらの条件の組み合わせによる検索

動的SQLを使用することで、検索条件の有無に応じて適切なSQLが生成され、不要な条件が含まれないクエリが実行されます。
