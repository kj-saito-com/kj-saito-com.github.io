---
layout: default
title: "Day 4: 例外処理の設計と実装"
---
# Day 4: 例外処理の設計と実装

## 学習の目的と背景

例外処理は、プログラムの実行中に発生する異常な状況を適切に処理するための重要なメカニズムです。適切な例外処理を行うことで、プログラムの堅牢性と保守性を向上させることができます。特に業務アプリケーションでは、ユーザー入力エラー、ネットワーク障害、データベース接続エラーなど、様々な例外状況に対応する必要があります。

本日の学習では、Javaの例外処理の基本から応用までを体系的に学び、チェック例外と非チェック例外の適切な使い分け、独自例外クラスの設計、効果的な例外処理パターンなどを習得することを目指します。

## 前提知識と準備

### 必要な前提知識
- Javaの基本構文（変数、制御構文、クラスなど）
- オブジェクト指向プログラミングの基本概念
- 継承とインターフェースの基本的な理解

### 環境準備
- Eclipse 2023-12 (4.30.0)
- Java 21

### プロジェクト作成手順

1. Eclipseを起動します。
2. 「File」→「New」→「Java Project」を選択します。
3. プロジェクト名を「JavaExceptions_Day4」と入力します。
4. 「JRE」の設定で「Use an execution environment JRE:」から「JavaSE-21」を選択します。
5. 「Finish」をクリックしてプロジェクトを作成します。

## 基本概念の解説

### 1. 例外の基本概念

例外（Exception）は、プログラムの実行中に発生する異常な状況を表すオブジェクトです。Javaでは、例外は`Throwable`クラスのサブクラスとして表現されます。

**用語解説: 例外（Exception）**  
プログラムの実行中に発生する異常な状況を表すオブジェクトです。通常の制御フローを中断し、適切なハンドラに制御を移します。

Javaの例外は、以下のような階層構造になっています：

```
Throwable
├── Error（システムエラー、通常はキャッチしない）
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
└── Exception（アプリケーションで処理可能な例外）
    ├── IOException（チェック例外）
    ├── SQLException（チェック例外）
    ├── RuntimeException（非チェック例外）
    │   ├── NullPointerException
    │   ├── IllegalArgumentException
    │   └── ...
    └── ...
```

### 2. チェック例外と非チェック例外

Javaの例外は、チェック例外と非チェック例外の2種類に分類されます。

**用語解説: チェック例外（Checked Exception）**  
コンパイル時にチェックされる例外です。メソッドがチェック例外をスローする可能性がある場合、そのメソッドの宣言に`throws`節を含めるか、`try-catch`ブロックで例外をキャッチする必要があります。`Exception`クラスのサブクラスで、`RuntimeException`のサブクラスではないものがチェック例外です。

**用語解説: 非チェック例外（Unchecked Exception）**  
コンパイル時にチェックされない例外です。メソッドが非チェック例外をスローする可能性がある場合でも、`throws`節を含める必要はありません。`RuntimeException`クラスのサブクラスと`Error`クラスのサブクラスが非チェック例外です。

#### チェック例外の例
- `IOException`：入出力操作に関連する例外
- `SQLException`：データベース操作に関連する例外
- `ClassNotFoundException`：クラスが見つからない場合の例外

#### 非チェック例外の例
- `NullPointerException`：nullオブジェクトのメソッドやフィールドにアクセスした場合の例外
- `IllegalArgumentException`：メソッドに不正な引数が渡された場合の例外
- `IndexOutOfBoundsException`：インデックスが範囲外の場合の例外

### 3. 例外処理の基本構文

#### try-catch-finally

`try-catch-finally`ブロックは、例外をキャッチして処理するための基本的な構文です。

```java
try {
    // 例外が発生する可能性のあるコード
    int result = 10 / 0;  // ArithmeticExceptionが発生
} catch (ArithmeticException e) {
    // ArithmeticExceptionが発生した場合の処理
    System.out.println("ゼロ除算が発生しました: " + e.getMessage());
} catch (Exception e) {
    // その他の例外が発生した場合の処理
    System.out.println("例外が発生しました: " + e.getMessage());
} finally {
    // 例外の発生有無にかかわらず実行される処理
    System.out.println("finallyブロックが実行されました");
}
```

#### try-with-resources

Java 7以降では、`try-with-resources`構文を使用して、自動的にリソースをクローズすることができます。この構文は、`AutoCloseable`インターフェースを実装したリソースに対して使用できます。

```java
try (FileReader reader = new FileReader("file.txt");
     BufferedReader bufferedReader = new BufferedReader(reader)) {
    // リソースを使用するコード
    String line = bufferedReader.readLine();
    System.out.println(line);
} catch (IOException e) {
    // 例外処理
    System.out.println("ファイル読み込みエラー: " + e.getMessage());
}
// リソースは自動的にクローズされる
```

### 4. 例外のスロー

`throw`キーワードを使用して、明示的に例外をスローすることができます。

```java
public void validateAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("年齢は0以上である必要があります");
    }
    if (age > 120) {
        throw new IllegalArgumentException("年齢は120以下である必要があります");
    }
    // 正常な処理
    System.out.println("年齢は有効です: " + age);
}
```

### 5. メソッドの例外宣言

メソッドがチェック例外をスローする可能性がある場合、メソッド宣言に`throws`節を含める必要があります。

```java
public void readFile(String fileName) throws IOException {
    FileReader reader = new FileReader(fileName);
    BufferedReader bufferedReader = new BufferedReader(reader);
    
    try {
        String line = bufferedReader.readLine();
        System.out.println(line);
    } finally {
        bufferedReader.close();
    }
}
```

### 6. 独自例外クラスの作成

特定のアプリケーションドメインに関連する例外を表現するために、独自の例外クラスを作成することができます。

