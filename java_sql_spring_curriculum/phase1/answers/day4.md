---
layout: default
title: "Day 4: 例外処理の設計と実装 - 解答例"
---
# Day 4: 例外処理の設計と実装 - 解答例

## 演習1: 基本的な例外処理

### 解答例
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
        BufferedReader reader = null;
        String firstLine = "";
        
        try {
            reader = new BufferedReader(new FileReader(fileName));
            firstLine = reader.readLine();
            if (firstLine == null) {
                firstLine = "";
            }
        } catch (IOException e) {
            System.err.println("ファイル読み込み中にエラーが発生しました: " + e.getMessage());
            throw e; // 例外を再スロー
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                    System.out.println("リソースを正常にクローズしました");
                } catch (IOException e) {
                    System.err.println("リソースのクローズ中にエラーが発生しました: " + e.getMessage());
                }
            }
        }
        
        return firstLine;
    }
    
    /**
     * try-with-resources構文を使用してファイルを読み込みます。
     * 
     * @param fileName 読み込むファイルのパス
     * @return ファイルの最初の行、ファイルが空の場合は空文字列
     * @throws IOException ファイル読み込みエラーが発生した場合
     */
    public static String readFirstLineWithTryWithResources(String fileName) throws IOException {
        String firstLine = "";
        
        try (BufferedReader reader = new BufferedReader(new FileReader(fileName))) {
            firstLine = reader.readLine();
            if (firstLine == null) {
                firstLine = "";
            }
        } catch (IOException e) {
            System.err.println("ファイル読み込み中にエラーが発生しました: " + e.getMessage());
            throw e; // 例外を再スロー
        }
        
        return firstLine;
    }
    
    /**
     * ファイルの内容を読み込み、各行を表示します。
     * 
     * @param fileName 読み込むファイルのパス
     */
    public static void readAndDisplayFile(String fileName) {
        try (BufferedReader reader = new BufferedReader(new FileReader(fileName))) {
            String line;
            int lineNumber = 1;
            
            System.out.println("ファイルの内容を表示します: " + fileName);
            while ((line = reader.readLine()) != null) {
                System.out.println(lineNumber + ": " + line);
                lineNumber++;
            }
            System.out.println("ファイルの読み込みが完了しました");
        } catch (IOException e) {
            if (e instanceof java.io.FileNotFoundException) {
                System.err.println("ファイルが見つかりません: " + fileName);
            } else {
                System.err.println("ファイル読み込み中にエラーが発生しました: " + e.getMessage());
            }
        }
    }
}
```

### 解説
この解答例では、ファイル操作における基本的な例外処理を実装しています。

1. **従来のtry-catch-finally構文**
   - `readFirstLineWithTraditional`メソッドでは、従来のtry-catch-finally構文を使用しています
   - finallyブロックでリソースを確実にクローズし、クローズ時のエラーも適切に処理しています
   - 発生した例外は上位層に再スローしています

2. **try-with-resources構文**
   - `readFirstLineWithTryWithResources`メソッドでは、Java 7以降で導入されたtry-with-resources構文を使用しています
   - リソースは自動的にクローズされるため、明示的なクローズ処理が不要です
   - コードがより簡潔で読みやすくなっています

3. **例外の種類に応じた処理**
   - `readAndDisplayFile`メソッドでは、`FileNotFoundException`と他のIOExceptionを区別して処理しています
   - ユーザーに対して適切なエラーメッセージを表示しています

この実装により、ファイル操作における例外を適切に処理し、リソースの確実なクローズを保証しています。

## 演習2: チェック例外と非チェック例外の使い分け

### 解答例
```java
package exceptions.advanced;

import java.util.HashMap;
import java.util.Map;

/**
 * データベース関連の例外を表すチェック例外
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
 * ユーザーが見つからない場合のチェック例外
 */
class UserNotFoundException extends Exception {
    public UserNotFoundException(String message) {
        super(message);
    }
    
    public UserNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}

/**
 * ユーザーデータが不正な場合の非チェック例外
 */
class InvalidUserDataException extends RuntimeException {
    public InvalidUserDataException(String message) {
        super(message);
    }
    
    public InvalidUserDataException(String message, Throwable cause) {
        super(message, cause);
    }
}

