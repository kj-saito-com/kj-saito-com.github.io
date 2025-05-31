---
layout: default
title: Day 13: MyBatisとPostgreSQLの連携
---
# Day 13: MyBatisとPostgreSQLの連携

## 学習の目的と背景

TERASOLUNAフレームワークでは、データアクセス層の実装にMyBatisを使用することが推奨されています。MyBatisはSQLマッピングフレームワークであり、JavaオブジェクトとSQLの間のマッピングを容易にします。本日の学習では、MyBatisとPostgreSQLを連携させ、TODOアプリケーションのデータアクセス層を実装する方法を学びます。

データアクセス層は、アプリケーションの中でも特に重要な部分です。適切に設計・実装されたデータアクセス層は、アプリケーションのパフォーマンスや保守性に大きく影響します。TERASOLUNAフレームワークの推奨アーキテクチャに従い、ドメイン層のRepositoryインターフェースとインフラストラクチャ層のMyBatis実装を適切に分離することで、テスタビリティと保守性を向上させることができます。

## 学習内容の解説

### 1. MyBatisの概要

#### 1.1 MyBatisとは

MyBatisは、SQLマッピングフレームワークであり、以下の特徴を持っています：

- JavaオブジェクトとSQLの間のマッピングを容易にする
- SQLをXMLファイルやアノテーションで管理できる
- 動的SQLの生成をサポートする
- 複雑なJOINやサブクエリも柔軟に記述できる
- Spring Frameworkとの統合が容易

#### 1.2 MyBatisの基本的な使い方

MyBatisの基本的な使い方は以下の通りです：

1. **マッパーインターフェースの定義**
   - Javaインターフェースとして定義
   - メソッド名とパラメータを定義

2. **SQLマッピングXMLの作成**
   - マッパーインターフェースに対応するXMLファイルを作成
   - SQL文とパラメータのマッピングを定義
   - 結果のマッピング方法を定義

3. **マッパーインターフェースの使用**
   - Spring FrameworkのDIコンテナからマッパーインターフェースを注入
   - マッパーインターフェースのメソッドを呼び出し

#### 1.3 MyBatisの主要コンポーネント

MyBatisの主要コンポーネントは以下の通りです：

1. **SqlSessionFactory**
   - SqlSessionを生成するファクトリ
   - アプリケーション起動時に1つだけ生成

2. **SqlSession**
   - データベースとの接続を表すオブジェクト
   - SQLの実行やトランザクション管理を行う
   - スレッドセーフではないため、メソッドスコープで使用

3. **マッパーインターフェース**
   - SQLを実行するためのJavaインターフェース
   - SqlSessionを通じて実装が自動的に生成される

4. **マッピングXML**
   - SQLとパラメータ、結果のマッピングを定義するXMLファイル
   - マッパーインターフェースと対応関係にある

### 2. PostgreSQLの概要

#### 2.1 PostgreSQLとは

PostgreSQLは、オープンソースのリレーショナルデータベース管理システム（RDBMS）であり、以下の特徴を持っています：

- 高い信頼性と安定性
- 豊富な機能（JSON型、全文検索、地理情報など）
- ACID準拠のトランザクション
- 拡張性と柔軟性
- 標準SQLへの高い準拠性

#### 2.2 PostgreSQLの基本的な使い方

PostgreSQLの基本的な使い方は以下の通りです：

1. **データベースの作成**
   ```sql
   CREATE DATABASE todo_app;
   ```

2. **テーブルの作成**
   ```sql
   CREATE TABLE users (
       username VARCHAR(50) PRIMARY KEY,
       password VARCHAR(100) NOT NULL,
       role VARCHAR(20) NOT NULL,
       enabled BOOLEAN NOT NULL
   );

   CREATE TABLE todo (
       todo_id VARCHAR(36) PRIMARY KEY,
       todo_title VARCHAR(100) NOT NULL,
       finished BOOLEAN NOT NULL,
       created_at TIMESTAMP NOT NULL,
       created_by VARCHAR(50) NOT NULL REFERENCES users(username),
       updated_at TIMESTAMP
   );
   ```