```java
// チェック例外の例
public class UserNotFoundException extends Exception {
    public UserNotFoundException(String message) {
        super(message);
    }
    
    public UserNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}

// 非チェック例外の例
public class InvalidUserDataException extends RuntimeException {
    public InvalidUserDataException(String message) {
        super(message);
    }
    
    public InvalidUserDataException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### 7. 例外の連鎖

例外の連鎖（Exception Chaining）は、ある例外が別の例外の原因である場合に、その関係を保持するための仕組みです。

```java
try {
    // データベース操作
    // ...
} catch (SQLException e) {
    // SQLExceptionをアプリケーション固有の例外に変換
    throw new DataAccessException("データベースアクセスエラー", e);
}
```

## Eclipse操作ガイド

### クラスの作成

1. プロジェクト「JavaExceptions_Day4」を右クリックします。
2. 「New」→「Class」を選択します。
3. 「Name」に作成するクラス名（例：`ExceptionDemo`）を入力します。
4. 「public static void main(String[] args)」にチェックを入れます。
5. 「Finish」をクリックしてクラスを作成します。

### 例外処理の自動生成

1. 例外が発生する可能性のあるコードを記述します。
2. エラーが表示された行にカーソルを置きます。
3. 「Ctrl + 1」を押して、クイックフィックスのメニューを表示します。
4. 「Surround with try/catch」または「Add throws declaration」を選択します。
5. Eclipseが自動的に例外処理コードを生成します。

### try-with-resourcesへの変換

1. 従来の`try-finally`ブロックを選択します。
2. 右クリックして「Refactor」→「Convert to try-with-resources」を選択します。
3. Eclipseが自動的に`try-with-resources`構文に変換します。

### 例外のデバッグ

1. 例外が発生する可能性のあるコードにブレークポイントを設定します（行番号の左側をダブルクリック）。
2. 右クリックして「Debug As」→「Java Application」を選択します。
3. デバッグパースペクティブで変数の値を確認します。
4. 「Step Over」（F6）などのボタンで実行を制御します。
5. 例外が発生した場合、デバッガは自動的に停止し、例外の詳細を表示します。

## コア演習（必須）

### 演習1: 基本的な例外処理（難易度：基本）

#### 課題
ファイル操作を行うプログラムを作成し、以下の例外処理を実装してください：
1. ファイルが存在しない場合の例外処理
2. ファイル読み込み中のエラーの例外処理
3. リソースの適切なクローズ（try-with-resources構文を使用）

#### 雛形コード
以下のクラスを作成してください：

```java
package exceptions.basic;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

/**
 * 基本的な例外処理を学ぶためのデモクラス
 */
public class FileReaderDemo {
    
    /**
     * 従来のtry-catch-finally構文を使用してファイルを読み込みます。
     * 
     * @param fileName 読み込むファイルのパス
     * @return ファイルの最初の行、ファイルが空の場合は空文字列
     * @throws IOException ファイル読み込みエラーが発生した場合
     */
    public static String readFirstLineWithTraditional(String fileName) throws IOException {
        // TODO: ここに従来のtry-catch-finally構文を使用したコードを実装
        // ヒント1: FileReaderとBufferedReaderを使用
        // ヒント2: finallyブロックでリソースをクローズ
        // ヒント3: ファイルが存在しない場合のエラーメッセージを表示
        
        return null; // 仮の戻り値
    }
    
    /**
     * try-with-resources構文を使用してファイルを読み込みます。
     * 
     * @param fileName 読み込むファイルのパス
     * @return ファイルの最初の行、ファイルが空の場合は空文字列
     * @throws IOException ファイル読み込みエラーが発生した場合
     */
    public static String readFirstLineWithTryWithResources(String fileName) throws IOException {
        // TODO: ここにtry-with-resources構文を使用したコードを実装
        // ヒント1: FileReaderとBufferedReaderをtry-with-resources構文で宣言
        // ヒント2: ファイルが存在しない場合のエラーメッセージを表示
        
        return null; // 仮の戻り値
    }
    
    /**
     * ファイルの内容を読み込み、各行を表示します。
     * 
     * @param fileName 読み込むファイルのパス
     */
    public static void readAndDisplayFile(String fileName) {
        // TODO: ここにファイルの内容を読み込み、表示するコードを実装
        // ヒント1: try-with-resources構文を使用
        // ヒント2: ファイルが存在しない場合と読み込みエラーの両方をキャッチ
        // ヒント3: 例外が発生した場合はエラーメッセージを表示
    }
    
    public static void main(String[] args) {
        // 存在するファイルのパス
        String existingFile = "sample.txt";
        
        // 存在しないファイルのパス
        String nonExistingFile = "non_existing_file.txt";
        
        // 従来のtry-catch-finally構文を使用したファイル読み込み
        System.out.println("===== 従来のtry-catch-finally構文 =====");
        try {
            String firstLine = readFirstLineWithTraditional(existingFile);
            System.out.println("最初の行: " + firstLine);
        } catch (IOException e) {
            System.out.println("エラーが発生しました: " + e.getMessage());
        }
        
        // try-with-resources構文を使用したファイル読み込み
        System.out.println("\n===== try-with-resources構文 =====");
        try {
            String firstLine = readFirstLineWithTryWithResources(existingFile);
            System.out.println("最初の行: " + firstLine);
        } catch (IOException e) {
            System.out.println("エラーが発生しました: " + e.getMessage());
        }
        
        // 存在しないファイルの読み込み
        System.out.println("\n===== 存在しないファイルの読み込み =====");
        readAndDisplayFile(nonExistingFile);
        
        // 存在するファイルの読み込み
        System.out.println("\n===== 存在するファイルの読み込み =====");
        readAndDisplayFile(existingFile);
    }
}
```

#### 実装ヒント
1. `readFirstLineWithTraditional`メソッドでは、`FileReader`と`BufferedReader`を使用して、ファイルの最初の行を読み込みます。`finally`ブロックでリソースをクローズします。
2. `readFirstLineWithTryWithResources`メソッドでは、`try-with-resources`構文を使用して、リソースの自動クローズを行います。
3. `readAndDisplayFile`メソッドでは、`try-with-resources`構文と`catch`ブロックを使用して、ファイルの内容を読み込み、表示します。ファイルが存在しない場合と読み込みエラーの両方をキャッチします。

### 演習2: チェック例外と非チェック例外の使い分け（難易度：応用）

#### 課題
ユーザー登録システムを実装し、以下の例外処理を行ってください：
1. チェック例外：ユーザーが見つからない場合（`UserNotFoundException`）
2. 非チェック例外：ユーザーデータが不正な場合（`InvalidUserDataException`）
3. 例外の連鎖：データベースエラーを適切な例外に変換

#### 雛形コード
以下のクラスを作成してください：

```java
package exceptions.advanced;

