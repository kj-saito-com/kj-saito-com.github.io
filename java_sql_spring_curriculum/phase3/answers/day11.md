---
layout: default
title: "Day 11: Spring Frameworkの基本概念とTERASOLUNAフレームワーク - 解答例"
---
# Day 11: Spring Frameworkの基本概念とTERASOLUNAフレームワーク - 解答例

## コア演習1: Spring DIコンテナの基本

### MessageService.java
```java
package com.example.di;

public interface MessageService {
    String getMessage();
}
```

### SimpleMessageService.java
```java
package com.example.di;

import org.springframework.stereotype.Service;

@Service
public class SimpleMessageService implements MessageService {
    
    @Override
    public String getMessage() {
        return "Hello, Spring DI!";
    }
}
```

### MessagePrinter.java
```java
package com.example.di;

import org.springframework.stereotype.Component;

@Component
public class MessagePrinter {
    
    private final MessageService messageService;
    
    // コンストラクタインジェクション
    public MessagePrinter(MessageService messageService) {
        this.messageService = messageService;
    }
    
    public void printMessage() {
        System.out.println(messageService.getMessage());
    }
}
```

### Application.java
```java
package com.example.di;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class Application {
    
    public static void main(String[] args) {
        // Spring DIコンテナを初期化
        try (AnnotationConfigApplicationContext context = 
                new AnnotationConfigApplicationContext(Application.class)) {
            
            // MessagePrinterを取得して実行
            MessagePrinter printer = context.getBean(MessagePrinter.class);
            printer.printMessage();
        }
    }
}
```

### 解説
この演習では、Spring DIコンテナの基本的な使い方を実装しています。

1. `MessageService`インターフェースを定義し、`SimpleMessageService`クラスで実装しています。
2. `@Service`アノテーションにより、`SimpleMessageService`はSpring DIコンテナによって管理されるコンポーネントとして認識されます。
3. `MessagePrinter`クラスは`@Component`アノテーションが付与され、コンストラクタインジェクションによって`MessageService`の実装を受け取ります。
4. `Application`クラスでは、`@Configuration`と`@ComponentScan`アノテーションを使用して、DIコンテナの設定を行っています。
5. `main`メソッドでは、`AnnotationConfigApplicationContext`を使用してDIコンテナを初期化し、`MessagePrinter`のインスタンスを取得して実行しています。

## コア演習2: Spring AOPの基本

### LoggingAspect.java
```java
package com.example.di;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.After;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {
    
    @Before("execution(* com.example.di.MessageService.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("メソッド実行前: " + joinPoint.getSignature().getName());
    }
    
    @After("execution(* com.example.di.MessagePrinter.*(..))")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println("メソッド実行後: " + joinPoint.getSignature().getName());
    }
}
```

### Application.java（更新版）
```java
package com.example.di;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@ComponentScan
@EnableAspectJAutoProxy
public class Application {
    
    public static void main(String[] args) {
        // Spring DIコンテナを初期化
        try (AnnotationConfigApplicationContext context = 
                new AnnotationConfigApplicationContext(Application.class)) {
            
            // MessagePrinterを取得して実行
            MessagePrinter printer = context.getBean(MessagePrinter.class);
            printer.printMessage();
        }
    }
}
```

### 解説
この演習では、Spring AOPの基本的な使い方を実装しています。

1. `LoggingAspect`クラスに`@Aspect`と`@Component`アノテーションを付与し、アスペクトとして定義しています。
2. `@Before`アドバイスを使用して、`MessageService`のメソッド実行前にログを出力します。
3. `@After`アドバイスを使用して、`MessagePrinter`のメソッド実行後にログを出力します。
4. `Application`クラスに`@EnableAspectJAutoProxy`アノテーションを追加して、AOPを有効化しています。

実行結果は以下のようになります：
```
メソッド実行前: getMessage
Hello, Spring DI!
メソッド実行後: printMessage
```

