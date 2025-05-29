# Day 4: 例外処理の設計と実装 - 解答例

## コア演習1: 基本的な例外処理

### 解答例
```java
package exception.basic;

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;

public class BasicExceptionHandling {

    /**
     * try-catch-finallyを使用したファイル読み込み
     */
    public List<String> readFileWithTryCatchFinally(String filePath) {
        BufferedReader reader = null;
        List<String> lines = new ArrayList<>();
        
        try {
            reader = new BufferedReader(new FileReader(filePath));
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
        } catch (FileNotFoundException e) {
            System.err.println("ファイルが見つかりません: " + filePath);
            System.err.println("エラー詳細: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("ファイル読み込み中にエラーが発生しました: " + filePath);
            System.err.println("エラー詳細: " + e.getMessage());
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                    System.out.println("リソースを正常にクローズしました");
                } catch (IOException e) {
                    System.err.println("リソースのクローズ中にエラーが発生しました");
                    System.err.println("エラー詳細: " + e.getMessage());
                }
            }
        }
        
        return lines;
    }
    
    /**
     * try-with-resourcesを使用したファイル読み込み
     */
    public List<String> readFileWithTryWithResources(String filePath) {
        List<String> lines = new ArrayList<>();
        
        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
            System.out.println("ファイルを正常に読み込みました: " + filePath);
        } catch (FileNotFoundException e) {
            System.err.println("ファイルが見つかりません: " + filePath);
            System.err.println("エラー詳細: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("ファイル読み込み中にエラーが発生しました: " + filePath);
            System.err.println("エラー詳細: " + e.getMessage());
        }
        
        return lines;
    }
    
    /**
     * 複数のリソースを使用するtry-with-resources
     */
    public void copyFile(String sourcePath, String destinationPath) {
        try (
            BufferedReader reader = new BufferedReader(new FileReader(sourcePath));
            java.io.BufferedWriter writer = new java.io.BufferedWriter(new java.io.FileWriter(destinationPath))
        ) {
            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();
            }
            System.out.println("ファイルを正常にコピーしました: " + sourcePath + " -> " + destinationPath);
        } catch (FileNotFoundException e) {
            System.err.println("ファイルが見つかりません: " + sourcePath);
            System.err.println("エラー詳細: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("ファイル操作中にエラーが発生しました");
            System.err.println("エラー詳細: " + e.getMessage());
        }
    }
    
    /**
     * 例外の再スロー
     */
    public List<String> readFileWithRethrow(String filePath) throws IOException {
        List<String> lines = new ArrayList<>();
        
        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
            System.out.println("ファイルを正常に読み込みました: " + filePath);
        } catch (FileNotFoundException e) {
            System.err.println("ファイルが見つかりません: " + filePath);
            // 例外を再スロー
            throw new FileNotFoundException("ファイルが存在しません: " + filePath);
        } catch (IOException e) {
            System.err.println("ファイル読み込み中にエラーが発生しました: " + filePath);
            // 例外を再スロー（追加情報付き）
            IOException newException = new IOException("ファイル読み込みエラー: " + filePath, e);
            throw newException;
        }
        
        return lines;
    }
    
    /**
     * 例外の抑制
     */
    public void demonstrateSuppressedException() {
        Exception mainException = new Exception("メイン例外");
        
        try {
            throw mainException;
        } catch (Exception e) {
            try {
                // クリーンアップ処理中に例外が発生
                throw new RuntimeException("クリーンアップ中の例外");
            } catch (RuntimeException cleanupException) {
                // クリーンアップ例外をメイン例外に抑制例外として追加
                e.addSuppressed(cleanupException);
            }
            
            // 抑制された例外の情報を表示
            System.err.println("メイン例外: " + e.getMessage());
            Throwable[] suppressed = e.getSuppressed();
            for (Throwable t : suppressed) {
                System.err.println("抑制された例外: " + t.getMessage());
            }
        }
    }
    
    /**
     * 例外の連鎖
     */
    public void demonstrateExceptionChaining() {
        try {
            // 低レベルの例外
            try {
                int result = 10 / 0;
            } catch (ArithmeticException e) {
                // 低レベルの例外を原因として、より高レベルの例外をスロー
                throw new IllegalStateException("計算処理中にエラーが発生しました", e);
            }
        } catch (IllegalStateException e) {
            System.err.println("キャッチした例外: " + e.getMessage());
            Throwable cause = e.getCause();
            if (cause != null) {
                System.err.println("原因となった例外: " + cause.getMessage());
                System.err.println("原因となった例外のクラス: " + cause.getClass().getName());
            }
        }
    }
    
    /**
     * マルチキャッチ
     */
    public void demonstrateMultiCatch() {
        try {
            // ランダムに例外をスロー
            int random = (int) (Math.random() * 3);
            if (random == 0) {
                throw new IllegalArgumentException("不正な引数");
            } else if (random == 1) {
                throw new NumberFormatException("数値フォーマットエラー");
            } else {
                throw new ArrayIndexOutOfBoundsException("配列インデックスエラー");
            }
        } catch (IllegalArgumentException | NumberFormatException e) {
            // 複数の例外型を一度にキャッチ
            System.err.println("引数または数値の例外: " + e.getMessage());
        } catch (RuntimeException e) {
            // その他のランタイム例外
            System.err.println("その他のランタイム例外: " + e.getMessage());
        }
    }
    
    /**
     * テスト用のサンプルファイルを作成
     */
    private void createSampleFile(String filePath, List<String> content) {
        try {
            Path path = Paths.get(filePath);
            Files.write(path, content);
            System.out.println("サンプルファイルを作成しました: " + filePath);
        } catch (IOException e) {
            System.err.println("サンプルファイルの作成に失敗しました: " + filePath);
            System.err.println("エラー詳細: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        BasicExceptionHandling demo = new BasicExceptionHandling();
        
        // サンプルファイルの作成
        String sampleFilePath = "sample.txt";
        List<String> sampleContent = List.of(
            "これはサンプルファイルの1行目です。",
            "これはサンプルファイルの2行目です。",
            "これはサンプルファイルの3行目です。"
        );
        demo.createSampleFile(sampleFilePath, sampleContent);
        
        // try-catch-finallyの例
        System.out.println("\n=== try-catch-finallyの例 ===");
        List<String> lines1 = demo.readFileWithTryCatchFinally(sampleFilePath);
        System.out.println("読み込んだ行数: " + lines1.size());
        
        // try-with-resourcesの例
        System.out.println("\n=== try-with-resourcesの例 ===");
        List<String> lines2 = demo.readFileWithTryWithResources(sampleFilePath);
        System.out.println("読み込んだ行数: " + lines2.size());
        
        // 存在しないファイルの読み込み
        System.out.println("\n=== 存在しないファイルの読み込み ===");
        List<String> lines3 = demo.readFileWithTryWithResources("non_existent.txt");
        System.out.println("読み込んだ行数: " + lines3.size());
        
        // ファイルのコピー
        System.out.println("\n=== ファイルのコピー ===");
        String destinationPath = "sample_copy.txt";
        demo.copyFile(sampleFilePath, destinationPath);
        
        // 例外の再スロー
        System.out.println("\n=== 例外の再スロー ===");
        try {
            List<String> lines4 = demo.readFileWithRethrow("non_existent.txt");
        } catch (IOException e) {
            System.err.println("メインメソッドでキャッチした例外: " + e.getMessage());
        }
        
        // 例外の抑制
        System.out.println("\n=== 例外の抑制 ===");
        demo.demonstrateSuppressedException();
        
        // 例外の連鎖
        System.out.println("\n=== 例外の連鎖 ===");
        demo.demonstrateExceptionChaining();
        
        // マルチキャッチ
        System.out.println("\n=== マルチキャッチ ===");
        demo.demonstrateMultiCatch();
    }
}
```

