---
layout: default
title: "Day 1: 入出力処理の基本と応用 - 解答例"
---
# Day 1: 入出力処理の基本と応用 - 解答例

## 演習1: 基本的なファイル入出力

### 解答例
```java
package io.basic;

import java.io.*;

/**
 * 基本的なファイル入出力を行うクラス
 */
public class FileNumbering {
    
    /**
     * 入力ファイルを読み込み、各行の先頭に行番号を付けて出力ファイルに書き出します。
     * 
     * @param inputFile 入力ファイルのパス
     * @param outputFile 出力ファイルのパス
     * @throws IOException 入出力エラーが発生した場合
     */
    public void numberLines(String inputFile, String outputFile) throws IOException {
        // BufferedReaderとBufferedWriterを使用
        try (BufferedReader reader = new BufferedReader(new FileReader(inputFile));
             BufferedWriter writer = new BufferedWriter(new FileWriter(outputFile))) {
            
            String line;
            int lineNumber = 1;
            
            // ファイルを1行ずつ読み込む
            while ((line = reader.readLine()) != null) {
                // 行番号を付けて書き出す
                writer.write(lineNumber + ": " + line);
                writer.newLine();
                lineNumber++;
            }
        }
    }
    
    public static void main(String[] args) {
        FileNumbering app = new FileNumbering();
        try {
            app.numberLines("input.txt", "output.txt");
            System.out.println("処理が完了しました。");
        } catch (IOException e) {
            System.err.println("エラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### 解説
この解答例では、以下の点に注目してください：

1. **try-with-resources文の使用**：
   - `BufferedReader`と`BufferedWriter`をtry-with-resources文で宣言することで、使用後に自動的にクローズされます。
   - これにより、例外が発生した場合でもリソースリークを防ぐことができます。

2. **行番号の管理**：
   - `lineNumber`変数を1で初期化し、1行読み込むごとにインクリメントしています。
   - これにより、各行に連番を付けることができます。

3. **書き込み処理**：
   - `writer.write()`メソッドで行番号とコロン、スペース、元の行の内容を連結して書き込んでいます。
   - `writer.newLine()`メソッドで改行を追加しています。

4. **例外処理**：
   - `IOException`をキャッチして適切なエラーメッセージを表示しています。
   - 実際のアプリケーションでは、より詳細なエラーハンドリングが必要な場合があります。

## 演習2: ログファイル解析

### 解答例
```java
package io.log;

import java.io.*;
import java.util.*;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * ログファイル解析を行うクラス
 */
public class LogAnalyzer {
    
    // ログの形式: IPアドレス - - [日時] "リクエスト" ステータスコード サイズ
    private static final Pattern LOG_PATTERN = 
            Pattern.compile("(\\S+) - - \\[(.+?)\\] \"(.+?)\" (\\d+) (\\d+|-)");
    
    /**
     * ログファイルを解析し、結果を表示します。
     * 
     * @param logFile ログファイルのパス
     * @throws IOException 入出力エラーが発生した場合
     */
    public void analyzeLog(String logFile) throws IOException {
        int totalAccess = 0;
        Map<String, Integer> statusCounts = new HashMap<>();
        Map<String, Integer> ipCounts = new HashMap<>();
        
        try (BufferedReader reader = new BufferedReader(new FileReader(logFile))) {
            String line;
            
            while ((line = reader.readLine()) != null) {
                Matcher matcher = LOG_PATTERN.matcher(line);
                
                if (matcher.find()) {
                    // IPアドレスとステータスコードを抽出
                    String ipAddress = matcher.group(1);
                    String statusCode = matcher.group(4);
                    
                    // 合計アクセス数をカウント
                    totalAccess++;
                    
                    // ステータスコード別のアクセス数をカウント
                    statusCounts.put(statusCode, statusCounts.getOrDefault(statusCode, 0) + 1);
                    
                    // IPアドレス別のアクセス数をカウント
                    ipCounts.put(ipAddress, ipCounts.getOrDefault(ipAddress, 0) + 1);
                }
            }
        }
        
        // 結果を表示
        displayResults(totalAccess, statusCounts, ipCounts);
    }
    