## コア演習3: TERASOLUNAフレームワークの構成要素の理解

### クラス図

```
[Controller]
- Webリクエストを受け付け、適切なServiceを呼び出す
- FormオブジェクトとDomain Modelの変換を行う
- 処理結果に応じたViewを選択する
 ↓ 利用
[Service]
- ビジネスロジックを実装する
- トランザクション境界となる
- Repositoryを利用してデータアクセスを行う
 ↓ 利用
[Repository]
- データアクセスの抽象化を提供する
- データベースやファイルなどの永続化層へのアクセスを担当
 ↓ 操作
[Domain Model]
- ビジネスデータとロジックをカプセル化する
- エンティティやValueObjectなどで構成される

[Form/DTO] ←→ [Domain Model]
- FormはWebリクエストのデータを保持する
- DTOはレイヤー間のデータ転送に使用される
- 相互に変換処理が必要
```

### 解説

TERASOLUNAフレームワークの主要構成要素とその役割は以下の通りです：

1. **Controller（コントローラ層）**
   - Webリクエストを受け付け、適切なServiceを呼び出す役割を担います
   - FormオブジェクトのバリデーションやDomain Modelへの変換を行います
   - 処理結果に応じたViewを選択します
   - `@Controller`アノテーションを使用します

2. **Service（サービス層）**
   - ビジネスロジックを実装する役割を担います
   - トランザクション境界となります（`@Transactional`）
   - 複数のRepositoryを組み合わせて処理を行うことがあります
   - `@Service`アノテーションを使用します

3. **Repository（リポジトリ層）**
   - データアクセスの抽象化を提供する役割を担います
   - データベースやファイルなどの永続化層へのアクセスを担当します
   - MyBatisやJPAなどのO/Rマッパーと連携します
   - `@Repository`アノテーションを使用します

4. **Domain Model（ドメインモデル）**
   - ビジネスデータとロジックをカプセル化する役割を担います
   - エンティティやValueObjectなどで構成されます
   - 永続化の詳細を知らない、純粋なJavaオブジェクトとして実装されます

5. **Form/DTO**
   - Formはユーザーからの入力データを保持します（`@RequestParam`、`@ModelAttribute`）
   - DTOはレイヤー間のデータ転送に使用されます
   - Domain Modelとの間で相互変換が必要です

これらの構成要素が連携することで、関心事の分離が実現され、保守性と拡張性が向上します。

## 発展演習: Spring MVCの基本

### UserController.java
```java
package com.example.web;

import java.util.List;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public String list(Model model) {
        List<User> users = userService.findAll();
        model.addAttribute("users", users);
        return "users/list";
    }
    
    @GetMapping("/{id}")
    public String detail(@PathVariable Long id, Model model) {
        User user = userService.findById(id)
                .orElseThrow(() -> new UserNotFoundException(id));
        model.addAttribute("user", user);
        return "users/detail";
    }
}
```

### UserService.java
```java
package com.example.web;

import java.util.List;
import java.util.Optional;

public interface UserService {
    
    List<User> findAll();
    
    Optional<User> findById(Long id);
}
```

### UserServiceImpl.java
```java
package com.example.web;

import java.util.List;
import java.util.Optional;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;
    
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
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
}
```

### UserRepository.java
```java
package com.example.web;

import java.util.List;
import java.util.Optional;

public interface UserRepository {
    
    List<User> findAll();
    
    Optional<User> findById(Long id);
}
```

