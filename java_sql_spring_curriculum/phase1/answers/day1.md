# Day 1: 入出力処理の基本と応用 - 解答例

## コア演習1: テキストファイル読み込みと処理

### 解答例
```java
package io.basic;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;

public class TextFileProcessor {

    /**
     * テキストファイルを読み込み、各行を処理します。
     * 
     * @param filePath 読み込むファイルのパス
     * @return 処理結果の文字列リスト
     * @throws IOException ファイル読み込みエラーが発生した場合
     */
    public List<String> processFile(String filePath) throws IOException {
        List<String> results = new ArrayList<>();
        
        // try-with-resourcesを使用してリソースを自動的にクローズ
        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            String line;
            int lineNumber = 1;
            
            // ファイルの各行を読み込む
            while ((line = reader.readLine()) != null) {
                // 行が空でない場合のみ処理
                if (!line.trim().isEmpty()) {
                    // 行番号と行の長さを含む形式で結果を追加
                    String result = String.format("Line %d: %s (Length: %d)", 
                                                 lineNumber, line, line.length());
                    results.add(result);
                }
                lineNumber++;
            }
        }
        
        return results;
    }

    /**
     * Java NIO.2 APIを使用してテキストファイルを読み込み、各行を処理します。
     * 
     * @param filePath 読み込むファイルのパス
     * @return 処理結果の文字列リスト
     * @throws IOException ファイル読み込みエラーが発生した場合
     */
    public List<String> processFileWithNIO(String filePath) throws IOException {
        List<String> results = new ArrayList<>();
        Path path = Paths.get(filePath);
        
        // Files.readAllLinesを使用して全行を一度に読み込む
        List<String> lines = Files.readAllLines(path);
        
        for (int i = 0; i < lines.size(); i++) {
            String line = lines.get(i);
            // 行が空でない場合のみ処理
            if (!line.trim().isEmpty()) {
                // 行番号と行の長さを含む形式で結果を追加
                String result = String.format("Line %d: %s (Length: %d)", 
                                             i + 1, line, line.length());
                results.add(result);
            }
        }
        
        return results;
    }

    public static void main(String[] args) {
        TextFileProcessor processor = new TextFileProcessor();
        String filePath = "sample.txt";  // 実際のファイルパスに変更してください
        
        try {
            // ファイルが存在しない場合は作成
            Path path = Paths.get(filePath);
            if (!Files.exists(path)) {
                List<String> sampleLines = List.of(
                    "これはサンプルテキストファイルです。",
                    "2行目のテキストです。",
                    "",  // 空行
                    "4行目は少し長めのテキストで、様々な処理の例を示すために使用します。",
                    "最後の行です。"
                );
                Files.write(path, sampleLines);
                System.out.println("サンプルファイルを作成しました: " + filePath);
            }
            
            System.out.println("従来のI/O APIを使用した処理結果:");
            List<String> results = processor.processFile(filePath);
            for (String result : results) {
                System.out.println(result);
            }
            
            System.out.println("\nNIO.2 APIを使用した処理結果:");
            List<String> nioResults = processor.processFileWithNIO(filePath);
            for (String result : nioResults) {
                System.out.println(result);
            }
            
        } catch (IOException e) {
            System.err.println("ファイル処理中にエラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

## コア演習2: バイナリファイルのコピーと処理

### 解答例
```java
package io.binary;

