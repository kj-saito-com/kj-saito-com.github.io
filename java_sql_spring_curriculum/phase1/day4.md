# Day 4: 例外処理の設計と実装

## 学習の目的と背景

例外処理は、プログラムの実行中に発生する予期しない状況や異常な状態を適切に扱うための重要なメカニズムです。適切な例外処理を行うことで、プログラムの堅牢性が向上し、エラーが発生した場合でも適切に対応できるようになります。

特に業務アプリケーションでは、ファイル操作、データベースアクセス、ネットワーク通信など、様々な外部リソースとの連携が必要となり、これらの処理では様々な例外が発生する可能性があります。例外を適切に処理することで、ユーザーに分かりやすいエラーメッセージを提示したり、システムの状態を正常に保ったりすることができます。

本日の学習では、Javaの例外処理の基本から、チェック例外と非チェック例外の違い、適切な例外設計のパターン、そして実務で役立つ例外処理のベストプラクティスまでを学びます。

## 前提知識と準備

### 必要な前提知識
- Javaの基本構文（変数、制御構文、クラス、オブジェクト）
- クラスの継承の概念
- Day 1で学習した入出力処理の基本

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

例外（Exception）は、プログラムの実行中に発生する異常な状態を表すオブジェクトです。Javaでは、例外は`Throwable`クラスのサブクラスとして表現されます。

<div class="term-box">
<strong>用語解説: 例外（Exception）</strong><br>
プログラムの実行中に発生する異常な状態を表すオブジェクトです。例外が発生すると、通常のプログラムの流れが中断され、例外を処理するコード（例外ハンドラ）に制御が移ります。例外は、`Throwable`クラスのサブクラスとして表現されます。
</div>

Javaの例外階層：

```
Throwable
├── Error（システムエラー、通常はキャッチしない）
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
└── Exception
    ├── チェック例外（コンパイラがチェックする例外）
    │   ├── IOException
    │   ├── SQLException
    │   └── ...
    └── RuntimeException（非チェック例外）
        ├── NullPointerException
        ├── IllegalArgumentException
        ├── IndexOutOfBoundsException
        └── ...
```

### 2. チェック例外と非チェック例外

Javaの例外は、大きく「チェック例外」と「非チェック例外」の2種類に分けられます。

<div class="term-box">
<strong>用語解説: チェック例外（Checked Exception）</strong><br>
コンパイル時にチェックされる例外です。メソッドがチェック例外をスローする可能性がある場合、そのメソッドの宣言に`throws`節を含めるか、`try-catch`ブロックで例外を処理する必要があります。`Exception`クラスのサブクラスで、`RuntimeException`のサブクラスでないものがチェック例外です。
</div>

<div class="term-box">
<strong>用語解説: 非チェック例外（Unchecked Exception）</strong><br>
コンパイル時にチェックされない例外です。メソッドが非チェック例外をスローする可能性がある場合でも、`throws`節を宣言する必要はありません。`RuntimeException`クラスとそのサブクラス、および`Error`クラスとそのサブクラスが非チェック例外です。
</div>

#### チェック例外の例
- `IOException`: 入出力操作に関する例外
- `SQLException`: データベース操作に関する例外
- `ClassNotFoundException`: クラスが見つからない場合の例外

#### 非チェック例外の例
- `NullPointerException`: nullオブジェクトに対してメソッドを呼び出した場合の例外
- `IllegalArgumentException`: メソッドに不正な引数が渡された場合の例外
- `IndexOutOfBoundsException`: 配列やリストの範囲外のインデックスにアクセスした場合の例外

#### 使い分けの指針

- **チェック例外**：
  - 回復可能なエラー
  - 呼び出し元に対応を強制したいエラー
  - 例：ファイルが見つからない、ネットワーク接続の問題

- **非チェック例外**：
  - プログラミングエラー（バグ）
  - 回復が困難または不可能なエラー
  - 例：NullPointerException、配列の範囲外アクセス

### 3. 例外処理の基本構文

#### try-catch-finally

```java
try {
    // 例外が発生する可能性のあるコード
    FileReader file = new FileReader("example.txt");
    BufferedReader reader = new BufferedReader(file);
    String line = reader.readLine();
    System.out.println(line);
} catch (FileNotFoundException e) {
    // FileNotFoundExceptionが発生した場合の処理
    System.err.println("ファイルが見つかりません: " + e.getMessage());
} catch (IOException e) {
    // IOExceptionが発生した場合の処理
    System.err.println("入出力エラーが発生しました: " + e.getMessage());
} finally {
    // 例外の発生有無にかかわらず実行される処理
    // リソースのクローズなど
    try {
        if (reader != null) reader.close();
    } catch (IOException e) {
        System.err.println("ファイルのクローズに失敗しました: " + e.getMessage());
    }
}
```

#### try-with-resources（Java 7以降）

リソースを自動的にクローズするための構文です。`AutoCloseable`インターフェースを実装したクラスで使用できます。

```java
try (
    FileReader file = new FileReader("example.txt");
    BufferedReader reader = new BufferedReader(file)
) {
    // 例外が発生する可能性のあるコード
    String line = reader.readLine();
    System.out.println(line);
} catch (FileNotFoundException e) {
    // FileNotFoundExceptionが発生した場合の処理
    System.err.println("ファイルが見つかりません: " + e.getMessage());
} catch (IOException e) {
    // IOExceptionが発生した場合の処理
    System.err.println("入出力エラーが発生しました: " + e.getMessage());
}
// finallyブロックは不要（リソースは自動的にクローズされる）
```

#### マルチキャッチ（Java 7以降）

複数の例外を1つの`catch`ブロックで処理できます。

```java
try {
    // 例外が発生する可能性のあるコード
} catch (FileNotFoundException | NullPointerException e) {
    // FileNotFoundExceptionまたはNullPointerExceptionが発生した場合の処理
    System.err.println("エラーが発生しました: " + e.getMessage());
}
```

### 4. 例外のスロー

`throw`キーワードを使用して、明示的に例外をスローできます。

```java
public void validateAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("年齢は0以上である必要があります");
    }
    // 正常な処理
}
```

メソッドが例外をスローする可能性がある場合、メソッド宣言に`throws`節を含める必要があります（チェック例外の場合）。

```java
public void readFile(String filename) throws IOException {
    FileReader file = new FileReader(filename);
    // ファイル読み込み処理
}
```

### 5. 独自例外クラスの作成

特定のエラー状況に対応するために、独自の例外クラスを作成できます。