## コア演習2: チェック例外と非チェック例外の使い分け

### 解答例
```java
package exception.design;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Pattern;

public class ExceptionDesignDemo {

    /**
     * ユーザー登録システムのデモ
     */
    public static class UserRegistrationSystem {
        
        // ユーザーリスト（データベースの代わり）
        private List<User> users = new ArrayList<>();
        
        // メールアドレスの正規表現パターン
        private static final Pattern EMAIL_PATTERN = 
            Pattern.compile("^[A-Za-z0-9+_.-]+@(.+)$");
        
        // パスワードの最小長
        private static final int MIN_PASSWORD_LENGTH = 8;
        
        /**
         * ユーザーを登録します。
         * 
         * @param username ユーザー名
         * @param email メールアドレス
         * @param password パスワード
         * @return 登録されたユーザー
         * @throws ValidationException 入力値が不正な場合
         * @throws DuplicateUserException ユーザーが既に存在する場合
         * @throws SystemException システムエラーが発生した場合
         */
        public User registerUser(String username, String email, String password) 
                throws ValidationException, DuplicateUserException, SystemException {
            
            // 入力値の検証（チェック例外）
            validateInput(username, email, password);
            
            // 重複チェック（チェック例外）
            checkDuplicateUser(username, email);
            
            try {
                // ユーザーの作成
                User user = new User(username, email, password);
                
                // データベースへの保存をシミュレート
                simulateDatabaseOperation();
                
                // ユーザーリストに追加
                users.add(user);
                
                // 確認メールの送信をシミュレート
                sendConfirmationEmail(user);
                
                return user;
            } catch (RuntimeException e) {
                // システムエラー（チェック例外）
                throw new SystemException("ユーザー登録中にシステムエラーが発生しました", e);
            }
        }
        
        /**
         * 入力値を検証します。
         * 
         * @param username ユーザー名
         * @param email メールアドレス
         * @param password パスワード
         * @throws ValidationException 入力値が不正な場合
         */
        private void validateInput(String username, String email, String password) throws ValidationException {
            List<String> errors = new ArrayList<>();
            
            // ユーザー名の検証
            if (username == null || username.trim().isEmpty()) {
                errors.add("ユーザー名は必須です");
            }
            
            // メールアドレスの検証
            if (email == null || email.trim().isEmpty()) {
                errors.add("メールアドレスは必須です");
            } else if (!EMAIL_PATTERN.matcher(email).matches()) {
                errors.add("メールアドレスの形式が不正です");
            }
            
            // パスワードの検証
            if (password == null || password.trim().isEmpty()) {
                errors.add("パスワードは必須です");
            } else if (password.length() < MIN_PASSWORD_LENGTH) {
                errors.add("パスワードは" + MIN_PASSWORD_LENGTH + "文字以上である必要があります");
            }
            
            // エラーがある場合は例外をスロー
            if (!errors.isEmpty()) {
                throw new ValidationException("入力値の検証に失敗しました", errors);
            }
        }
        
        /**
         * ユーザーの重複をチェックします。
         * 
         * @param username ユーザー名
         * @param email メールアドレス
         * @throws DuplicateUserException ユーザーが既に存在する場合
         */
        private void checkDuplicateUser(String username, String email) throws DuplicateUserException {
            for (User user : users) {
                if (user.getUsername().equals(username)) {
                    throw new DuplicateUserException("ユーザー名 '" + username + "' は既に使用されています");
                }
                if (user.getEmail().equals(email)) {
                    throw new DuplicateUserException("メールアドレス '" + email + "' は既に使用されています");
                }
            }
        }
        
        /**
         * データベース操作をシミュレートします。
         */
        private void simulateDatabaseOperation() {
            // ランダムにデータベースエラーをシミュレート
            if (Math.random() < 0.1) {
                throw new DatabaseException("データベース接続エラー");
            }
        }
        
        /**
         * 確認メールの送信をシミュレートします。
         * 
         * @param user ユーザー
         */
        private void sendConfirmationEmail(User user) {
            // ランダムにメール送信エラーをシミュレート
            if (Math.random() < 0.1) {
                throw new EmailException("メール送信エラー: " + user.getEmail());
            }
            
            System.out.println("確認メールを送信しました: " + user.getEmail());
        }
        
        /**
         * 登録済みのユーザーを取得します。
         * 
         * @return ユーザーリスト
         */
        public List<User> getUsers() {
            return new ArrayList<>(users);
        }
    }
    
    /**
     * ユーザークラス
     */
    public static class User {
        private String username;
        private String email;
        private String password;
        
        public User(String username, String email, String password) {
            this.username = username;
            this.email = email;
            this.password = password;
        }
        
        public String getUsername() { return username; }
        public String getEmail() { return email; }
        
        @Override
        public String toString() {
            return "User{username='" + username + "', email='" + email + "'}";
        }
    }
    
    /**
     * 入力値検証例外（チェック例外）
     */
    public static class ValidationException extends Exception {
        private List<String> errors;
        
        public ValidationException(String message, List<String> errors) {
            super(message);
            this.errors = errors;
        }
        
        public List<String> getErrors() {
            return errors;
        }
    }
    
    /**
     * 重複ユーザー例外（チェック例外）
     */
    public static class DuplicateUserException extends Exception {
        public DuplicateUserException(String message) {
            super(message);
        }
    }
    
    /**
     * システム例外（チェック例外）
     */
    public static class SystemException extends Exception {
        public SystemException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    /**
     * データベース例外（非チェック例外）
     */
    public static class DatabaseException extends RuntimeException {
        public DatabaseException(String message) {
            super(message);
        }
    }
    
    /**
     * メール例外（非チェック例外）
     */
    public static class EmailException extends RuntimeException {
        public EmailException(String message) {
            super(message);
        }
    }

    public static void main(String[] args) {
        UserRegistrationSystem system = new UserRegistrationSystem();
        
        // 正常なユーザー登録
        try {
            User user1 = system.registerUser("user1", "user1@example.com", "password123");
            System.out.println("ユーザーを登録しました: " + user1);
        } catch (ValidationException e) {
            System.err.println("入力値の検証エラー: " + e.getMessage());
            System.err.println("エラー詳細:");
            for (String error : e.getErrors()) {
                System.err.println("- " + error);
            }
        } catch (DuplicateUserException e) {
            System.err.println("重複エラー: " + e.getMessage());
        } catch (SystemException e) {
            System.err.println("システムエラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        }
        
        // 不正な入力値でのユーザー登録
        try {
            User user2 = system.registerUser("", "invalid-email", "short");
            System.out.println("ユーザーを登録しました: " + user2);
        } catch (ValidationException e) {
            System.err.println("入力値の検証エラー: " + e.getMessage());
            System.err.println("エラー詳細:");
            for (String error : e.getErrors()) {
                System.err.println("- " + error);
            }
        } catch (DuplicateUserException e) {
            System.err.println("重複エラー: " + e.getMessage());
        } catch (SystemException e) {
            System.err.println("システムエラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        }
        
        // 重複ユーザーでのユーザー登録
        try {
            // 最初に正常なユーザーを登録
            User user3 = system.registerUser("user3", "user3@example.com", "password123");
            System.out.println("ユーザーを登録しました: " + user3);
            
            // 同じユーザー名で再登録
            User user4 = system.registerUser("user3", "another@example.com", "password456");
            System.out.println("ユーザーを登録しました: " + user4);
        } catch (ValidationException e) {
            System.err.println("入力値の検証エラー: " + e.getMessage());
            System.err.println("エラー詳細:");
            for (String error : e.getErrors()) {
                System.err.println("- " + error);
            }
        } catch (DuplicateUserException e) {
            System.err.println("重複エラー: " + e.getMessage());
        } catch (SystemException e) {
            System.err.println("システムエラー: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        }
        
        // 登録済みのユーザーを表示
        System.out.println("\n登録済みのユーザー:");
        for (User user : system.getUsers()) {
            System.out.println("- " + user);
        }
    }
}
```