/**
 * ユーザークラス
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
    
    public String getId() {
        return id;
    }
    
    public String getName() {
        return name;
    }
    
    public String getEmail() {
        return email;
    }
    
    public int getAge() {
        return age;
    }
    
    @Override
    public String toString() {
        return "User{id='" + id + "', name='" + name + "', email='" + email + "', age=" + age + "}";
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
        // データベース接続チェック
        if (!databaseConnected) {
            throw new DatabaseException("データベースに接続できません");
        }
        
        // ユーザーデータのバリデーション
        validateUserData(user);
        
        // ユーザーの登録
        users.put(user.getId(), user);
        System.out.println("ユーザーを登録しました: " + user);
    }
    
    /**
     * ユーザーデータを検証します。
     * 
     * @param user 検証するユーザー
     * @throws InvalidUserDataException ユーザーデータが不正な場合
     */
    private void validateUserData(User user) {
        if (user == null) {
            throw new InvalidUserDataException("ユーザーデータがnullです");
        }
        
        if (user.getId() == null || user.getId().trim().isEmpty()) {
            throw new InvalidUserDataException("ユーザーIDは必須です");
        }
        
        if (user.getName() == null || user.getName().trim().isEmpty()) {
            throw new InvalidUserDataException("ユーザー名は必須です");
        }
        
        if (user.getEmail() == null || user.getEmail().trim().isEmpty()) {
            throw new InvalidUserDataException("メールアドレスは必須です");
        }
        
        if (user.getAge() < 0) {
            throw new InvalidUserDataException("年齢は0以上である必要があります");
        }
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
        // データベース接続チェック
        if (!databaseConnected) {
            throw new DatabaseException("データベースに接続できません");
        }
        
        // ユーザーの検索
        User user = users.get(userId);
        if (user == null) {
            throw new UserNotFoundException("ユーザーID '" + userId + "' のユーザーが見つかりません");
        }
        
        return user;
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
        // ユーザー登録システムの作成
        UserRegistrationSystem system = new UserRegistrationSystem();
        
        // ユーザー登録のテスト
        System.out.println("===== ユーザー登録のテスト =====");
        
        // 正常なユーザーの登録
        try {
            User user1 = new User("user1", "山田太郎", "yamada@example.com", 30);
            system.registerUser(user1);
            System.out.println("ユーザーが正常に登録されました: " + user1);
        } catch (InvalidUserDataException e) {
            System.out.println("不正なユーザーデータ: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
        
        // 不正なユーザーの登録（nullユーザー）
        try {
            system.registerUser(null);
            System.out.println("ユーザーが正常に登録されました");
        } catch (InvalidUserDataException e) {
            System.out.println("不正なユーザーデータ: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
        
        // 不正なユーザーの登録（IDなし）
        try {
            User user2 = new User("", "鈴木花子", "suzuki@example.com", 25);
            system.registerUser(user2);
            System.out.println("ユーザーが正常に登録されました: " + user2);
        } catch (InvalidUserDataException e) {
            System.out.println("不正なユーザーデータ: " + e.getMessage());
        } catch (DatabaseException e) {
            System.out.println("データベースエラー: " + e.getMessage());
        }
        
        // 不正なユーザーの登録（負の年齢）
        try {
            User user3 = new User("user3", "佐藤次郎", "sato@example.com", -5);
            system.registerUser(user3);
            System.out.println("ユーザーが正常に登録されました: " + user3);
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

### 解説
この解答例では、チェック例外と非チェック例外の適切な使い分けを実装しています。

1. **チェック例外（UserNotFoundException, DatabaseException）**
   - `UserNotFoundException`はユーザーが見つからない場合に使用するチェック例外です
   - `DatabaseException`はデータベース接続エラーを表すチェック例外です
   - 呼び出し元に明示的な処理を強制するため、チェック例外として設計しています
   - 回復可能なエラー状態を表現しています

2. **非チェック例外（InvalidUserDataException）**
   - ユーザーデータが不正な場合に使用する非チェック例外です
   - プログラミングエラーを表現するため、非チェック例外として設計しています
   - 呼び出し元は通常このエラーを回避するべきです

3. **例外の連鎖**
   - 各例外クラスには、原因となる例外を保持するコンストラクタを実装しています
   - これにより、デバッグ時に役立つ情報を提供しています

4. **例外処理の責任分担**
   - データ検証は`validateUserData`メソッドに集約されています
   - 各メソッドは自身の責任範囲の例外のみをスローしています

この実装により、異なる種類のエラー状況に対して適切な例外型を使い分け、呼び出し元に必要な情報を提供しています。

## 演習3: 例外の変換とラッピング

### 解答例
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
    public FileOperationException(String message) {
        super(message);
    }
    
    public FileOperationException(String message, Throwable cause) {
        super(message, cause);
    }
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
        List<String> lines = new ArrayList<>();
        
        try (BufferedReader reader = new BufferedReader(new FileReader(fileName))) {
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
            return lines;
        } catch (IOException e) {
            // IOExceptionをFileOperationExceptionに変換
            if (e instanceof java.io.FileNotFoundException) {
                throw new FileOperationException("ファイルが見つかりません: " + fileName, e);
            } else {
                throw new FileOperationException("ファイル読み込み中にエラーが発生しました: " + fileName, e);
            }
        }
    }
    
    /**
     * 文字列のリストをファイルに書き込みます。
     * 
     * @param lines 書き込む文字列のリスト
     * @param fileName 書き込むファイルのパス
     * @throws FileOperationException ファイル操作エラーが発生した場合
     */
    public static void writeLines(List<String> lines, String fileName) throws FileOperationException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(fileName))) {
            for (String line : lines) {
                writer.write(line);
                writer.newLine();
            }
        } catch (IOException e) {
            // IOExceptionをFileOperationExceptionに変換
            throw new FileOperationException("ファイル書き込み中にエラーが発生しました: " + fileName, e);
        }
    }
    
    /**
     * ファイルをコピーします。
     * 
     * @param sourceFileName コピー元ファイルのパス
     * @param targetFileName コピー先ファイルのパス
     * @throws FileOperationException ファイル操作エラーが発生した場合
     */
    public static void copyFile(String sourceFileName, String targetFileName) throws FileOperationException {
        try {
            // ファイルの内容を読み込み
            List<String> lines = readLines(sourceFileName);
            
            // ファイルに内容を書き込み
            writeLines(lines, targetFileName);
        } catch (FileOperationException e) {
            // FileOperationExceptionを適切なメッセージを追加して再スロー
            throw new FileOperationException("ファイルコピー中にエラーが発生しました: " + sourceFileName + " -> " + targetFileName, e);
        }
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

### 解説
この解答例では、例外の変換とラッピングを実装しています。

1. **例外クラスの設計**
   - `FileOperationException`クラスを定義し、ファイル操作に関連する例外を表現しています
   - メッセージのみを受け取るコンストラクタと、メッセージと原因例外を受け取るコンストラクタを実装しています

2. **例外の変換**
   - 低レベルの例外（`IOException`）を、より意味のある高レベルの例外（`FileOperationException`）に変換しています
   - 変換時に元の例外を原因（cause）として保持することで、デバッグ情報を失わないようにしています
   - `FileNotFoundException`と他の`IOException`を区別して、より具体的なエラーメッセージを提供しています

3. **例外メッセージの強化**
   - 変換後の例外には、より具体的なエラーメッセージを設定しています
   - ファイルパスなどのコンテキスト情報を含めることで、問題の特定を容易にしています

4. **例外の再スロー**
   - `copyFile`メソッドでは、キャッチした例外を適切に処理した後、より具体的なメッセージを追加して再スローしています
   - これにより、呼び出し元に対してより詳細なエラー情報を提供しています

この実装により、低レベルの例外を適切に変換・ラッピングし、呼び出し元に対してより意味のある例外情報を提供しています。また、例外の連鎖を使用することで、元の例外情報も保持しています。

## 発展演習1: 例外フィルタリング（難易度：発展）

### 解答例
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
            // 例外の型を表示
            System.out.println("キャッチした例外: " + e.getClass().getSimpleName());
            System.out.println("例外メッセージ: " + e.getMessage());
            
            // exceptionTypeに基づいて、特定の例外だけを処理
            if (exceptionType.equals("io") && e instanceof IOException) {
                System.out.println("IOExceptionを処理しました");
            } else if (exceptionType.equals("sql") && e instanceof SQLException) {
                System.out.println("SQLExceptionを処理しました");
            } else if (exceptionType.equals("all")) {
                System.out.println("すべての例外を処理しました");
            } else {
                // 処理しない例外は再スロー
                System.out.println("この例外は処理対象外のため、再スローします");
                throw e;
            }
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
        } catch (FileNotFoundException | SQLException e) {
            // FileNotFoundExceptionとSQLExceptionを1つのcatchブロックで処理
            System.out.println("マルチキャッチで捕捉した例外: " + e.getClass().getSimpleName());
            System.out.println("例外メッセージ: " + e.getMessage());
            System.out.println("FileNotFoundExceptionまたはSQLExceptionを処理しました");
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
            System.out.println();
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
            System.out.println();
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
            System.out.println();
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
            System.out.println();
        }
    }
}
```

### 解説
この解答例では、例外フィルタリングとマルチキャッチ構文を実装しています。

1. **例外フィルタリング**
   - `processExceptionsWithFiltering`メソッドでは、`exceptionType`パラメータに基づいて、特定の例外だけを処理しています
   - `"io"`の場合は`IOException`だけを処理し、`"sql"`の場合は`SQLException`だけを処理し、`"all"`の場合はすべての例外を処理しています
   - 処理対象外の例外は再スローしています

2. **マルチキャッチ構文**
   - `processExceptionsWithMultiCatch`メソッドでは、Java 7で導入されたマルチキャッチ構文を使用しています
   - `FileNotFoundException | SQLException`のように、複数の例外タイプを1つの`catch`ブロックで処理しています
   - これにより、同じ処理を行う例外を効率的にキャッチできます

3. **ランダム例外生成**
   - `throwRandomException`メソッドでは、4種類の例外（`FileNotFoundException`、`IOException`、`SQLException`、`IllegalArgumentException`）からランダムに1つを選択してスローしています
   - これにより、様々な例外パターンをテストできます

4. **テストコード**
   - `main`メソッドでは、様々な条件で例外フィルタリングとマルチキャッチをテストしています
   - 各テストケースで複数回実行することで、ランダム性を確保しています

この実装により、特定の条件に一致する例外だけを処理し、それ以外の例外は上位の呼び出し元に伝播させる方法を示しています。また、マルチキャッチ構文を使用して、複数の例外タイプを効率的に処理する方法も示しています。

## 発展演習2: 例外処理のパフォーマンス測定（難易度：発展）

### 解答例
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
     * @throws IllegalArgumentException パスがnullまたは空の場合
     */
    public static int calculateDepthWithExceptions(String path) {
        if (path == null || path.isEmpty()) {
            throw new IllegalArgumentException("パスがnullまたは空です");
        }
        
        // スラッシュの数をカウント
        int depth = 0;
        for (int i = 0; i < path.length(); i++) {
            if (path.charAt(i) == '/' || path.charAt(i) == '\\') {
                depth++;
            }
        }
        
        return depth;
    }
    
    /**
     * 戻り値を使用してエラーを表現する方法でディレクトリの深さを計算します。
     * 
     * @param path パス
     * @return ディレクトリの深さ（エラーの場合は-1）
     */
    public static int calculateDepthWithReturnValue(String path) {
        if (path == null || path.isEmpty()) {
            return -1;  // エラーを表す戻り値
        }
        
        // スラッシュの数をカウント
        int depth = 0;
        for (int i = 0; i < path.length(); i++) {
            if (path.charAt(i) == '/' || path.charAt(i) == '\\') {
                depth++;
            }
        }
        
        return depth;
    }
    
    /**
     * 例外をスローして処理する方法のパフォーマンスを測定します。
     * 
     * @param errorRate エラー率（0.0～1.0）
     * @return 実行時間（ミリ秒）
     */
    public static long measureExceptionPerformance(double errorRate) {
        // 開始時間を記録
        long startTime = System.currentTimeMillis();
        
        // ITERATIONS回繰り返し
        for (int i = 0; i < ITERATIONS; i++) {
            try {
                // errorRateの確率でnullを渡し、それ以外は有効なパスを渡す
                String path = (Math.random() < errorRate) ? null : "/path/to/some/directory";
                int depth = calculateDepthWithExceptions(path);
                
                // 結果を使用（最適化防止）
                if (depth < 0) {
                    System.out.println("Negative depth: " + depth);
                }
            } catch (IllegalArgumentException e) {
                // 例外をキャッチして処理
                // 実際の処理は行わない（パフォーマンス測定のため）
            }
        }
        
        // 終了時間を記録し、実行時間を計算
        long endTime = System.currentTimeMillis();
        return endTime - startTime;
    }
    
    /**
     * 戻り値を使用してエラーを表現する方法のパフォーマンスを測定します。
     * 
     * @param errorRate エラー率（0.0～1.0）
     * @return 実行時間（ミリ秒）
     */
    public static long measureReturnValuePerformance(double errorRate) {
        // 開始時間を記録
        long startTime = System.currentTimeMillis();
        
        // ITERATIONS回繰り返し
        for (int i = 0; i < ITERATIONS; i++) {
            // errorRateの確率でnullを渡し、それ以外は有効なパスを渡す
            String path = (Math.random() < errorRate) ? null : "/path/to/some/directory";
            int depth = calculateDepthWithReturnValue(path);
            
            // 戻り値をチェックしてエラー処理
            if (depth < 0) {
                // エラー処理
                // 実際の処理は行わない（パフォーマンス測定のため）
            } else {
                // 正常処理
                // 結果を使用（最適化防止）
                if (depth < 0) {
                    System.out.println("Negative depth: " + depth);
                }
            }
        }
        
        // 終了時間を記録し、実行時間を計算
        long endTime = System.currentTimeMillis();
        return endTime - startTime;
    }
    
    public static void main(String[] args) {
        // エラー率の配列
        double[] errorRates = {0.0, 0.01, 0.1, 0.5, 1.0};
        
        System.out.println("===== 例外処理のパフォーマンス測定 =====");
        System.out.println("繰り返し回数: " + ITERATIONS);
        System.out.println();
        
        System.out.println("エラー率\t例外処理時間(ms)\t戻り値処理時間(ms)\t比率(例外/戻り値)");
        System.out.println("--------------------------------------------------------------");
        
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
        System.out.println("1. エラー率が低い場合（0.0, 0.01）、例外処理と戻り値を使用した処理の性能差は小さい");
        System.out.println("2. エラー率が高くなるほど（0.1, 0.5, 1.0）、例外処理の性能が戻り値を使用した処理に比べて低下する");
        System.out.println("3. エラー率が1.0（すべての呼び出しでエラー）の場合、例外処理は戻り値を使用した処理に比べて数倍～数十倍遅くなる可能性がある");
        System.out.println("4. 例外処理は、スタックトレースの生成やJVMの最適化の妨げになるため、頻繁に発生するエラー状況では性能に影響を与える");
        System.out.println("5. 例外は「例外的な状況」に使用し、通常の制御フローには使用しないことが推奨される");
    }
}
```

