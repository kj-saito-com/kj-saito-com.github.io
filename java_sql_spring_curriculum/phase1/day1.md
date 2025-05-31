---
layout: default
title: Day 1: 入出力処理の基本と応用
---
# Day 1: 入出力処理の基本と応用

## 学習の目的と背景

入出力処理はあらゆるプログラムの基本となる重要な要素です。ファイルの読み書き、ネットワーク通信、ユーザーとのインタラクションなど、多くの場面で入出力処理が必要となります。特に業務システムでは、ログファイルの出力や分析、CSVやExcelファイルの処理、設定ファイルの読み込みなど、様々な入出力処理が求められます。

本日の学習では、Javaにおける入出力処理の基本から応用までを体系的に学び、実際のコードを書きながら理解を深めていきます。特にファイル入出力の効率的な実装方法や、実務でよく使われるログファイル解析の手法を習得することを目指します。

## 前提知識と準備

### 必要な前提知識
- Javaの基本構文（変数、制御構文、例外処理など）
- クラスとオブジェクトの基本概念
- 例外処理の基本（try-catch文）

### 環境準備
- Eclipse 2023-12 (4.30.0)
- Java 21

### プロジェクト作成手順

1. Eclipseを起動します。
2. 「File」→「New」→「Java Project」を選択します。
3. プロジェクト名を「JavaIO_Day1」と入力します。
4. 「JRE」の設定で「Use an execution environment JRE:」から「JavaSE-21」を選択します。
5. 「Finish」をクリックしてプロジェクトを作成します。

## 基本概念の解説

### 1. Javaの入出力ストリーム

Javaの入出力処理は「ストリーム」という概念に基づいています。ストリームとは、データの流れを表す抽象的な概念で、入力ストリーム（InputStream）と出力ストリーム（OutputStream）の2種類があります。

**用語解説: ストリーム（Stream）**  
データの連続的な流れを表す抽象概念です。入力ストリームはプログラムにデータを読み込む流れ、出力ストリームはプログラムからデータを書き出す流れを表します。Javaの入出力APIでは、様々な種類のストリームクラスが提供されています。

#### 入出力ストリームの種類

Javaの入出力ストリームは大きく分けて以下の4種類があります：

1. **バイトストリーム**：バイナリデータを扱うためのストリーム
   - 入力：`InputStream`（抽象クラス）とそのサブクラス
   - 出力：`OutputStream`（抽象クラス）とそのサブクラス

2. **文字ストリーム**：テキストデータを扱うためのストリーム
   - 入力：`Reader`（抽象クラス）とそのサブクラス
   - 出力：`Writer`（抽象クラス）とそのサブクラス

### 2. ファイル入出力の基本

#### ファイルからの読み込み（基本）

ファイルからテキストを読み込む最も基本的な方法は、`FileReader`と`BufferedReader`を組み合わせる方法です。

