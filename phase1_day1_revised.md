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

![Eclipseプロジェクト作成](https://example.com/eclipse_project_creation.png)

## 基本概念の解説

### 1. Javaの入出力ストリーム

Javaの入出力処理は「ストリーム」という概念に基づいています。ストリームとは、データの流れを表す抽象的な概念で、入力ストリーム（InputStream）と出力ストリーム（OutputStream）の2種類があります。

<div class="term-box">
<strong>用語解説: ストリーム（Stream）</strong><br>
データの連続的な流れを表す抽象概念です。入力ストリームはプログラムにデータを読み込む流れ、出力ストリームはプログラムからデータを書き出す流れを表します。Javaの入出力APIでは、様々な種類のストリームクラスが提供されています。
</div>

#### 入出力ストリームの種類

Javaの入出力ストリームは大きく分けて以下の4種類があります：

1. **バイトストリーム**：バイナリデータを扱うためのストリーム
   - 入力：`InputStream`（抽象クラス）とそのサブクラス
   - 出力：`OutputStream`（抽象クラス）とそのサブクラス

2. **文字ストリーム**：テキストデータを扱うためのストリーム
   - 入力：`Reader`（抽象クラス）とそのサブクラス
   - 出力：`Writer`（抽象クラス）とそのサブクラス

![Javaストリーム階層](https://example.com/java_stream_hierarchy.png)

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

<div class="term-box">
<strong>用語解説: try-with-resources</strong><br>
Java 7から導入された構文で、リソース（ここではBufferedReader）を自動的にクローズするための仕組みです。try文の括弧内でリソースを宣言すると、tryブロックの終了時に自動的にclose()メソッドが呼び出されます。
</div>

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

<div class="term-box">
<strong>用語解説: バッファリング</strong><br>
データを一時的に蓄えてから一括して処理する技術です。ディスクやネットワークなどの遅いデバイスへのアクセス回数を減らし、処理効率を向上させます。
</div>

バッファリングを使用すると、入出力処理の効率が大幅に向上します。例えば、`FileReader`/`FileWriter`だけを使うよりも、`BufferedReader`/`BufferedWriter`を組み合わせて使うほうが効率的です。

#### バッファリングの効果（計算量の観点から）

<div class="term-box">
<strong>用語解説: オーダー記法（Big O表記）</strong><br>
アルゴリズムの実行時間や必要なスペースが入力サイズによってどのように変化するかを表す記法です。例えば、O(1)は定数時間、O(n)は入力サイズに比例する時間を意味します。
</div>

バッファなしの場合、1バイトごとにディスクアクセスが発生するため、n文字の処理にO(n)回のディスクアクセスが必要です。一方、バッファありの場合、バッファサイズを b とすると、O(n/b)回のディスクアクセスで済みます。

例えば、1MBのファイルを1バイトずつ読み込むと1,048,576回のディスクアクセスが必要ですが、8KBのバッファを使用すると約128回のディスクアクセスで済みます。

### 4. Java NIOの基本

Java NIO（New I/O）は、Java 1.4から導入された高性能な入出力APIです。従来のIOと比較して、以下の特徴があります：

- ノンブロッキングI/O
- バッファ指向
- チャネルの概念
- セレクタの概念

<div class="term-box">
<strong>用語解説: ノンブロッキングI/O</strong><br>
従来のI/O（ブロッキングI/O）では、読み書きの操作中にスレッドが待機状態になりますが、ノンブロッキングI/Oでは、データの準備ができていなくても処理を続行できます。これにより、1つのスレッドで複数の入出力操作を効率的に処理できます。
</div>

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

<div class="term-box">
<strong>用語解説: 文字エンコーディング</strong><br>
文字をバイト列に変換する方式です。代表的なものにUTF-8、UTF-16、Shift-JIS、ISO-8859-1などがあります。異なるエンコーディング間で変換を行うと文字化けの原因になります。
</div>

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

![Eclipseクラス作成](https://example.com/eclipse_class_creation.png)

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

![Eclipseデバッグ](https://example.com/eclipse_debug.png)

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
            configManager.setProperty("app.theme", "dark");
            
            // プロパティを保存
            configManager.save("Application Configuration");
            System.out.println("設定を保存しました。");
            
            // プロパティを表示
            System.out.println("\n現在の設定:");
            configManager.listProperties();
            
        } catch (IOException e) {
            System.err.println("エラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

#### 実装ヒント
1. `Properties`クラスを使用して、キーと値のペアを管理します。
2. `load()`メソッドでは、`FileInputStream`と`Properties.load()`メソッドを使用してプロパティファイルを読み込みます。
3. `getProperty()`メソッドでは、`Properties.getProperty()`メソッドを使用してプロパティ値を取得します。
4. `setProperty()`メソッドでは、`Properties.setProperty()`メソッドを使用してプロパティ値を設定します。
5. `save()`メソッドでは、`FileOutputStream`と`Properties.store()`メソッドを使用してプロパティファイルに保存します。
6. `listProperties()`メソッドでは、`Properties.stringPropertyNames()`メソッドを使用してキーの一覧を取得し、各キーに対応する値を表示します。

## 解答例

### 演習1: 基本的なファイル入出力の解答例

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
        try (BufferedReader reader = new BufferedReader(new FileReader(inputFile));
             BufferedWriter writer = new BufferedWriter(new FileWriter(outputFile))) {
            
            String line;
            int lineNumber = 1;
            
            while ((line = reader.readLine()) != null) {
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

### 演習2: ログファイル解析の解答例

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
        Map<String, Integer> statusCounts = new HashMap<>();
        Map<String, Integer> ipCounts = new HashMap<>();
        int totalAccess = 0;
        
        try (BufferedReader reader = new BufferedReader(new FileReader(logFile))) {
            String line;
            
            while ((line = reader.readLine()) != null) {
                Matcher matcher = LOG_PATTERN.matcher(line);
                
                if (matcher.find()) {
                    String ip = matcher.group(1);
                    String statusCode = matcher.group(4);
                    
                    // 合計アクセス数をカウント
                    totalAccess++;
                    
                    // ステータスコード別のカウント
                    statusCounts.put(statusCode, statusCounts.getOrDefault(statusCode, 0) + 1);
                    
                    // IPアドレス別のカウント
                    ipCounts.put(ip, ipCounts.getOrDefault(ip, 0) + 1);
                }
            }
        }
        
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
        System.out.println("===== ログ解析結果 =====");
        System.out.println("合計アクセス数: " + totalAccess);
        
        System.out.println("\n--- ステータスコード別アクセス数 ---");
        for (Map.Entry<String, Integer> entry : statusCounts.entrySet()) {
            System.out.printf("ステータスコード %s: %d回 (%.1f%%)\n", 
                    entry.getKey(), entry.getValue(), 
                    (double) entry.getValue() / totalAccess * 100);
        }
        
        System.out.println("\n--- IPアドレス別アクセス数 ---");
        for (Map.Entry<String, Integer> entry : ipCounts.entrySet()) {
            System.out.printf("IP %s: %d回 (%.1f%%)\n", 
                    entry.getKey(), entry.getValue(), 
                    (double) entry.getValue() / totalAccess * 100);
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

### 演習3: CSVファイル処理の解答例

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
                    // データ行は条件に合致するもののみ出力
                    String[] fields = line.split(",");
                    if (fields.length >= 3) {
                        try {
                            int age = Integer.parseInt(fields[2]);
                            if (age >= 30) {
                                writer.write(line);
                                writer.newLine();
                            }
                        } catch (NumberFormatException e) {
                            System.err.println("年齢の解析に失敗しました: " + fields[2]);
                        }
                    }
                }
            }
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

## 解説と補足

### 入出力処理の効率化

入出力処理は、プログラムの実行速度に大きな影響を与える要素の一つです。特にディスクI/Oは、メモリアクセスと比較して非常に遅いため、効率的な入出力処理を実装することが重要です。

#### バッファリングの重要性

バッファリングを使用することで、ディスクアクセスの回数を減らし、入出力処理の効率を大幅に向上させることができます。例えば、1MBのファイルを1バイトずつ読み書きする場合：

- バッファなし: 1,048,576回のディスクアクセス
- 8KBバッファ: 約128回のディスクアクセス

この差は、ファイルサイズが大きくなるほど顕著になります。

#### 適切なバッファサイズの選択

バッファサイズは大きければ良いというわけではありません。適切なバッファサイズは、以下の要素によって異なります：

- メモリ使用量の制約
- ファイルサイズ
- アクセスパターン（シーケンシャルかランダムか）
- ハードウェアの特性

一般的には、8KB〜64KBのバッファサイズが多くの場合に適しています。

### 文字エンコーディングの重要性

テキストファイルを扱う際は、適切な文字エンコーディングを指定することが重要です。特に日本語などの非ASCII文字を含むファイルを扱う場合は注意が必要です。

#### 主な文字エンコーディング

- **UTF-8**: 最も広く使われている国際的な文字エンコーディング。1〜4バイトで文字を表現。
- **UTF-16**: 2〜4バイトで文字を表現。Javaの内部表現はUTF-16。
- **Shift-JIS**: 日本語Windows環境でよく使われる。
- **EUC-JP**: UNIXシステムでよく使われる日本語エンコーディング。
- **ISO-8859-1**: 西ヨーロッパ言語用のエンコーディング。

#### エンコーディング指定の方法

```java
// 読み込み時にエンコーディングを指定
BufferedReader reader = new BufferedReader(
    new InputStreamReader(new FileInputStream("input.txt"), StandardCharsets.UTF_8));

// 書き込み時にエンコーディングを指定
BufferedWriter writer = new BufferedWriter(
    new OutputStreamWriter(new FileOutputStream("output.txt"), StandardCharsets.UTF_8));
```

### Java IOとNIOの使い分け

Java IOとNIOは、それぞれ異なる特性を持っており、用途に応じて使い分けることが重要です。

#### Java IO（旧API）の特徴

- シンプルで使いやすい
- ブロッキングI/O
- ストリーム指向
- 単一スレッドでの使用に適している

#### Java NIO（新API）の特徴

- より複雑だが柔軟性が高い
- ノンブロッキングI/Oをサポート
- バッファとチャネル指向
- セレクタを使用した多重I/O
- 高負荷のサーバーアプリケーションに適している

#### 使い分けの指針

- 単純なファイル操作: Java IO
- 高性能が必要な場合: Java NIO
- ノンブロッキングI/Oが必要な場合: Java NIO
- 多数の接続を処理するサーバー: Java NIO

### 正規表現の活用

ログファイルの解析など、テキスト処理では正規表現が非常に役立ちます。Javaでは`java.util.regex`パッケージを使用して正規表現を扱うことができます。

#### 主な正規表現のパターン

- `\d`: 数字1文字
- `\w`: 英数字またはアンダースコア1文字
- `\s`: 空白文字1文字
- `+`: 直前のパターンの1回以上の繰り返し
- `*`: 直前のパターンの0回以上の繰り返し
- `?`: 直前のパターンの0回または1回の出現
- `()`: グループ化
- `[]`: 文字クラス（いずれか1文字にマッチ）
- `^`: 行の先頭
- `$`: 行の末尾

#### 正規表現の使用例

```java
Pattern pattern = Pattern.compile("(\\d+)-(\\d+)-(\\d+)");
Matcher matcher = pattern.matcher("2025-05-28");
if (matcher.matches()) {
    String year = matcher.group(1);   // "2025"
    String month = matcher.group(2);  // "05"
    String day = matcher.group(3);    // "28"
}
```

## 実務での活用シーン

### 1. ログファイル解析

実務では、アプリケーションのログファイルを解析して、エラーの発生状況や性能のボトルネックを特定することがよくあります。例えば：

- Webサーバーのアクセスログを解析して、アクセス数やエラー率を集計
- アプリケーションのエラーログを解析して、頻発するエラーを特定
- パフォーマンスログを解析して、処理時間の長いリクエストを特定

### 2. データ変換処理

異なるシステム間でデータを連携する際に、データ形式の変換処理が必要になることがよくあります。例えば：

- CSVファイルをXMLやJSONに変換
- レガシーシステムのデータを新システム用に変換
- 外部システムからのデータを取り込み、内部形式に変換

### 3. 設定ファイル管理

多くのアプリケーションでは、設定情報を外部ファイルに保存し、実行時に読み込む方式が採用されています。例えば：

- アプリケーションの動作パラメータを設定ファイルから読み込み
- 環境ごと（開発、テスト、本番）に異なる設定を管理
- ユーザー設定を保存・読み込み

### 4. バッチ処理

定期的に実行されるバッチ処理では、大量のデータを効率的に処理する必要があります。例えば：

- 日次の売上データを集計して報告書を生成
- 大量のトランザクションデータを処理して請求書を作成
- システムのバックアップデータを作成

### 5. ファイル転送・同期

システム間でファイルを転送したり、ディレクトリを同期したりする処理も、入出力処理の重要な応用例です。例えば：

- FTPサーバーとのファイル送受信
- クラウドストレージとのファイル同期
- バックアップシステムへのデータ転送

## 参考資料

1. Java公式ドキュメント: [Java I/O, NIO, and NIO.2](https://docs.oracle.com/javase/tutorial/essential/io/index.html)
2. 書籍: 「Effective Java 第3版」（Joshua Bloch著）
3. 書籍: 「Java Performance: The Definitive Guide」（Scott Oaks著）
4. Oracle技術記事: [Java I/O Performance](https://www.oracle.com/technical-resources/articles/javase/perftuning.html)
5. Java正規表現ガイド: [Java Regex Tutorial](https://www.vogella.com/tutorials/JavaRegularExpressions/article.html)