## コア演習3: 例外の変換とラッピング

### 解答例
```java
package exception.wrapper;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

/**
 * ファイル操作ユーティリティクラス
 */
public class FileUtils {
    
    /**
     * ファイルの内容を文字列として読み込みます。
     * 
     * @param filePath ファイルパス
     * @return ファイルの内容
     * @throws FileOperationException ファイル操作中にエラーが発生した場合
     */
    public static String readFileAsString(String filePath) throws FileOperationException {
        try {
            Path path = Paths.get(filePath);
            return new String(Files.readAllBytes(path), StandardCharsets.UTF_8);
        } catch (IOException e) {
            // IOExceptionをFileOperationExceptionにラップ
            throw new FileOperationException("ファイルの読み込みに失敗しました: " + filePath, e);
        }
    }
    
    /**
     * ファイルの内容を行のリストとして読み込みます。
     * 
     * @param filePath ファイルパス
     * @return 行のリスト
     * @throws FileOperationException ファイル操作中にエラーが発生した場合
     */
    public static List<String> readFileAsLines(String filePath) throws FileOperationException {
        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            List<String> lines = new ArrayList<>();
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
            return lines;
        } catch (FileNotFoundException e) {
            // FileNotFoundExceptionを特定のFileOperationExceptionにラップ
            throw new FileOperationException("ファイルが見つかりません: " + filePath, e, FileOperationException.ErrorType.FILE_NOT_FOUND);
        } catch (IOException e) {
            // IOExceptionを一般的なFileOperationExceptionにラップ
            throw new FileOperationException("ファイルの読み込み中にエラーが発生しました: " + filePath, e);
        }
    }
    
    /**
     * プロパティファイルを読み込みます。
     * 
     * @param filePath プロパティファイルのパス
     * @return プロパティオブジェクト
     * @throws ConfigurationException 設定ファイルの読み込み中にエラーが発生した場合
     */
    public static Properties loadProperties(String filePath) throws ConfigurationException {
        Properties properties = new Properties();
        
        try (InputStream input = new FileInputStream(filePath)) {
            properties.load(input);
            return properties;
        } catch (IOException e) {
            // IOExceptionをConfigurationExceptionにラップ
            throw new ConfigurationException("プロパティファイルの読み込みに失敗しました: " + filePath, e);
        }
    }
    
    /**
     * ファイルの内容を文字列として書き込みます。
     * 
     * @param filePath ファイルパス
     * @param content 書き込む内容
     * @throws FileOperationException ファイル操作中にエラーが発生した場合
     */
    public static void writeStringToFile(String filePath, String content) throws FileOperationException {
        try {
            Path path = Paths.get(filePath);
            Files.write(path, content.getBytes(StandardCharsets.UTF_8));
        } catch (IOException e) {
            // IOExceptionをFileOperationExceptionにラップ
            throw new FileOperationException("ファイルの書き込みに失敗しました: " + filePath, e);
        }
    }
    
    /**
     * ファイルの内容を行のリストとして書き込みます。
     * 
     * @param filePath ファイルパス
     * @param lines 書き込む行のリスト
     * @throws FileOperationException ファイル操作中にエラーが発生した場合
     */
    public static void writeLinesToFile(String filePath, List<String> lines) throws FileOperationException {
        try {
            Path path = Paths.get(filePath);
            Files.write(path, lines, StandardCharsets.UTF_8);
        } catch (IOException e) {
            // IOExceptionをFileOperationExceptionにラップ
            throw new FileOperationException("ファイルの書き込みに失敗しました: " + filePath, e);
        }
    }
    
    /**
     * ディレクトリを作成します。
     * 
     * @param directoryPath ディレクトリパス
     * @throws FileOperationException ディレクトリ作成中にエラーが発生した場合
     */
    public static void createDirectory(String directoryPath) throws FileOperationException {
        try {
            Path path = Paths.get(directoryPath);
            Files.createDirectories(path);
        } catch (IOException e) {
            // IOExceptionをFileOperationExceptionにラップ
            throw new FileOperationException("ディレクトリの作成に失敗しました: " + directoryPath, e);
        }
    }
    
    /**
     * ファイルを削除します。
     * 
     * @param filePath ファイルパス
     * @throws FileOperationException ファイル削除中にエラーが発生した場合
     */
    public static void deleteFile(String filePath) throws FileOperationException {
        try {
            Path path = Paths.get(filePath);
            Files.deleteIfExists(path);
        } catch (IOException e) {
            // IOExceptionをFileOperationExceptionにラップ
            throw new FileOperationException("ファイルの削除に失敗しました: " + filePath, e);
        }
    }
    
    /**
     * ファイルをコピーします。
     * 
     * @param sourcePath コピー元ファイルパス
     * @param destinationPath コピー先ファイルパス
     * @throws FileOperationException ファイルコピー中にエラーが発生した場合
     */
    public static void copyFile(String sourcePath, String destinationPath) throws FileOperationException {
        try {
            Path source = Paths.get(sourcePath);
            Path destination = Paths.get(destinationPath);
            Files.copy(source, destination, java.nio.file.StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            // IOExceptionをFileOperationExceptionにラップ
            throw new FileOperationException("ファイルのコピーに失敗しました: " + sourcePath + " -> " + destinationPath, e);
        }
    }
    
    /**
     * ファイルが存在するかチェックします。
     * 
     * @param filePath ファイルパス
     * @return ファイルが存在する場合はtrue
     */
    public static boolean fileExists(String filePath) {
        File file = new File(filePath);
        return file.exists() && file.isFile();
    }
    
    /**
     * ディレクトリが存在するかチェックします。
     * 
     * @param directoryPath ディレクトリパス
     * @return ディレクトリが存在する場合はtrue
     */
    public static boolean directoryExists(String directoryPath) {
        File directory = new File(directoryPath);
        return directory.exists() && directory.isDirectory();
    }
    
    /**
     * ファイル操作例外クラス
     */
    public static class FileOperationException extends Exception {
        
        /**
         * エラータイプ
         */
        public enum ErrorType {
            FILE_NOT_FOUND,
            PERMISSION_DENIED,
            IO_ERROR,
            UNKNOWN
        }
        
        private ErrorType errorType;
        
        public FileOperationException(String message) {
            super(message);
            this.errorType = ErrorType.UNKNOWN;
        }
        
        public FileOperationException(String message, Throwable cause) {
            super(message, cause);
            this.errorType = getErrorTypeFromCause(cause);
        }
        
        public FileOperationException(String message, Throwable cause, ErrorType errorType) {
            super(message, cause);
            this.errorType = errorType;
        }
        
        public ErrorType getErrorType() {
            return errorType;
        }
        
        /**
         * 原因となった例外からエラータイプを推測します。
         */
        private ErrorType getErrorTypeFromCause(Throwable cause) {
            if (cause instanceof FileNotFoundException) {
                return ErrorType.FILE_NOT_FOUND;
            } else if (cause instanceof java.nio.file.AccessDeniedException) {
                return ErrorType.PERMISSION_DENIED;
            } else if (cause instanceof IOException) {
                return ErrorType.IO_ERROR;
            } else {
                return ErrorType.UNKNOWN;
            }
        }
    }
    
    /**
     * 設定ファイル例外クラス
     */
    public static class ConfigurationException extends Exception {
        
        public ConfigurationException(String message) {
            super(message);
        }
        
        public ConfigurationException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    public static void main(String[] args) {
        // サンプルファイルの作成
        String sampleFilePath = "sample.txt";
        List<String> sampleContent = List.of(
            "これはサンプルファイルの1行目です。",
            "これはサンプルファイルの2行目です。",
            "これはサンプルファイルの3行目です。"
        );
        
        try {
            // ファイルの書き込み
            System.out.println("=== ファイルの書き込み ===");
            writeLinesToFile(sampleFilePath, sampleContent);
            System.out.println("ファイルを書き込みました: " + sampleFilePath);
            
            // ファイルの読み込み（文字列として）
            System.out.println("\n=== ファイルの読み込み（文字列として） ===");
            String content = readFileAsString(sampleFilePath);
            System.out.println("ファイルの内容:\n" + content);
            
            // ファイルの読み込み（行のリストとして）
            System.out.println("\n=== ファイルの読み込み（行のリストとして） ===");
            List<String> lines = readFileAsLines(sampleFilePath);
            System.out.println("ファイルの行数: " + lines.size());
            for (int i = 0; i < lines.size(); i++) {
                System.out.println((i + 1) + ": " + lines.get(i));
            }
            
            // 存在しないファイルの読み込み
            System.out.println("\n=== 存在しないファイルの読み込み ===");
            try {
                readFileAsLines("non_existent.txt");
            } catch (FileOperationException e) {
                System.err.println("エラー: " + e.getMessage());
                System.err.println("エラータイプ: " + e.getErrorType());
                if (e.getCause() != null) {
                    System.err.println("原因: " + e.getCause().getMessage());
                }
            }
            
            // ディレクトリの作成
            System.out.println("\n=== ディレクトリの作成 ===");
            String directoryPath = "sample_dir";
            createDirectory(directoryPath);
            System.out.println("ディレクトリを作成しました: " + directoryPath);
            
            // ファイルのコピー
            System.out.println("\n=== ファイルのコピー ===");
            String destinationPath = directoryPath + "/sample_copy.txt";
            copyFile(sampleFilePath, destinationPath);
            System.out.println("ファイルをコピーしました: " + sampleFilePath + " -> " + destinationPath);
            
            // コピーされたファイルの内容を確認
            List<String> copiedLines = readFileAsLines(destinationPath);
            System.out.println("コピーされたファイルの行数: " + copiedLines.size());
            
            // プロパティファイルの作成と読み込み
            System.out.println("\n=== プロパティファイルの作成と読み込み ===");
            String propertiesFilePath = "config.properties";
            List<String> propertiesContent = List.of(
                "# サンプル設定ファイル",
                "app.name=サンプルアプリケーション",
                "app.version=1.0.0",
                "app.debug=true"
            );
            writeLinesToFile(propertiesFilePath, propertiesContent);
            
            try {
                Properties properties = loadProperties(propertiesFilePath);
                System.out.println("読み込んだプロパティ:");
                System.out.println("- app.name: " + properties.getProperty("app.name"));
                System.out.println("- app.version: " + properties.getProperty("app.version"));
                System.out.println("- app.debug: " + properties.getProperty("app.debug"));
            } catch (ConfigurationException e) {
                System.err.println("設定ファイルの読み込みエラー: " + e.getMessage());
                if (e.getCause() != null) {
                    System.err.println("原因: " + e.getCause().getMessage());
                }
            }
            
            // ファイルの削除
            System.out.println("\n=== ファイルの削除 ===");
            deleteFile(sampleFilePath);
            System.out.println("ファイルを削除しました: " + sampleFilePath);
            
            deleteFile(destinationPath);
            System.out.println("ファイルを削除しました: " + destinationPath);
            
            deleteFile(propertiesFilePath);
            System.out.println("ファイルを削除しました: " + propertiesFilePath);
            
            // ディレクトリの削除
            java.io.File dir = new java.io.File(directoryPath);
            dir.delete();
            System.out.println("ディレクトリを削除しました: " + directoryPath);
            
        } catch (FileOperationException e) {
            System.err.println("ファイル操作エラー: " + e.getMessage());
            System.err.println("エラータイプ: " + e.getErrorType());
            if (e.getCause() != null) {
                System.err.println("原因: " + e.getCause().getMessage());
            }
        }
    }
}
```