```java
try (BufferedReader reader = new BufferedReader(new FileReader("input.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

**用語解説: try-with-resources**  
Java 7から導入された構文で、リソース（ここではBufferedReader）を自動的にクローズするための仕組みです。try文の括弧内でリソースを宣言すると、tryブロックの終了時に自動的にclose()メソッドが呼び出されます。

#### ファイルへの書き込み（基本）

ファイルにテキストを書き込む基本的な方法は、`FileWriter`と`BufferedWriter`を組み合わせる方法です。

```java
try (BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
    writer.write("Hello, Java I/O!");
    writer.newLine();  // 改行を追加
    writer.write("This is a test.");
} catch (IOException e) {
    e.printStackTrace();
}
```

### 3. バッファリングの重要性

**用語解説: バッファリング**  
データを一時的に蓄えてから一括して処理する技術です。ディスクやネットワークなどの遅いデバイスへのアクセス回数を減らし、処理効率を向上させます。

バッファリングを使用すると、入出力処理の効率が大幅に向上します。例えば、`FileReader`/`FileWriter`だけを使うよりも、`BufferedReader`/`BufferedWriter`を組み合わせて使うほうが効率的です。

#### バッファリングの効果（計算量の観点から）

**用語解説: オーダー記法（Big O表記）**  
アルゴリズムの実行時間や必要なスペースが入力サイズによってどのように変化するかを表す記法です。例えば、O(1)は定数時間、O(n)は入力サイズに比例する時間を意味します。

バッファなしの場合、1バイトごとにディスクアクセスが発生するため、n文字の処理にO(n)回のディスクアクセスが必要です。一方、バッファありの場合、バッファサイズを b とすると、O(n/b)回のディスクアクセスで済みます。

例えば、1MBのファイルを1バイトずつ読み込むと1,048,576回のディスクアクセスが必要ですが、8KBのバッファを使用すると約128回のディスクアクセスで済みます。

### 4. Java NIOの基本

Java NIO（New I/O）は、Java 1.4から導入された高性能な入出力APIです。従来のIOと比較して、以下の特徴があります：

- ノンブロッキングI/O
- バッファ指向
- チャネルの概念
- セレクタの概念

**用語解説: ノンブロッキングI/O**  
従来のI/O（ブロッキングI/O）では、読み書きの操作中にスレッドが待機状態になりますが、ノンブロッキングI/Oでは、データの準備ができていなくても処理を続行できます。これにより、1つのスレッドで複数の入出力操作を効率的に処理できます。

#### NIOを使用したファイル読み込み

```java
try {
    Path path = Paths.get("input.txt");
    List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);
    for (String line : lines) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

#### NIOを使用したファイル書き込み

```java
try {
    Path path = Paths.get("output.txt");
    List<String> lines = Arrays.asList("Hello, Java NIO!", "This is a test.");
    Files.write(path, lines, StandardCharsets.UTF_8);
} catch (IOException e) {
    e.printStackTrace();
}
```

### 5. 文字エンコーディング

**用語解説: 文字エンコーディング**  
文字をバイト列に変換する方式です。代表的なものにUTF-8、UTF-16、Shift-JIS、ISO-8859-1などがあります。異なるエンコーディング間で変換を行うと文字化けの原因になります。

ファイルの読み書きを行う際は、適切な文字エンコーディングを指定することが重要です。特に日本語を含むファイルを扱う場合は注意が必要です。

```java
// UTF-8エンコーディングでファイルを読み込む
try (BufferedReader reader = new BufferedReader(
        new InputStreamReader(new FileInputStream("input.txt"), StandardCharsets.UTF_8))) {
    // 処理
}

// UTF-8エンコーディングでファイルに書き込む
try (BufferedWriter writer = new BufferedWriter(
        new OutputStreamWriter(new FileOutputStream("output.txt"), StandardCharsets.UTF_8))) {
    // 処理
}
```

## Eclipse操作ガイド

### クラスの作成

1. プロジェクト「JavaIO_Day1」を右クリックします。
2. 「New」→「Class」を選択します。
3. 「Name」に作成するクラス名（例：`FileIOBasic`）を入力します。
4. 「public static void main(String[] args)」にチェックを入れます。
5. 「Finish」をクリックしてクラスを作成します。

### ファイルの作成と配置

1. プロジェクト「JavaIO_Day1」を右クリックします。
2. 「New」→「File」を選択します。
3. 「File name」に作成するファイル名（例：`sample.log`）を入力します。
4. 「Finish」をクリックしてファイルを作成します。
5. エディタでファイルを開き、内容を入力して保存します。

### プログラムの実行

1. 実行したいクラスのエディタ上で右クリックします。
2. 「Run As」→「Java Application」を選択します。
3. コンソール画面に実行結果が表示されます。

### デバッグの基本