import java.util.HashMap;
import java.util.Map;

/**
 * ユーザーが見つからない場合のチェック例外
 */
class UserNotFoundException extends Exception {
    // TODO: ここにコンストラクタを実装
    // ヒント1: 文字列メッセージを受け取るコンストラクタ
    // ヒント2: 文字列メッセージと原因例外を受け取るコンストラクタ
}

/**
 * ユーザーデータが不正な場合の非チェック例外
 */
class InvalidUserDataException extends RuntimeException {
    // TODO: ここにコンストラクタを実装
    // ヒント1: 文字列メッセージを受け取るコンストラクタ
    // ヒント2: 文字列メッセージと原因例外を受け取るコンストラクタ
}

/**
 * データベースエラーを表すチェック例外
 */
class DatabaseException extends Exception {
    public DatabaseException(String message) {
        super(message);
    }
    
    public DatabaseException(String message, Throwable cause) {
        super(message, cause);
    }
}

/**
 * ユーザー情報を表すクラス
 */
class User {
    private String id;
    private String name;
    private String email;
    private int age;
    
    public User(String id, String name, String email, int age) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.age = age;
    }
    
    // ゲッター
    public String getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public int getAge() { return age; }
    
    @Override
    public String toString() {
        return "User [id=" + id + ", name=" + name + ", email=" + email + ", age=" + age + "]";
    }
}

/**
 * ユーザー登録システムを表すクラス
 */
public class UserRegistrationSystem {
    
    private Map<String, User> users = new HashMap<>();
    private boolean databaseConnected = true;  // データベース接続状態（デモ用）
    
    /**
     * ユーザーを登録します。
     * 
     * @param user 登録するユーザー
     * @throws InvalidUserDataException ユーザーデータが不正な場合
     * @throws DatabaseException データベースエラーが発生した場合
     */
    public void registerUser(User user) throws DatabaseException {
        // TODO: ここにユーザー登録処理を実装
        // ヒント1: ユーザーデータのバリデーション（名前、メール、年齢）
        // ヒント2: バリデーションエラーの場合はInvalidUserDataExceptionをスロー
        // ヒント3: データベース接続エラーの場合はDatabaseExceptionをスロー
        // ヒント4: 正常な場合はusersマップにユーザーを追加
    }
    
    /**
     * ユーザーIDでユーザーを検索します。
     * 
     * @param userId 検索するユーザーID
     * @return 見つかったユーザー
     * @throws UserNotFoundException ユーザーが見つからない場合
     * @throws DatabaseException データベースエラーが発生した場合
     */
    public User findUserById(String userId) throws UserNotFoundException, DatabaseException {
        // TODO: ここにユーザー検索処理を実装
        // ヒント1: データベース接続エラーの場合はDatabaseExceptionをスロー
        // ヒント2: ユーザーが見つからない場合はUserNotFoundExceptionをスロー
        // ヒント3: 正常な場合はusersマップからユーザーを返す
        
        return null; // 仮の戻り値
    }
    
    /**
     * データベース接続状態を設定します（デモ用）。
     * 
     * @param connected 接続状態
     */
    public void setDatabaseConnected(boolean connected) {
        this.databaseConnected = connected;
    }
    