    /**
     * 解析結果を表示します。
     * 
     * @param totalAccess 合計アクセス数
     * @param statusCounts ステータスコード別のアクセス数
     * @param ipCounts IPアドレス別のアクセス数
     */
    private void displayResults(int totalAccess, Map<String, Integer> statusCounts, Map<String, Integer> ipCounts) {
        System.out.println("===== アクセスログ解析結果 =====");
        System.out.println("合計アクセス数: " + totalAccess);
        
        System.out.println("\nステータスコード別アクセス数:");
        for (Map.Entry<String, Integer> entry : statusCounts.entrySet()) {
            System.out.printf("  %s: %d回 (%.1f%%)\n", 
                             entry.getKey(), 
                             entry.getValue(), 
                             (entry.getValue() * 100.0 / totalAccess));
        }
        
        System.out.println("\nIPアドレス別アクセス数:");
        for (Map.Entry<String, Integer> entry : ipCounts.entrySet()) {
            System.out.printf("  %s: %d回 (%.1f%%)\n", 
                             entry.getKey(), 
                             entry.getValue(), 
                             (entry.getValue() * 100.0 / totalAccess));
        }
    }
    
    public static void main(String[] args) {
        LogAnalyzer analyzer = new LogAnalyzer();
        try {
            analyzer.analyzeLog("access.log");
        } catch (IOException e) {
            System.err.println("エラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### 解説
この解答例では、以下の点に注目してください：

1. **正規表現パターンの使用**：
   - `LOG_PATTERN`という正規表現パターンを定義して、ログ行からIPアドレスやステータスコードなどの情報を抽出しています。
   - 正規表現のグループ化機能を使って、必要な情報を個別に取得しています。

2. **データ集計**：
   - `Map<String, Integer>`を使用して、ステータスコード別とIPアドレス別のアクセス数をカウントしています。
   - `getOrDefault`メソッドを使って、既存のカウントに1を加算しています。

3. **結果の表示**：
   - 合計アクセス数、ステータスコード別アクセス数、IPアドレス別アクセス数を整形して表示しています。
   - パーセンテージも計算して表示しています。

4. **例外処理**：
   - `IOException`をキャッチして適切なエラーメッセージを表示しています。

## 演習3: CSVファイル処理

### 解答例
```java
package io.csv;

import java.io.*;
import java.util.*;

/**
 * CSVファイル処理を行うクラス
 */
public class CsvProcessor {
    
    /**
     * CSVファイルを読み込み、条件に合致するレコードを抽出して別のCSVファイルに書き出します。
     * 条件: 年齢が30歳以上のレコード
     * 
     * @param inputCsv 入力CSVファイルのパス
     * @param outputCsv 出力CSVファイルのパス
     * @throws IOException 入出力エラーが発生した場合
     */
    public void processCSV(String inputCsv, String outputCsv) throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader(inputCsv));
             BufferedWriter writer = new BufferedWriter(new FileWriter(outputCsv))) {
            
            String line;
            boolean isFirstLine = true;
            
            while ((line = reader.readLine()) != null) {
                if (isFirstLine) {
                    // ヘッダー行はそのまま出力
                    writer.write(line);
                    writer.newLine();
                    isFirstLine = false;
                } else {
                    // データ行を処理
                    String[] fields = line.split(",");
                    
                    // 年齢（3番目のフィールド）が30以上かチェック
                    if (fields.length >= 3 && isAgeOver30(fields[2])) {
                        writer.write(line);
                        writer.newLine();
                    }
                }
            }
        }
    }
    
    /**
     * 年齢が30歳以上かどうかをチェックします。
     * 
     * @param ageStr 年齢の文字列
     * @return 30歳以上の場合はtrue、それ以外はfalse
     */
    private boolean isAgeOver30(String ageStr) {
        try {
            int age = Integer.parseInt(ageStr.trim());
            return age >= 30;
        } catch (NumberFormatException e) {
            // 年齢が数値でない場合はfalseを返す
            return false;
        }
    }
    
    public static void main(String[] args) {
        CsvProcessor processor = new CsvProcessor();
        try {
            processor.processCSV("users.csv", "filtered_users.csv");
            System.out.println("処理が完了しました。");
        } catch (IOException e) {
            System.err.println("エラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### 解説
この解答例では、以下の点に注目してください：

1. **CSVファイルの読み込みと書き込み**：
   - `BufferedReader`と`BufferedWriter`を使用して、CSVファイルを効率的に読み書きしています。
   - try-with-resourcesを使用して、リソースの解放を確実に行っています。

2. **ヘッダー行の処理**：
   - 1行目はヘッダー行として扱い、条件に関係なくそのまま出力しています。
   - `isFirstLine`フラグを使って、ヘッダー行かどうかを判断しています。

3. **データ行の処理**：
   - `String.split(",")`を使用して、CSVの各フィールドに分割しています。
   - 年齢（3番目のフィールド）が30以上のレコードのみを抽出しています。

4. **年齢の判定**：
   - `isAgeOver30`メソッドで、年齢が30歳以上かどうかを判定しています。
   - 数値に変換できない場合は`NumberFormatException`をキャッチして、falseを返しています。

5. **例外処理**：
   - `IOException`をキャッチして適切なエラーメッセージを表示しています。

## 発展演習1: バイナリファイルのコピー

### 解答例
```java
package io.binary;

import java.io.*;

/**
 * バイナリファイルのコピーを行うクラス
 */
public class BinaryFileCopier {
    
    /**
     * バイナリファイルをコピーします。
     * 
     * @param sourceFile コピー元ファイルのパス
     * @param targetFile コピー先ファイルのパス
     * @param bufferSize バッファサイズ（バイト単位）
     * @return コピーしたバイト数
     * @throws IOException 入出力エラーが発生した場合
     */
    public long copyFile(String sourceFile, String targetFile, int bufferSize) throws IOException {
        long bytesCopied = 0;
        
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(sourceFile));
             BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(targetFile))) {
            
            byte[] buffer = new byte[bufferSize];
            int bytesRead;
            
            while ((bytesRead = bis.read(buffer)) != -1) {
                bos.write(buffer, 0, bytesRead);
                bytesCopied += bytesRead;
            }
        }
        
        return bytesCopied;
    }
    
    /**
     * 異なるバッファサイズでのコピー性能を比較します。
     * 
     * @param sourceFile コピー元ファイルのパス
     * @throws IOException 入出力エラーが発生した場合
     */
    public void compareBufferSizes(String sourceFile) throws IOException {
        int[] bufferSizes = {1024, 8192, 32768, 131072};
        
        System.out.println("バッファサイズによるパフォーマンス比較:");
        System.out.println("-----------------------------------");
        
        for (int bufferSize : bufferSizes) {
            String targetFile = "copy_" + bufferSize + "_" + new File(sourceFile).getName();
            
            long startTime = System.currentTimeMillis();
            long bytesCopied = copyFile(sourceFile, targetFile, bufferSize);
            long endTime = System.currentTimeMillis();
            
            System.out.printf("バッファサイズ: %6d バイト, コピー時間: %4d ミリ秒, コピーサイズ: %d バイト%n", 
                             bufferSize, (endTime - startTime), bytesCopied);
        }
    }
    
    public static void main(String[] args) {
        BinaryFileCopier copier = new BinaryFileCopier();
        try {
            // 任意のバイナリファイル（画像やPDFなど）のパスを指定
            String sourceFile = "sample.jpg";
            
            // 異なるバッファサイズでの性能比較
            copier.compareBufferSizes(sourceFile);
            
            // 最適なバッファサイズでコピー
            long bytesCopied = copier.copyFile(sourceFile, "copied_" + sourceFile, 8192);
            System.out.println("コピーが完了しました。コピーしたバイト数: " + bytesCopied);
        } catch (IOException e) {
            System.err.println("エラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### 解説
この解答例では、以下の点に注目してください：

1. **バッファリングストリームの使用**：
   - `BufferedInputStream`と`BufferedOutputStream`を使用して、バッファリングを有効にしています。
   - これにより、ディスクアクセスの回数を減らし、パフォーマンスを向上させています。

2. **バッファサイズの指定**：
   - `bufferSize`パラメータで、バッファサイズを指定できるようにしています。
   - 異なるバッファサイズでのパフォーマンス比較も行っています。

3. **バイト単位の読み書き**：
   - `read`メソッドと`write`メソッドを使って、バイト単位でデータを転送しています。
   - 読み込んだバイト数を返す`read`メソッドの戻り値を使って、ファイルの終端を検出しています。

4. **パフォーマンス計測**：
   - `System.currentTimeMillis()`を使って、コピー処理の実行時間を計測しています。
   - 異なるバッファサイズでのパフォーマンスを比較して表示しています。

5. **例外処理**：
   - `IOException`をキャッチして適切なエラーメッセージを表示しています。

## 発展演習2: プロパティファイルの操作

### 解答例
```java
package io.properties;

import java.io.*;
import java.util.*;

/**
 * プロパティファイルを操作するクラス
 */
public class ConfigManager {
    
    private Properties properties;
    private String configFile;
    
    /**
     * コンストラクタ
     * 
     * @param configFile プロパティファイルのパス
     */
    public ConfigManager(String configFile) {
        this.configFile = configFile;
        this.properties = new Properties();
    }
    
    /**
     * プロパティファイルを読み込みます。
     * 
     * @throws IOException 入出力エラーが発生した場合
     */
    public void load() throws IOException {
        try (FileInputStream fis = new FileInputStream(configFile)) {
            properties.load(fis);
        }
    }
    
    /**
     * プロパティ値を取得します。
     * 
     * @param key プロパティのキー
     * @return プロパティの値（キーが存在しない場合はnull）
     */
    public String getProperty(String key) {
        return properties.getProperty(key);
    }
    
    /**
     * プロパティ値を取得します。キーが存在しない場合はデフォルト値を返します。
     * 
     * @param key プロパティのキー
     * @param defaultValue デフォルト値
     * @return プロパティの値（キーが存在しない場合はデフォルト値）
     */
    public String getProperty(String key, String defaultValue) {
        return properties.getProperty(key, defaultValue);
    }
    
    /**
     * プロパティ値を設定します。
     * 
     * @param key プロパティのキー
     * @param value プロパティの値
     */
    public void setProperty(String key, String value) {
        properties.setProperty(key, value);
    }
    
    /**
     * プロパティファイルに変更を保存します。
     * 
     * @param comments コメント（ファイルの先頭に追加される）
     * @throws IOException 入出力エラーが発生した場合
     */
    public void save(String comments) throws IOException {
        try (FileOutputStream fos = new FileOutputStream(configFile)) {
            properties.store(fos, comments);
        }
    }
    
    /**
     * すべてのプロパティを表示します。
     */
    public void listProperties() {
        System.out.println("===== プロパティ一覧 =====");
        for (String key : properties.stringPropertyNames()) {
            System.out.println(key + " = " + properties.getProperty(key));
        }
    }
    
    public static void main(String[] args) {
        ConfigManager configManager = new ConfigManager("config.properties");
        try {
            // 既存のプロパティファイルがあれば読み込む
            try {
                configManager.load();
                System.out.println("既存の設定を読み込みました。");
            } catch (FileNotFoundException e) {
                System.out.println("新しい設定ファイルを作成します。");
            }
            
            // プロパティを設定
            configManager.setProperty("app.name", "MyApp");
            configManager.setProperty("app.version", "1.0.0");
            configManager.setProperty("app.language", "ja");
            configManager.setProperty("db.url", "jdbc:mysql://localhost:3306/mydb");
            configManager.setProperty("db.user", "user");
            configManager.setProperty("db.password", "password");
            
            // プロパティを保存
            configManager.save("アプリケーション設定");
            System.out.println("設定を保存しました。");
            
            // すべてのプロパティを表示
            configManager.listProperties();
            
            // 特定のプロパティを取得
            System.out.println("\n特定のプロパティ値:");
            System.out.println("アプリ名: " + configManager.getProperty("app.name"));
            System.out.println("データベースURL: " + configManager.getProperty("db.url"));
            System.out.println("存在しないプロパティ: " + 
                              configManager.getProperty("nonexistent", "デフォルト値"));
            
        } catch (IOException e) {
            System.err.println("設定ファイル操作中にエラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### 解説
この解答例では、以下の点に注目してください：

1. **Propertiesクラスの使用**：
   - Javaの標準APIである`Properties`クラスを使用して、プロパティの管理を行っています。
   - キーと値のペアを簡単に扱うことができます。

2. **ファイル入出力**：
   - `FileInputStream`と`FileOutputStream`を使用して、プロパティファイルの読み書きを行っています。
   - try-with-resourcesを使用して、リソースの解放を確実に行っています。

3. **プロパティの操作**：
   - `getProperty`メソッドと`setProperty`メソッドを使用して、プロパティの取得と設定を行っています。
   - デフォルト値を指定できるオーバーロードメソッドも提供しています。

4. **プロパティの一覧表示**：
   - `stringPropertyNames`メソッドを使用して、すべてのプロパティキーを取得しています。
   - 各キーに対応する値を表示しています。

5. **例外処理**：
   - `IOException`や`FileNotFoundException`をキャッチして適切なエラーメッセージを表示しています。
   - 既存のファイルがない場合は新規作成するなど、柔軟な対応を行っています。