3. **データの操作**
   ```sql
   -- 挿入
   INSERT INTO users VALUES ('user1', 'password1', 'ROLE_USER', true);
   
   -- 検索
   SELECT * FROM users WHERE username = 'user1';
   
   -- 更新
   UPDATE users SET password = 'newpassword' WHERE username = 'user1';
   
   -- 削除
   DELETE FROM users WHERE username = 'user1';
   ```

#### 2.3 PostgreSQLの主要な機能

PostgreSQLの主要な機能は以下の通りです：

1. **データ型**
   - 基本型（INTEGER, VARCHAR, BOOLEAN, TIMESTAMP など）
   - 配列型
   - JSON型
   - 地理情報型

2. **インデックス**
   - B-tree（デフォルト）
   - Hash
   - GiST
   - GIN

3. **制約**
   - PRIMARY KEY
   - FOREIGN KEY
   - UNIQUE
   - CHECK
   - NOT NULL

4. **トランザクション**
   - BEGIN
   - COMMIT
   - ROLLBACK
   - SAVEPOINT

### 3. MyBatisとPostgreSQLの連携

#### 3.1 MyBatisの設定

MyBatisをSpring Bootで使用するための設定は以下の通りです：

1. **依存関係の追加**
   ```xml
   <dependency>
       <groupId>org.mybatis.spring.boot</groupId>
       <artifactId>mybatis-spring-boot-starter</artifactId>
       <version>2.2.0</version>
   </dependency>
   <dependency>
       <groupId>org.postgresql</groupId>
       <artifactId>postgresql</artifactId>
       <scope>runtime</scope>
   </dependency>
   ```

2. **データソースの設定**
   ```properties
   # application.properties
   spring.datasource.url=jdbc:postgresql://localhost:5432/todo_app
   spring.datasource.username=postgres
   spring.datasource.password=postgres
   spring.datasource.driver-class-name=org.postgresql.Driver
   ```

3. **MyBatisの設定**
   ```properties
   # application.properties
   mybatis.config-location=classpath:mybatis-config.xml
   mybatis.mapper-locations=classpath*:/META-INF/mybatis/**/*.xml
   ```

4. **MyBatis設定ファイル**
   ```xml
   <!-- mybatis-config.xml -->
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <settings>
           <setting name="mapUnderscoreToCamelCase" value="true" />
       </settings>
       <typeAliases>
           <package name="com.example.todo.domain.model" />
       </typeAliases>
   </configuration>
   ```

#### 3.2 マッパーインターフェースの作成

マッパーインターフェースは以下のように作成します：

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

#### 3.3 マッピングXMLの作成

マッピングXMLは以下のように作成します：

```xml
<!-- TodoRepositoryMapper.xml -->
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

#### 3.4 Repositoryインターフェースの作成

Repositoryインターフェースは以下のように作成します：

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

#### 3.5 Repositoryの実装

Repositoryの実装は以下のように作成します：

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

### 4. 動的SQLの活用

#### 4.1 動的SQLとは

動的SQLとは、条件に応じてSQL文を動的に生成する機能です。MyBatisでは、以下のような動的SQL要素を提供しています：

- `<if>`：条件分岐
- `<choose>`, `<when>`, `<otherwise>`：複数条件の分岐
- `<where>`：WHERE句の生成
- `<set>`：SET句の生成
- `<foreach>`：コレクションの繰り返し処理

#### 4.2 動的SQLの例

動的SQLの例は以下の通りです：

```xml
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
        <if test="createdBy != null">
            created_by = #{createdBy}
        </if>
        <if test="finished != null">
            AND finished = #{finished}
        </if>
        <if test="todoTitle != null and todoTitle != ''">
            AND todo_title LIKE '%' || #{todoTitle} || '%'
        </if>
    </where>
    ORDER BY
        created_at DESC