import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class BinaryFileProcessor {

    /**
     * バイナリファイルをコピーし、コピー前後のファイルサイズとMD5ハッシュを比較します。
     * 
     * @param sourcePath コピー元ファイルのパス
     * @param targetPath コピー先ファイルのパス
     * @param bufferSize バッファサイズ（バイト単位）
     * @return コピーの結果（成功/失敗）と詳細情報を含む文字列
     */
    public String copyAndVerifyFile(String sourcePath, String targetPath, int bufferSize) {
        StringBuilder result = new StringBuilder();
        
        try {
            File sourceFile = new File(sourcePath);
            File targetFile = new File(targetPath);
            
            // コピー元ファイルの存在確認
            if (!sourceFile.exists()) {
                return "エラー: コピー元ファイルが存在しません: " + sourcePath;
            }
            
            // コピー元ファイルのサイズとMD5ハッシュを取得
            long sourceSize = sourceFile.length();
            String sourceMD5 = calculateMD5(sourcePath);
            
            result.append("コピー元ファイル: ").append(sourcePath).append("\n");
            result.append("サイズ: ").append(sourceSize).append(" バイト\n");
            result.append("MD5ハッシュ: ").append(sourceMD5).append("\n\n");
            
            // コピー先ディレクトリが存在しない場合は作成
            File targetDir = targetFile.getParentFile();
            if (targetDir != null && !targetDir.exists()) {
                targetDir.mkdirs();
            }
            
            // ファイルコピーの開始時間
            long startTime = System.currentTimeMillis();
            
            // バッファリングストリームを使用してファイルをコピー
            try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(sourceFile));
                 BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(targetFile))) {
                
                byte[] buffer = new byte[bufferSize];
                int bytesRead;
                
                while ((bytesRead = bis.read(buffer)) != -1) {
                    bos.write(buffer, 0, bytesRead);
                }
                
                // バッファ内のデータを確実に書き出す
                bos.flush();
            }
            
            // コピーの終了時間と所要時間
            long endTime = System.currentTimeMillis();
            long duration = endTime - startTime;
            
            // コピー先ファイルのサイズとMD5ハッシュを取得
            long targetSize = targetFile.length();
            String targetMD5 = calculateMD5(targetPath);
            
            result.append("コピー先ファイル: ").append(targetPath).append("\n");
            result.append("サイズ: ").append(targetSize).append(" バイト\n");
            result.append("MD5ハッシュ: ").append(targetMD5).append("\n\n");
            
            // コピー結果の検証
            boolean sizeMatch = (sourceSize == targetSize);
            boolean md5Match = sourceMD5.equals(targetMD5);
            
            result.append("コピー所要時間: ").append(duration).append(" ミリ秒\n");
            result.append("サイズ一致: ").append(sizeMatch ? "はい" : "いいえ").append("\n");
            result.append("MD5ハッシュ一致: ").append(md5Match ? "はい" : "いいえ").append("\n\n");
            
            if (sizeMatch && md5Match) {
                result.append("結果: コピー成功（ファイルの整合性が確認されました）");
            } else {
                result.append("結果: コピー失敗（ファイルの整合性が確認できませんでした）");
            }
            
        } catch (IOException | NoSuchAlgorithmException e) {
            result.append("エラー: ファイル処理中に例外が発生しました: ").append(e.getMessage());
            e.printStackTrace();
        }
        
        return result.toString();
    }
    
    /**
     * Java NIO.2 APIを使用してバイナリファイルをコピーし、コピー前後のファイルサイズとMD5ハッシュを比較します。
     * 
     * @param sourcePath コピー元ファイルのパス
     * @param targetPath コピー先ファイルのパス
     * @return コピーの結果（成功/失敗）と詳細情報を含む文字列
     */
    public String copyAndVerifyFileWithNIO(String sourcePath, String targetPath) {
        StringBuilder result = new StringBuilder();
        
        try {
            Path source = Paths.get(sourcePath);
            Path target = Paths.get(targetPath);
            
            // コピー元ファイルの存在確認
            if (!Files.exists(source)) {
                return "エラー: コピー元ファイルが存在しません: " + sourcePath;
            }
            
            // コピー元ファイルのサイズとMD5ハッシュを取得
            long sourceSize = Files.size(source);
            String sourceMD5 = calculateMD5(sourcePath);
            
            result.append("コピー元ファイル: ").append(sourcePath).append("\n");
            result.append("サイズ: ").append(sourceSize).append(" バイト\n");
            result.append("MD5ハッシュ: ").append(sourceMD5).append("\n\n");
            
            // コピー先ディレクトリが存在しない場合は作成
            Path targetDir = target.getParent();
            if (targetDir != null && !Files.exists(targetDir)) {
                Files.createDirectories(targetDir);
            }
            
            // ファイルコピーの開始時間
            long startTime = System.currentTimeMillis();
            
            // NIO.2 APIを使用してファイルをコピー
            Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
            
            // コピーの終了時間と所要時間
            long endTime = System.currentTimeMillis();
            long duration = endTime - startTime;
            
            // コピー先ファイルのサイズとMD5ハッシュを取得
            long targetSize = Files.size(target);
            String targetMD5 = calculateMD5(targetPath);
            
            result.append("コピー先ファイル: ").append(targetPath).append("\n");
            result.append("サイズ: ").append(targetSize).append(" バイト\n");
            result.append("MD5ハッシュ: ").append(targetMD5).append("\n\n");
            
            // コピー結果の検証
            boolean sizeMatch = (sourceSize == targetSize);
            boolean md5Match = sourceMD5.equals(targetMD5);
            
            result.append("コピー所要時間: ").append(duration).append(" ミリ秒\n");
            result.append("サイズ一致: ").append(sizeMatch ? "はい" : "いいえ").append("\n");
            result.append("MD5ハッシュ一致: ").append(md5Match ? "はい" : "いいえ").append("\n\n");
            
            if (sizeMatch && md5Match) {
                result.append("結果: コピー成功（ファイルの整合性が確認されました）");
            } else {
                result.append("結果: コピー失敗（ファイルの整合性が確認できませんでした）");
            }
            
        } catch (IOException | NoSuchAlgorithmException e) {
            result.append("エラー: ファイル処理中に例外が発生しました: ").append(e.getMessage());
            e.printStackTrace();
        }
        
        return result.toString();
    }
    
    /**
     * ファイルのMD5ハッシュを計算します。
     * 
     * @param filePath ハッシュを計算するファイルのパス
     * @return MD5ハッシュ（16進数文字列）
     * @throws IOException ファイル読み込みエラーが発生した場合
     * @throws NoSuchAlgorithmException MD5アルゴリズムが利用できない場合
     */
    private String calculateMD5(String filePath) throws IOException, NoSuchAlgorithmException {
        MessageDigest md = MessageDigest.getInstance("MD5");
        
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(filePath))) {
            byte[] buffer = new byte[8192];
            int bytesRead;
            
            while ((bytesRead = bis.read(buffer)) != -1) {
                md.update(buffer, 0, bytesRead);
            }
        }
        
        byte[] digest = md.digest();
        
        // バイト配列を16進数文字列に変換
        StringBuilder sb = new StringBuilder();
        for (byte b : digest) {
            sb.append(String.format("%02x", b & 0xff));
        }
        
        return sb.toString();
    }

    public static void main(String[] args) {
        BinaryFileProcessor processor = new BinaryFileProcessor();
        
        // サンプルバイナリファイルの作成（実際のアプリケーションでは既存のファイルを使用）
        try {
            String sampleFilePath = "sample_binary.dat";
            createSampleBinaryFile(sampleFilePath, 1024 * 1024); // 1MBのサンプルファイル
            
            System.out.println("従来のI/O APIを使用したコピー結果:");
            String result1 = processor.copyAndVerifyFile(sampleFilePath, "copy_io_sample.dat", 8192);
            System.out.println(result1);
            
            System.out.println("\n\nNIO.2 APIを使用したコピー結果:");
            String result2 = processor.copyAndVerifyFileWithNIO(sampleFilePath, "copy_nio_sample.dat");
            System.out.println(result2);
            
            // バッファサイズによるパフォーマンス比較
            System.out.println("\n\nバッファサイズによるパフォーマンス比較:");
            int[] bufferSizes = {1024, 4096, 8192, 16384, 32768, 65536};
            for (int bufferSize : bufferSizes) {
                String targetPath = "copy_buffer_" + bufferSize + ".dat";
                long startTime = System.currentTimeMillis();
                processor.copyAndVerifyFile(sampleFilePath, targetPath, bufferSize);
                long endTime = System.currentTimeMillis();
                System.out.printf("バッファサイズ: %6d バイト, 所要時間: %4d ミリ秒%n", 
                                 bufferSize, (endTime - startTime));
            }
            
        } catch (IOException e) {
            System.err.println("サンプルファイル作成中にエラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    /**
     * テスト用のサンプルバイナリファイルを作成します。
     * 
     * @param filePath 作成するファイルのパス
     * @param size ファイルサイズ（バイト単位）
     * @throws IOException ファイル作成エラーが発生した場合
     */
    private static void createSampleBinaryFile(String filePath, int size) throws IOException {
        try (FileOutputStream fos = new FileOutputStream(filePath)) {
            byte[] data = new byte[size];
            // ランダムなバイトデータを生成
            for (int i = 0; i < size; i++) {
                data[i] = (byte) (Math.random() * 256);
            }
            fos.write(data);
        }
        System.out.println("サンプルバイナリファイルを作成しました: " + filePath + " (" + size + " バイト)");
    }
}
```

## コア演習3: ログファイル解析

### 解答例
```java
package io.log;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class LogAnalyzer {

    // ログエントリを表すクラス
    public static class LogEntry {
        private LocalDateTime timestamp;
        private String level;
        private String message;
        
        public LogEntry(LocalDateTime timestamp, String level, String message) {
            this.timestamp = timestamp;
            this.level = level;
            this.message = message;
        }
        
        public LocalDateTime getTimestamp() { return timestamp; }
        public String getLevel() { return level; }
        public String getMessage() { return message; }
        
        @Override
        public String toString() {
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
            return String.format("[%s] %s: %s", 
                                timestamp.format(formatter), level, message);
        }
    }
    
    // ログ解析結果を表すクラス
    public static class LogAnalysisResult {
        private int totalEntries;
        private Map<String, Integer> levelCounts;
        private List<LogEntry> errorEntries;
        private LocalDateTime startTime;
        private LocalDateTime endTime;
        
        public LogAnalysisResult() {
            this.totalEntries = 0;
            this.levelCounts = new HashMap<>();
            this.errorEntries = new ArrayList<>();
            this.startTime = null;
            this.endTime = null;
        }
        
        public void incrementTotal() {
            this.totalEntries++;
        }
        
        public void incrementLevelCount(String level) {
            levelCounts.put(level, levelCounts.getOrDefault(level, 0) + 1);
        }
        
        public void addErrorEntry(LogEntry entry) {
            errorEntries.add(entry);
        }
        
        public void updateTimeRange(LocalDateTime timestamp) {
            if (startTime == null || timestamp.isBefore(startTime)) {
                startTime = timestamp;
            }
            if (endTime == null || timestamp.isAfter(endTime)) {
                endTime = timestamp;
            }
        }
        
        public int getTotalEntries() { return totalEntries; }
        public Map<String, Integer> getLevelCounts() { return levelCounts; }
        public List<LogEntry> getErrorEntries() { return errorEntries; }
        public LocalDateTime getStartTime() { return startTime; }
        public LocalDateTime getEndTime() { return endTime; }
    }

    // ログファイルを解析するメソッド
    public LogAnalysisResult analyzeLogFile(String logFilePath) throws IOException {
        LogAnalysisResult result = new LogAnalysisResult();
        
        // ログエントリのパターン: [yyyy-MM-dd HH:mm:ss] LEVEL: Message
        Pattern pattern = Pattern.compile("\\[(\\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}:\\d{2})\\] (\\w+): (.+)");
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        
        try (BufferedReader reader = new BufferedReader(new FileReader(logFilePath))) {
            String line;
            
            while ((line = reader.readLine()) != null) {
                Matcher matcher = pattern.matcher(line);
                
                if (matcher.matches()) {
                    try {
                        // タイムスタンプ、ログレベル、メッセージを抽出
                        String timestampStr = matcher.group(1);
                        String level = matcher.group(2);
                        String message = matcher.group(3);
                        
                        LocalDateTime timestamp = LocalDateTime.parse(timestampStr, formatter);
                        
                        // ログエントリを作成
                        LogEntry entry = new LogEntry(timestamp, level, message);
                        
                        // 解析結果を更新
                        result.incrementTotal();
                        result.incrementLevelCount(level);
                        result.updateTimeRange(timestamp);
                        
                        // エラーまたは警告レベルのログを記録
                        if ("ERROR".equals(level) || "WARN".equals(level)) {
                            result.addErrorEntry(entry);
                        }
                    } catch (DateTimeParseException e) {
                        System.err.println("タイムスタンプの解析に失敗しました: " + e.getMessage());
                    }
                }
            }
        }
        
        return result;
    }
    
    // 解析結果を文字列形式で取得するメソッド
    public String getAnalysisReport(LogAnalysisResult result) {
        StringBuilder report = new StringBuilder();
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        
        report.append("===== ログ解析レポート =====\n\n");
        
        // 基本情報
        report.append("総ログエントリ数: ").append(result.getTotalEntries()).append("\n");
        if (result.getStartTime() != null && result.getEndTime() != null) {
            report.append("ログ期間: ").append(result.getStartTime().format(formatter))
                  .append(" から ").append(result.getEndTime().format(formatter)).append("\n");
        }
        
        // ログレベル別のカウント
        report.append("\nログレベル別カウント:\n");
        for (Map.Entry<String, Integer> entry : result.getLevelCounts().entrySet()) {
            report.append("- ").append(entry.getKey()).append(": ")
                  .append(entry.getValue()).append("\n");
        }
        
        // エラーと警告のログ
        List<LogEntry> errorEntries = result.getErrorEntries();
        report.append("\nエラーと警告 (").append(errorEntries.size()).append(" 件):\n");
        for (LogEntry entry : errorEntries) {
            report.append("- ").append(entry.toString()).append("\n");
        }
        
        return report.toString();
    }

    public static void main(String[] args) {
        LogAnalyzer analyzer = new LogAnalyzer();
        String logFilePath = "sample_log.txt";
        
        try {
            // サンプルログファイルが存在しない場合は作成
            Path path = Paths.get(logFilePath);
            if (!Files.exists(path)) {
                createSampleLogFile(logFilePath);
            }
            
            // ログファイルを解析
            LogAnalysisResult result = analyzer.analyzeLogFile(logFilePath);
            
            // 解析レポートを表示
            String report = analyzer.getAnalysisReport(result);
            System.out.println(report);
            
        } catch (IOException e) {
            System.err.println("ログファイル処理中にエラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    // サンプルログファイルを作成するメソッド
    private static void createSampleLogFile(String filePath) throws IOException {
        List<String> logLines = List.of(
            "[2025-05-28 08:00:00] INFO: アプリケーションを起動しました。",
            "[2025-05-28 08:01:15] DEBUG: 設定ファイルを読み込みました。",
            "[2025-05-28 08:02:30] INFO: ユーザー「admin」がログインしました。",
            "[2025-05-28 08:05:45] WARN: データベース接続が遅延しています。",
            "[2025-05-28 08:10:20] ERROR: データベースクエリの実行に失敗しました。SQL構文エラー。",
            "[2025-05-28 08:15:10] INFO: バックアップ処理を開始します。",
            "[2025-05-28 08:20:05] DEBUG: バックアップファイルを作成しました: backup_20250528.zip",
            "[2025-05-28 08:25:30] INFO: バックアップ処理が完了しました。",
            "[2025-05-28 08:30:15] WARN: メモリ使用率が80%を超えています。",
            "[2025-05-28 08:35:40] INFO: キャッシュをクリアしました。",
            "[2025-05-28 08:40:25] DEBUG: 定期メンテナンスタスクを実行しています。",
            "[2025-05-28 08:45:50] ERROR: ファイル「/data/reports/monthly.pdf」が見つかりません。",
            "[2025-05-28 08:50:10] INFO: ユーザー「user123」がログインしました。",
            "[2025-05-28 08:55:35] INFO: レポート生成処理を開始します。",
            "[2025-05-28 09:00:00] INFO: レポート生成処理が完了しました。"
        );
        
        Files.write(Paths.get(filePath), logLines);
        System.out.println("サンプルログファイルを作成しました: " + filePath);
    }
}
```

## 発展演習1: 設定ファイル読み込みと検証

### 解答例
```java
package io.config;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;
import java.util.Set;
import java.util.TreeSet;

public class ConfigFileProcessor {

    // 設定ファイルの検証結果を表すクラス
    public static class ValidationResult {
        private boolean valid;
        private Map<String, String> missingProperties;
        private Map<String, String> invalidProperties;
        
        public ValidationResult() {
            this.valid = true;
            this.missingProperties = new HashMap<>();
            this.invalidProperties = new HashMap<>();
        }
        
        public void addMissingProperty(String key, String description) {
            missingProperties.put(key, description);
            valid = false;
        }
        
        public void addInvalidProperty(String key, String description) {
            invalidProperties.put(key, description);
            valid = false;
        }
        
        public boolean isValid() { return valid; }
        public Map<String, String> getMissingProperties() { return missingProperties; }
        public Map<String, String> getInvalidProperties() { return invalidProperties; }
    }

    /**
     * 設定ファイルを読み込み、Propertiesオブジェクトとして返します。
     * 
     * @param configFilePath 設定ファイルのパス
     * @return 読み込まれた設定（Properties）
     * @throws IOException ファイル読み込みエラーが発生した場合
     */
    public Properties loadConfigFile(String configFilePath) throws IOException {
        Properties properties = new Properties();
        
        try (FileInputStream fis = new FileInputStream(configFilePath)) {
            properties.load(fis);
        }
        
        return properties;
    }
    
    /**
     * 設定ファイルを検証します。
     * 
     * @param properties 検証する設定（Properties）
     * @param requiredProperties 必須プロパティの定義（キーと説明）
     * @param validators プロパティの検証ルール（キーと検証関数）
     * @return 検証結果
     */
    public ValidationResult validateConfig(
            Properties properties, 
            Map<String, String> requiredProperties,
            Map<String, PropertyValidator> validators) {
        
        ValidationResult result = new ValidationResult();
        
        // 必須プロパティの存在チェック
        for (Map.Entry<String, String> entry : requiredProperties.entrySet()) {
            String key = entry.getKey();
            String description = entry.getValue();
            
            if (!properties.containsKey(key) || properties.getProperty(key).trim().isEmpty()) {
                result.addMissingProperty(key, description);
            }
        }
        
        // 各プロパティの値の検証
        for (Map.Entry<String, PropertyValidator> entry : validators.entrySet()) {
            String key = entry.getKey();
            PropertyValidator validator = entry.getValue();
            
            if (properties.containsKey(key)) {
                String value = properties.getProperty(key);
                if (!validator.isValid(value)) {
                    result.addInvalidProperty(key, validator.getErrorMessage(value));
                }
            }
        }
        
        return result;
    }
    
    /**
     * 設定ファイルを作成または更新します。
     * 
     * @param configFilePath 設定ファイルのパス
     * @param properties 設定内容（Properties）
     * @param comments ファイル先頭に追加するコメント
     * @throws IOException ファイル書き込みエラーが発生した場合
     */
    public void saveConfigFile(String configFilePath, Properties properties, String comments) throws IOException {
        try (FileOutputStream fos = new FileOutputStream(configFilePath)) {
            properties.store(fos, comments);
        }
    }
    
    /**
     * 設定ファイルの内容を整形して文字列として返します。
     * 
     * @param properties 設定内容（Properties）
     * @return 整形された設定内容の文字列
     */
    public String formatConfig(Properties properties) {
        StringBuilder sb = new StringBuilder();
        sb.append("===== 設定内容 =====\n\n");
        
        // キーをアルファベット順にソート
        Set<String> keys = new TreeSet<>();
        for (Object key : properties.keySet()) {
            keys.add((String) key);
        }
        
        // 各設定項目を出力
        for (String key : keys) {
            String value = properties.getProperty(key);
            sb.append(key).append(" = ").append(value).append("\n");
        }
        
        return sb.toString();
    }
    
    /**
     * 検証結果を整形して文字列として返します。
     * 
     * @param result 検証結果
     * @return 整形された検証結果の文字列
     */
    public String formatValidationResult(ValidationResult result) {
        StringBuilder sb = new StringBuilder();
        sb.append("===== 設定ファイル検証結果 =====\n\n");
        
        if (result.isValid()) {
            sb.append("検証結果: 有効（すべての設定が正しく設定されています）\n");
        } else {
            sb.append("検証結果: 無効（以下の問題が見つかりました）\n\n");
            
            // 不足しているプロパティ
            Map<String, String> missingProps = result.getMissingProperties();
            if (!missingProps.isEmpty()) {
                sb.append("不足しているプロパティ:\n");
                for (Map.Entry<String, String> entry : missingProps.entrySet()) {
                    sb.append("- ").append(entry.getKey())
                      .append(": ").append(entry.getValue()).append("\n");
                }
                sb.append("\n");
            }
            
            // 無効なプロパティ値
            Map<String, String> invalidProps = result.getInvalidProperties();
            if (!invalidProps.isEmpty()) {
                sb.append("無効なプロパティ値:\n");
                for (Map.Entry<String, String> entry : invalidProps.entrySet()) {
                    sb.append("- ").append(entry.getKey())
                      .append(": ").append(entry.getValue()).append("\n");
                }
            }
        }
        
        return sb.toString();
    }

    // プロパティ検証インターフェース
    public interface PropertyValidator {
        boolean isValid(String value);
        String getErrorMessage(String value);
    }
    
    // 数値範囲検証クラス
    public static class NumberRangeValidator implements PropertyValidator {
        private int min;
        private int max;
        
        public NumberRangeValidator(int min, int max) {
            this.min = min;
            this.max = max;
        }
        
        @Override
        public boolean isValid(String value) {
            try {
                int intValue = Integer.parseInt(value);
                return intValue >= min && intValue <= max;
            } catch (NumberFormatException e) {
                return false;
            }
        }
        
        @Override
        public String getErrorMessage(String value) {
            return String.format("値「%s」は有効な数値ではないか、%d〜%d の範囲外です。", value, min, max);
        }
    }
    
    // 列挙値検証クラス
    public static class EnumValidator implements PropertyValidator {
        private Set<String> validValues;
        
        public EnumValidator(String... validValues) {
            this.validValues = new TreeSet<>();
            for (String value : validValues) {
                this.validValues.add(value.toLowerCase());
            }
        }
        
        @Override
        public boolean isValid(String value) {
            return validValues.contains(value.toLowerCase());
        }
        
        @Override
        public String getErrorMessage(String value) {
            return String.format("値「%s」は有効な選択肢ではありません。有効な値: %s", 
                                value, String.join(", ", validValues));
        }
    }
    
    // 正規表現検証クラス
    public static class RegexValidator implements PropertyValidator {
        private String pattern;
        private String description;
        
        public RegexValidator(String pattern, String description) {
            this.pattern = pattern;
            this.description = description;
        }
        
        @Override
        public boolean isValid(String value) {
            return value.matches(pattern);
        }
        
        @Override
        public String getErrorMessage(String value) {
            return String.format("値「%s」は%sの形式ではありません。", value, description);
        }
    }

    public static void main(String[] args) {
        ConfigFileProcessor processor = new ConfigFileProcessor();
        String configFilePath = "app_config.properties";
        
        try {
            // サンプル設定ファイルが存在しない場合は作成
            Path path = Paths.get(configFilePath);
            if (!Files.exists(path)) {
                createSampleConfigFile(configFilePath);
            }
            
            // 設定ファイルを読み込む
            Properties config = processor.loadConfigFile(configFilePath);
            System.out.println("設定ファイルを読み込みました: " + configFilePath);
            System.out.println(processor.formatConfig(config));
            
            // 必須プロパティの定義
            Map<String, String> requiredProperties = new HashMap<>();
            requiredProperties.put("app.name", "アプリケーション名");
            requiredProperties.put("app.version", "アプリケーションバージョン");
            requiredProperties.put("db.url", "データベース接続URL");
            requiredProperties.put("db.user", "データベースユーザー名");
            requiredProperties.put("db.password", "データベースパスワード");
            requiredProperties.put("log.level", "ログレベル");
            requiredProperties.put("server.port", "サーバーポート番号");
            
            // 検証ルールの定義
            Map<String, PropertyValidator> validators = new HashMap<>();
            validators.put("app.version", new RegexValidator("\\d+\\.\\d+\\.\\d+", "セマンティックバージョニング"));
            validators.put("log.level", new EnumValidator("DEBUG", "INFO", "WARN", "ERROR"));
            validators.put("server.port", new NumberRangeValidator(1024, 65535));
            validators.put("db.connection.pool.size", new NumberRangeValidator(1, 100));
            
            // 設定ファイルを検証
            ValidationResult result = processor.validateConfig(config, requiredProperties, validators);
            System.out.println(processor.formatValidationResult(result));
            
            // 検証に失敗した場合は修正例を示す
            if (!result.isValid()) {
                System.out.println("\n===== 修正例 =====\n");
                
                // 不足しているプロパティを追加
                for (String key : result.getMissingProperties().keySet()) {
                    if ("app.name".equals(key)) config.setProperty(key, "SampleApplication");
                    else if ("app.version".equals(key)) config.setProperty(key, "1.0.0");
                    else if ("db.url".equals(key)) config.setProperty(key, "jdbc:postgresql://localhost:5432/sampledb");
                    else if ("db.user".equals(key)) config.setProperty(key, "dbuser");
                    else if ("db.password".equals(key)) config.setProperty(key, "dbpassword");
                    else if ("log.level".equals(key)) config.setProperty(key, "INFO");
                    else if ("server.port".equals(key)) config.setProperty(key, "8080");
                }
                
                // 無効なプロパティを修正
                for (String key : result.getInvalidProperties().keySet()) {
                    if ("app.version".equals(key)) config.setProperty(key, "1.0.0");
                    else if ("log.level".equals(key)) config.setProperty(key, "INFO");
                    else if ("server.port".equals(key)) config.setProperty(key, "8080");
                    else if ("db.connection.pool.size".equals(key)) config.setProperty(key, "10");
                }
                
                // 修正後の設定を保存
                String fixedConfigPath = "fixed_" + configFilePath;
                processor.saveConfigFile(fixedConfigPath, config, "Fixed configuration");
                System.out.println("修正後の設定ファイルを保存しました: " + fixedConfigPath);
                
                // 修正後の設定を表示
                System.out.println(processor.formatConfig(config));
                
                // 再検証
                ValidationResult fixedResult = processor.validateConfig(config, requiredProperties, validators);
                System.out.println(processor.formatValidationResult(fixedResult));
            }
            
        } catch (IOException e) {
            System.err.println("設定ファイル処理中にエラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    // サンプル設定ファイルを作成するメソッド
    private static void createSampleConfigFile(String filePath) throws IOException {
        Properties properties = new Properties();
        
        // 基本設定
        properties.setProperty("app.name", "SampleApplication");
        properties.setProperty("app.version", "1.0"); // 無効な形式（正しくは1.0.0）
        
        // データベース設定
        properties.setProperty("db.url", "jdbc:postgresql://localhost:5432/sampledb");
        properties.setProperty("db.user", "dbuser");
        // db.passwordが欠落している
        properties.setProperty("db.connection.pool.size", "200"); // 範囲外（1-100）
        
        // ログ設定
        properties.setProperty("log.level", "TRACE"); // 無効な値（DEBUG/INFO/WARN/ERROR）
        properties.setProperty("log.file", "/var/log/app.log");
        
        // サーバー設定
        properties.setProperty("server.host", "localhost");
        // server.portが欠落している
        
        try (FileOutputStream fos = new FileOutputStream(filePath)) {
            properties.store(fos, "Sample Configuration File");
        }
        
        System.out.println("サンプル設定ファイルを作成しました: " + filePath);
    }
}
```

## 発展演習2: CSVファイル処理

### 解答例
```java
package io.csv;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class CsvProcessor {

    // CSVファイルの列定義を表すクラス
    public static class CsvColumnDefinition {
        private String name;
        private String type;
        private boolean required;
        
        public CsvColumnDefinition(String name, String type, boolean required) {
            this.name = name;
            this.type = type;
            this.required = required;
        }
        
        public String getName() { return name; }
        public String getType() { return type; }
        public boolean isRequired() { return required; }
    }
    
    // CSVデータの検証エラーを表すクラス
    public static class CsvValidationError {
        private int rowIndex;
        private String columnName;
        private String value;
        private String errorMessage;
        
        public CsvValidationError(int rowIndex, String columnName, String value, String errorMessage) {
            this.rowIndex = rowIndex;
            this.columnName = columnName;
            this.value = value;
            this.errorMessage = errorMessage;
        }
        
        public int getRowIndex() { return rowIndex; }
        public String getColumnName() { return columnName; }
        public String getValue() { return value; }
        public String getErrorMessage() { return errorMessage; }
        
        @Override
        public String toString() {
            return String.format("行 %d, 列 '%s': 値 '%s' - %s", 
                                rowIndex + 1, columnName, value, errorMessage);
        }
    }
    
    // CSVデータの処理結果を表すクラス
    public static class CsvProcessingResult {
        private List<Map<String, String>> validRecords;
        private List<CsvValidationError> errors;
        
        public CsvProcessingResult() {
            this.validRecords = new ArrayList<>();
            this.errors = new ArrayList<>();
        }
        
        public void addValidRecord(Map<String, String> record) {
            validRecords.add(record);
        }
        
        public void addError(CsvValidationError error) {
            errors.add(error);
        }
        
        public List<Map<String, String>> getValidRecords() { return validRecords; }
        public List<CsvValidationError> getErrors() { return errors; }
        public boolean hasErrors() { return !errors.isEmpty(); }
        public int getValidRecordCount() { return validRecords.size(); }
        public int getErrorCount() { return errors.size(); }
    }

    /**
     * CSVファイルを読み込み、検証して処理します。
     * 
     * @param csvFilePath CSVファイルのパス
     * @param columnDefinitions 列定義のリスト
     * @param delimiter 区切り文字（カンマやタブなど）
     * @return 処理結果
     * @throws IOException ファイル読み込みエラーが発生した場合
     */
    public CsvProcessingResult processCsvFile(
            String csvFilePath, 
            List<CsvColumnDefinition> columnDefinitions,
            char delimiter) throws IOException {
        
        CsvProcessingResult result = new CsvProcessingResult();
        
        try (BufferedReader reader = new BufferedReader(new FileReader(csvFilePath))) {
            String line;
            int rowIndex = 0;
            
            // ヘッダー行を読み込む
            line = reader.readLine();
            if (line == null) {
                throw new IOException("CSVファイルが空です。");
            }
            
            // ヘッダー列を解析
            String[] headers = parseCsvLine(line, delimiter);
            
            // 列定義と実際のヘッダーの対応を確認
            Map<String, Integer> headerIndexMap = new HashMap<>();
            for (int i = 0; i < headers.length; i++) {
                headerIndexMap.put(headers[i], i);
            }
            
            // 必須列が存在するか確認
            for (CsvColumnDefinition colDef : columnDefinitions) {
                if (colDef.isRequired() && !headerIndexMap.containsKey(colDef.getName())) {
                    result.addError(new CsvValidationError(
                        0, colDef.getName(), null, "必須列がCSVファイルに存在しません。"));
                }
            }
            
            // 各データ行を処理
            while ((line = reader.readLine()) != null) {
                rowIndex++;
                
                // 空行はスキップ
                if (line.trim().isEmpty()) {
                    continue;
                }
                
                // 行を解析
                String[] values = parseCsvLine(line, delimiter);
                
                // 列数が一致しない場合はエラー
                if (values.length != headers.length) {
                    result.addError(new CsvValidationError(
                        rowIndex, null, null, 
                        String.format("列数が一致しません。期待: %d, 実際: %d", 
                                     headers.length, values.length)));
                    continue;
                }
                
                // 行データをマップに変換
                Map<String, String> record = new HashMap<>();
                boolean rowValid = true;
                
                for (CsvColumnDefinition colDef : columnDefinitions) {
                    String columnName = colDef.getName();
                    
                    // 列が存在しない場合はスキップ
                    if (!headerIndexMap.containsKey(columnName)) {
                        continue;
                    }
                    
                    int columnIndex = headerIndexMap.get(columnName);
                    String value = values[columnIndex];
                    
                    // 必須チェック
                    if (colDef.isRequired() && (value == null || value.trim().isEmpty())) {
                        result.addError(new CsvValidationError(
                            rowIndex, columnName, value, "必須項目が空です。"));
                        rowValid = false;
                        continue;
                    }
                    
                    // 型チェック
                    if (value != null && !value.trim().isEmpty()) {
                        boolean typeValid = validateValueType(value, colDef.getType());
                        if (!typeValid) {
                            result.addError(new CsvValidationError(
                                rowIndex, columnName, value, 
                                String.format("値の型が不正です。期待: %s", colDef.getType())));
                            rowValid = false;
                            continue;
                        }
                    }
                    
                    // 有効な値をレコードに追加
                    record.put(columnName, value);
                }
                
                // 行が有効な場合は結果に追加
                if (rowValid) {
                    result.addValidRecord(record);
                }
            }
        }
        
        return result;
    }
    
    /**
     * 値の型を検証します。
     * 
     * @param value 検証する値
     * @param type 期待される型（"string", "integer", "decimal", "date"など）
     * @return 型が一致する場合はtrue、それ以外はfalse
     */
    private boolean validateValueType(String value, String type) {
        switch (type.toLowerCase()) {
            case "string":
                return true; // 文字列は常に有効
                
            case "integer":
                try {
                    Integer.parseInt(value);
                    return true;
                } catch (NumberFormatException e) {
                    return false;
                }
                
            case "decimal":
                try {
                    Double.parseDouble(value);
                    return true;
                } catch (NumberFormatException e) {
                    return false;
                }
                
            case "date":
                // 簡易的な日付形式チェック（YYYY-MM-DD）
                return value.matches("\\d{4}-\\d{2}-\\d{2}");
                
            case "boolean":
                String lowerValue = value.toLowerCase();
                return lowerValue.equals("true") || lowerValue.equals("false") ||
                       lowerValue.equals("yes") || lowerValue.equals("no") ||
                       lowerValue.equals("1") || lowerValue.equals("0");
                
            default:
                return false; // 未知の型
        }
    }
    
    /**
     * CSV行を解析して配列に分割します。
     * 
     * @param line 解析する行
     * @param delimiter 区切り文字
     * @return 分割された値の配列
     */
    private String[] parseCsvLine(String line, char delimiter) {
        List<String> values = new ArrayList<>();
        StringBuilder sb = new StringBuilder();
        boolean inQuotes = false;
        
        for (int i = 0; i < line.length(); i++) {
            char c = line.charAt(i);
            
            if (c == '"') {
                // 引用符内の引用符（エスケープされた引用符）
                if (inQuotes && i + 1 < line.length() && line.charAt(i + 1) == '"') {
                    sb.append('"');
                    i++; // 次の引用符をスキップ
                } else {
                    // 引用符の開始または終了
                    inQuotes = !inQuotes;
                }
            } else if (c == delimiter && !inQuotes) {
                // 区切り文字（引用符の外）
                values.add(sb.toString());
                sb.setLength(0); // StringBuilder をクリア
            } else {
                // 通常の文字
                sb.append(c);
            }
        }
        
        // 最後の値を追加
        values.add(sb.toString());
        
        return values.toArray(new String[0]);
    }
    
    /**
     * 処理結果をCSVファイルに書き込みます。
     * 
     * @param outputFilePath 出力ファイルのパス
     * @param records 書き込むレコードのリスト
     * @param columnOrder 列の順序
     * @param delimiter 区切り文字
     * @throws IOException ファイル書き込みエラーが発生した場合
     */
    public void writeRecordsToCsv(
            String outputFilePath, 
            List<Map<String, String>> records,
            List<String> columnOrder,
            char delimiter) throws IOException {
        
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(outputFilePath))) {
            // ヘッダー行を書き込む
            writer.write(String.join(String.valueOf(delimiter), columnOrder));
            writer.newLine();
            
            // データ行を書き込む
            for (Map<String, String> record : records) {
                List<String> values = new ArrayList<>();
                
                for (String column : columnOrder) {
                    String value = record.getOrDefault(column, "");
                    // 値に区切り文字や引用符が含まれる場合は引用符で囲む
                    if (value.contains(String.valueOf(delimiter)) || value.contains("\"") || value.contains("\n")) {
                        value = "\"" + value.replace("\"", "\"\"") + "\"";
                    }
                    values.add(value);
                }
                
                writer.write(String.join(String.valueOf(delimiter), values));
                writer.newLine();
            }
        }
    }
    
    /**
     * 処理結果のサマリーを文字列形式で取得します。
     * 
     * @param result 処理結果
     * @return サマリー文字列
     */
    public String getProcessingSummary(CsvProcessingResult result) {
        StringBuilder summary = new StringBuilder();
        summary.append("===== CSV処理サマリー =====\n\n");
        
        summary.append("有効レコード数: ").append(result.getValidRecordCount()).append("\n");
        summary.append("エラー数: ").append(result.getErrorCount()).append("\n\n");
        
        if (result.hasErrors()) {
            summary.append("エラー詳細:\n");
            for (CsvValidationError error : result.getErrors()) {
                summary.append("- ").append(error.toString()).append("\n");
            }
        }
        
        return summary.toString();
    }

    public static void main(String[] args) {
        CsvProcessor processor = new CsvProcessor();
        String csvFilePath = "sample_data.csv";
        
        try {
            // サンプルCSVファイルが存在しない場合は作成
            Path path = Paths.get(csvFilePath);
            if (!Files.exists(path)) {
                createSampleCsvFile(csvFilePath);
            }
            
            // 列定義を設定
            List<CsvColumnDefinition> columnDefinitions = Arrays.asList(
                new CsvColumnDefinition("ID", "integer", true),
                new CsvColumnDefinition("Name", "string", true),
                new CsvColumnDefinition("Age", "integer", false),
                new CsvColumnDefinition("Email", "string", true),
                new CsvColumnDefinition("JoinDate", "date", false),
                new CsvColumnDefinition("Active", "boolean", false)
            );
            
            // CSVファイルを処理
            CsvProcessingResult result = processor.processCsvFile(csvFilePath, columnDefinitions, ',');
            
            // 処理結果のサマリーを表示
            System.out.println(processor.getProcessingSummary(result));
            
            // 有効なレコードを別のCSVファイルに書き出す
            if (result.getValidRecordCount() > 0) {
                String outputFilePath = "valid_records.csv";
                List<String> columnOrder = Arrays.asList("ID", "Name", "Age", "Email", "JoinDate", "Active");
                
                processor.writeRecordsToCsv(outputFilePath, result.getValidRecords(), columnOrder, ',');
                System.out.println("有効なレコードを書き出しました: " + outputFilePath);
                
                // 有効なレコードの内容を表示
                System.out.println("\n有効なレコード:");
                for (Map<String, String> record : result.getValidRecords()) {
                    System.out.println(record);
                }
            }
            
        } catch (IOException e) {
            System.err.println("CSVファイル処理中にエラーが発生しました: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    // サンプルCSVファイルを作成するメソッド
    private static void createSampleCsvFile(String filePath) throws IOException {
        List<String> lines = new ArrayList<>();
        
        // ヘッダー行
        lines.add("ID,Name,Age,Email,JoinDate,Active");
        
        // データ行
        lines.add("1,\"Yamada, Taro\",30,taro.yamada@example.com,2023-04-01,true");
        lines.add("2,Hanako Sato,25,hanako.sato@example.com,2023-05-15,true");
        lines.add("3,Jiro Suzuki,,jiro.suzuki@example.com,2023-06-20,false");
        lines.add("4,Sachiko Tanaka,invalid,sachiko.tanaka@example.com,2023-07-10,yes");
        lines.add("5,Ichiro Watanabe,45,ichiro.watanabe@example.com,invalid-date,1");
        lines.add("6,Yumiko Ito,28,,2023-09-05,true");  // メールアドレスが空（必須項目）
        lines.add("7,Takeshi Kato,33,takeshi.kato@example.com,2023-10-12,");
        
        Files.write(Paths.get(filePath), lines);
        System.out.println("サンプルCSVファイルを作成しました: " + filePath);
    }
}
```