```java
// チェック例外の例
public class BusinessException extends Exception {
    public BusinessException(String message) {
        super(message);
    }
    
    public BusinessException(String message, Throwable cause) {
        super(message, cause);
    }
}

// 非チェック例外の例
public class ValidationException extends RuntimeException {
    public ValidationException(String message) {
        super(message);
    }
    
    public ValidationException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### 6. 例外設計のパターン

#### 例外の変換（Exception Translation）

低レベルの例外を、より意味のある高レベルの例外に変換するパターンです。

```java
public void saveUser(User user) throws BusinessException {
    try {
        // データベース操作
        userDao.save(user);
    } catch (SQLException e) {
        // SQLExceptionをBusinessExceptionに変換
        throw new BusinessException("ユーザーの保存に失敗しました", e);
    }
}
```

#### 例外のラッピング（Exception Wrapping）

元の例外を新しい例外の原因（cause）として保持するパターンです。

```java
try {
    // 何らかの処理
} catch (Exception e) {
    throw new RuntimeException("処理に失敗しました", e);
}
```

#### 例外の再スロー（Exception Rethrowing）

例外をキャッチした後、何らかの処理を行ってから再スローするパターンです。

```java
try {
    // 何らかの処理
} catch (IOException e) {
    // ログ出力など
    logger.error("入出力エラーが発生しました", e);
    throw e; // 例外を再スロー
}
```

### 7. 例外処理のベストプラクティス

1. **早期に失敗する（Fail Fast）**：
   - エラーが発生したら、できるだけ早く例外をスローする
   - 無効な状態でプログラムを継続させない

2. **適切な例外を使用する**：
   - チェック例外と非チェック例外を適切に使い分ける
   - 具体的な例外クラスを使用する（`Exception`ではなく`IllegalArgumentException`など）

3. **意味のあるメッセージを提供する**：
   - 例外メッセージには、エラーの原因と対処方法を含める
   - デバッグに役立つ情報（パラメータ値など）を含める

4. **例外を無視しない**：
   - 空の`catch`ブロックは避ける
   - 最低でもログ出力を行う

5. **リソースを適切に解放する**：
   - `try-with-resources`を使用する
   - `finally`ブロックでリソースを確実に解放する

6. **例外の連鎖を維持する**：
   - 例外を変換する場合は、元の例外を原因（cause）として保持する

## Eclipse操作ガイド

### 例外処理の自動生成

Eclipseには、例外処理コードを自動生成する機能があります：

1. 例外をスローするメソッド（例：`FileReader`のコンストラクタ）を入力します。
2. エラーが表示されたら、行の左端に表示される電球アイコンをクリックします。
3. 「Surround with try/catch」または「Add throws declaration」を選択します。

![Eclipse例外処理自動生成](https://example.com/eclipse_exception_handling.png)

### try-with-resourcesの変換

既存の`try-catch-finally`ブロックを`try-with-resources`に変換できます：

1. `try`ブロック内のリソース宣言部分を選択します。
2. 右クリックして「Refactor」→「Convert to try-with-resources」を選択します。

### 例外のデバッグ

例外が発生した場合のデバッグ方法：

1. 「Window」→「Preferences」→「Java」→「Debug」→「Suspend execution on uncaught exceptions」にチェックを入れます。
2. デバッグモードでプログラムを実行します。
3. 未キャッチの例外が発生すると、その場所で実行が一時停止します。
4. 「Variables」ビューで例外オブジェクトの詳細を確認できます。
5. 「Debug」ビューでスタックトレースを確認できます。

![Eclipseデバッグ例外](https://example.com/eclipse_debug_exception.png)

## コア演習（必須）

### 演習1: 基本的な例外処理（難易度：基本）

#### 課題
ファイルの読み込みと書き込みを行うプログラムを作成し、適切な例外処理を実装してください。以下の要件を満たす必要があります：
1. 存在しないファイルを読み込もうとした場合、適切なエラーメッセージを表示する
2. ファイルの読み込み中にエラーが発生した場合、適切なエラーメッセージを表示する
3. リソース（ファイル）を確実に解放する

#### 雛形コード
```java
package exception.basic;

import java.io.*;

/**
 * 基本的な例外処理を練習するクラス
 */
public class BasicExceptionHandling {

    /**
     * ファイルを読み込み、内容を表示します。
     * 
     * @param filename 読み込むファイルのパス
     */
    public void readFile(String filename) {
        // TODO: try-with-resourcesを使用して、ファイルを読み込み、内容を表示する
        // TODO: FileNotFoundExceptionとIOExceptionを適切に処理する
    }

    /**
     * ファイルに文字列を書き込みます。
     * 
     * @param filename 書き込むファイルのパス
     * @param content 書き込む内容
     */
    public void writeFile(String filename, String content) {
        // TODO: try-with-resourcesを使用して、ファイルに文字列を書き込む
        // TODO: IOExceptionを適切に処理する
    }

    public static void main(String[] args) {
        BasicExceptionHandling demo = new BasicExceptionHandling();
        
        // 存在するファイルの作成
        demo.writeFile("sample.txt", "これはサンプルファイルです。\n例外処理の練習に使用します。");
        
        // 存在するファイルの読み込み
        System.out.println("=== 存在するファイルの読み込み ===");
        demo.readFile("sample.txt");
        
        // 存在しないファイルの読み込み
        System.out.println("\n=== 存在しないファイルの読み込み ===");
        demo.readFile("nonexistent.txt");
    }
}
```

#### 実装ヒント
- `try-with-resources`構文を使用して、リソースを自動的にクローズします。
- `FileReader`と`BufferedReader`を使用してファイルを読み込みます。
- `FileWriter`と`BufferedWriter`を使用してファイルに書き込みます。
- `FileNotFoundException`と`IOException`を適切に処理します。
- 例外が発生した場合は、エラーメッセージを表示します（`System.err.println`を使用）。

### 演習2: チェック例外と非チェック例外の使い分け（難易度：応用）

#### 課題
ユーザー登録システムを模したプログラムを作成し、チェック例外と非チェック例外を適切に使い分けてください。以下の要件を満たす必要があります：
1. ユーザー名が無効（nullまたは空）の場合、非チェック例外（`IllegalArgumentException`）をスローする
2. ユーザー名が既に使用されている場合、チェック例外（独自の`UserAlreadyExistsException`）をスローする
3. データベース接続エラーが発生した場合、チェック例外（独自の`DatabaseException`）をスローする

#### 雛形コード
```java
package exception.advanced;

import java.util.*;

/**
 * チェック例外と非チェック例外の使い分けを練習するクラス
 */
public class UserRegistrationSystem {

    // ユーザーデータベースの代わりとなるマップ
    private Map<String, User> userDatabase = new HashMap<>();
    
    // データベース接続状態を模擬するフラグ
    private boolean databaseConnected = true;

    /**
     * ユーザーが既に存在する場合にスローされるチェック例外
     */
    public static class UserAlreadyExistsException extends Exception {
        // TODO: コンストラクタを実装
    }

    /**
     * データベースエラーが発生した場合にスローされるチェック例外
     */
    public static class DatabaseException extends Exception {
        // TODO: コンストラクタを実装
    }

    /**
     * ユーザークラス
     */
    public static class User {
        private String username;
        private String email;
        
        public User(String username, String email) {
            this.username = username;
            this.email = email;
        }
        
        public String getUsername() { return username; }
        public String getEmail() { return email; }
        
        @Override
        public String toString() {
            return "User{username='" + username + "', email='" + email + "'}";
        }
    }

    /**
     * ユーザーを登録します。
     * 
     * @param username ユーザー名
     * @param email メールアドレス
     * @throws UserAlreadyExistsException ユーザー名が既に使用されている場合
     * @throws DatabaseException データベース接続エラーが発生した場合
     * @throws IllegalArgumentException ユーザー名またはメールアドレスが無効な場合
     */
    public void registerUser(String username, String email) throws UserAlreadyExistsException, DatabaseException {
        // TODO: ユーザー名とメールアドレスの検証（nullまたは空の場合はIllegalArgumentExceptionをスロー）
        
        // TODO: データベース接続状態のチェック（接続されていない場合はDatabaseExceptionをスロー）
        
        // TODO: ユーザー名の重複チェック（既に存在する場合はUserAlreadyExistsExceptionをスロー）
        
        // TODO: ユーザーの登録
        
        System.out.println("ユーザーが正常に登録されました: " + username);
    }