</select>
```

#### 4.3 バッチ処理

MyBatisでは、バッチ処理を以下のように実装できます：

```xml
<insert id="createBatch" parameterType="java.util.List">
    INSERT INTO todo
        (todo_id, todo_title, finished, created_at, created_by)
    VALUES
    <foreach collection="list" item="item" separator=",">
        (#{item.todoId}, #{item.todoTitle}, #{item.finished}, #{item.createdAt}, #{item.createdBy})
    </foreach>
</insert>
```

## 実践課題

### コア演習1: PostgreSQLのテーブル作成

**目的**: TODOアプリケーションで使用するテーブルをPostgreSQLに作成する

**手順**:
1. PostgreSQLに接続する
2. TODOアプリケーション用のデータベースを作成する
3. usersテーブルとtodoテーブルを作成する

**雛形**:

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

### コア演習2: MyBatisのマッパーインターフェースとマッピングXMLの作成

**目的**: TODOアプリケーションで使用するMyBatisのマッパーインターフェースとマッピングXMLを作成する

**手順**:
1. TodoRepositoryMapperインターフェースを作成する
2. TodoRepositoryMapper.xmlを作成する
3. UserRepositoryMapperインターフェースを作成する
4. UserRepositoryMapper.xmlを作成する

**雛形**:

```java
// TodoRepositoryMapper.java
package com.example.todo.domain.repository.mybatis;

import com.example.todo.domain.model.Todo;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;

@Mapper
public interface TodoRepositoryMapper {
    // TODO: メソッドを定義する
}
```

```xml
<!-- TodoRepositoryMapper.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.todo.domain.repository.mybatis.TodoRepositoryMapper">
    <!-- TODO: SQLマッピングを定義する -->
</mapper>
```

### コア演習3: Repositoryインターフェースと実装の作成

**目的**: TODOアプリケーションで使用するRepositoryインターフェースと実装を作成する

**手順**:
1. TodoRepositoryインターフェースを作成する
2. TodoRepositoryImplクラスを作成する
3. UserRepositoryインターフェースを作成する
4. UserRepositoryImplクラスを作成する

**雛形**:

```java
// TodoRepository.java
package com.example.todo.domain.repository;

import com.example.todo.domain.model.Todo;

import java.util.List;
import java.util.Optional;

public interface TodoRepository {
    // TODO: メソッドを定義する
}
```

```java
// TodoRepositoryImpl.java
package com.example.todo.domain.repository;

import com.example.todo.domain.model.Todo;
import com.example.todo.domain.repository.mybatis.TodoRepositoryMapper;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public class TodoRepositoryImpl implements TodoRepository {
    // TODO: 実装する
}
```

### 発展演習: 動的SQLを使用した検索機能の実装

**目的**: 動的SQLを使用して、条件に応じたTODOの検索機能を実装する

**手順**:
1. 検索条件を表すクラスを作成する
2. TodoRepositoryMapperに検索メソッドを追加する
3. TodoRepositoryMapper.xmlに動的SQLを使用した検索用のSQLマッピングを追加する
4. TodoRepositoryとTodoRepositoryImplに検索メソッドを追加する

**雛形**:

```java
// TodoSearchCriteria.java
package com.example.todo.domain.model;

public class TodoSearchCriteria {
    private String createdBy;
    private Boolean finished;
    private String todoTitle;
    
    // TODO: getterとsetterを実装する
}
```

```java
// TodoRepositoryMapper.java（追加）
List<Todo> findByCondition(@Param("criteria") TodoSearchCriteria criteria);
```

```xml
<!-- TodoRepositoryMapper.xml（追加） -->
<select id="findByCondition" resultMap="todoResultMap">
    <!-- TODO: 動的SQLを実装する -->
</select>
```

## 解説と補足

### MyBatisを使用する利点

MyBatisを使用する主な利点は以下の通りです：

1. **SQLの可視性**
   - SQLをXMLファイルに記述するため、SQLの可視性が高い
   - 複雑なSQLも記述しやすい
   - SQLのチューニングが容易

2. **動的SQLの柔軟性**
   - 条件に応じてSQLを動的に生成できる
   - 複雑な検索条件にも対応できる

3. **マッピングの柔軟性**
   - JavaオブジェクトとSQLの結果のマッピングを柔軟に設定できる
   - 1対多、多対多の関係も扱いやすい

4. **パフォーマンス**
   - 必要なSQLだけを実行するため、パフォーマンスが良い
   - キャッシュ機能も提供されている

### MyBatisとJPAの比較

MyBatisとJPAの主な違いは以下の通りです：

1. **アプローチ**
   - MyBatis: SQLマッピングアプローチ（SQLを明示的に記述）
   - JPA: オブジェクト関係マッピング（ORMアプローチ）（SQLを自動生成）

2. **学習曲線**
   - MyBatis: SQLの知識があれば比較的簡単
   - JPA: 概念の理解に時間がかかる場合がある

3. **制御**
   - MyBatis: SQLを完全に制御できる
   - JPA: 自動生成されるSQLの制御が難しい場合がある

4. **適用場面**
   - MyBatis: 複雑なSQLを使用する場合、既存のデータベースを使用する場合
   - JPA: 新規開発で、シンプルなCRUD操作が中心の場合

### PostgreSQLを使用する利点

PostgreSQLを使用する主な利点は以下の通りです：

1. **機能の豊富さ**
   - JSON型、配列型、地理情報型などの豊富なデータ型
   - 全文検索、ウィンドウ関数などの高度な機能
   - 拡張機能による機能の追加

2. **標準SQLへの準拠**
   - SQL標準に高い準拠性を持つ
   - 他のデータベースからの移行が容易

3. **信頼性と安定性**
   - ACID準拠のトランザクション
   - 高い信頼性と安定性
   - 障害耐性

4. **オープンソース**
   - オープンソースで無償で使用可能
   - 活発なコミュニティによるサポート
   - 多くの企業での採用実績

### リポジトリパターンの重要性

リポジトリパターンは、以下の理由で重要です：

1. **関心事の分離**
   - データアクセスロジックをビジネスロジックから分離
   - 各層の責務を明確化

2. **テスタビリティの向上**
   - モックやスタブを使用したテストが容易
   - ビジネスロジックの単体テストが容易

3. **技術変更の容易性**
   - データアクセス技術の変更（例：MyBatisからJPAへ）が容易
   - インターフェースを変更せずに実装を変更可能

4. **コードの再利用性**
   - 共通のデータアクセスロジックを再利用可能
   - 複数のサービスから同じリポジトリを使用可能

## 実務での活用シーン

### 1. エンタープライズアプリケーション開発

エンタープライズアプリケーション開発では、以下のような場面でMyBatisとPostgreSQLの連携が活用されます：

- 複雑なデータモデルを持つアプリケーション
- 大量のデータを扱うアプリケーション
- 複雑な検索条件を持つアプリケーション
- レポーティング機能を持つアプリケーション

### 2. マイクロサービスアーキテクチャ

マイクロサービスアーキテクチャでは、以下のような場面でMyBatisとPostgreSQLの連携が活用されます：

- データアクセスを担当するマイクロサービス
- 複数のマイクロサービスからアクセスされるデータベース
- トランザクション管理が必要なマイクロサービス
- データの整合性が重要なマイクロサービス

### 3. レガシーシステムのリファクタリング

レガシーシステムのリファクタリングでは、以下のような場面でMyBatisとPostgreSQLの連携が活用されます：

- 既存のSQLを活用したリファクタリング
- データベース構造を変更せずにアプリケーションを刷新
- 段階的なリファクタリング
- パフォーマンスの改善

### 4. データ分析アプリケーション

データ分析アプリケーションでは、以下のような場面でMyBatisとPostgreSQLの連携が活用されます：

- 複雑な集計クエリを使用するアプリケーション
- 大量のデータを分析するアプリケーション
- レポート生成機能を持つアプリケーション
- データの可視化機能を持つアプリケーション