## 発展演習1: 例外フィルタリング

### 解答例
```java
package exception.filter;

import java.io.FileNotFoundException;
import java.io.IOException;
import java.nio.file.AccessDeniedException;
import java.util.ArrayList;
import java.util.List;
import java.util.function.Predicate;

public class ExceptionFilteringDemo {

    /**
     * 例外フィルターインターフェース
     */
    @FunctionalInterface
    interface ExceptionFilter<E extends Exception> {
        boolean shouldHandle(E exception);
    }
    
    /**
     * 例外ハンドラーインターフェース
     */
    @FunctionalInterface
    interface ExceptionHandler<E extends Exception> {
        void handle(E exception) throws Exception;
    }
    
    /**
     * 例外処理ユーティリティクラス
     */
    static class ExceptionUtils {
        
        /**
         * 特定の例外のみをキャッチして処理します。
         * 
         * @param <E> 例外の型
         * @param exceptionType キャッチする例外の型
         * @param handler 例外ハンドラー
         * @return 例外ハンドラーラッパー
         */
        public static <E extends Exception> ExceptionHandler<Exception> handleOnly(
                Class<E> exceptionType, ExceptionHandler<E> handler) {
            
            return exception -> {
                if (exceptionType.isInstance(exception)) {
                    handler.handle(exceptionType.cast(exception));
                } else {
                    throw exception;
                }
            };
        }
        
        /**
         * 条件に一致する例外のみをキャッチして処理します。
         * 
         * @param <E> 例外の型
         * @param exceptionType キャッチする例外の型
         * @param filter 例外フィルター
         * @param handler 例外ハンドラー
         * @return 例外ハンドラーラッパー
         */
        public static <E extends Exception> ExceptionHandler<Exception> handleIf(
                Class<E> exceptionType, ExceptionFilter<E> filter, ExceptionHandler<E> handler) {
            
            return exception -> {
                if (exceptionType.isInstance(exception)) {
                    E typedException = exceptionType.cast(exception);
                    if (filter.shouldHandle(typedException)) {
                        handler.handle(typedException);
                    } else {
                        throw exception;
                    }
                } else {
                    throw exception;
                }
            };
        }
        
        /**
         * 複数の例外ハンドラーを連鎖させます。
         * 
         * @param handlers 例外ハンドラーのリスト
         * @return 連鎖された例外ハンドラー
         */
        @SafeVarargs
        public static ExceptionHandler<Exception> chain(ExceptionHandler<Exception>... handlers) {
            return exception -> {
                Exception currentException = exception;
                
                for (ExceptionHandler<Exception> handler : handlers) {
                    try {
                        handler.handle(currentException);
                        return; // 例外が処理された
                    } catch (Exception e) {
                        currentException = e; // 次のハンドラーに渡す
                    }
                }
                
                throw currentException; // すべてのハンドラーが例外を処理できなかった
            };
        }
        
        /**
         * 例外を変換します。
         * 
         * @param <E> 元の例外の型
         * @param <T> 変換後の例外の型
         * @param exceptionType 元の例外の型
         * @param mapper 例外マッパー
         * @return 例外ハンドラー
         */
        public static <E extends Exception, T extends Exception> ExceptionHandler<Exception> mapException(
                Class<E> exceptionType, ExceptionMapper<E, T> mapper) {
            
            return exception -> {
                if (exceptionType.isInstance(exception)) {
                    throw mapper.map(exceptionType.cast(exception));
                } else {
                    throw exception;
                }
            };
        }
    }
    
    /**
     * 例外マッパーインターフェース
     */
    @FunctionalInterface
    interface ExceptionMapper<E extends Exception, T extends Exception> {
        T map(E exception) throws T;
    }
    
    /**
     * ファイル操作例外クラス
     */
    static class FileOperationException extends Exception {
        
        public enum ErrorType {
            FILE_NOT_FOUND,
            PERMISSION_DENIED,
            IO_ERROR,
            UNKNOWN
        }
        
        private ErrorType errorType;
        
        public FileOperationException(String message, ErrorType errorType) {
            super(message);
            this.errorType = errorType;
        }
        
        public FileOperationException(String message, Throwable cause, ErrorType errorType) {
            super(message, cause);
            this.errorType = errorType;
        }
        
        public ErrorType getErrorType() {
            return errorType;
        }
    }
    
    /**
     * ファイル操作を実行します。
     * 
     * @param filePath ファイルパス
     * @throws IOException ファイル操作中にエラーが発生した場合
     */
    public static void performFileOperation(String filePath) throws IOException {
        double random = Math.random();
        
        if (random < 0.25) {
            throw new FileNotFoundException("ファイルが見つかりません: " + filePath);
        } else if (random < 0.5) {
            throw new AccessDeniedException(filePath, null, "アクセスが拒否されました");
        } else if (random < 0.75) {
            throw new IOException("ファイル操作中にエラーが発生しました: " + filePath);
        } else {
            System.out.println("ファイル操作が成功しました: " + filePath);
        }
    }
    
    /**
     * 例外フィルタリングを使用してファイル操作を実行します。
     * 
     * @param filePath ファイルパス
     */
    public static void executeWithExceptionFiltering(String filePath) {
        try {
            performFileOperation(filePath);
        } catch (Exception e) {
            try {
                // 例外ハンドラーチェーンを作成
                ExceptionHandler<Exception> handlerChain = ExceptionUtils.chain(
                    // FileNotFoundExceptionのみを処理
                    ExceptionUtils.handleOnly(FileNotFoundException.class, ex -> {
                        System.err.println("ファイルが見つかりません: " + ex.getMessage());
                        // 代替ファイルを使用するなどの回復処理
                    }),
                    
                    // AccessDeniedExceptionのみを処理
                    ExceptionUtils.handleOnly(AccessDeniedException.class, ex -> {
                        System.err.println("アクセスが拒否されました: " + ex.getMessage());
                        // 権限の要求や別の方法でのアクセスを試みるなどの回復処理
                    }),
                    
                    // 特定の条件を満たすIOExceptionのみを処理
                    ExceptionUtils.handleIf(IOException.class, 
                        ex -> ex.getMessage().contains("操作中"), // フィルター条件
                        ex -> {
                            System.err.println("IO操作中のエラー: " + ex.getMessage());
                            // 再試行などの回復処理
                        }
                    ),
                    
                    // その他のIOExceptionを変換
                    ExceptionUtils.mapException(IOException.class, 
                        ex -> new FileOperationException("ファイル操作に失敗しました", ex, 
                                FileOperationException.ErrorType.IO_ERROR))
                );
                
                // 例外ハンドラーチェーンを実行
                handlerChain.handle(e);
                
            } catch (FileOperationException ex) {
                System.err.println("変換された例外: " + ex.getMessage());
                System.err.println("エラータイプ: " + ex.getErrorType());
            } catch (Exception ex) {
                System.err.println("未処理の例外: " + ex.getMessage());
            }
        }
    }
    
    /**
     * 例外フィルタリングを使用して複数のファイル操作を実行します。
     * 
     * @param filePaths ファイルパスのリスト
     * @return 成功したファイル操作の数
     */
    public static int executeMultipleOperations(List<String> filePaths) {
        int successCount = 0;
        List<Exception> exceptions = new ArrayList<>();
        
        for (String filePath : filePaths) {
            try {
                performFileOperation(filePath);
                successCount++;
            } catch (Exception e) {
                exceptions.add(e);
            }
        }
        
        // 例外の集計と処理
        if (!exceptions.isEmpty()) {
            System.err.println("\n=== 例外の集計 ===");
            
            // FileNotFoundExceptionの数をカウント
            long fileNotFoundCount = exceptions.stream()
                    .filter(e -> e instanceof FileNotFoundException)
                    .count();
            
            // AccessDeniedExceptionの数をカウント
            long accessDeniedCount = exceptions.stream()
                    .filter(e -> e instanceof AccessDeniedException)
                    .count();
            
            // その他のIOExceptionの数をカウント
            long otherIOCount = exceptions.stream()
                    .filter(e -> e instanceof IOException && 
                           !(e instanceof FileNotFoundException) && 
                           !(e instanceof AccessDeniedException))
                    .count();
            
            System.err.println("ファイルが見つからない例外: " + fileNotFoundCount);
            System.err.println("アクセス拒否例外: " + accessDeniedCount);
            System.err.println("その他のIO例外: " + otherIOCount);
            
            // 最初の数個の例外の詳細を表示
            System.err.println("\n最初の3つの例外の詳細:");
            exceptions.stream()
                    .limit(3)
                    .forEach(e -> System.err.println("- " + e.getClass().getSimpleName() + ": " + e.getMessage()));
        }
        
        return successCount;
    }
    
    /**
     * 例外フィルタリングを使用して例外を分類します。
     * 
     * @param exceptions 例外のリスト
     * @return 分類された例外のマップ
     */
    public static void categorizeExceptions(List<Exception> exceptions) {
        System.out.println("\n=== 例外の分類 ===");
        
        // 例外フィルターの定義
        Predicate<Exception> isFileNotFound = e -> e instanceof FileNotFoundException;
        Predicate<Exception> isAccessDenied = e -> e instanceof AccessDeniedException;
        Predicate<Exception> isIOException = e -> e instanceof IOException && 
                                                 !(e instanceof FileNotFoundException) && 
                                                 !(e instanceof AccessDeniedException);
        
        // 例外の分類
        List<Exception> fileNotFoundExceptions = new ArrayList<>();
        List<Exception> accessDeniedExceptions = new ArrayList<>();
        List<Exception> otherIOExceptions = new ArrayList<>();
        List<Exception> otherExceptions = new ArrayList<>();
        
        for (Exception e : exceptions) {
            if (isFileNotFound.test(e)) {
                fileNotFoundExceptions.add(e);
            } else if (isAccessDenied.test(e)) {
                accessDeniedExceptions.add(e);
            } else if (isIOException.test(e)) {
                otherIOExceptions.add(e);
            } else {
                otherExceptions.add(e);
            }
        }
        
        // 分類結果の表示
        System.out.println("ファイルが見つからない例外: " + fileNotFoundExceptions.size());
        System.out.println("アクセス拒否例外: " + accessDeniedExceptions.size());
        System.out.println("その他のIO例外: " + otherIOExceptions.size());
        System.out.println("その他の例外: " + otherExceptions.size());
    }

    public static void main(String[] args) {
        // 単一のファイル操作
        System.out.println("=== 単一のファイル操作 ===");
        executeWithExceptionFiltering("example.txt");
        
        // 複数のファイル操作
        System.out.println("\n=== 複数のファイル操作 ===");
        List<String> filePaths = List.of(
            "file1.txt",
            "file2.txt",
            "file3.txt",
            "file4.txt",
            "file5.txt"
        );
        
        int successCount = executeMultipleOperations(filePaths);
        System.out.println("\n成功したファイル操作: " + successCount + "/" + filePaths.size());
        
        // 例外の収集と分類
        System.out.println("\n=== 例外の収集と分類 ===");
        List<Exception> collectedExceptions = new ArrayList<>();
        
        for (int i = 0; i < 20; i++) {
            try {
                performFileOperation("test" + i + ".txt");
            } catch (Exception e) {
                collectedExceptions.add(e);
            }
        }
        
        System.out.println("収集された例外の数: " + collectedExceptions.size());
        categorizeExceptions(collectedExceptions);
    }
}
```