1. ブレークポイントを設定したい行の左端をダブルクリックします（青い丸が表示されます）。
2. クラスのエディタ上で右クリックし、「Debug As」→「Java Application」を選択します。
3. デバッグパースペクティブに切り替わり、ブレークポイントで実行が一時停止します。
4. 「Step Over」（F6）、「Step Into」（F5）、「Step Return」（F7）などのボタンで実行を制御します。
5. 変数の値は「Variables」ビューで確認できます。

## コア演習（必須）

### 演習1: 基本的なファイル入出力（難易度：基本）

#### 課題
テキストファイルを読み込み、各行の先頭に行番号を付けて別のファイルに書き出すプログラムを作成してください。

#### 雛形コード
以下のクラスを作成してください：

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
        // TODO: ここに処理を実装
        // ヒント1: BufferedReaderとBufferedWriterを使用する
        // ヒント2: 行番号は1から始める
        // ヒント3: 行番号の後にはコロンとスペースを入れる（例: "1: この行は..."）
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

#### 実装ヒント
1. `BufferedReader`を使って入力ファイルを1行ずつ読み込みます。
2. `BufferedWriter`を使って出力ファイルに書き込みます。
3. 行番号を管理するための変数を用意し、1行読み込むごとにインクリメントします。
4. try-with-resourcesを使用して、リソースの解放を確実に行います。

#### テスト用入力ファイル（input.txt）
```
これは1行目です。
これは2行目です。
これは3行目です。
日本語も正しく処理できるようにしましょう。
```

### 演習2: ログファイル解析（難易度：応用）

#### 課題
アクセスログファイルを解析し、以下の情報を抽出して表示するプログラムを作成してください：
1. 合計アクセス数
2. ステータスコード別のアクセス数（200, 404, 500など）
3. アクセス元IPアドレスの一覧とそれぞれのアクセス数