### UserRepositoryImpl.java（インメモリ実装）
```java
package com.example.web;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

import org.springframework.stereotype.Repository;

@Repository
public class UserRepositoryImpl implements UserRepository {
    
    private final Map<Long, User> userMap = new HashMap<>();
    
    public UserRepositoryImpl() {
        // サンプルデータの初期化
        User user1 = new User();
        user1.setId(1L);
        user1.setName("山田太郎");
        user1.setEmail("yamada@example.com");
        
        User user2 = new User();
        user2.setId(2L);
        user2.setName("鈴木花子");
        user2.setEmail("suzuki@example.com");
        
        userMap.put(user1.getId(), user1);
        userMap.put(user2.getId(), user2);
    }
    
    @Override
    public List<User> findAll() {
        return new ArrayList<>(userMap.values());
    }
    
    @Override
    public Optional<User> findById(Long id) {
        return Optional.ofNullable(userMap.get(id));
    }
}
```

### User.java
```java
package com.example.web;

public class User {
    
    private Long id;
    private String name;
    private String email;
    
    // Getters and Setters
    public Long getId() {
        return id;
    }
    
    public void setId(Long id) {
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public String getEmail() {
        return email;
    }
    
    public void setEmail(String email) {
        this.email = email;
    }
}
```

### UserNotFoundException.java
```java
package com.example.web;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {
    
    private static final long serialVersionUID = 1L;
    
    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
}
```

### users/list.jsp
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ユーザー一覧</title>
</head>
<body>
    <h1>ユーザー一覧</h1>
    
    <table border="1">
        <thead>
            <tr>
                <th>ID</th>
                <th>名前</th>
                <th>メールアドレス</th>
                <th>操作</th>
            </tr>
        </thead>
        <tbody>
            <c:forEach items="${users}" var="user">
                <tr>
                    <td>${user.id}</td>
                    <td>${user.name}</td>
                    <td>${user.email}</td>
                    <td>
                        <a href="${pageContext.request.contextPath}/users/${user.id}">詳細</a>
                    </td>
                </tr>
            </c:forEach>
        </tbody>
    </table>
</body>
</html>
```

### users/detail.jsp
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ユーザー詳細</title>
</head>
<body>
    <h1>ユーザー詳細</h1>
    
    <table border="1">
        <tr>
            <th>ID</th>
            <td>${user.id}</td>
        </tr>
        <tr>
            <th>名前</th>
            <td>${user.name}</td>
        </tr>
        <tr>
            <th>メールアドレス</th>
            <td>${user.email}</td>
        </tr>
    </table>
    
    <p><a href="${pageContext.request.contextPath}/users">一覧に戻る</a></p>
</body>
</html>
```

### 解説

この発展演習では、Spring MVCを使用した簡単なユーザー管理機能を実装しています。

1. **Controller層**
   - `UserController`クラスは、ユーザー一覧と詳細表示のエンドポイントを提供します
   - `@Controller`と`@RequestMapping`アノテーションを使用して、リクエストマッピングを定義しています
   - `Model`オブジェクトを使用して、ビューに渡すデータを設定しています

2. **Service層**
   - `UserService`インターフェースと`UserServiceImpl`クラスは、ビジネスロジックを提供します
   - `@Service`と`@Transactional`アノテーションを使用して、サービスとトランザクション境界を定義しています
   - 読み取り専用の操作には`@Transactional(readOnly = true)`を使用しています

3. **Repository層**
   - `UserRepository`インターフェースと`UserRepositoryImpl`クラスは、データアクセスを提供します
   - 簡略化のため、インメモリのマップを使用していますが、実際のアプリケーションではデータベースを使用します
   - `@Repository`アノテーションを使用して、リポジトリとして定義しています

4. **Domain Model**
   - `User`クラスは、ユーザー情報を表現するドメインモデルです
   - シンプルなJavaクラスとして実装されています

5. **例外処理**
   - `UserNotFoundException`クラスは、ユーザーが見つからない場合の例外を表現します
   - `@ResponseStatus`アノテーションを使用して、HTTPステータスコードを設定しています

6. **View**
   - JSPを使用して、ユーザー一覧と詳細表示のビューを実装しています
   - JSTLを使用して、動的なコンテンツを生成しています

この実装により、MVCパターンに基づいた、保守性と拡張性の高いWebアプリケーションが実現されています。
