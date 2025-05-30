# Day 17: Repository層の実装

## 学習の目的と背景

TERASOLUNAフレームワークにおいて、Repository層はデータアクセスを担当する重要な層です。Service層とデータストア（データベースなど）の間に位置し、データの永続化と取得を担当します。

本日の学習では、TERASOLUNAフレームワークにおけるRepository層の実装方法を学び、MyBatisを使用したToDoアプリケーションのRepository層を実装します。これにより、データアクセスの適切な設計と実装、SQLマッピング、トランザクション管理などの重要な概念を理解することを目指します。

## 学習内容の解説

### 1. Repository層の役割と責務

#### Repository層とは

Repository層は、データアクセスを担当する層です。以下の特徴があります：

1. **データアクセスの抽象化**
   - データストアへのアクセス方法の隠蔽
   - データアクセス技術の変更に対する影響の局所化
   - テスト容易性の向上

2. **永続化操作の提供**
   - データの検索
   - データの作成
   - データの更新
   - データの削除

3. **トランザクション管理のサポート**
   - Service層のトランザクション境界内での動作
   - データの一貫性確保

4. **データマッピング**
   - データストアの形式とドメインモデルの形式の変換
   - 複雑なクエリ結果の変換

#### Repository層の責務

TERASOLUNAフレームワークにおけるRepository層の主な責務は以下の通りです：

1. **データアクセスの実装**
   - SQLの実行
   - データの取得
   - データの永続化

2. **データマッピング**
   - データベースのレコードとドメインオブジェクトの変換
   - 複雑なクエリ結果の変換

3. **検索条件の処理**
   - 動的クエリの構築
   - 検索条件の適用

4. **ページネーションの処理**
   - 件数制限
   - オフセット指定
   - ソート順指定

5. **例外ハンドリング**
   - データアクセス例外の処理
   - 適切な例外への変換

### 2. Repository層の設計指針

#### レイヤ化アーキテクチャにおけるRepository層

TERASOLUNAフレームワークでは、レイヤ化アーキテクチャを採用しており、Repository層は以下の位置づけになります：

1. **Service層からの呼び出し**
   - Service層はRepository層のメソッドを呼び出す
   - Repository層はService層に依存しない

2. **データストアへのアクセス**
   - Repository層はデータストア（データベースなど）にアクセスする
   - データストアの詳細はRepository層内に隠蔽される

3. **Domain層との連携**
   - Repository層はDomain層のオブジェクトを操作する
   - Domain層のオブジェクトはビジネスルールを持つ

#### Repository層の設計指針

TERASOLUNAフレームワークにおけるRepository層の設計指針は以下の通りです：

1. **インターフェースと実装の分離**
   - リポジトリインターフェースの定義
   - リポジトリ実装クラスの作成
   - 依存性の注入（DI）による結合度の低減

2. **ドメインオブジェクト単位での設計**
   - エンティティごとにリポジトリを作成
   - 関連するエンティティの操作を集約

3. **メソッド名の命名規則**
   - 操作の種類を明確に表現
   - 検索条件を明確に表現
   - 戻り値の型を考慮

4. **例外ハンドリングの一貫性**
   - データアクセス例外の適切な処理
   - 例外の変換と伝播

5. **テスト容易性の確保**
   - モック化可能な設計
   - 依存性の明確化
   - インターフェースの適切な抽象化

### 3. MyBatisを使用したRepository層の実装

#### MyBatisとは

MyBatisは、Javaのデータアクセスフレームワークの一つで、SQLとJavaオブジェクトのマッピングを行います。以下の特徴があります：

1. **SQLとJavaの分離**
   - SQLをXMLファイルに記述
   - Javaコードとの分離による保守性の向上

2. **柔軟なマッピング**
   - 複雑なSQLの記述が可能
   - 動的SQLの構築が容易
   - 複雑なオブジェクトグラフのマッピングが可能

3. **シンプルな設定**
   - 最小限の設定で利用可能
   - アノテーションベースの設定も可能

4. **パフォーマンスの最適化**
   - ステートメントのキャッシュ
   - バッチ処理のサポート
   - 遅延ロードのサポート

#### MyBatisの基本的な使い方

MyBatisの基本的な使い方は以下の通りです：

**1. Mapperインターフェースの定義**

```java
public interface TodoMapper {
    
    List<Todo> findAll();
    
    Todo findById(Long id);
    
    void insert(Todo todo);
    
    void update(Todo todo);
    
    void delete(Long id);
}
```

**2. マッピングXMLファイルの作成**

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
</mapper>
```

**3. MyBatisの設定**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true" />
        <setting name="lazyLoadingEnabled" value="true" />
        <setting name="aggressiveLazyLoading" value="false" />
        <setting name="defaultExecutorType" value="REUSE" />
        <setting name="jdbcTypeForNull" value="NULL" />
    </settings>
    <typeAliases>
        <package name="com.example.todo.domain.model" />
    </typeAliases>
    <typeHandlers>
        <package name="com.example.todo.domain.repository.typehandler" />
    </typeHandlers>
</configuration>
```

**4. Spring Bootでの設定**

```java
@Configuration
@MapperScan("com.example.todo.domain.repository")
public class MyBatisConfig {
    
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        sessionFactory.setTypeAliasesPackage("com.example.todo.domain.model");
        sessionFactory.setConfigLocation(new ClassPathResource("mybatis-config.xml"));
        sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:com/example/todo/domain/repository/**/*.xml"));
        return sessionFactory.getObject();
    }
    
    @Bean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

#### 動的SQLの構築

MyBatisでは、動的SQLを構築するための様々な要素が提供されています。

```xml
<select id="findByCondition" parameterType="com.example.todo.domain.repository.todo.TodoSearchCriteria" resultMap="todoResultMap">
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
    ORDER BY
        <choose>
            <when test="orderBy != null and orderBy == 'title'">
                title
            </when>
            <when test="orderBy != null and orderBy == 'createdAt'">
                created_at
            </when>
            <otherwise>
                id
            </otherwise>
        </choose>
        <if test="ascending != null and ascending == true">
            ASC
        </if>
        <if test="ascending == null or ascending == false">
            DESC
        </if>