#### 雛形コード
以下のクラスを作成してください：

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
        // TODO: ここに処理を実装
        // ヒント1: BufferedReaderを使用してログファイルを1行ずつ読み込む
        // ヒント2: 正規表現パターン(LOG_PATTERN)を使用して各行を解析
        // ヒント3: Map<String, Integer>を使用してステータスコード別、IPアドレス別のカウントを管理
    }
    
    /**
     * 解析結果を表示します。
     * 
     * @param totalAccess 合計アクセス数
     * @param statusCounts ステータスコード別のアクセス数
     * @param ipCounts IPアドレス別のアクセス数
     */
    private void displayResults(int totalAccess, Map<String, Integer> statusCounts, Map<String, Integer> ipCounts) {
        // TODO: ここに処理を実装
        // ヒント: System.out.printlnを使用して結果を整形して表示
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

#### 実装ヒント
1. `BufferedReader`を使ってログファイルを1行ずつ読み込みます。
2. 正規表現パターン`LOG_PATTERN`を使用して、各行からIPアドレスとステータスコードを抽出します。
3. `Map<String, Integer>`を使用して、ステータスコード別とIPアドレス別のアクセス数をカウントします。
4. 解析結果を整形して表示します。

#### テスト用ログファイル（access.log）
```
192.168.1.1 - - [28/May/2025:10:15:32 +0900] "GET /index.html HTTP/1.1" 200 1234
192.168.1.2 - - [28/May/2025:10:16:45 +0900] "GET /about.html HTTP/1.1" 200 2345
192.168.1.1 - - [28/May/2025:10:17:21 +0900] "GET /contact.html HTTP/1.1" 200 3456
192.168.1.3 - - [28/May/2025:10:18:10 +0900] "GET /nonexistent.html HTTP/1.1" 404 567
192.168.1.2 - - [28/May/2025:10:19:05 +0900] "GET /index.html HTTP/1.1" 200 1234
192.168.1.4 - - [28/May/2025:10:20:30 +0900] "GET /error.html HTTP/1.1" 500 789
192.168.1.1 - - [28/May/2025:10:21:15 +0900] "GET /products.html HTTP/1.1" 200 4567
192.168.1.3 - - [28/May/2025:10:22:40 +0900] "GET /about.html HTTP/1.1" 200 2345
192.168.1.4 - - [28/May/2025:10:23:55 +0900] "GET /nonexistent.html HTTP/1.1" 404 567
192.168.1.2 - - [28/May/2025:10:25:12 +0900] "GET /contact.html HTTP/1.1" 200 3456
```

### 演習3: CSVファイル処理（難易度：応用）

#### 課題
CSVファイルを読み込み、特定の条件に合致するレコードを抽出して別のCSVファイルに書き出すプログラムを作成してください。

#### 雛形コード
以下のクラスを作成してください：

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
        // TODO: ここに処理を実装
        // ヒント1: BufferedReaderとBufferedWriterを使用する
        // ヒント2: String.split(",")を使用してCSVの各フィールドを分割
        // ヒント3: 1行目はヘッダーとして扱い、そのまま出力する
        // ヒント4: 2行目以降は年齢（3番目のフィールド）が30以上のレコードのみ出力
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

#### 実装ヒント
1. `BufferedReader`を使って入力CSVファイルを1行ずつ読み込みます。
2. 1行目はヘッダーとして扱い、そのまま出力します。
3. 2行目以降は、`String.split(",")`を使用して各フィールドに分割します。
4. 年齢（3番目のフィールド）が30以上のレコードのみを`BufferedWriter`を使って出力CSVファイルに書き出します。
5. try-with-resourcesを使用して、リソースの解放を確実に行います。

#### テスト用CSVファイル（users.csv）
```
id,name,age,email
1,山田太郎,35,yamada@example.com
2,佐藤花子,28,sato@example.com
3,鈴木一郎,42,suzuki@example.com
4,田中美咲,25,tanaka@example.com
5,伊藤健太,31,ito@example.com
6,渡辺裕子,39,watanabe@example.com
7,小林誠,27,kobayashi@example.com
8,加藤洋平,33,kato@example.com
9,松本さくら,24,matsumoto@example.com
10,井上大輔,45,inoue@example.com
```

## 発展演習（オプション）

### 発展演習1: バイナリファイルのコピー（難易度：発展）

#### 課題
バイナリファイル（画像やPDFなど）を効率的にコピーするプログラムを作成してください。バッファリングを使用して、大きなファイルでも効率よくコピーできるようにしてください。

#### 雛形コード
以下のクラスを作成してください：

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
        // TODO: ここに処理を実装
        // ヒント1: FileInputStreamとFileOutputStreamを使用する
        // ヒント2: BufferedInputStreamとBufferedOutputStreamでラップする
        // ヒント3: 指定されたバッファサイズの byte[] を用意し、read/writeを繰り返す
        // ヒント4: コピーしたバイト数を返す
        return 0; // 仮の戻り値
    }
    
    /**
     * 異なるバッファサイズでのコピー性能を比較します。
     * 
     * @param sourceFile コピー元ファイルのパス
     * @throws IOException 入出力エラーが発生した場合
     */
    public void compareBufferSizes(String sourceFile) throws IOException {
        // TODO: ここに処理を実装
        // ヒント1: 異なるバッファサイズ（例: 1KB, 8KB, 64KB, 1MB）でコピーを実行
        // ヒント2: 各コピーの実行時間を計測して表示
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

#### 実装ヒント
1. `FileInputStream`と`FileOutputStream`を使用してバイナリファイルを読み書きします。
2. `BufferedInputStream`と`BufferedOutputStream`でラップして、バッファリングを有効にします。
3. 指定されたバッファサイズの`byte[]`配列を用意し、`read`/`write`メソッドを使って一定量ずつデータを転送します。
4. 異なるバッファサイズでのコピー性能を比較するために、`System.currentTimeMillis()`などを使って実行時間を計測します。

### 発展演習2: プロパティファイルの操作（難易度：発展）

#### 課題
アプリケーションの設定を管理するためのプロパティファイル操作クラスを作成してください。プロパティの読み込み、更新、保存ができるようにしてください。

#### 雛形コード
以下のクラスを作成してください：

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
        // TODO: ここに処理を実装
        // ヒント: FileInputStreamとPropertiesクラスのloadメソッドを使用
    }
    
    /**
     * プロパティ値を取得します。
     * 
     * @param key プロパティのキー
     * @return プロパティの値（キーが存在しない場合はnull）
     */
    public String getProperty(String key) {
        // TODO: ここに処理を実装
        // ヒント: PropertiesクラスのgetPropertyメソッドを使用
        return null; // 仮の戻り値
    }
    
    /**
     * プロパティ値を取得します。キーが存在しない場合はデフォルト値を返します。
     * 
     * @param key プロパティのキー
     * @param defaultValue デフォルト値
     * @return プロパティの値（キーが存在しない場合はデフォルト値）
     */
    public String getProperty(String key, String defaultValue) {
        // TODO: ここに処理を実装
        // ヒント: PropertiesクラスのgetPropertyメソッドを使用
        return null; // 仮の戻り値
    }
    
    /**
     * プロパティ値を設定します。
     * 
     * @param key プロパティのキー
     * @param value プロパティの値
     */
    public void setProperty(String key, String value) {
        // TODO: ここに処理を実装
        // ヒント: PropertiesクラスのsetPropertyメソッドを使用
    }
    
    /**
     * プロパティファイルに変更を保存します。
     * 
     * @param comments コメント（ファイルの先頭に追加される）
     * @throws IOException 入出力エラーが発生した場合
     */
    public void save(String comments) throws IOException {
        // TODO: ここに処理を実装
        // ヒント: FileOutputStreamとPropertiesクラスのstoreメソッドを使用
    }
    
    /**
     * すべてのプロパティを表示します。
     */
    public void listProperties() {
        // TODO: ここに処理を実装
        // ヒント: PropertiesクラスのstringPropertyNamesメソッドを使用してキーの一覧を取得し、
        // 各キーに対応する値を表示
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
            System.out.println("アプリケーション名: " + configManager.getProperty("app.name"));
            System.out.println("データベースURL: " + configManager.getProperty("db.url"));
            System.out.println("存在しないプロパティ: " + configManager.getProperty("nonexistent", "デフォルト値"));
        } catch (IOException e) {
            System.err.println("エラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

#### 実装ヒント
1. `Properties`クラスを使用してプロパティを管理します。
2. `load()`メソッドでは、`FileInputStream`と`Properties.load()`メソッドを使用してプロパティファイルを読み込みます。
3. `getProperty()`メソッドでは、`Properties.getProperty()`メソッドを使用してプロパティ値を取得します。
4. `setProperty()`メソッドでは、`Properties.setProperty()`メソッドを使用してプロパティ値を設定します。
5. `save()`メソッドでは、`FileOutputStream`と`Properties.store()`メソッドを使用してプロパティファイルに保存します。
6. `listProperties()`メソッドでは、`Properties.stringPropertyNames()`メソッドを使用してキーの一覧を取得し、各キーに対応する値を表示します。

## 解説と補足

### 入出力処理の効率化

ファイル入出力処理を効率化するためには、以下の点に注意することが重要です：

1. **適切なバッファサイズの選択**：
   - 小さすぎるバッファサイズはディスクアクセスの回数が増えて非効率
   - 大きすぎるバッファサイズはメモリ消費が増大
   - 一般的には8KB〜64KBが適切とされる

2. **リソースの適切な解放**：
   - try-with-resourcesを使用して確実にクローズする
   - 明示的にクローズしない場合、ファイルハンドルのリークが発生する可能性がある

3. **適切なストリームの選択**：
   - テキストデータには文字ストリーム（Reader/Writer）
   - バイナリデータにはバイトストリーム（InputStream/OutputStream）
   - 大量のデータを一度に処理する場合はNIOの`Files`クラスのメソッド

4. **文字エンコーディングの明示的な指定**：
   - プラットフォームのデフォルトエンコーディングに依存しない
   - 国際化対応のためにUTF-8を使用することが推奨される

### 実務での入出力処理の例

実務では、以下のような入出力処理がよく使われます：

1. **ログファイルの解析**：
   - アクセスログやエラーログを解析して統計情報を抽出
   - 特定のパターンに一致するログエントリを検索

2. **データの変換**：
   - CSVファイルからデータベースへのインポート
   - データベースからCSVファイルへのエクスポート
   - あるフォーマットから別のフォーマットへの変換

3. **設定ファイルの管理**：
   - アプリケーションの設定をプロパティファイルやXMLファイルで管理
   - 実行時に設定を読み込み、必要に応じて更新

4. **一時ファイルの作成と管理**：
   - 処理の中間結果を一時ファイルに保存
   - 処理完了後に一時ファイルを削除

5. **大容量ファイルの処理**：
   - 一度にメモリに読み込めない大きなファイルを分割して処理
   - ストリーム処理によるメモリ効率の向上

### パフォーマンスに関する考察

入出力処理のパフォーマンスは、アプリケーション全体のパフォーマンスに大きな影響を与えます。以下の点に注意することで、パフォーマンスを向上させることができます：

1. **ディスクアクセスの最小化**：
   - バッファリングの活用
   - 必要なデータのみを読み込む（全ファイルを読み込まない）
   - 書き込みの場合は、小さな書き込みを集約して一度に書き込む

2. **適切なデータ構造の選択**：
   - 大量のデータを処理する場合は、メモリ効率の良いデータ構造を選択
   - 検索が頻繁に行われる場合は、ハッシュマップなどの効率的な検索が可能なデータ構造を使用

3. **並列処理の活用**：
   - 複数のファイルを同時に処理する場合は、並列処理を検討
   - ただし、同一ファイルへの並列アクセスには注意が必要

4. **NIOの活用**：
   - チャネルとバッファを使用した効率的なI/O
   - メモリマップドファイルによる大容量ファイルの効率的な処理

## 実務での活用シーン

### 1. ログ解析システム

Webサーバーやアプリケーションサーバーのログファイルを解析して、アクセス統計やエラー分析を行うシステムを開発する場合、効率的なファイル入出力処理が必要です。

```java
// 日次ログファイルを解析して統計情報を生成
public void generateDailyStatistics(String logFile, String outputFile) throws IOException {
    Map<String, Integer> urlCounts = new HashMap<>();
    Map<String, Integer> errorCounts = new HashMap<>();
    
    try (BufferedReader reader = new BufferedReader(new FileReader(logFile))) {
        String line;
        while ((line = reader.readLine()) != null) {
            // ログエントリを解析
            LogEntry entry = LogParser.parse(line);
            
            // URLごとのアクセス数をカウント
            String url = entry.getUrl();
            urlCounts.put(url, urlCounts.getOrDefault(url, 0) + 1);
            
            // エラーの場合はエラーコードごとにカウント
            if (entry.getStatusCode() >= 400) {
                String errorCode = String.valueOf(entry.getStatusCode());
                errorCounts.put(errorCode, errorCounts.getOrDefault(errorCode, 0) + 1);
            }
        }
    }
    
    // 統計情報をファイルに出力
    try (BufferedWriter writer = new BufferedWriter(new FileWriter(outputFile))) {
        writer.write("URL別アクセス数:\n");
        for (Map.Entry<String, Integer> entry : urlCounts.entrySet()) {
            writer.write(entry.getKey() + ": " + entry.getValue() + "\n");
        }
        
        writer.write("\nエラーコード別件数:\n");
        for (Map.Entry<String, Integer> entry : errorCounts.entrySet()) {
            writer.write(entry.getKey() + ": " + entry.getValue() + "\n");
        }
    }
}
```

### 2. データ変換ツール

異なるシステム間でデータを連携する際に、あるフォーマットから別のフォーマットにデータを変換するツールを開発する場合、効率的なファイル入出力処理が必要です。

```java
// CSVファイルをJSONファイルに変換
public void convertCsvToJson(String csvFile, String jsonFile) throws IOException {
    List<Map<String, String>> records = new ArrayList<>();
    
    try (BufferedReader reader = new BufferedReader(new FileReader(csvFile))) {
        String line = reader.readLine();
        if (line == null) {
            return; // 空のファイル
        }
        
        // ヘッダー行を解析
        String[] headers = line.split(",");
        
        // データ行を解析
        while ((line = reader.readLine()) != null) {
            String[] values = line.split(",");
            Map<String, String> record = new HashMap<>();
            
            for (int i = 0; i < headers.length && i < values.length; i++) {
                record.put(headers[i], values[i]);
            }
            
            records.add(record);
        }
    }
    
    // JSONファイルに出力
    try (BufferedWriter writer = new BufferedWriter(new FileWriter(jsonFile))) {
        writer.write("[\n");
        
        for (int i = 0; i < records.size(); i++) {
            writer.write("  {\n");
            
            Map<String, String> record = records.get(i);
            int j = 0;
            for (Map.Entry<String, String> entry : record.entrySet()) {
                writer.write("    \"" + entry.getKey() + "\": \"" + entry.getValue() + "\"");
                if (j < record.size() - 1) {
                    writer.write(",");
                }
                writer.write("\n");
                j++;
            }
            
            writer.write("  }");
            if (i < records.size() - 1) {
                writer.write(",");
            }
            writer.write("\n");
        }
        
        writer.write("]\n");
    }
}
```

### 3. 設定管理システム

アプリケーションの設定を管理するシステムを開発する場合、プロパティファイルやXMLファイルを読み書きする効率的な入出力処理が必要です。

```java
// 複数環境（開発、テスト、本番）の設定を管理
public class EnvironmentConfigManager {
    private Map<String, Properties> envConfigs = new HashMap<>();
    private String configDir;
    
    public EnvironmentConfigManager(String configDir) {
        this.configDir = configDir;
    }
    
    // 全環境の設定を読み込む
    public void loadAllEnvironments() throws IOException {
        String[] environments = {"dev", "test", "prod"};
        
        for (String env : environments) {
            Properties props = new Properties();
            String configFile = configDir + "/" + env + ".properties";
            
            try (FileInputStream fis = new FileInputStream(configFile)) {
                props.load(fis);
                envConfigs.put(env, props);
                System.out.println(env + "環境の設定を読み込みました。");
            } catch (FileNotFoundException e) {
                System.out.println(env + "環境の設定ファイルが見つかりません。");
            }
        }
    }
    
    // 特定の環境の設定を取得
    public Properties getEnvironmentConfig(String environment) {
        return envConfigs.get(environment);
    }
    
    // 全環境の特定の設定値を比較
    public void comparePropertyAcrossEnvironments(String propertyKey) {
        System.out.println("プロパティ「" + propertyKey + "」の環境別設定値:");
        
        for (Map.Entry<String, Properties> entry : envConfigs.entrySet()) {
            String env = entry.getKey();
            Properties props = entry.getValue();
            String value = props.getProperty(propertyKey, "未設定");
            
            System.out.println(env + ": " + value);
        }
    }
    
    // 特定の環境の設定を更新して保存
    public void updateAndSaveEnvironmentConfig(String environment, String key, String value) throws IOException {
        Properties props = envConfigs.get(environment);
        if (props == null) {
            System.out.println(environment + "環境の設定が読み込まれていません。");
            return;
        }
        
        props.setProperty(key, value);
        
        String configFile = configDir + "/" + environment + ".properties";
        try (FileOutputStream fos = new FileOutputStream(configFile)) {
            props.store(fos, environment + "環境の設定 - 更新日時: " + new Date());
            System.out.println(environment + "環境の設定を更新しました。");
        }
    }
}
```

これらの実務での活用シーンを参考に、入出力処理の重要性と効率的な実装方法を理解してください。