    /**
     * 登録されているすべてのユーザーを表示します。
     */
    public void listAllUsers() {
        System.out.println("=== 登録ユーザー一覧 ===");
        if (userDatabase.isEmpty()) {
            System.out.println("登録ユーザーはいません。");
        } else {
            for (User user : userDatabase.values()) {
                System.out.println(user);
            }
        }
    }

    /**
     * データベース接続状態を設定します（テスト用）。
     */
    public void setDatabaseConnected(boolean connected) {
        this.databaseConnected = connected;
    }

    public static void main(String[] args) {
        UserRegistrationSystem system = new UserRegistrationSystem();
        
        // 正常なケース
        try {
            system.registerUser("alice", "alice@example.com");
        } catch (UserAlreadyExistsException e) {
            System.err.println("エラー: " + e.getMessage());
        } catch (DatabaseException e) {
            System.err.println("データベースエラー: " + e.getMessage());
        } catch (IllegalArgumentException e) {
            System.err.println("入力エラー: " + e.getMessage());
        }
        
        // 重複ユーザー名のケース
        try {
            system.registerUser("alice", "alice2@example.com");
        } catch (UserAlreadyExistsException e) {
            System.err.println("エラー: " + e.getMessage());
        } catch (DatabaseException e) {
            System.err.println("データベースエラー: " + e.getMessage());
        } catch (IllegalArgumentException e) {
            System.err.println("入力エラー: " + e.getMessage());
        }
        
        // 無効な入力のケース
        try {
            system.registerUser("", "bob@example.com");
        } catch (UserAlreadyExistsException e) {
            System.err.println("エラー: " + e.getMessage());
        } catch (DatabaseException e) {
            System.err.println("データベースエラー: " + e.getMessage());
        } catch (IllegalArgumentException e) {
            System.err.println("入力エラー: " + e.getMessage());
        }
        
        // データベース接続エラーのケース
        system.setDatabaseConnected(false);
        try {
            system.registerUser("bob", "bob@example.com");
        } catch (UserAlreadyExistsException e) {
            System.err.println("エラー: " + e.getMessage());
        } catch (DatabaseException e) {
            System.err.println("データベースエラー: " + e.getMessage());
        } catch (IllegalArgumentException e) {
            System.err.println("入力エラー: " + e.getMessage());
        }
        
        // 登録ユーザー一覧の表示
        system.listAllUsers();
    }
}
```

#### 実装ヒント
- `UserAlreadyExistsException`と`DatabaseException`のコンストラクタを実装します。少なくとも、メッセージを受け取るコンストラクタと、メッセージと原因（cause）を受け取るコンストラクタを実装してください。
- `registerUser`メソッドでは、以下の順序で検証を行います：
  1. ユーザー名とメールアドレスの検証（nullまたは空の場合は`IllegalArgumentException`をスロー）
  2. データベース接続状態のチェック（接続されていない場合は`DatabaseException`をスロー）
  3. ユーザー名の重複チェック（既に存在する場合は`UserAlreadyExistsException`をスロー）
  4. ユーザーの登録（`userDatabase`に追加）
- 例外メッセージには、エラーの原因を明確に示す情報を含めてください。

### 演習3: 例外の変換とラッピング（難易度：応用）

#### 課題
ファイル操作を行うユーティリティクラスを作成し、低レベルの例外（`IOException`など）を、より意味のある業務例外に変換してください。以下の要件を満たす必要があります：
1. ファイルが見つからない場合、独自の`FileOperationException`をスローする
2. ファイルの読み込み中にエラーが発生した場合、`FileOperationException`をスローし、元の例外を原因（cause）として保持する
3. ファイルの内容が無効な場合、独自の`InvalidContentException`をスローする

#### 雛形コード
```java
package exception.advanced;

import java.io.*;
import java.util.*;

/**
 * 例外の変換とラッピングを練習するクラス
 */
public class FileOperationUtil {

    /**
     * ファイル操作に関する例外
     */
    public static class FileOperationException extends Exception {
        // TODO: コンストラクタを実装
    }

    /**
     * ファイルの内容が無効な場合の例外
     */
    public static class InvalidContentException extends Exception {
        // TODO: コンストラクタを実装
    }

    /**
     * ファイルからユーザーリストを読み込みます。
     * 各行は「ユーザー名,メールアドレス」の形式である必要があります。
     * 
     * @param filename 読み込むファイルのパス
     * @return ユーザーのリスト
     * @throws FileOperationException ファイル操作に失敗した場合
     * @throws InvalidContentException ファイルの内容が無効な場合
     */
    public List<UserRegistrationSystem.User> loadUsersFromFile(String filename) throws FileOperationException, InvalidContentException {
        // TODO: ファイルからユーザーリストを読み込む
        // TODO: IOExceptionをFileOperationExceptionに変換する
        // TODO: ファイルの各行を解析し、無効な形式の場合はInvalidContentExceptionをスローする
        
        return null; // 仮の戻り値
    }

    /**
     * ユーザーリストをファイルに保存します。
     * 各ユーザーは「ユーザー名,メールアドレス」の形式で保存されます。
     * 
     * @param users 保存するユーザーのリスト
     * @param filename 保存先ファイルのパス
     * @throws FileOperationException ファイル操作に失敗した場合
     */
    public void saveUsersToFile(List<UserRegistrationSystem.User> users, String filename) throws FileOperationException {
        // TODO: ユーザーリストをファイルに保存する
        // TODO: IOExceptionをFileOperationExceptionに変換する
    }

    /**
     * テスト用のユーザーデータを作成します。
     */
    private List<UserRegistrationSystem.User> createSampleUsers() {
        List<UserRegistrationSystem.User> users = new ArrayList<>();
        users.add(new UserRegistrationSystem.User("alice", "alice@example.com"));
        users.add(new UserRegistrationSystem.User("bob", "bob@example.com"));
        users.add(new UserRegistrationSystem.User("charlie", "charlie@example.com"));
        return users;
    }