    public static void main(String[] args) {
        UserRegistrationSystem system = new UserRegistrationSystem();
        
        // ユーザー登録のテスト
        System.out.println("===== ユーザー登録のテスト =====");
        
        // 正常なユーザーデータ
        User validUser = new User("user1", "山田太郎", "yamada@example.com", 30);
        
        // 不正なユーザーデータ（年齢が負の値）
        User invalidAgeUser = new User("user2", "佐藤花子", "sato@example.com", -5);
        
        // 不正なユーザーデータ（名前が空）
        User invalidNameUser = new User("user3", "", "tanaka@example.com", 25);
        
        // 正常なユーザーの登録
        try {
            system.registerUser(validUser);
            System.out.println("ユーザーが正常に登録されました: " + validUser);
        } catch (InvalidUserDataException e) {
            System.out.println("不正なユーザーデータ: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
        
        // 不正な年齢のユーザーの登録
        try {
            system.registerUser(invalidAgeUser);
            System.out.println("ユーザーが正常に登録されました: " + invalidAgeUser);
        } catch (InvalidUserDataException e) {
            System.out.println("不正なユーザーデータ: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
        
        // 不正な名前のユーザーの登録
        try {
            system.registerUser(invalidNameUser);
            System.out.println("ユーザーが正常に登録されました: " + invalidNameUser);
        } catch (InvalidUserDataException e) {
            System.out.println("不正なユーザーデータ: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
        
        // ユーザー検索のテスト
        System.out.println("\n===== ユーザー検索のテスト =====");
        
        // 存在するユーザーの検索
        try {
            User foundUser = system.findUserById("user1");
            System.out.println("ユーザーが見つかりました: " + foundUser);
        } catch (UserNotFoundException e) {
            System.out.println("ユーザーが見つかりません: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
        
        // 存在しないユーザーの検索
        try {
            User foundUser = system.findUserById("non_existing_user");
            System.out.println("ユーザーが見つかりました: " + foundUser);
        } catch (UserNotFoundException e) {
            System.out.println("ユーザーが見つかりません: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
        
        // データベース接続エラーのテスト
        System.out.println("\n===== データベース接続エラーのテスト =====");
        
        // データベース接続を切断
        system.setDatabaseConnected(false);
        
        // データベース接続エラー時のユーザー登録
        try {
            User newUser = new User("user4", "伊藤健太", "ito@example.com", 40);
            system.registerUser(newUser);
            System.out.println("ユーザーが正常に登録されました: " + newUser);
        } catch (InvalidUserDataException e) {
            System.out.println("不正なユーザーデータ: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
        
        // データベース接続エラー時のユーザー検索
        try {
            User foundUser = system.findUserById("user1");
            System.out.println("ユーザーが見つかりました: " + foundUser);
        } catch (UserNotFoundException e) {
            System.out.println("ユーザーが見つかりません: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
    }
}
```

#### 実装ヒント
1. `UserNotFoundException`クラスと`InvalidUserDataException`クラスに、適切なコンストラクタを実装します。
2. `registerUser`メソッドでは、ユーザーデータのバリデーションを行い、不正な場合は`InvalidUserDataException`をスローします。データベース接続エラーの場合は`DatabaseException`をスローします。
3. `findUserById`メソッドでは、ユーザーが見つからない場合は`UserNotFoundException`をスローします。データベース接続エラーの場合は`DatabaseException`をスローします。
4. 例外の連鎖を使用して、低レベルの例外（例：`SQLException`）を高レベルの例外（例：`DatabaseException`）に変換します。

### 演習3: 例外の変換とラッピング（難易度：応用）

#### 課題
ファイル操作ユーティリティを実装し、以下の例外処理を行ってください：
1. 低レベルの例外（`IOException`）を高レベルの例外（`FileOperationException`）に変換
2. 例外の連鎖を使用して、元の例外情報を保持
3. 適切な例外メッセージとスタックトレースの提供

#### 雛形コード
以下のクラスを作成してください：

```java
package exceptions.advanced;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * ファイル操作に関連する例外を表すクラス
 */
class FileOperationException extends Exception {
    // TODO: ここにコンストラクタを実装
    // ヒント1: 文字列メッセージを受け取るコンストラクタ
    // ヒント2: 文字列メッセージと原因例外を受け取るコンストラクタ
}

/**
 * ファイル操作ユーティリティクラス
 */
public class FileUtils {
    
    /**
     * ファイルの内容を読み込み、文字列のリストとして返します。
     * 
     * @param fileName 読み込むファイルのパス
     * @return ファイルの各行を含む文字列のリスト
     * @throws FileOperationException ファイル操作エラーが発生した場合
     */
    public static List<String> readLines(String fileName) throws FileOperationException {
        // TODO: ここにファイル読み込み処理を実装
        // ヒント1: try-with-resources構文を使用
        // ヒント2: IOExceptionをキャッチし、FileOperationExceptionに変換
        // ヒント3: 例外メッセージには具体的な情報を含める
        
        return null; // 仮の戻り値
    }
    
    /**
     * 文字列のリストをファイルに書き込みます。
     * 
     * @param lines 書き込む文字列のリスト
     * @param fileName 書き込むファイルのパス
     * @throws FileOperationException ファイル操作エラーが発生した場合
     */
    public static void writeLines(List<String> lines, String fileName) throws FileOperationException {
        // TODO: ここにファイル書き込み処理を実装
        // ヒント1: try-with-resources構文を使用
        // ヒント2: IOExceptionをキャッチし、FileOperationExceptionに変換
        // ヒント3: 例外メッセージには具体的な情報を含める
    }
    
    /**
     * ファイルをコピーします。
     * 
     * @param sourceFileName コピー元ファイルのパス
     * @param targetFileName コピー先ファイルのパス
     * @throws FileOperationException ファイル操作エラーが発生した場合
     */
    public static void copyFile(String sourceFileName, String targetFileName) throws FileOperationException {
        // TODO: ここにファイルコピー処理を実装
        // ヒント1: readLinesメソッドとwriteLinesメソッドを使用
        // ヒント2: FileOperationExceptionをキャッチし、適切なメッセージを追加して再スロー
    }
    
    /**
     * ファイルが存在するかどうかを確認します。
     * 
     * @param fileName 確認するファイルのパス
     * @return ファイルが存在する場合はtrue、存在しない場合はfalse
     */
    public static boolean fileExists(String fileName) {
        File file = new File(fileName);
        return file.exists() && file.isFile();
    }
    
    public static void main(String[] args) {
        // テスト用のファイル名
        String sourceFileName = "source.txt";
        String targetFileName = "target.txt";
        String nonExistingFileName = "non_existing_file.txt";
        
        // テスト用のデータ
        List<String> testData = new ArrayList<>();
        testData.add("これはテストデータの1行目です。");
        testData.add("これはテストデータの2行目です。");
        testData.add("これはテストデータの3行目です。");
        
        // ファイル書き込みのテスト
        System.out.println("===== ファイル書き込みのテスト =====");
        try {
            FileUtils.writeLines(testData, sourceFileName);
            System.out.println("ファイルへの書き込みが成功しました: " + sourceFileName);
        } catch (FileOperationException e) {
            System.out.println("ファイル操作エラー: " + e.getMessage());
            System.out.println("原因: " + e.getCause());
        }
        
        // ファイル読み込みのテスト（存在するファイル）
        System.out.println("\n===== ファイル読み込みのテスト（存在するファイル） =====");
        try {
            List<String> lines = FileUtils.readLines(sourceFileName);
            System.out.println("ファイルからの読み込みが成功しました: " + sourceFileName);
            System.out.println("読み込んだ行数: " + lines.size());
            for (String line : lines) {
                System.out.println(line);
            }
        } catch (FileOperationException e) {
            System.out.println("ファイル操作エラー: " + e.getMessage());
            System.out.println("原因: " + e.getCause());
        }
        
        // ファイル読み込みのテスト（存在しないファイル）
        System.out.println("\n===== ファイル読み込みのテスト（存在しないファイル） =====");
        try {
            List<String> lines = FileUtils.readLines(nonExistingFileName);
            System.out.println("ファイルからの読み込みが成功しました: " + nonExistingFileName);
            System.out.println("読み込んだ行数: " + lines.size());
        } catch (FileOperationException e) {
            System.out.println("ファイル操作エラー: " + e.getMessage());
            System.out.println("原因: " + e.getCause());
        }
        
        // ファイルコピーのテスト
        System.out.println("\n===== ファイルコピーのテスト =====");
        try {
            FileUtils.copyFile(sourceFileName, targetFileName);
            System.out.println("ファイルのコピーが成功しました: " + sourceFileName + " -> " + targetFileName);
            
            if (FileUtils.fileExists(targetFileName)) {
                List<String> lines = FileUtils.readLines(targetFileName);
                System.out.println("コピー先ファイルの内容:");
                for (String line : lines) {
                    System.out.println(line);
                }
            }
        } catch (FileOperationException e) {
            System.out.println("ファイル操作エラー: " + e.getMessage());
            System.out.println("原因: " + e.getCause());
        }
        
        // 存在しないファイルのコピーのテスト
        System.out.println("\n===== 存在しないファイルのコピーのテスト =====");
        try {
            FileUtils.copyFile(nonExistingFileName, targetFileName);
            System.out.println("ファイルのコピーが成功しました: " + nonExistingFileName + " -> " + targetFileName);
        } catch (FileOperationException e) {
            System.out.println("ファイル操作エラー: " + e.getMessage());
            System.out.println("原因: " + e.getCause());
        }
    }
}
```

#### 実装ヒント
1. `FileOperationException`クラスに、適切なコンストラクタを実装します。
2. `readLines`メソッドでは、`try-with-resources`構文を使用してファイルを読み込み、`IOException`をキャッチして`FileOperationException`に変換します。
3. `writeLines`メソッドでは、`try-with-resources`構文を使用してファイルに書き込み、`IOException`をキャッチして`FileOperationException`に変換します。
4. `copyFile`メソッドでは、`readLines`メソッドと`writeLines`メソッドを使用してファイルをコピーし、`FileOperationException`をキャッチして適切なメッセージを追加して再スローします。

## 発展演習（オプション）

### 発展演習1: 例外フィルタリング（難易度：発展）

#### 課題
Java 7で導入された例外フィルタリング機能を使用して、特定の例外だけをキャッチするプログラムを作成してください。以下の要件を満たす必要があります：

1. 複数の例外をキャッチし、特定の条件に一致する例外だけを処理する
2. マルチキャッチ構文を使用して、複数の例外タイプを1つの`catch`ブロックで処理する
3. 例外の再スローを行い、処理しない例外は上位の呼び出し元に伝播させる

#### 雛形コード
以下のクラスを作成してください：

```java
package exceptions.advanced;

import java.io.FileNotFoundException;
import java.io.IOException;
import java.sql.SQLException;
import java.util.Arrays;
import java.util.List;

/**
 * 例外フィルタリングを学ぶためのデモクラス
 */
public class ExceptionFilteringDemo {
    
    /**
     * 特定の条件に一致する例外だけを処理します。
     * 
     * @param exceptionType 処理する例外のタイプ（"io", "sql", "all"のいずれか）
     * @throws Exception 処理されなかった例外
     */
    public static void processExceptionsWithFiltering(String exceptionType) throws Exception {
        try {
            // ランダムに例外をスロー
            throwRandomException();
        } catch (Exception e) {
            // TODO: ここに例外フィルタリングのコードを実装
            // ヒント1: exceptionTypeに基づいて、特定の例外だけを処理
            // ヒント2: 処理しない例外は再スロー
            // ヒント3: 例外の型と内容を表示
            
            throw e; // 仮の実装（すべての例外を再スロー）
        }
    }
    
    /**
     * マルチキャッチ構文を使用して、複数の例外タイプを1つのcatchブロックで処理します。
     * 
     * @throws Exception 処理されなかった例外
     */
    public static void processExceptionsWithMultiCatch() throws Exception {
        try {
            // ランダムに例外をスロー
            throwRandomException();
        } catch (/* TODO: ここにマルチキャッチの例外リストを追加 */) {
            // TODO: ここにマルチキャッチのコードを実装
            // ヒント1: FileNotFoundExceptionとSQLExceptionを1つのcatchブロックで処理
            // ヒント2: 例外の型と内容を表示
        } catch (Exception e) {
            // その他の例外の処理
            System.out.println("その他の例外が発生しました: " + e.getClass().getSimpleName());
            System.out.println("例外メッセージ: " + e.getMessage());
            throw e;
        }
    }
    
    /**
     * ランダムに例外をスローします。
     * 
     * @throws FileNotFoundException ファイルが見つからない場合
     * @throws IOException 入出力エラーが発生した場合
     * @throws SQLException SQLエラーが発生した場合
     * @throws IllegalArgumentException 不正な引数が渡された場合
     */
    private static void throwRandomException() throws FileNotFoundException, IOException, SQLException, IllegalArgumentException {
        List<Exception> exceptions = Arrays.asList(
            new FileNotFoundException("ファイルが見つかりません"),
            new IOException("入出力エラーが発生しました"),
            new SQLException("SQLエラーが発生しました"),
            new IllegalArgumentException("不正な引数が渡されました")
        );
        
        // ランダムに例外を選択
        int index = (int) (Math.random() * exceptions.size());
        Exception exception = exceptions.get(index);
        
        // 選択した例外をスロー
        if (exception instanceof FileNotFoundException) {
            throw (FileNotFoundException) exception;
        } else if (exception instanceof IOException) {
            throw (IOException) exception;
        } else if (exception instanceof SQLException) {
            throw (SQLException) exception;
        } else if (exception instanceof IllegalArgumentException) {
            throw (IllegalArgumentException) exception;
        }
    }
    
    public static void main(String[] args) {
        // 例外フィルタリングのテスト
        System.out.println("===== 例外フィルタリングのテスト =====");
        
        // IOExceptionのみを処理
        System.out.println("\n--- IOExceptionのみを処理 ---");
        for (int i = 0; i < 5; i++) {
            try {
                processExceptionsWithFiltering("io");
                System.out.println("例外は発生しませんでした");
            } catch (Exception e) {
                System.out.println("キャッチされなかった例外: " + e.getClass().getSimpleName());
                System.out.println("例外メッセージ: " + e.getMessage());
            }
        }
        
        // SQLExceptionのみを処理
        System.out.println("\n--- SQLExceptionのみを処理 ---");
        for (int i = 0; i < 5; i++) {
            try {
                processExceptionsWithFiltering("sql");
                System.out.println("例外は発生しませんでした");
            } catch (Exception e) {
                System.out.println("キャッチされなかった例外: " + e.getClass().getSimpleName());
                System.out.println("例外メッセージ: " + e.getMessage());
            }
        }
        
        // すべての例外を処理
        System.out.println("\n--- すべての例外を処理 ---");
        for (int i = 0; i < 5; i++) {
            try {
                processExceptionsWithFiltering("all");
                System.out.println("例外は発生しませんでした");
            } catch (Exception e) {
                System.out.println("キャッチされなかった例外: " + e.getClass().getSimpleName());
                System.out.println("例外メッセージ: " + e.getMessage());
            }
        }
        
        // マルチキャッチのテスト
        System.out.println("\n===== マルチキャッチのテスト =====");
        for (int i = 0; i < 5; i++) {
            try {
                processExceptionsWithMultiCatch();
                System.out.println("例外は発生しませんでした");
            } catch (Exception e) {
                System.out.println("キャッチされなかった例外: " + e.getClass().getSimpleName());
                System.out.println("例外メッセージ: " + e.getMessage());
            }
        }
    }
}
```

#### 実装ヒント
1. `processExceptionsWithFiltering`メソッドでは、`exceptionType`パラメータに基づいて、特定の例外だけを処理します。例えば、`exceptionType`が`"io"`の場合は`IOException`だけを処理し、それ以外の例外は再スローします。
2. `processExceptionsWithMultiCatch`メソッドでは、マルチキャッチ構文を使用して、`FileNotFoundException`と`SQLException`を1つの`catch`ブロックで処理します。
3. 例外の型を確認するには、`instanceof`演算子や`getClass().getSimpleName()`メソッドを使用します。

### 発展演習2: 例外処理のパフォーマンス測定（難易度：発展）

#### 課題
例外処理のパフォーマンスを測定するプログラムを作成してください。以下の処理のパフォーマンスを比較してください：

1. 例外をスローして処理する方法
2. 戻り値を使用してエラーを表現する方法
3. 例外をキャッチする頻度の違いによるパフォーマンスの変化

#### 雛形コード
以下のクラスを作成してください：

```java
package exceptions.performance;

/**
 * 例外処理のパフォーマンスを測定するデモクラス
 */
public class ExceptionPerformanceDemo {
    
    // 繰り返し回数
    private static final int ITERATIONS = 1000000;
    
    /**
     * 例外をスローして処理する方法でディレクトリの深さを計算します。
     * 
     * @param path パス
     * @return ディレクトリの深さ
     */
    public static int calculateDepthWithExceptions(String path) {
        // TODO: ここに例外をスローして処理する方法を実装
        // ヒント1: パスがnullまたは空の場合は例外をスロー
        // ヒント2: パスの深さを計算（スラッシュの数をカウント）
        
        return 0; // 仮の戻り値
    }
    
    /**
     * 戻り値を使用してエラーを表現する方法でディレクトリの深さを計算します。
     * 
     * @param path パス
     * @return ディレクトリの深さ（エラーの場合は-1）
     */
    public static int calculateDepthWithReturnValue(String path) {
        // TODO: ここに戻り値を使用してエラーを表現する方法を実装
        // ヒント1: パスがnullまたは空の場合は-1を返す
        // ヒント2: パスの深さを計算（スラッシュの数をカウント）
        
        return 0; // 仮の戻り値
    }
    
    /**
     * 例外をスローして処理する方法のパフォーマンスを測定します。
     * 
     * @param errorRate エラー率（0.0～1.0）
     * @return 実行時間（ミリ秒）
     */
    public static long measureExceptionPerformance(double errorRate) {
        // TODO: ここに例外をスローして処理する方法のパフォーマンス測定を実装
        // ヒント1: 開始時間を記録
        // ヒント2: ITERATIONS回繰り返し
        // ヒント3: errorRateの確率でnullを渡し、それ以外は有効なパスを渡す
        // ヒント4: 例外をキャッチして処理
        // ヒント5: 終了時間を記録し、実行時間を計算
        
        return 0; // 仮の戻り値
    }
    
    /**
     * 戻り値を使用してエラーを表現する方法のパフォーマンスを測定します。
     * 
     * @param errorRate エラー率（0.0～1.0）
     * @return 実行時間（ミリ秒）
     */
    public static long measureReturnValuePerformance(double errorRate) {
        // TODO: ここに戻り値を使用してエラーを表現する方法のパフォーマンス測定を実装
        // ヒント1: 開始時間を記録
        // ヒント2: ITERATIONS回繰り返し
        // ヒント3: errorRateの確率でnullを渡し、それ以外は有効なパスを渡す
        // ヒント4: 戻り値をチェックしてエラー処理
        // ヒント5: 終了時間を記録し、実行時間を計算
        
        return 0; // 仮の戻り値
    }
    
    public static void main(String[] args) {
        // エラー率の配列
        double[] errorRates = {0.0, 0.01, 0.1, 0.5, 1.0};
        
        System.out.println("===== 例外処理のパフォーマンス測定 =====");
        System.out.println("繰り返し回数: " + ITERATIONS);
        
        System.out.println("\n--- 例外をスローして処理する方法 vs 戻り値を使用してエラーを表現する方法 ---");
        System.out.println("エラー率\t例外処理(ms)\t戻り値(ms)\t比率(例外/戻り値)");
        
        for (double errorRate : errorRates) {
            // 例外をスローして処理する方法のパフォーマンスを測定
            long exceptionTime = measureExceptionPerformance(errorRate);
            
            // 戻り値を使用してエラーを表現する方法のパフォーマンスを測定
            long returnValueTime = measureReturnValuePerformance(errorRate);
            
            // 比率を計算
            double ratio = (double) exceptionTime / returnValueTime;
            
            // 結果を表示
            System.out.printf("%.2f\t%d\t\t%d\t\t%.2f%n", errorRate, exceptionTime, returnValueTime, ratio);
        }
        
        System.out.println("\n--- 考察 ---");
        // TODO: ここに測定結果の考察を記述
    }
}
```

#### 実装ヒント
1. `calculateDepthWithExceptions`メソッドでは、パスがnullまたは空の場合は例外をスローし、それ以外の場合はパスの深さを計算します。
2. `calculateDepthWithReturnValue`メソッドでは、パスがnullまたは空の場合は-1を返し、それ以外の場合はパスの深さを計算します。
3. `measureExceptionPerformance`メソッドでは、指定されたエラー率で`calculateDepthWithExceptions`メソッドを呼び出し、例外をキャッチして処理します。実行時間を測定して返します。
4. `measureReturnValuePerformance`メソッドでは、指定されたエラー率で`calculateDepthWithReturnValue`メソッドを呼び出し、戻り値をチェックしてエラー処理します。実行時間を測定して返します。
5. 測定結果を考察し、例外処理と戻り値を使用したエラー処理のパフォーマンスの違いを説明します。

## 解説と補足

### チェック例外と非チェック例外の使い分け

チェック例外と非チェック例外は、それぞれ異なる状況で使用することが適切です。以下に、それぞれの使い分けの指針を示します。

#### チェック例外を使用すべき場合

1. **回復可能なエラー**：
   - プログラムが回復可能なエラー状況を表す場合
   - 例：ファイルが見つからない、ネットワーク接続が切断された

2. **呼び出し元に通知すべきエラー**：
   - 呼び出し元がエラーを認識し、適切に対応する必要がある場合
   - 例：ユーザーが見つからない、認証に失敗した

3. **ビジネスロジックの一部としてのエラー**：
   - エラーがビジネスロジックの一部として予想される場合
   - 例：残高不足、在庫切れ

#### 非チェック例外を使用すべき場合

1. **プログラミングエラー**：
   - プログラマのミスによるエラーを表す場合
   - 例：NullPointerException、IndexOutOfBoundsException

2. **回復不可能なエラー**：
   - プログラムが回復不可能なエラー状況を表す場合
   - 例：OutOfMemoryError、StackOverflowError

3. **前提条件違反**：
   - メソッドの前提条件が満たされていない場合
   - 例：IllegalArgumentException、IllegalStateException

### 例外設計のベストプラクティス

例外を適切に設計し、使用するためのベストプラクティスを以下に示します。

1. **具体的な例外クラスを使用する**：
   - 汎用的な`Exception`や`RuntimeException`ではなく、より具体的な例外クラスを使用する
   - 例：`IllegalArgumentException`、`FileNotFoundException`

2. **意味のある例外メッセージを提供する**：
   - 例外の原因と対処方法を明確に示す例外メッセージを提供する
   - 例：`"ユーザーID 'user123'が見つかりません"`

3. **例外の連鎖を使用する**：
   - 低レベルの例外を高レベルの例外に変換する際に、元の例外を原因として保持する
   - 例：`throw new ServiceException("サービスエラー", originalException);`

4. **例外をドキュメント化する**：
   - メソッドがスローする可能性のある例外を`@throws`タグでドキュメント化する
   - 例：`@throws IllegalArgumentException 引数が無効な場合`

5. **例外をキャッチして無視しない**：
   - 例外をキャッチした場合は、適切に処理するか、ログに記録するか、再スローする
   - 例外を単に無視することは避ける

6. **finally ブロックでリソースをクローズする**：
   - リソースを使用する場合は、`finally`ブロックでリソースをクローズするか、`try-with-resources`構文を使用する
   - 例：ファイル、データベース接続、ネットワーク接続

### 例外処理のアンチパターン

例外処理において避けるべきアンチパターンを以下に示します。

1. **例外の乱用**：
   - 通常の制御フローに例外を使用する
   - 例：ループの終了条件として例外を使用する

2. **例外の抑制**：
   - 例外をキャッチして何もしない（空の`catch`ブロック）
   - 例：`catch (Exception e) {}`

3. **過度に細かい例外処理**：
   - 小さなコードブロックごとに`try-catch`を使用する
   - 代わりに、より大きな機能単位で例外を処理する

4. **汎用的な例外のスロー**：
   - 具体的な例外クラスではなく、汎用的な`Exception`や`RuntimeException`をスローする
   - 代わりに、より具体的な例外クラスを使用する

5. **例外の過度な変換**：
   - 例外を何度も変換し、元の例外情報を失う
   - 代わりに、例外の連鎖を使用して元の例外情報を保持する

### try-with-resourcesの利点

Java 7で導入された`try-with-resources`構文には、以下のような利点があります。

1. **リソースの自動クローズ**：
   - `try`ブロックの終了時に、自動的にリソースがクローズされる
   - 明示的な`finally`ブロックが不要になる

2. **コードの簡潔さ**：
   - 従来の`try-finally`ブロックよりも簡潔に記述できる
   - 特に複数のリソースを使用する場合に有効

3. **例外の抑制**：
   - リソースのクローズ時に例外が発生した場合、元の例外が優先され、クローズ時の例外は抑制される
   - 抑制された例外は、`getSuppressed()`メソッドで取得できる

4. **可読性の向上**：
   - リソースの宣言と使用が一箇所にまとまり、コードの可読性が向上する

## 実務での活用シーン

### 1. 外部リソースアクセス

ファイル、データベース、ネットワークなどの外部リソースにアクセスする際には、様々な例外が発生する可能性があります。適切な例外処理を行うことで、エラー状況を適切に処理し、リソースのリークを防ぐことができます。

```java
// データベース接続の例
public List<User> getAllUsers() throws DatabaseException {
    try (Connection conn = dataSource.getConnection();
         PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users");
         ResultSet rs = stmt.executeQuery()) {
        
        List<User> users = new ArrayList<>();
        while (rs.next()) {
            User user = new User();
            user.setId(rs.getString("id"));
            user.setName(rs.getString("name"));
            user.setEmail(rs.getString("email"));
            users.add(user);
        }
        return users;
    } catch (SQLException e) {
        throw new DatabaseException("ユーザー情報の取得に失敗しました", e);
    }
}
```

### 2. ビジネスロジックのバリデーション

ビジネスロジックにおけるバリデーションエラーは、適切な例外として表現することができます。チェック例外を使用することで、呼び出し元に明示的にエラー処理を要求することができます。

```java
// 送金処理の例
public void transfer(Account fromAccount, Account toAccount, BigDecimal amount) throws InsufficientFundsException, AccountNotFoundException {
    // アカウントの存在確認
    if (fromAccount == null) {
        throw new AccountNotFoundException("送金元アカウントが見つかりません");
    }
    if (toAccount == null) {
        throw new AccountNotFoundException("送金先アカウントが見つかりません");
    }
    
    // 残高確認
    if (fromAccount.getBalance().compareTo(amount) < 0) {
        throw new InsufficientFundsException("残高不足です。現在の残高: " + fromAccount.getBalance());
    }
    
    // 送金処理
    fromAccount.withdraw(amount);
    toAccount.deposit(amount);
    
    // トランザクション記録
    transactionRepository.recordTransfer(fromAccount, toAccount, amount);
}
```

### 3. 階層化されたエラー処理

多層アーキテクチャ（例：コントローラ、サービス、リポジトリ）では、各層で適切な例外処理を行うことが重要です。低レベルの例外を高レベルの例外に変換することで、抽象化レベルに適した例外を提供することができます。

```java
// リポジトリ層
public User findById(String userId) throws RepositoryException {
    try {
        // データベースアクセス
        return jdbcTemplate.queryForObject(
            "SELECT * FROM users WHERE id = ?",
            new Object[]{userId},
            userRowMapper
        );
    } catch (EmptyResultDataAccessException e) {
        return null;  // ユーザーが見つからない場合はnullを返す
    } catch (DataAccessException e) {
        throw new RepositoryException("ユーザー情報の取得に失敗しました", e);
    }
}

// サービス層
public User getUserById(String userId) throws UserNotFoundException, ServiceException {
    try {
        User user = userRepository.findById(userId);
        if (user == null) {
            throw new UserNotFoundException("ユーザーID '" + userId + "' が見つかりません");
        }
        return user;
    } catch (RepositoryException e) {
        throw new ServiceException("ユーザー情報の取得中にエラーが発生しました", e);
    }
}

// コントローラ層
@GetMapping("/users/{userId}")
public ResponseEntity<UserDto> getUser(@PathVariable String userId) {
    try {
        User user = userService.getUserById(userId);
        return ResponseEntity.ok(userMapper.toDto(user));
    } catch (UserNotFoundException e) {
        return ResponseEntity.notFound().build();
    } catch (ServiceException e) {
        logger.error("ユーザー情報の取得中にエラーが発生しました", e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
    }
}
```

### 4. リトライメカニズム

一時的なエラー（例：ネットワーク障害、データベース接続エラー）が発生した場合、リトライメカニズムを実装することで、処理の成功率を向上させることができます。

```java
// リトライメカニズムの例
public <T> T executeWithRetry(Callable<T> task, int maxRetries, long retryDelayMs) throws Exception {
    int retryCount = 0;
    Exception lastException = null;
    
    while (retryCount <= maxRetries) {
        try {
            return task.call();
        } catch (Exception e) {
            lastException = e;
            
            // リトライ可能なエラーかどうかを判断
            if (!isRetryableException(e) || retryCount >= maxRetries) {
                throw e;
            }
            
            // リトライ前に待機
            retryCount++;
            logger.warn("エラーが発生しました。リトライします ({}/{}): {}", retryCount, maxRetries, e.getMessage());
            Thread.sleep(retryDelayMs * retryCount);  // 指数バックオフ
        }
    }
    
    // ここには到達しないはずだが、コンパイラのために必要
    throw lastException;
}

// リトライ可能な例外かどうかを判断
private boolean isRetryableException(Exception e) {
    return e instanceof IOException ||
           e instanceof SQLException ||
           e instanceof TimeoutException;
}

// 使用例
public User getUserFromExternalService(String userId) throws ServiceException {
    try {
        return executeWithRetry(
            () -> externalService.getUser(userId),
            3,  // 最大リトライ回数
            1000  // リトライ間隔（ミリ秒）
        );
    } catch (Exception e) {
        throw new ServiceException("外部サービスからのユーザー情報取得に失敗しました", e);
    }
}
```

### 5. グレースフルデグラデーション

一部の機能がエラーになった場合でも、システム全体が停止しないように、グレースフルデグラデーション（優雅な機能低下）を実装することができます。

```java
// グレースフルデグラデーションの例
public UserProfileResponse getUserProfile(String userId) {
    UserProfileResponse response = new UserProfileResponse();
    
    // 基本ユーザー情報の取得
    try {
        User user = userService.getUserById(userId);
        response.setUser(user);
    } catch (Exception e) {
        logger.error("ユーザー情報の取得に失敗しました", e);
        response.setUserError("ユーザー情報の取得に失敗しました");
    }
    
    // ユーザーの注文履歴の取得
    try {
        List<Order> orders = orderService.getOrdersByUserId(userId);
        response.setOrders(orders);
    } catch (Exception e) {
        logger.error("注文履歴の取得に失敗しました", e);
        response.setOrdersError("注文履歴の取得に失敗しました");
    }
    
    // ユーザーのレビュー履歴の取得
    try {
        List<Review> reviews = reviewService.getReviewsByUserId(userId);
        response.setReviews(reviews);
    } catch (Exception e) {
        logger.error("レビュー履歴の取得に失敗しました", e);
        response.setReviewsError("レビュー履歴の取得に失敗しました");
    }
    
    return response;
}
```

これらの実務での活用シーンを参考に、適切な例外処理を設計し、実装してください。適切な例外処理は、プログラムの堅牢性と保守性を向上させる重要な要素です。