</select>
```

#### ページネーションの実装

MyBatisでは、ページネーションを実装するための様々な方法があります。

```xml
<select id="findPage" parameterType="map" resultMap="todoResultMap">
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
```

#### バッチ処理の実装

MyBatisでは、バッチ処理を実装するための様々な方法があります。

```xml
<insert id="batchInsert" parameterType="java.util.List">
    INSERT INTO todo (
        title,
        description,
        completed,
        created_at,
        updated_at,
        completed_at,
        user_id,
        version
    ) VALUES
    <foreach collection="list" item="item" separator=",">
        (
            #{item.title},
            #{item.description},
            #{item.completed},
            #{item.createdAt},
            #{item.updatedAt},
            #{item.completedAt},
            #{item.userId},
            1
        )
    </foreach>
</insert>

<update id="batchUpdate" parameterType="java.util.List">
    <foreach collection="list" item="item" separator=";">
        UPDATE
            todo
        SET
            title = #{item.title},
            description = #{item.description},
            completed = #{item.completed},
            updated_at = #{item.updatedAt},
            completed_at = #{item.completedAt},
            version = version + 1
        WHERE
            id = #{item.id}
            AND version = #{item.version}
    </foreach>
</update>
```

### 4. TERASOLUNAフレームワークにおけるRepository層の実装パターン

#### 基本的な実装パターン

TERASOLUNAフレームワークにおけるRepository層の基本的な実装パターンは以下の通りです：

**1. リポジトリインターフェース**

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

**2. リポジトリ実装クラス**

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

**3. Mapperインターフェース**

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

**4. 検索条件クラス**

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

#### 複雑なクエリの実装

複雑なクエリを実装する場合は、SQLを直接記述することができます。

```xml
<select id="findWithUserInfo" resultMap="todoWithUserResultMap">
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
        u.id as user_id,
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

<resultMap id="todoWithUserResultMap" type="com.example.todo.domain.model.Todo">
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
```

### 5. ToDoアプリケーションのRepository層の実装

#### TodoRepository

ToDoアプリケーションのRepository層の実装例を示します。

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

#### UserRepository

ユーザー管理のRepository層の実装例を示します。

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

## 実践課題（演習）

### 課題1: TodoRepositoryの実装

以下の要件に基づいて、TodoRepositoryを実装してください。

**要件:**
1. ToDoの一覧取得機能
2. ToDoの詳細取得機能
3. ToDoの新規作成機能
4. ToDoの更新機能
5. ToDoの削除機能
6. ユーザーIDによるフィルタリング機能
7. 完了状態によるフィルタリング機能
8. タイトルの重複チェック機能
9. ページネーション機能
10. 検索機能

### 課題2: UserRepositoryの実装

以下の要件に基づいて、UserRepositoryを実装してください。

**要件:**
1. ユーザーの一覧取得機能
2. ユーザーの詳細取得機能
3. ユーザーの新規作成機能
4. ユーザーの更新機能
5. ユーザーの削除機能
6. ユーザー名による検索機能
7. ユーザー名の重複チェック機能
8. ページネーション機能
9. 検索機能

### 課題3: Repository層のテスト

以下の要件に基づいて、TodoRepositoryのテストを実装してください。

**要件:**
1. 一覧取得機能のテスト
2. 詳細取得機能のテスト
3. 新規作成機能のテスト
4. 更新機能のテスト
5. 削除機能のテスト
6. フィルタリング機能のテスト
7. 重複チェック機能のテスト
8. ページネーション機能のテスト
9. 検索機能のテスト
10. 楽観的ロックのテスト

### 課題4: 複雑なクエリの実装

以下の要件に基づいて、複雑なクエリを実装してください。

**要件:**
1. ToDoとユーザー情報を結合して取得する機能
2. 期限切れのToDoを取得する機能
3. 特定の期間内に作成されたToDoを取得する機能
4. 特定のユーザーの完了率を計算する機能
5. 特定のユーザーの未完了のToDoを期限順に取得する機能

## 実務での活用シーン

### 1. 業務アプリケーション開発

Repository層の実装は、業務アプリケーション開発の中核となります。以下のような場面で活用されます：

- **企業の基幹システム開発**
  - 複雑なデータアクセスの抽象化
  - 複数のデータソースの連携
  - パフォーマンスの最適化

- **金融システム開発**
  - 大量データの効率的な処理
  - 複雑な検索条件の処理
  - 監査証跡の記録

- **ECサイトの開発**
  - 商品検索機能の実装
  - 注文履歴の管理
  - 在庫管理

### 2. データ連携

Repository層の実装は、データ連携においても重要です：

- **外部システムとのデータ連携**
  - データの取得と変換
  - データの送信と受信
  - エラーハンドリング

- **バッチ処理**
  - 大量データの一括処理
  - 定期的なデータ更新
  - データの集計と分析

- **データ移行**
  - 旧システムからのデータ移行
  - データの変換と検証
  - 整合性の確保

### 3. API開発

Repository層の実装は、API開発においても重要です：

- **RESTful API**
  - リソース指向のデータアクセス
  - ページネーションの実装
  - フィルタリングの実装

- **GraphQL API**
  - 柔軟なデータ取得
  - 関連データの効率的な取得
  - データローダの実装

- **マイクロサービス**
  - サービス間のデータ連携
  - データの整合性確保
  - 分散トランザクション