    /**
     * テスト用の無効なユーザーデータファイルを作成します。
     */
    private void createInvalidUserFile(String filename) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            writer.write("alice,alice@example.com\n");
            writer.write("bob,bob@example.com\n");
            writer.write("invalid_line\n"); // 無効な形式の行
            writer.write("charlie,charlie@example.com\n");
        }
    }

    public static void main(String[] args) {
        FileOperationUtil util = new FileOperationUtil();
        
        // ユーザーリストの保存
        try {
            List<UserRegistrationSystem.User> users = util.createSampleUsers();
            util.saveUsersToFile(users, "users.txt");
            System.out.println("ユーザーリストを保存しました。");
        } catch (FileOperationException e) {
            System.err.println("ファイル操作エラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        }
        
        // ユーザーリストの読み込み
        try {
            List<UserRegistrationSystem.User> users = util.loadUsersFromFile("users.txt");
            System.out.println("読み込んだユーザーリスト:");
            for (UserRegistrationSystem.User user : users) {
                System.out.println(user);
            }
        } catch (FileOperationException e) {
            System.err.println("ファイル操作エラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        } catch (InvalidContentException e) {
            System.err.println("無効な内容: " + e.getMessage());
        }
        
        // 存在しないファイルの読み込み
        try {
            util.loadUsersFromFile("nonexistent.txt");
        } catch (FileOperationException e) {
            System.err.println("ファイル操作エラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        } catch (InvalidContentException e) {
            System.err.println("無効な内容: " + e.getMessage());
        }
        
        // 無効な内容のファイルの読み込み
        try {
            util.createInvalidUserFile("invalid_users.txt");
            util.loadUsersFromFile("invalid_users.txt");
        } catch (FileOperationException e) {
            System.err.println("ファイル操作エラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        } catch (InvalidContentException e) {
            System.err.println("無効な内容: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("IOエラー: " + e.getMessage());
        }
    }
}
```

#### 実装ヒント
- `FileOperationException`と`InvalidContentException`のコンストラクタを実装します。少なくとも、メッセージを受け取るコンストラクタと、メッセージと原因（cause）を受け取るコンストラクタを実装してください。
- `loadUsersFromFile`メソッドでは、以下の処理を行います：
  1. `try-with-resources`を使用してファイルを読み込みます。
  2. `FileNotFoundException`が発生した場合、適切なメッセージを含む`FileOperationException`に変換します。
  3. その他の`IOException`が発生した場合、元の例外を原因（cause）として含む`FileOperationException`に変換します。
  4. ファイルの各行を解析し、無効な形式の場合は`InvalidContentException`をスローします。
- `saveUsersToFile`メソッドでは、以下の処理を行います：
  1. `try-with-resources`を使用してファイルに書き込みます。
  2. `IOException`が発生した場合、元の例外を原因（cause）として含む`FileOperationException`に変換します。
- 例外メッセージには、エラーの原因を明確に示す情報を含めてください。

## 発展演習（オプション）

### 発展演習1: 例外フィルタリングとロギング（難易度：発展）

#### 課題
例外をフィルタリングし、適切にログ出力するユーティリティクラスを作成してください。以下の要件を満たす必要があります：
1. 例外の種類に応じて、異なるログレベル（INFO, WARNING, ERROR, FATAL）で出力する
2. 特定の例外は無視する（ログ出力しない）
3. 例外のスタックトレースを整形して出力する

#### 雛形コード
```java
package exception.expert;

import java.io.*;
import java.util.*;
import java.util.function.Predicate;

/**
 * 例外フィルタリングとロギングを行うユーティリティクラス
 */
public class ExceptionLogger {

    // ログレベルの定義
    public enum LogLevel {
        INFO, WARNING, ERROR, FATAL
    }

    // 例外フィルタの定義
    private Map<Predicate<Throwable>, LogLevel> filters = new HashMap<>();
    
    // 無視する例外の定義
    private List<Predicate<Throwable>> ignoreFilters = new ArrayList<>();
    
    // ログ出力先
    private PrintStream logOutput = System.err;

    /**
     * コンストラクタ
     */
    public ExceptionLogger() {
        // デフォルトのフィルタを設定
        
        // RuntimeExceptionはWARNINGレベルでログ出力
        addFilter(e -> e instanceof RuntimeException, LogLevel.WARNING);
        
        // IOExceptionはERRORレベルでログ出力
        addFilter(e -> e instanceof IOException, LogLevel.ERROR);
        
        // OutOfMemoryErrorはFATALレベルでログ出力
        addFilter(e -> e instanceof OutOfMemoryError, LogLevel.FATAL);
        
        // その他の例外はINFOレベルでログ出力（デフォルト）
    }

    /**
     * 例外フィルタを追加します。
     * 
     * @param filter 例外を判定する述語
     * @param level ログレベル
     */
    public void addFilter(Predicate<Throwable> filter, LogLevel level) {
        filters.put(filter, level);
    }

    /**
     * 無視する例外フィルタを追加します。
     * 
     * @param filter 無視する例外を判定する述語
     */
    public void addIgnoreFilter(Predicate<Throwable> filter) {
        ignoreFilters.add(filter);
    }

    /**
     * ログ出力先を設定します。
     * 
     * @param output ログ出力先
     */
    public void setLogOutput(PrintStream output) {
        this.logOutput = output;
    }

    /**
     * 例外をログ出力します。
     * 
     * @param e ログ出力する例外
     * @param message 追加のメッセージ
     */
    public void log(Throwable e, String message) {
        // TODO: 無視フィルタに一致する場合は何もしない
        
        // TODO: 適切なログレベルを決定
        
        // TODO: ログを出力
        // フォーマット: [レベル] メッセージ: 例外メッセージ
        // スタックトレースも出力
    }

    /**
     * 例外に適用するログレベルを決定します。
     * 
     * @param e 例外
     * @return ログレベル
     */
    private LogLevel determineLogLevel(Throwable e) {
        // TODO: フィルタを順に適用し、一致するものがあればそのログレベルを返す
        // TODO: 一致するものがなければデフォルトのINFOを返す
        
        return LogLevel.INFO; // 仮の戻り値
    }

    /**
     * スタックトレースを文字列に変換します。
     * 
     * @param e 例外
     * @return スタックトレースの文字列表現
     */
    private String formatStackTrace(Throwable e) {
        // TODO: スタックトレースを文字列に変換
        // StringWriterとPrintWriterを使用すると便利
        
        return ""; // 仮の戻り値
    }

    public static void main(String[] args) {
        ExceptionLogger logger = new ExceptionLogger();
        
        // FileNotFoundExceptionを無視するフィルタを追加
        logger.addIgnoreFilter(e -> e instanceof FileNotFoundException);
        
        // IllegalArgumentExceptionはERRORレベルでログ出力するフィルタを追加
        logger.addFilter(e -> e instanceof IllegalArgumentException, LogLevel.ERROR);
        
        // 様々な例外をログ出力
        try {
            // FileNotFoundException（無視される）
            new FileInputStream("nonexistent.txt");
        } catch (Exception e) {
            logger.log(e, "ファイルの読み込みに失敗しました");
        }
        
        try {
            // IllegalArgumentException（ERRORレベル）
            Integer.parseInt("not_a_number");
        } catch (Exception e) {
            logger.log(e, "数値の解析に失敗しました");
        }
        
        try {
            // NullPointerException（WARNINGレベル）
            String s = null;
            s.length();
        } catch (Exception e) {
            logger.log(e, "nullポインタが発生しました");
        }
        
        try {
            // IOException（ERRORレベル）
            throw new IOException("I/Oエラーが発生しました");
        } catch (Exception e) {
            logger.log(e, "I/O操作に失敗しました");
        }
    }
}
```

#### 実装ヒント
- `log`メソッドでは、まず無視フィルタに一致するかどうかをチェックします。一致する場合は何もしません。
- `determineLogLevel`メソッドでは、フィルタを順に適用し、一致するものがあればそのログレベルを返します。一致するものがなければデフォルトの`INFO`を返します。
- `formatStackTrace`メソッドでは、`StringWriter`と`PrintWriter`を使用して、例外のスタックトレースを文字列に変換します。
- ログ出力の形式は、`[レベル] メッセージ: 例外メッセージ`とし、その後にスタックトレースを出力します。

### 発展演習2: 例外処理のパフォーマンス測定（難易度：発展）

#### 課題
例外処理のパフォーマンスを測定するプログラムを作成してください。以下の要件を満たす必要があります：
1. 例外をスローする場合と、通常の条件分岐を使用する場合のパフォーマンスを比較する
2. 例外のスタックトレースの有無によるパフォーマンスの違いを測定する
3. 測定結果をグラフィカルに表示する（オプション）

#### 雛形コード
```java
package exception.expert;

import java.io.*;
import java.util.*;

/**
 * 例外処理のパフォーマンスを測定するクラス
 */
public class ExceptionPerformanceTester {

    // 測定回数
    private static final int ITERATIONS = 1_000_000;
    
    /**
     * 例外をスローする方法でディレクトリの存在をチェックします。
     * 
     * @param path チェックするパス
     * @return ディレクトリが存在する場合はtrue
     */
    public boolean checkDirectoryWithException(String path) {
        try {
            // ディレクトリを開いてみる
            File dir = new File(path);
            if (dir.listFiles() == null) {
                return false;
            }
            return true;
        } catch (SecurityException e) {
            return false;
        }
    }
    
    /**
     * 条件分岐を使用する方法でディレクトリの存在をチェックします。
     * 
     * @param path チェックするパス
     * @return ディレクトリが存在する場合はtrue
     */
    public boolean checkDirectoryWithCondition(String path) {
        File dir = new File(path);
        return dir.isDirectory() && dir.exists();
    }
    
    /**
     * スタックトレース付きの例外をスローします。
     */
    public void throwExceptionWithStackTrace() {
        try {
            throw new Exception("Test exception with stack trace");
        } catch (Exception e) {
            // 何もしない
        }
    }
    
    /**
     * スタックトレースなしの例外をスローします。
     */
    public void throwExceptionWithoutStackTrace() {
        try {
            Exception e = new Exception("Test exception without stack trace");
            e.setStackTrace(new StackTraceElement[0]); // 空のスタックトレース
            throw e;
        } catch (Exception e) {
            // 何もしない
        }
    }
    
    /**
     * パフォーマンスを測定します。
     * 
     * @param runnable 測定対象の処理
     * @param iterations 繰り返し回数
     * @return 実行時間（ミリ秒）
     */
    public long measurePerformance(Runnable runnable, int iterations) {
        // ウォームアップ
        for (int i = 0; i < 1000; i++) {
            runnable.run();
        }
        
        // 測定
        long startTime = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            runnable.run();
        }
        long endTime = System.nanoTime();
        
        return (endTime - startTime) / 1_000_000; // ナノ秒からミリ秒に変換
    }
    
    /**
     * 測定結果を表示します。
     * 
     * @param title タイトル
     * @param time1 1つ目の測定結果
     * @param label1 1つ目のラベル
     * @param time2 2つ目の測定結果
     * @param label2 2つ目のラベル
     */
    public void displayResults(String title, long time1, String label1, long time2, String label2) {
        System.out.println("=== " + title + " ===");
        System.out.println(label1 + ": " + time1 + "ms");
        System.out.println(label2 + ": " + time2 + "ms");
        System.out.println("比率（" + label2 + "/" + label1 + "）: " + (double) time2 / time1);
        System.out.println();
    }
    
    /**
     * グラフィカルに結果を表示します（オプション）。
     * 
     * @param title タイトル
     * @param results 測定結果のマップ（ラベル -> 時間）
     */
    public void displayGraphicalResults(String title, Map<String, Long> results) {
        // TODO: 結果をグラフィカルに表示（オプション）
        // 例: 簡易的なアスキーアートグラフを表示
        System.out.println("=== " + title + " ===");
        
        // 最大値を求める
        long maxValue = 0;
        for (long value : results.values()) {
            maxValue = Math.max(maxValue, value);
        }
        
        // スケールを計算（最大値を50文字に収める）
        double scale = 50.0 / maxValue;
        
        // グラフを表示
        for (Map.Entry<String, Long> entry : results.entrySet()) {
            String label = entry.getKey();
            long value = entry.getValue();
            int barLength = (int) (value * scale);
            
            System.out.printf("%-30s | %-10d | %s%n", label, value, "#".repeat(barLength));
        }
        System.out.println();
    }

    public static void main(String[] args) {
        ExceptionPerformanceTester tester = new ExceptionPerformanceTester();
        
        // 存在するディレクトリと存在しないディレクトリのパス
        String existingDir = System.getProperty("user.dir"); // 現在のディレクトリ
        String nonExistingDir = "/path/to/nonexistent/directory";
        
        // 1. 例外をスローする場合と条件分岐を使用する場合の比較（存在するディレクトリ）
        long exceptionTime1 = tester.measurePerformance(() -> tester.checkDirectoryWithException(existingDir), ITERATIONS);
        long conditionTime1 = tester.measurePerformance(() -> tester.checkDirectoryWithCondition(existingDir), ITERATIONS);
        tester.displayResults("存在するディレクトリの場合", conditionTime1, "条件分岐", exceptionTime1, "例外スロー");
        
        // 2. 例外をスローする場合と条件分岐を使用する場合の比較（存在しないディレクトリ）
        long exceptionTime2 = tester.measurePerformance(() -> tester.checkDirectoryWithException(nonExistingDir), ITERATIONS);
        long conditionTime2 = tester.measurePerformance(() -> tester.checkDirectoryWithCondition(nonExistingDir), ITERATIONS);
        tester.displayResults("存在しないディレクトリの場合", conditionTime2, "条件分岐", exceptionTime2, "例外スロー");
        
        // 3. スタックトレースの有無による比較
        long withStackTraceTime = tester.measurePerformance(() -> tester.throwExceptionWithStackTrace(), ITERATIONS);
        long withoutStackTraceTime = tester.measurePerformance(() -> tester.throwExceptionWithoutStackTrace(), ITERATIONS);
        tester.displayResults("スタックトレースの有無", withoutStackTraceTime, "スタックトレースなし", withStackTraceTime, "スタックトレースあり");
        
        // 4. グラフィカル表示（オプション）
        Map<String, Long> results = new LinkedHashMap<>();
        results.put("条件分岐（存在する）", conditionTime1);
        results.put("例外スロー（存在する）", exceptionTime1);
        results.put("条件分岐（存在しない）", conditionTime2);
        results.put("例外スロー（存在しない）", exceptionTime2);
        results.put("スタックトレースなし", withoutStackTraceTime);
        results.put("スタックトレースあり", withStackTraceTime);
        tester.displayGraphicalResults("パフォーマンス比較", results);
    }
}
```

#### 実装ヒント
- `measurePerformance`メソッドでは、まずウォームアップを行い、JVMの最適化を促します。その後、指定された回数だけ処理を実行し、実行時間を測定します。
- `displayResults`メソッドでは、2つの測定結果を比較し、比率を計算して表示します。
- `displayGraphicalResults`メソッドでは、測定結果をグラフィカルに表示します。簡易的なアスキーアートグラフを表示する実装例を提供していますが、より高度な表示方法を実装しても構いません。
- 例外処理のパフォーマンスに影響を与える要素として、以下の点に注目してください：
  - 例外がスローされる頻度（正常ケースと異常ケースの比率）
  - スタックトレースの生成コスト
  - 例外オブジェクトの生成コスト
  - キャッチブロックの実行コスト

## 解答例

### 演習1: 基本的な例外処理の解答例

```java
package exception.basic;

import java.io.*;

/**
 * 基本的な例外処理を練習するクラス
 */
public class BasicExceptionHandling {

    /**
     * ファイルを読み込み、内容を表示します。
     * 
     * @param filename 読み込むファイルのパス
     */
    public void readFile(String filename) {
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (FileNotFoundException e) {
            System.err.println("ファイルが見つかりません: " + filename);
            System.err.println("エラーの詳細: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("ファイルの読み込み中にエラーが発生しました: " + filename);
            System.err.println("エラーの詳細: " + e.getMessage());
        }
    }

    /**
     * ファイルに文字列を書き込みます。
     * 
     * @param filename 書き込むファイルのパス
     * @param content 書き込む内容
     */
    public void writeFile(String filename, String content) {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            writer.write(content);
        } catch (IOException e) {
            System.err.println("ファイルの書き込み中にエラーが発生しました: " + filename);
            System.err.println("エラーの詳細: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        BasicExceptionHandling demo = new BasicExceptionHandling();
        
        // 存在するファイルの作成
        demo.writeFile("sample.txt", "これはサンプルファイルです。\n例外処理の練習に使用します。");
        
        // 存在するファイルの読み込み
        System.out.println("=== 存在するファイルの読み込み ===");
        demo.readFile("sample.txt");
        
        // 存在しないファイルの読み込み
        System.out.println("\n=== 存在しないファイルの読み込み ===");
        demo.readFile("nonexistent.txt");
    }
}
```

### 演習2: チェック例外と非チェック例外の使い分けの解答例

```java
package exception.advanced;

import java.util.*;

/**
 * チェック例外と非チェック例外の使い分けを練習するクラス
 */
public class UserRegistrationSystem {

    // ユーザーデータベースの代わりとなるマップ
    private Map<String, User> userDatabase = new HashMap<>();
    
    // データベース接続状態を模擬するフラグ
    private boolean databaseConnected = true;

    /**
     * ユーザーが既に存在する場合にスローされるチェック例外
     */
    public static class UserAlreadyExistsException extends Exception {
        public UserAlreadyExistsException(String message) {
            super(message);
        }
        
        public UserAlreadyExistsException(String message, Throwable cause) {
            super(message, cause);
        }
    }

    /**
     * データベースエラーが発生した場合にスローされるチェック例外
     */
    public static class DatabaseException extends Exception {
        public DatabaseException(String message) {
            super(message);
        }
        
        public DatabaseException(String message, Throwable cause) {
            super(message, cause);
        }
    }

    /**
     * ユーザークラス
     */
    public static class User {
        private String username;
        private String email;
        
        public User(String username, String email) {
            this.username = username;
            this.email = email;
        }
        
        public String getUsername() { return username; }
        public String getEmail() { return email; }
        
        @Override
        public String toString() {
            return "User{username='" + username + "', email='" + email + "'}";
        }
    }

    /**
     * ユーザーを登録します。
     * 
     * @param username ユーザー名
     * @param email メールアドレス
     * @throws UserAlreadyExistsException ユーザー名が既に使用されている場合
     * @throws DatabaseException データベース接続エラーが発生した場合
     * @throws IllegalArgumentException ユーザー名またはメールアドレスが無効な場合
     */
    public void registerUser(String username, String email) throws UserAlreadyExistsException, DatabaseException {
        // ユーザー名とメールアドレスの検証
        if (username == null || username.isEmpty()) {
            throw new IllegalArgumentException("ユーザー名は必須です");
        }
        if (email == null || email.isEmpty()) {
            throw new IllegalArgumentException("メールアドレスは必須です");
        }
        
        // データベース接続状態のチェック
        if (!databaseConnected) {
            throw new DatabaseException("データベースに接続できません");
        }
        
        // ユーザー名の重複チェック
        if (userDatabase.containsKey(username)) {
            throw new UserAlreadyExistsException("ユーザー名 '" + username + "' は既に使用されています");
        }
        
        // ユーザーの登録
        userDatabase.put(username, new User(username, email));
        
        System.out.println("ユーザーが正常に登録されました: " + username);
    }

    /**
     * 登録されているすべてのユーザーを表示します。
     */
    public void listAllUsers() {
        System.out.println("=== 登録ユーザー一覧 ===");
        if (userDatabase.isEmpty()) {
            System.out.println("登録ユーザーはいません。");
        } else {
            for (User user : userDatabase.values()) {
                System.out.println(user);
            }
        }
    }

    /**
     * データベース接続状態を設定します（テスト用）。
     */
    public void setDatabaseConnected(boolean connected) {
        this.databaseConnected = connected;
    }

    public static void main(String[] args) {
        UserRegistrationSystem system = new UserRegistrationSystem();
        
        // 正常なケース
        try {
            system.registerUser("alice", "alice@example.com");
        } catch (UserAlreadyExistsException e) {
            System.err.println("エラー: " + e.getMessage());
        } catch (DatabaseException e) {
            System.err.println("データベースエラー: " + e.getMessage());
        } catch (IllegalArgumentException e) {
            System.err.println("入力エラー: " + e.getMessage());
        }
        
        // 重複ユーザー名のケース
        try {
            system.registerUser("alice", "alice2@example.com");
        } catch (UserAlreadyExistsException e) {
            System.err.println("エラー: " + e.getMessage());
        } catch (DatabaseException e) {
            System.err.println("データベースエラー: " + e.getMessage());
        } catch (IllegalArgumentException e) {
            System.err.println("入力エラー: " + e.getMessage());
        }
        
        // 無効な入力のケース
        try {
            system.registerUser("", "bob@example.com");
        } catch (UserAlreadyExistsException e) {
            System.err.println("エラー: " + e.getMessage());
        } catch (DatabaseException e) {
            System.err.println("データベースエラー: " + e.getMessage());
        } catch (IllegalArgumentException e) {
            System.err.println("入力エラー: " + e.getMessage());
        }
        
        // データベース接続エラーのケース
        system.setDatabaseConnected(false);
        try {
            system.registerUser("bob", "bob@example.com");
        } catch (UserAlreadyExistsException e) {
            System.err.println("エラー: " + e.getMessage());
        } catch (DatabaseException e) {
            System.err.println("データベースエラー: " + e.getMessage());
        } catch (IllegalArgumentException e) {
            System.err.println("入力エラー: " + e.getMessage());
        }
        
        // 登録ユーザー一覧の表示
        system.listAllUsers();
    }
}
```

### 演習3: 例外の変換とラッピングの解答例

```java
package exception.advanced;

import java.io.*;
import java.util.*;

/**
 * 例外の変換とラッピングを練習するクラス
 */
public class FileOperationUtil {

    /**
     * ファイル操作に関する例外
     */
    public static class FileOperationException extends Exception {
        public FileOperationException(String message) {
            super(message);
        }
        
        public FileOperationException(String message, Throwable cause) {
            super(message, cause);
        }
    }

    /**
     * ファイルの内容が無効な場合の例外
     */
    public static class InvalidContentException extends Exception {
        public InvalidContentException(String message) {
            super(message);
        }
        
        public InvalidContentException(String message, Throwable cause) {
            super(message, cause);
        }
    }

    /**
     * ファイルからユーザーリストを読み込みます。
     * 各行は「ユーザー名,メールアドレス」の形式である必要があります。
     * 
     * @param filename 読み込むファイルのパス
     * @return ユーザーのリスト
     * @throws FileOperationException ファイル操作に失敗した場合
     * @throws InvalidContentException ファイルの内容が無効な場合
     */
    public List<UserRegistrationSystem.User> loadUsersFromFile(String filename) throws FileOperationException, InvalidContentException {
        List<UserRegistrationSystem.User> users = new ArrayList<>();
        
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line;
            int lineNumber = 0;
            
            while ((line = reader.readLine()) != null) {
                lineNumber++;
                
                // 空行はスキップ
                if (line.trim().isEmpty()) {
                    continue;
                }
                
                // 行を解析
                String[] parts = line.split(",");
                if (parts.length != 2) {
                    throw new InvalidContentException(
                            "無効な形式の行です（行 " + lineNumber + "）: " + line + 
                            "。期待される形式: 'ユーザー名,メールアドレス'");
                }
                
                String username = parts[0].trim();
                String email = parts[1].trim();
                
                // ユーザーをリストに追加
                users.add(new UserRegistrationSystem.User(username, email));
            }
        } catch (FileNotFoundException e) {
            throw new FileOperationException("ファイルが見つかりません: " + filename, e);
        } catch (IOException e) {
            throw new FileOperationException("ファイルの読み込み中にエラーが発生しました: " + filename, e);
        }
        
        return users;
    }

    /**
     * ユーザーリストをファイルに保存します。
     * 各ユーザーは「ユーザー名,メールアドレス」の形式で保存されます。
     * 
     * @param users 保存するユーザーのリスト
     * @param filename 保存先ファイルのパス
     * @throws FileOperationException ファイル操作に失敗した場合
     */
    public void saveUsersToFile(List<UserRegistrationSystem.User> users, String filename) throws FileOperationException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            for (UserRegistrationSystem.User user : users) {
                writer.write(user.getUsername() + "," + user.getEmail());
                writer.newLine();
            }
        } catch (IOException e) {
            throw new FileOperationException("ファイルの書き込み中にエラーが発生しました: " + filename, e);
        }
    }

    /**
     * テスト用のユーザーデータを作成します。
     */
    private List<UserRegistrationSystem.User> createSampleUsers() {
        List<UserRegistrationSystem.User> users = new ArrayList<>();
        users.add(new UserRegistrationSystem.User("alice", "alice@example.com"));
        users.add(new UserRegistrationSystem.User("bob", "bob@example.com"));
        users.add(new UserRegistrationSystem.User("charlie", "charlie@example.com"));
        return users;
    }

    /**
     * テスト用の無効なユーザーデータファイルを作成します。
     */
    private void createInvalidUserFile(String filename) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            writer.write("alice,alice@example.com\n");
            writer.write("bob,bob@example.com\n");
            writer.write("invalid_line\n"); // 無効な形式の行
            writer.write("charlie,charlie@example.com\n");
        }
    }

    public static void main(String[] args) {
        FileOperationUtil util = new FileOperationUtil();
        
        // ユーザーリストの保存
        try {
            List<UserRegistrationSystem.User> users = util.createSampleUsers();
            util.saveUsersToFile(users, "users.txt");
            System.out.println("ユーザーリストを保存しました。");
        } catch (FileOperationException e) {
            System.err.println("ファイル操作エラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        }
        
        // ユーザーリストの読み込み
        try {
            List<UserRegistrationSystem.User> users = util.loadUsersFromFile("users.txt");
            System.out.println("読み込んだユーザーリスト:");
            for (UserRegistrationSystem.User user : users) {
                System.out.println(user);
            }
        } catch (FileOperationException e) {
            System.err.println("ファイル操作エラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        } catch (InvalidContentException e) {
            System.err.println("無効な内容: " + e.getMessage());
        }
        
        // 存在しないファイルの読み込み
        try {
            util.loadUsersFromFile("nonexistent.txt");
        } catch (FileOperationException e) {
            System.err.println("ファイル操作エラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        } catch (InvalidContentException e) {
            System.err.println("無効な内容: " + e.getMessage());
        }
        
        // 無効な内容のファイルの読み込み
        try {
            util.createInvalidUserFile("invalid_users.txt");
            util.loadUsersFromFile("invalid_users.txt");
        } catch (FileOperationException e) {
            System.err.println("ファイル操作エラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        } catch (InvalidContentException e) {
            System.err.println("無効な内容: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("IOエラー: " + e.getMessage());
        }
    }
}
```

## 解説と補足

### 例外処理の目的

例外処理の主な目的は以下の通りです：

1. **エラー処理の一元化**：
   - エラー処理コードを通常のコードから分離し、可読性を向上させる
   - エラー処理を忘れることを防ぐ（特にチェック例外の場合）

2. **プログラムの堅牢性向上**：
   - 予期しない状況でもプログラムがクラッシュせず、適切に対応できるようにする
   - リソースの適切な解放を保証する

3. **エラー情報の伝達**：
   - エラーの原因と状況を詳細に伝える
   - スタックトレースにより、エラーの発生場所を特定できる

### チェック例外と非チェック例外の使い分け

チェック例外と非チェック例外の使い分けについては、様々な意見がありますが、一般的には以下のような指針が役立ちます：

#### チェック例外を使用すべき場合
- 呼び出し元が回復可能なエラー
- 呼び出し元に対応を強制したいエラー
- ビジネスロジックの一部として扱うべきエラー
- 例：ファイルが見つからない、ネットワーク接続の問題、ビジネスルール違反

#### 非チェック例外を使用すべき場合
- プログラミングエラー（バグ）
- 回復が困難または不可能なエラー
- 通常の操作では発生しないはずのエラー
- 例：NullPointerException、IndexOutOfBoundsException、IllegalArgumentException

### 例外処理のアンチパターン

例外処理において避けるべき一般的なアンチパターンは以下の通りです：

1. **空のcatchブロック**：
   ```java
   try {
       // 何らかの処理
   } catch (Exception e) {
       // 何もしない（エラーを無視）
   }
   ```
   問題点：エラーが発生しても気づかず、デバッグが困難になる

2. **例外の丸呑み**：
   ```java
   try {
       // 何らかの処理
   } catch (Exception e) {
       System.err.println("エラーが発生しました");
       // 元の例外情報が失われる
   }
   ```
   問題点：元の例外情報が失われ、根本原因の特定が困難になる

3. **例外を使った制御フロー**：
   ```java
   try {
       // 通常の処理の一部として例外を使用
       throw new Exception("次の処理へ");
   } catch (Exception e) {
       // 別の処理へ分岐
   }
   ```
   問題点：可読性が低下し、パフォーマンスにも悪影響を与える

4. **過度に細かい例外キャッチ**：
   ```java
   try {
       // 何らかの処理
   } catch (FileNotFoundException e) {
       // 処理1
   } catch (EOFException e) {
       // 処理2
   } catch (IOException e) {
       // 処理3
   }
   ```
   問題点：コードが冗長になり、メンテナンス性が低下する（マルチキャッチや例外の階層を活用すべき）

### try-with-resourcesの利点

Java 7で導入された`try-with-resources`構文には、以下の利点があります：

1. **リソースの自動クローズ**：
   - `finally`ブロックでのリソースクローズを忘れることがない
   - 複数のリソースを扱う場合でも簡潔に記述できる

2. **例外の抑制**：
   - リソースのクローズ時に例外が発生した場合、元の例外が優先され、クローズ時の例外は抑制される（Suppressed Exception）
   - 抑制された例外は`getSuppressed()`メソッドで取得できる

3. **コードの簡潔さ**：
   - `finally`ブロックが不要になり、コードが簡潔になる
   - ネストされた`try-catch`ブロックが減少する

### 例外処理とパフォーマンス

例外処理はパフォーマンスに影響を与える可能性があります。以下の点に注意してください：

1. **例外のコスト**：
   - 例外オブジェクトの生成
   - スタックトレースの収集
   - 例外ハンドラの検索

2. **正常系と異常系**：
   - 正常系（例外が発生しない場合）では、例外処理のオーバーヘッドはほとんどない
   - 異常系（例外が発生する場合）では、例外処理のコストが高くなる

3. **パフォーマンス最適化**：
   - 頻繁に発生する状況では、例外ではなく条件分岐を使用する
   - 例外は本当に例外的な状況のために使用する
   - スタックトレースが不要な場合は、`fillInStackTrace()`をオーバーライドして無効化することも検討する

### 例外処理のベストプラクティス（詳細）

1. **具体的な例外をキャッチする**：
   - `Exception`ではなく、より具体的な例外クラスをキャッチする
   - 例外の種類に応じて適切に処理する

2. **例外の連鎖を維持する**：
   - 例外を変換する場合は、元の例外を原因（cause）として保持する
   - 例：`throw new BusinessException("処理に失敗しました", originalException);`

3. **意味のあるメッセージを提供する**：
   - 例外メッセージには、エラーの原因と対処方法を含める
   - コンテキスト情報（パラメータ値など）を含める

4. **ログ出力を適切に行う**：
   - 例外をキャッチした場合は、適切なログレベルでログ出力する
   - スタックトレースも含めてログ出力する

5. **リソースを適切に解放する**：
   - `try-with-resources`を使用する
   - `finally`ブロックでリソースを確実に解放する

6. **例外を適切に文書化する**：
   - メソッドがスローする可能性のある例外を`@throws`タグで文書化する
   - 例外がスローされる条件を明記する

## 実務での活用シーン

### 1. 外部リソースアクセス

ファイル、データベース、ネットワークなどの外部リソースにアクセスする際には、様々な例外が発生する可能性があります。適切な例外処理を行うことで、エラーが発生した場合でも適切に対応できます。

```java
public void saveData(Data data) {
    try (Connection conn = dataSource.getConnection();
         PreparedStatement stmt = conn.prepareStatement("INSERT INTO data VALUES (?, ?)")) {
        stmt.setString(1, data.getId());
        stmt.setString(2, data.getValue());
        stmt.executeUpdate();
    } catch (SQLException e) {
        throw new DataAccessException("データの保存に失敗しました: " + data.getId(), e);
    }
}
```

### 2. ビジネスロジックのバリデーション

ビジネスルールに違反する入力や状態を検出し、適切な例外をスローすることで、エラーを早期に検出できます。

```java
public void transferMoney(Account from, Account to, BigDecimal amount) throws InsufficientFundsException {
    if (amount.compareTo(BigDecimal.ZERO) <= 0) {
        throw new IllegalArgumentException("振込金額は0より大きい必要があります");
    }
    
    if (from.getBalance().compareTo(amount) < 0) {
        throw new InsufficientFundsException("残高不足です: 必要金額 " + amount + ", 残高 " + from.getBalance());
    }
    
    from.withdraw(amount);
    to.deposit(amount);
}
```

### 3. 階層化されたエラー処理

多層アーキテクチャ（例：コントローラ、サービス、リポジトリ）では、各層で適切な例外処理を行い、上位層に適切な例外を伝播させることが重要です。

```java
// リポジトリ層
public class UserRepository {
    public User findById(String id) throws DataAccessException {
        try {
            // データベースアクセス
        } catch (SQLException e) {
            throw new DataAccessException("ユーザーの取得に失敗しました: " + id, e);
        }
    }
}

// サービス層
public class UserService {
    public User getUser(String id) throws BusinessException {
        try {
            return userRepository.findById(id);
        } catch (DataAccessException e) {
            throw new BusinessException("ユーザー情報の取得に失敗しました", e);
        }
    }
}

// コントローラ層
public class UserController {
    public Response getUserInfo(String id) {
        try {
            User user = userService.getUser(id);
            return Response.ok(user).build();
        } catch (BusinessException e) {
            logger.error("ユーザー情報の取得エラー", e);
            return Response.status(Response.Status.INTERNAL_SERVER_ERROR)
                    .entity("エラーが発生しました: " + e.getMessage())
                    .build();
        }
    }
}
```

### 4. リトライメカニズム

一時的なエラー（ネットワーク接続の問題など）が発生した場合に、自動的にリトライするメカニズムを実装できます。

```java
public Response callExternalService() {
    int maxRetries = 3;
    int retryCount = 0;
    
    while (retryCount < maxRetries) {
        try {
            return externalService.call();
        } catch (IOException e) {
            retryCount++;
            if (retryCount >= maxRetries) {
                throw new ServiceUnavailableException("外部サービスの呼び出しに失敗しました", e);
            }
            logger.warn("外部サービスの呼び出しに失敗しました。リトライします（{}/{}）", retryCount, maxRetries);
            try {
                Thread.sleep(1000 * retryCount); // バックオフ
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt();
                throw new ServiceUnavailableException("リトライ中に中断されました", ie);
            }
        }
    }
    
    // ここには到達しないはず
    throw new IllegalStateException("予期しない状態");
}
```

### 5. グレースフルデグラデーション

一部の機能がエラーになった場合でも、システム全体が停止しないように、グレースフルデグラデーション（優雅な機能低下）を実装できます。

```java
public Dashboard getDashboard(User user) {
    Dashboard dashboard = new Dashboard();
    
    try {
        dashboard.setUserProfile(userService.getUserProfile(user.getId()));
    } catch (Exception e) {
        logger.error("ユーザープロファイルの取得に失敗しました", e);
        dashboard.setUserProfileAvailable(false);
    }
    
    try {
        dashboard.setRecentActivities(activityService.getRecentActivities(user.getId()));
    } catch (Exception e) {
        logger.error("最近のアクティビティの取得に失敗しました", e);
        dashboard.setRecentActivitiesAvailable(false);
    }
    
    try {
        dashboard.setNotifications(notificationService.getUnreadNotifications(user.getId()));
    } catch (Exception e) {
        logger.error("通知の取得に失敗しました", e);
        dashboard.setNotificationsAvailable(false);
    }
    
    return dashboard;
}
```

## 参考資料

1. Java公式ドキュメント: [Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/index.html)
2. Java公式ドキュメント: [The try-with-resources Statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)
3. 書籍: 「Effective Java 第3版」（Joshua Bloch著） - 特に項目69-77（例外）
4. 書籍: 「Clean Code」（Robert C. Martin著） - 第7章（エラー処理）
5. Oracle技術記事: [Java Exception Handling](https://www.oracle.com/technical-resources/articles/java/exceptions.html)
6. Baeldung: [Java Exception Handling Best Practices](https://www.baeldung.com/java-exception-handling-best-practices)
7. Baeldung: [Performance Effects of Exceptions in Java](https://www.baeldung.com/java-exceptions-performance)