### 解説
この解答例では、例外処理と戻り値を使用したエラー処理のパフォーマンスを比較しています。

1. **ディレクトリの深さ計算**
   - `calculateDepthWithExceptions`メソッドでは、パスがnullまたは空の場合に例外をスローしています
   - `calculateDepthWithReturnValue`メソッドでは、パスがnullまたは空の場合に-1を返しています
   - どちらのメソッドも、パスの深さをスラッシュの数でカウントしています

2. **パフォーマンス測定**
   - `measureExceptionPerformance`メソッドでは、例外をスローして処理する方法のパフォーマンスを測定しています
   - `measureReturnValuePerformance`メソッドでは、戻り値を使用してエラーを表現する方法のパフォーマンスを測定しています
   - どちらのメソッドも、指定されたエラー率でエラーを発生させ、処理時間を測定しています

3. **エラー率の影響**
   - 様々なエラー率（0.0, 0.01, 0.1, 0.5, 1.0）でパフォーマンスを測定し、比較しています
   - エラー率が高くなるほど、例外処理の性能が戻り値を使用した処理に比べて低下することを示しています

4. **考察**
   - エラー率が低い場合、例外処理と戻り値を使用した処理の性能差は小さいことを説明しています
   - エラー率が高くなるほど、例外処理の性能が低下することを説明しています
   - 例外処理は「例外的な状況」に使用し、通常の制御フローには使用しないことを推奨しています

この実装により、例外処理と戻り値を使用したエラー処理のパフォーマンスの違いを示し、適切な使い分けの指針を提供しています。