## 発展演習2: 例外処理のパフォーマンス測定

### 解答例
```java
package exception.performance;

import java.io.FileNotFoundException;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import java.util.function.Supplier;

public class ExceptionPerformanceDemo {

    /**
     * 結果クラス
     */
    static class Result<T> {
        private final T value;
        private final Exception error;
        
        private Result(T value, Exception error) {
            this.value = value;
            this.error = error;
        }
        
        public static <T> Result<T> success(T value) {
            return new Result<>(value, null);
        }
        
        public static <T> Result<T> failure(Exception error) {
            return new Result<>(null, error);
        }
        
        public boolean isSuccess() {
            return error == null;
        }
        
        public T getValue() {
            if (!isSuccess()) {
                throw new IllegalStateException("結果が失敗の場合は値を取得できません");
            }
            return value;
        }
        
        public Exception getError() {
            if (isSuccess()) {
                throw new IllegalStateException("結果が成功の場合はエラーを取得できません");
            }
            return error;
        }
        
        public <U> Result<U> map(ThrowingFunction<T, U> mapper) {
            if (!isSuccess()) {
                return Result.failure(error);
            }
            
            try {
                return Result.success(mapper.apply(value));
            } catch (Exception e) {
                return Result.failure(e);
            }
        }
        
        public T getOrElse(T defaultValue) {
            return isSuccess() ? value : defaultValue;
        }
        
        public T getOrElseGet(Supplier<T> supplier) {
            return isSuccess() ? value : supplier.get();
        }
        
        public void ifSuccess(ThrowingConsumer<T> consumer) {
            if (isSuccess()) {
                try {
                    consumer.accept(value);
                } catch (Exception e) {
                    // 例外を無視
                }
            }
        }
        
        public void ifFailure(ThrowingConsumer<Exception> consumer) {
            if (!isSuccess()) {
                try {
                    consumer.accept(error);
                } catch (Exception e) {
                    // 例外を無視
                }
            }
        }
    }
    
    /**
     * 例外をスローできる関数インターフェース
     */
    @FunctionalInterface
    interface ThrowingFunction<T, R> {
        R apply(T t) throws Exception;
    }
    
    /**
     * 例外をスローできるコンシューマーインターフェース
     */
    @FunctionalInterface
    interface ThrowingConsumer<T> {
        void accept(T t) throws Exception;
    }
    
    /**
     * 例外をスローできるサプライヤーインターフェース
     */
    @FunctionalInterface
    interface ThrowingSupplier<T> {
        T get() throws Exception;
    }
    
    /**
     * 例外を使用した処理
     */
    public static String processWithExceptions(String input) throws IOException {
        if (input == null) {
            throw new NullPointerException("入力がnullです");
        }
        
        if (input.isEmpty()) {
            throw new IllegalArgumentException("入力が空です");
        }
        
        if (input.equals("not_found")) {
            throw new FileNotFoundException("ファイルが見つかりません");
        }
        
        if (input.equals("io_error")) {
            throw new IOException("IO操作中にエラーが発生しました");
        }
        
        return "処理結果: " + input;
    }
    
    /**
     * 結果オブジェクトを使用した処理
     */
    public static Result<String> processWithResult(String input) {
        if (input == null) {
            return Result.failure(new NullPointerException("入力がnullです"));
        }
        
        if (input.isEmpty()) {
            return Result.failure(new IllegalArgumentException("入力が空です"));
        }
        
        if (input.equals("not_found")) {
            return Result.failure(new FileNotFoundException("ファイルが見つかりません"));
        }
        
        if (input.equals("io_error")) {
            return Result.failure(new IOException("IO操作中にエラーが発生しました"));
        }
        
        return Result.success("処理結果: " + input);
    }
    
    /**
     * Optionalを使用した処理
     */
    public static Optional<String> processWithOptional(String input) {
        if (input == null || input.isEmpty() || input.equals("not_found") || input.equals("io_error")) {
            return Optional.empty();
        }
        
        return Optional.of("処理結果: " + input);
    }
    
    /**
     * 例外を使用した処理のパフォーマンス測定
     */
    public static void measureExceptionPerformance() {
        System.out.println("=== 例外を使用した処理のパフォーマンス測定 ===");
        
        // テストデータの準備
        List<String> inputs = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            inputs.add("input" + i);
        }
        
        // 正常系のみのケース
        long startTime1 = System.nanoTime();
        
        for (String input : inputs) {
            try {
                String result = processWithExceptions(input);
            } catch (IOException e) {
                // 例外を無視
            }
        }
        
        long endTime1 = System.nanoTime();
        long duration1 = (endTime1 - startTime1) / 1_000_000; // ミリ秒に変換
        
        // 例外が発生するケース（10%の確率）
        inputs.clear();
        for (int i = 0; i < 900; i++) {
            inputs.add("input" + i);
        }
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                inputs.add("not_found");
            } else {
                inputs.add("io_error");
            }
        }
        
        long startTime2 = System.nanoTime();
        
        for (String input : inputs) {
            try {
                String result = processWithExceptions(input);
            } catch (IOException e) {
                // 例外を無視
            }
        }
        
        long endTime2 = System.nanoTime();
        long duration2 = (endTime2 - startTime2) / 1_000_000; // ミリ秒に変換
        
        System.out.println("正常系のみ: " + duration1 + " ms");
        System.out.println("例外発生あり（10%）: " + duration2 + " ms");
        System.out.println("パフォーマンス比: " + (double) duration2 / duration1);
    }
    
    /**
     * 結果オブジェクトを使用した処理のパフォーマンス測定
     */
    public static void measureResultPerformance() {
        System.out.println("\n=== 結果オブジェクトを使用した処理のパフォーマンス測定 ===");
        
        // テストデータの準備
        List<String> inputs = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            inputs.add("input" + i);
        }
        
        // 正常系のみのケース
        long startTime1 = System.nanoTime();
        
        for (String input : inputs) {
            Result<String> result = processWithResult(input);
            if (result.isSuccess()) {
                String value = result.getValue();
            }
        }
        
        long endTime1 = System.nanoTime();
        long duration1 = (endTime1 - startTime1) / 1_000_000; // ミリ秒に変換
        
        // エラーが発生するケース（10%の確率）
        inputs.clear();
        for (int i = 0; i < 900; i++) {
            inputs.add("input" + i);
        }
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                inputs.add("not_found");
            } else {
                inputs.add("io_error");
            }
        }
        
        long startTime2 = System.nanoTime();
        
        for (String input : inputs) {
            Result<String> result = processWithResult(input);
            if (result.isSuccess()) {
                String value = result.getValue();
            }
        }
        
        long endTime2 = System.nanoTime();
        long duration2 = (endTime2 - startTime2) / 1_000_000; // ミリ秒に変換
        
        System.out.println("正常系のみ: " + duration1 + " ms");
        System.out.println("エラー発生あり（10%）: " + duration2 + " ms");
        System.out.println("パフォーマンス比: " + (double) duration2 / duration1);
    }
    
    /**
     * Optionalを使用した処理のパフォーマンス測定
     */
    public static void measureOptionalPerformance() {
        System.out.println("\n=== Optionalを使用した処理のパフォーマンス測定 ===");
        
        // テストデータの準備
        List<String> inputs = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            inputs.add("input" + i);
        }
        
        // 正常系のみのケース
        long startTime1 = System.nanoTime();
        
        for (String input : inputs) {
            Optional<String> result = processWithOptional(input);
            if (result.isPresent()) {
                String value = result.get();
            }
        }
        
        long endTime1 = System.nanoTime();
        long duration1 = (endTime1 - startTime1) / 1_000_000; // ミリ秒に変換
        
        // 空のOptionalが返るケース（10%の確率）
        inputs.clear();
        for (int i = 0; i < 900; i++) {
            inputs.add("input" + i);
        }
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                inputs.add("not_found");
            } else {
                inputs.add("io_error");
            }
        }
        
        long startTime2 = System.nanoTime();
        
        for (String input : inputs) {
            Optional<String> result = processWithOptional(input);
            if (result.isPresent()) {
                String value = result.get();
            }
        }
        
        long endTime2 = System.nanoTime();
        long duration2 = (endTime2 - startTime2) / 1_000_000; // ミリ秒に変換
        
        System.out.println("正常系のみ: " + duration1 + " ms");
        System.out.println("空のOptionalあり（10%）: " + duration2 + " ms");
        System.out.println("パフォーマンス比: " + (double) duration2 / duration1);
    }
    
    /**
     * 例外のスタックトレースのパフォーマンス測定
     */
    public static void measureStackTracePerformance() {
        System.out.println("\n=== 例外のスタックトレースのパフォーマンス測定 ===");
        
        // スタックトレースあり
        long startTime1 = System.nanoTime();
        
        for (int i = 0; i < 1000; i++) {
            try {
                throw new RuntimeException("テスト例外");
            } catch (RuntimeException e) {
                // スタックトレースを取得
                StackTraceElement[] stackTrace = e.getStackTrace();
            }
        }
        
        long endTime1 = System.nanoTime();
        long duration1 = (endTime1 - startTime1) / 1_000_000; // ミリ秒に変換
        
        // スタックトレースなし
        long startTime2 = System.nanoTime();
        
        for (int i = 0; i < 1000; i++) {
            try {
                throw new RuntimeException("テスト例外") {
                    // スタックトレースを抑制
                    @Override
                    public synchronized Throwable fillInStackTrace() {
                        return this;
                    }
                };
            } catch (RuntimeException e) {
                // スタックトレースを取得
                StackTraceElement[] stackTrace = e.getStackTrace();
            }
        }
        
        long endTime2 = System.nanoTime();
        long duration2 = (endTime2 - startTime2) / 1_000_000; // ミリ秒に変換
        
        System.out.println("スタックトレースあり: " + duration1 + " ms");
        System.out.println("スタックトレースなし: " + duration2 + " ms");
        System.out.println("パフォーマンス比: " + (double) duration1 / duration2);
    }
    
    /**
     * 例外処理の深さのパフォーマンス測定
     */
    public static void measureExceptionDepthPerformance() {
        System.out.println("\n=== 例外処理の深さのパフォーマンス測定 ===");
        
        // 浅い例外処理
        long startTime1 = System.nanoTime();
        
        for (int i = 0; i < 1000; i++) {
            try {
                if (i % 10 == 0) {
                    throw new RuntimeException("テスト例外");
                }
            } catch (RuntimeException e) {
                // 例外を処理
            }
        }
        
        long endTime1 = System.nanoTime();
        long duration1 = (endTime1 - startTime1) / 1_000_000; // ミリ秒に変換
        
        // 深い例外処理
        long startTime2 = System.nanoTime();
        
        for (int i = 0; i < 1000; i++) {
            try {
                method1(i);
            } catch (RuntimeException e) {
                // 例外を処理
            }
        }
        
        long endTime2 = System.nanoTime();
        long duration2 = (endTime2 - startTime2) / 1_000_000; // ミリ秒に変換
        
        System.out.println("浅い例外処理: " + duration1 + " ms");
        System.out.println("深い例外処理: " + duration2 + " ms");
        System.out.println("パフォーマンス比: " + (double) duration2 / duration1);
    }
    
    private static void method1(int i) {
        method2(i);
    }
    
    private static void method2(int i) {
        method3(i);
    }
    
    private static void method3(int i) {
        method4(i);
    }
    
    private static void method4(int i) {
        method5(i);
    }
    
    private static void method5(int i) {
        if (i % 10 == 0) {
            throw new RuntimeException("テスト例外");
        }
    }
    
    /**
     * try-finallyとtry-with-resourcesのパフォーマンス比較
     */
    public static void compareTryFinallyAndTryWithResources() {
        System.out.println("\n=== try-finallyとtry-with-resourcesのパフォーマンス比較 ===");
        
        // try-finally
        long startTime1 = System.nanoTime();
        
        for (int i = 0; i < 10000; i++) {
            CustomResource resource = new CustomResource();
            try {
                resource.use();
            } finally {
                resource.close();
            }
        }
        
        long endTime1 = System.nanoTime();
        long duration1 = (endTime1 - startTime1) / 1_000_000; // ミリ秒に変換
        
        // try-with-resources
        long startTime2 = System.nanoTime();
        
        for (int i = 0; i < 10000; i++) {
            try (CustomResource resource = new CustomResource()) {
                resource.use();
            }
        }
        
        long endTime2 = System.nanoTime();
        long duration2 = (endTime2 - startTime2) / 1_000_000; // ミリ秒に変換
        
        System.out.println("try-finally: " + duration1 + " ms");
        System.out.println("try-with-resources: " + duration2 + " ms");
        System.out.println("パフォーマンス比: " + (double) duration1 / duration2);
    }
    
    /**
     * カスタムリソースクラス
     */
    static class CustomResource implements AutoCloseable {
        public void use() {
            // リソースを使用
        }
        
        @Override
        public void close() {
            // リソースをクローズ
        }
    }

    public static void main(String[] args) {
        // 例外を使用した処理のパフォーマンス測定
        measureExceptionPerformance();
        
        // 結果オブジェクトを使用した処理のパフォーマンス測定
        measureResultPerformance();
        
        // Optionalを使用した処理のパフォーマンス測定
        measureOptionalPerformance();
        
        // 例外のスタックトレースのパフォーマンス測定
        measureStackTracePerformance();
        
        // 例外処理の深さのパフォーマンス測定
        measureExceptionDepthPerformance();
        
        // try-finallyとtry-with-resourcesのパフォーマンス比較
        compareTryFinallyAndTryWithResources();
        
        // 結論
        System.out.println("\n=== 結論 ===");
        System.out.println("1. 例外は正常系のフローよりも大幅に遅い");
        System.out.println("2. 結果オブジェクトやOptionalは例外よりも高速");
        System.out.println("3. スタックトレースの生成は例外処理のコストの大部分を占める");
        System.out.println("4. 例外処理の深さが深いほどパフォーマンスが低下する");
        System.out.println("5. try-with-resourcesはtry-finallyと同等以上のパフォーマンス");
        System.out.println("6. 例外はエラー処理のために使用し、制御フローには使用しない");
    }
}
```
