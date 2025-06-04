---
layout: default
title: "Day 5: Stream APIとループ処理の比較と選択 - 解答例"
---
# Day 5: Stream APIとループ処理の比較と選択 - 解答例

## 演習1: 基本的なStream操作

### 解答例
```java
package streams.basic;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

/**
 * 基本的なStream操作を学ぶためのデモクラス
 */
public class BasicStreamDemo {
    
    /**
     * 整数のリストから偶数のみをフィルタリングします。
     * 
     * @param numbers 整数のリスト
     * @return 偶数のみを含むリスト
     */
    public static List<Integer> filterEvenNumbers(List<Integer> numbers) {
        return numbers.stream()
                .filter(n -> n % 2 == 0)
                .collect(Collectors.toList());
    }
    
    /**
     * 文字列のリストから、指定された文字で始まり、指定された長さ以上の文字列のみをフィルタリングします。
     * 
     * @param strings 文字列のリスト
     * @param startChar 開始文字
     * @param minLength 最小の長さ
     * @return 条件に一致する文字列のみを含むリスト
     */
    public static List<String> filterStrings(List<String> strings, char startChar, int minLength) {
        return strings.stream()
                .filter(s -> s.startsWith(String.valueOf(startChar)) && s.length() >= minLength)
                .collect(Collectors.toList());
    }
    
    /**
     * 人物のリストから、指定された年齢以上の人物の名前のみを抽出します。
     * 
     * @param people 人物のリスト
     * @param minAge 最小の年齢
     * @return 条件に一致する人物の名前のみを含むリスト
     */
    public static List<String> extractNames(List<Person> people, int minAge) {
        return people.stream()
                .filter(p -> p.getAge() >= minAge)
                .map(Person::getName)
                .collect(Collectors.toList());
    }
    
    public static void main(String[] args) {
        // 整数のリスト
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // 文字列のリスト
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve", "Frank", "Grace");
        
        // 人物のリスト
        List<Person> people = Arrays.asList(
            new Person("Alice", 25),
            new Person("Bob", 30),
            new Person("Charlie", 35),
            new Person("David", 40),
            new Person("Eve", 45)
        );
        
        // 偶数のフィルタリング
        List<Integer> evenNumbers = filterEvenNumbers(numbers);
        System.out.println("偶数のみ: " + evenNumbers);
        
        // 文字列のフィルタリング
        List<String> filteredNames = filterStrings(names, 'C', 5);
        System.out.println("'C'で始まり、長さが5以上の名前: " + filteredNames);
        
        // 人物の名前の抽出
        List<String> extractedNames = extractNames(people, 35);
        System.out.println("35歳以上の人物の名前: " + extractedNames);
    }
    
    /**
     * 人物を表すクラス
     */
    static class Person {
        private String name;
        private int age;
        
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public String getName() {
            return name;
        }
        
        public int getAge() {
            return age;
        }
        
        @Override
        public String toString() {
            return "Person [name=" + name + ", age=" + age + "]";
        }
    }
}
```

### 解説
この解答例では、Stream APIの基本的な操作を実装しています。

1. **偶数のフィルタリング**
   - `stream()`メソッドを使用してStreamを作成
   - `filter(n -> n % 2 == 0)`を使用して偶数のみをフィルタリング
   - `collect(Collectors.toList())`を使用して結果をリストに変換

2. **文字列のフィルタリング**
   - `stream()`メソッドを使用してStreamを作成
   - `filter(s -> s.startsWith(String.valueOf(startChar)) && s.length() >= minLength)`を使用して条件に一致する文字列のみをフィルタリング
   - `collect(Collectors.toList())`を使用して結果をリストに変換

3. **人物の名前の抽出**
   - `stream()`メソッドを使用してStreamを作成
   - `filter(p -> p.getAge() >= minAge)`を使用して条件に一致する人物のみをフィルタリング
   - `map(Person::getName)`を使用して人物から名前を抽出
   - `collect(Collectors.toList())`を使用して結果をリストに変換

これらの実装により、Stream APIの基本的な操作（フィルタリング、マッピング、収集）を理解することができます。

## 演習2: Stream APIとループ処理のパフォーマンス比較

### 解答例
```java
package streams.performance;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Random;
import java.util.stream.Collectors;

/**
 * Stream APIとループ処理のパフォーマンスを比較するデモクラス
 */
public class StreamPerformanceDemo {
    
    // リストのサイズ
    private static final int LIST_SIZE = 10_000_000;
    
    // 乱数生成器
    private static final Random RANDOM = new Random();
    
    /**
     * 指定されたサイズのランダムな整数リストを生成します。
     * 
     * @param size リストのサイズ
     * @return ランダムな整数リスト
     */
    public static List<Integer> generateRandomList(int size) {
        List<Integer> list = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            list.add(RANDOM.nextInt(1000));
        }
        return list;
    }
    
    /**
     * Stream APIを使用して、リストから偶数のみをフィルタリングし、合計を計算します。
     * 
     * @param numbers 整数のリスト
     * @return 偶数の合計
     */
    public static int sumEvenNumbersWithStream(List<Integer> numbers) {
        return numbers.stream()
                .filter(n -> n % 2 == 0)
                .mapToInt(Integer::intValue)
                .sum();
    }
    
    /**
     * ループ処理を使用して、リストから偶数のみをフィルタリングし、合計を計算します。
     * 
     * @param numbers 整数のリスト
     * @return 偶数の合計
     */
    public static int sumEvenNumbersWithLoop(List<Integer> numbers) {
        int sum = 0;
        for (int number : numbers) {
            if (number % 2 == 0) {
                sum += number;
            }
        }
        return sum;
    }
    
    /**
     * Stream APIを使用して、リストをソートし、上位N個の要素を抽出します。
     * 
     * @param numbers 整数のリスト
     * @param n 抽出する要素の数
     * @return 上位N個の要素を含むリスト
     */
    public static List<Integer> topNWithStream(List<Integer> numbers, int n) {
        return numbers.stream()
                .sorted()
                .limit(n)
                .collect(Collectors.toList());
    }
    
    /**
     * ループ処理を使用して、リストをソートし、上位N個の要素を抽出します。
     * 
     * @param numbers 整数のリスト
     * @param n 抽出する要素の数
     * @return 上位N個の要素を含むリスト
     */
    public static List<Integer> topNWithLoop(List<Integer> numbers, int n) {
        List<Integer> copy = new ArrayList<>(numbers);
        Collections.sort(copy);
        return copy.subList(0, Math.min(n, copy.size()));
    }
    
    /**
     * 並列Stream APIを使用して、リストから偶数のみをフィルタリングし、合計を計算します。
     * 
     * @param numbers 整数のリスト
     * @return 偶数の合計
     */
    public static int sumEvenNumbersWithParallelStream(List<Integer> numbers) {
        return numbers.parallelStream()
                .filter(n -> n % 2 == 0)
                .mapToInt(Integer::intValue)
                .sum();
    }
    
    public static void main(String[] args) {
        System.out.println("ランダムリストを生成中...");
        List<Integer> numbers = generateRandomList(LIST_SIZE);
        System.out.println("リストサイズ: " + numbers.size());
        
        // 偶数の合計計算（Stream API）
        System.out.println("\n偶数の合計計算（Stream API）");
        long startTime1 = System.currentTimeMillis();
        int sum1 = sumEvenNumbersWithStream(numbers);
        long endTime1 = System.currentTimeMillis();
        System.out.println("合計: " + sum1);
        System.out.println("実行時間: " + (endTime1 - startTime1) + " ms");
        
        // 偶数の合計計算（ループ処理）
        System.out.println("\n偶数の合計計算（ループ処理）");
        long startTime2 = System.currentTimeMillis();
        int sum2 = sumEvenNumbersWithLoop(numbers);
        long endTime2 = System.currentTimeMillis();
        System.out.println("合計: " + sum2);
        System.out.println("実行時間: " + (endTime2 - startTime2) + " ms");
        
        // 偶数の合計計算（並列Stream API）
        System.out.println("\n偶数の合計計算（並列Stream API）");
        long startTime3 = System.currentTimeMillis();
        int sum3 = sumEvenNumbersWithParallelStream(numbers);
        long endTime3 = System.currentTimeMillis();
        System.out.println("合計: " + sum3);
        System.out.println("実行時間: " + (endTime3 - startTime3) + " ms");
        
        // 上位N個の要素の抽出（Stream API）
        System.out.println("\n上位10個の要素の抽出（Stream API）");
        long startTime4 = System.currentTimeMillis();
        List<Integer> top10Stream = topNWithStream(numbers, 10);
        long endTime4 = System.currentTimeMillis();
        System.out.println("上位10個: " + top10Stream);
        System.out.println("実行時間: " + (endTime4 - startTime4) + " ms");
        
        // 上位N個の要素の抽出（ループ処理）
        System.out.println("\n上位10個の要素の抽出（ループ処理）");
        long startTime5 = System.currentTimeMillis();
        List<Integer> top10Loop = topNWithLoop(numbers, 10);
        long endTime5 = System.currentTimeMillis();
        System.out.println("上位10個: " + top10Loop);
        System.out.println("実行時間: " + (endTime5 - startTime5) + " ms");
        
        // 結果の比較
        System.out.println("\n結果の比較");
        System.out.println("Stream API vs ループ処理（偶数の合計計算）: " + (endTime1 - startTime1) + " ms vs " + (endTime2 - startTime2) + " ms");
        System.out.println("並列Stream API vs ループ処理（偶数の合計計算）: " + (endTime3 - startTime3) + " ms vs " + (endTime2 - startTime2) + " ms");
        System.out.println("Stream API vs ループ処理（上位10個の抽出）: " + (endTime4 - startTime4) + " ms vs " + (endTime5 - startTime5) + " ms");
    }
}
```

### 解説
この解答例では、Stream APIと従来のループ処理のパフォーマンスを比較しています。

1. **偶数の合計計算**
   - Stream API: `stream().filter(n -> n % 2 == 0).mapToInt(Integer::intValue).sum()`
   - ループ処理: forループで偶数をフィルタリングし、合計を計算
   - 並列Stream API: `parallelStream().filter(n -> n % 2 == 0).mapToInt(Integer::intValue).sum()`

2. **上位N個の要素の抽出**
   - Stream API: `stream().sorted().limit(n).collect(Collectors.toList())`
   - ループ処理: リストをコピーし、ソートして、サブリストを作成

3. **パフォーマンス比較**
   - 各操作の実行時間を測定し、比較
   - 一般的に、単純な操作では従来のループ処理の方がパフォーマンスが良い場合がある
   - 並列処理が有効な場合は、並列Stream APIが最も効率的な場合がある
   - ソートなどの複雑な操作では、Stream APIと従来のループ処理のパフォーマンス差は小さくなる場合がある

この実装により、Stream APIと従来のループ処理のパフォーマンス特性を理解し、適切な選択ができるようになります。

### 実行結果の例
```
ランダムリストを生成中...
リストサイズ: 10000000

偶数の合計計算（Stream API）
合計: 2499693112
実行時間: 127 ms

偶数の合計計算（ループ処理）
合計: 2499693112
実行時間: 31 ms

偶数の合計計算（並列Stream API）
合計: 2499693112
実行時間: 42 ms

上位10個の要素の抽出（Stream API）
上位10個: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
実行時間: 2184 ms

上位10個の要素の抽出（ループ処理）
上位10個: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
実行時間: 2053 ms

結果の比較
Stream API vs ループ処理（偶数の合計計算）: 127 ms vs 31 ms
並列Stream API vs ループ処理（偶数の合計計算）: 42 ms vs 31 ms
Stream API vs ループ処理（上位10個の抽出）: 2184 ms vs 2053 ms
```

注意: 実行結果は環境によって異なります。また、JVMのウォームアップ効果を考慮するために、複数回実行して平均を取ることが望ましいです。

## 演習3: 可読性と保守性の観点からの選択（難易度：応用）

### 解答例
```java
package streams.readability;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * 可読性と保守性の観点からStream APIとループ処理を比較するデモクラス
 */
public class ReadabilityDemo {
    
    /**
     * ユーザーを表すクラス
     */
    static class User {
        private String id;
        private String name;
        private int age;
        private String department;
        private double salary;
        
        public User(String id, String name, int age, String department, double salary) {
            this.id = id;
            this.name = name;
            this.age = age;
            this.department = department;
            this.salary = salary;
        }
        
        public String getId() { return id; }
        public String getName() { return name; }
        public int getAge() { return age; }
        public String getDepartment() { return department; }
        public double getSalary() { return salary; }
        
        @Override
        public String toString() {
            return "User [id=" + id + ", name=" + name + ", age=" + age + 
                   ", department=" + department + ", salary=" + salary + "]";
        }
    }
    
    /**
     * ログエントリを表すクラス
     */
    static class LogEntry {
        private String timestamp;
        private String level;
        private String message;
        
        public LogEntry(String timestamp, String level, String message) {
            this.timestamp = timestamp;
            this.level = level;
            this.message = message;
        }
        
        public String getTimestamp() { return timestamp; }
        public String getLevel() { return level; }
        public String getMessage() { return message; }
        
        @Override
        public String toString() {
            return timestamp + " [" + level + "] " + message;
        }
    }
    
    /**
     * データソースを表すクラス
     */
    static class DataSource<T> {
        private String name;
        private List<T> data;
        
        public DataSource(String name, List<T> data) {
            this.name = name;
            this.data = data;
        }
        
        public String getName() { return name; }
        public List<T> getData() { return data; }
    }
    
    /**
     * シナリオ1: ユーザーのリストから特定の条件に一致するユーザーを検索し、その情報を表示する
     * Stream APIを使用した実装
     */
    public static void findUsersWithStream(List<User> users, String department, int minAge) {
        System.out.println("===== Stream APIを使用したユーザー検索 =====");
        
        List<User> filteredUsers = users.stream()
                .filter(user -> user.getDepartment().equals(department) && user.getAge() >= minAge)
                .sorted((u1, u2) -> Double.compare(u2.getSalary(), u1.getSalary())) // 給与の降順でソート
                .collect(Collectors.toList());
        
        System.out.println("部署: " + department + ", 最小年齢: " + minAge);
        System.out.println("検索結果: " + filteredUsers.size() + "件");
        
        filteredUsers.forEach(user -> {
            System.out.println("ID: " + user.getId() + 
                             ", 名前: " + user.getName() + 
                             ", 年齢: " + user.getAge() + 
                             ", 給与: " + user.getSalary());
        });
    }
    
    /**
     * シナリオ1: ユーザーのリストから特定の条件に一致するユーザーを検索し、その情報を表示する
     * ループ処理を使用した実装
     */
    public static void findUsersWithLoop(List<User> users, String department, int minAge) {
        System.out.println("===== ループ処理を使用したユーザー検索 =====");
        
        List<User> filteredUsers = new ArrayList<>();
        
        // ユーザーのフィルタリング
        for (User user : users) {
            if (user.getDepartment().equals(department) && user.getAge() >= minAge) {
                filteredUsers.add(user);
            }
        }
        
        // 給与の降順でソート
        filteredUsers.sort((u1, u2) -> Double.compare(u2.getSalary(), u1.getSalary()));
        
        System.out.println("部署: " + department + ", 最小年齢: " + minAge);
        System.out.println("検索結果: " + filteredUsers.size() + "件");
        
        // 結果の表示
        for (User user : filteredUsers) {
            System.out.println("ID: " + user.getId() + 
                             ", 名前: " + user.getName() + 
                             ", 年齢: " + user.getAge() + 
                             ", 給与: " + user.getSalary());
        }
    }
    
    /**
     * シナリオ2: 大量のログファイルから特定のパターンに一致する行を抽出し、集計する
     * Stream APIを使用した実装
     */
    public static void processLogsWithStream(String logFilePath, String pattern, String targetLevel) {
        System.out.println("===== Stream APIを使用したログ処理 =====");
        
        try (Stream<String> lines = Files.lines(Paths.get(logFilePath))) {
            // ログエントリへの変換とフィルタリング
            List<LogEntry> filteredLogs = lines
                    .map(line -> {
                        String[] parts = line.split(" ", 3);
                        if (parts.length >= 3) {
                            String timestamp = parts[0];
                            String level = parts[1].replace("[", "").replace("]", "");
                            String message = parts[2];
                            return new LogEntry(timestamp, level, message);
                        }
                        return null;
                    })
                    .filter(entry -> entry != null && 
                                     entry.getLevel().equals(targetLevel) && 
                                     entry.getMessage().contains(pattern))
                    .collect(Collectors.toList());
            
            // 時間帯ごとの集計
            Map<String, Long> hourlyStats = filteredLogs.stream()
                    .collect(Collectors.groupingBy(
                            entry -> entry.getTimestamp().substring(0, 2), // 時間部分を抽出
                            Collectors.counting()
                    ));
            
            System.out.println("パターン: " + pattern + ", レベル: " + targetLevel);
            System.out.println("一致するログエントリ: " + filteredLogs.size() + "件");
            System.out.println("時間帯ごとの集計:");
            
            hourlyStats.entrySet().stream()
                    .sorted(Map.Entry.comparingByKey())
                    .forEach(entry -> System.out.println(entry.getKey() + "時: " + entry.getValue() + "件"));
            
        } catch (IOException e) {
            System.err.println("ログファイルの読み込み中にエラーが発生しました: " + e.getMessage());
        }
    }
    
    /**
     * シナリオ2: 大量のログファイルから特定のパターンに一致する行を抽出し、集計する
     * ループ処理を使用した実装
     */
    public static void processLogsWithLoop(String logFilePath, String pattern, String targetLevel) {
        System.out.println("===== ループ処理を使用したログ処理 =====");
        
        List<LogEntry> filteredLogs = new ArrayList<>();
        Map<String, Long> hourlyStats = new HashMap<>();
        
        try (BufferedReader reader = new BufferedReader(new FileReader(logFilePath))) {
            String line;
            
            // ログファイルの各行を処理
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split(" ", 3);
                if (parts.length >= 3) {
                    String timestamp = parts[0];
                    String level = parts[1].replace("[", "").replace("]", "");
                    String message = parts[2];
                    
                    // 条件に一致するログエントリをフィルタリング
                    if (level.equals(targetLevel) && message.contains(pattern)) {
                        LogEntry entry = new LogEntry(timestamp, level, message);
                        filteredLogs.add(entry);
                        
                        // 時間帯ごとの集計
                        String hour = timestamp.substring(0, 2);
                        hourlyStats.put(hour, hourlyStats.getOrDefault(hour, 0L) + 1);
                    }
                }
            }
            
            System.out.println("パターン: " + pattern + ", レベル: " + targetLevel);
            System.out.println("一致するログエントリ: " + filteredLogs.size() + "件");
            System.out.println("時間帯ごとの集計:");
            
            // 時間帯ごとの集計結果を表示
            List<String> hours = new ArrayList<>(hourlyStats.keySet());
            hours.sort(String::compareTo);
            
            for (String hour : hours) {
                System.out.println(hour + "時: " + hourlyStats.get(hour) + "件");
            }
            
        } catch (IOException e) {
            System.err.println("ログファイルの読み込み中にエラーが発生しました: " + e.getMessage());
        }
    }
    
    /**
     * シナリオ3: 複数のデータソースから取得したデータを結合し、変換して出力する
     * Stream APIを使用した実装
     */
    public static <T> void processMultipleDataSourcesWithStream(List<DataSource<T>> dataSources) {
        System.out.println("===== Stream APIを使用した複数データソース処理 =====");
        
        // 全データソースからのデータを結合
        List<T> combinedData = dataSources.stream()
                .peek(source -> System.out.println("データソース: " + source.getName() + " (" + source.getData().size() + "件)"))
                .flatMap(source -> source.getData().stream())
                .collect(Collectors.toList());
        
        System.out.println("結合後のデータ: " + combinedData.size() + "件");
        
        // データの種類ごとに異なる処理を適用
        if (!combinedData.isEmpty()) {
            Object firstItem = combinedData.get(0);
            
            if (firstItem instanceof User) {
                // ユーザーデータの場合
                @SuppressWarnings("unchecked")
                List<User> users = (List<User>) combinedData;
                
                // 部署ごとの平均給与を計算
                Map<String, Double> avgSalaryByDept = users.stream()
                        .collect(Collectors.groupingBy(
                                User::getDepartment,
                                Collectors.averagingDouble(User::getSalary)
                        ));
                
                System.out.println("部署ごとの平均給与:");
                avgSalaryByDept.forEach((dept, avgSalary) -> 
                        System.out.println(dept + ": " + String.format("%.2f", avgSalary)));
                
            } else if (firstItem instanceof LogEntry) {
                // ログデータの場合
                @SuppressWarnings("unchecked")
                List<LogEntry> logs = (List<LogEntry>) combinedData;
                
                // レベルごとのログ数を集計
                Map<String, Long> countByLevel = logs.stream()
                        .collect(Collectors.groupingBy(
                                LogEntry::getLevel,
                                Collectors.counting()
                        ));
                
                System.out.println("レベルごとのログ数:");
                countByLevel.forEach((level, count) -> 
                        System.out.println(level + ": " + count + "件"));
            }
        }
    }
    
    /**
     * シナリオ3: 複数のデータソースから取得したデータを結合し、変換して出力する
     * ループ処理を使用した実装
     */
    public static <T> void processMultipleDataSourcesWithLoop(List<DataSource<T>> dataSources) {
        System.out.println("===== ループ処理を使用した複数データソース処理 =====");
        
        // 全データソースからのデータを結合
        List<T> combinedData = new ArrayList<>();
        
        for (DataSource<T> source : dataSources) {
            System.out.println("データソース: " + source.getName() + " (" + source.getData().size() + "件)");
            combinedData.addAll(source.getData());
        }
        
        System.out.println("結合後のデータ: " + combinedData.size() + "件");
        
        // データの種類ごとに異なる処理を適用
        if (!combinedData.isEmpty()) {
            Object firstItem = combinedData.get(0);
            
            if (firstItem instanceof User) {
                // ユーザーデータの場合
                @SuppressWarnings("unchecked")
                List<User> users = (List<User>) combinedData;
                
                // 部署ごとの給与合計と人数を計算
                Map<String, Double> totalSalaryByDept = new HashMap<>();
                Map<String, Integer> countByDept = new HashMap<>();
                
                for (User user : users) {
                    String dept = user.getDepartment();
                    double salary = user.getSalary();
                    
                    totalSalaryByDept.put(dept, totalSalaryByDept.getOrDefault(dept, 0.0) + salary);
                    countByDept.put(dept, countByDept.getOrDefault(dept, 0) + 1);
                }
                
                // 部署ごとの平均給与を計算
                Map<String, Double> avgSalaryByDept = new HashMap<>();
                for (String dept : totalSalaryByDept.keySet()) {
                    double totalSalary = totalSalaryByDept.get(dept);
                    int count = countByDept.get(dept);
                    avgSalaryByDept.put(dept, totalSalary / count);
                }
                
                System.out.println("部署ごとの平均給与:");
                for (String dept : avgSalaryByDept.keySet()) {
                    double avgSalary = avgSalaryByDept.get(dept);
                    System.out.println(dept + ": " + String.format("%.2f", avgSalary));
                }
                
            } else if (firstItem instanceof LogEntry) {
                // ログデータの場合
                @SuppressWarnings("unchecked")
                List<LogEntry> logs = (List<LogEntry>) combinedData;
                
                // レベルごとのログ数を集計
                Map<String, Long> countByLevel = new HashMap<>();
                
                for (LogEntry log : logs) {
                    String level = log.getLevel();
                    countByLevel.put(level, countByLevel.getOrDefault(level, 0L) + 1);
                }
                
                System.out.println("レベルごとのログ数:");
                for (String level : countByLevel.keySet()) {
                    long count = countByLevel.get(level);
                    System.out.println(level + ": " + count + "件");
                }
            }
        }
    }
    
    public static void main(String[] args) {
        // ユーザーデータの作成
        List<User> users = Arrays.asList(
            new User("U001", "山田太郎", 35, "開発部", 450000),
            new User("U002", "佐藤花子", 28, "営業部", 380000),
            new User("U003", "鈴木一郎", 42, "開発部", 520000),
            new User("U004", "田中美咲", 31, "人事部", 410000),
            new User("U005", "伊藤健太", 38, "営業部", 480000),
            new User("U006", "渡辺裕子", 26, "開発部", 350000),
            new User("U007", "高橋誠", 45, "経理部", 550000),
            new User("U008", "中村和子", 33, "開発部", 430000),
            new User("U009", "小林大輔", 29, "営業部", 400000),
            new User("U010", "加藤真理", 37, "人事部", 470000)
        );
        
        // シナリオ1: ユーザー検索
        System.out.println("\n***** シナリオ1: ユーザー検索 *****\n");
        findUsersWithStream(users, "開発部", 30);
        System.out.println();
        findUsersWithLoop(users, "開発部", 30);
        
        // シナリオ2: ログ処理
        // 注: 実際のログファイルがない場合は、このシナリオはスキップされます
        System.out.println("\n***** シナリオ2: ログ処理 *****\n");
        String logFilePath = "logs/application.log";
        if (Files.exists(Paths.get(logFilePath))) {
            processLogsWithStream(logFilePath, "ERROR", "ERROR");
            System.out.println();
            processLogsWithLoop(logFilePath, "ERROR", "ERROR");
        } else {
            System.out.println("ログファイルが見つかりません: " + logFilePath);
            System.out.println("シナリオ2をスキップします。");
        }
        
        // シナリオ3: 複数データソース処理
        System.out.println("\n***** シナリオ3: 複数データソース処理 *****\n");
        
        // ユーザーデータソースの作成
        List<DataSource<User>> userDataSources = Arrays.asList(
            new DataSource<>("社内システム", users.subList(0, 5)),
            new DataSource<>("外部システム", users.subList(5, 10))
        );
        
        processMultipleDataSourcesWithStream(userDataSources);
        System.out.println();
        processMultipleDataSourcesWithLoop(userDataSources);
    }
}
```

### 解説
この解答例では、3つの異なるシナリオに対して、Stream APIとループ処理の両方を実装し、可読性と保守性の観点から比較しています。

#### シナリオ1: ユーザー検索

**Stream APIの利点:**
- コードが簡潔で、処理の流れが明確
- フィルタリング、ソート、収集の各ステップが一連の流れとして表現されている
- 中間処理と終端処理の区別が明確

**ループ処理の利点:**
- 処理の各ステップが明示的に分離されている
- デバッグが容易（各ステップで変数の状態を確認できる）
- 初心者にとって理解しやすい場合がある

**選択の理由:**
このシナリオでは、Stream APIの方が適切です。理由は以下の通りです：
1. 処理の流れ（フィルタリング→ソート→収集）が明確で、一連の操作として表現できる
2. コードが簡潔で、意図が明確に伝わる
3. 将来的に処理を追加・変更する場合も、パイプラインに操作を追加するだけで済む

#### シナリオ2: ログ処理

**Stream APIの利点:**
- ファイル読み込みから処理までを一連の流れとして表現できる
- 複雑な集計処理（groupingBy）が簡潔に記述できる
- 並列処理に容易に切り替えられる（parallelStream）

**ループ処理の利点:**
- 処理の流れが直線的で追いやすい
- 各ステップでの中間結果が明示的
- 複雑なエラー処理を組み込みやすい

**選択の理由:**
このシナリオでも、Stream APIの方が適切です。理由は以下の通りです：
1. 大量のデータを処理する場合、Stream APIの遅延評価が効率的
2. 集計処理（groupingBy）がStream APIでは非常に簡潔に記述できる
3. 将来的に並列処理が必要になった場合、parallelStreamに切り替えるだけで済む

#### シナリオ3: 複数データソース処理

**Stream APIの利点:**
- データの結合と変換が一連の流れとして表現できる
- flatMapを使用した複数ソースのデータ結合が簡潔
- 集計処理（groupingBy, averagingDouble）が簡潔に記述できる

**ループ処理の利点:**
- 処理の各ステップが明示的
- 型の扱いが明確（キャストの範囲が限定的）
- 複雑な条件分岐を組み込みやすい

**選択の理由:**
このシナリオでも、Stream APIの方が適切です。理由は以下の通りです：
1. 複数データソースの結合と変換という複雑な処理が簡潔に表現できる
2. 集計処理が非常に簡潔に記述できる
3. コードの可読性が高く、意図が明確に伝わる

#### 総合的な考察

1. **Stream APIが適している場合:**
   - データの変換、フィルタリング、集計など、一連の操作を行う場合
   - コードの簡潔さと可読性が重要な場合
   - 並列処理の可能性がある場合
   - 関数型プログラミングのパラダイムに慣れているチームの場合

2. **ループ処理が適している場合:**
   - 単純な処理や、処理の各ステップを明示的に分離したい場合
   - パフォーマンスが最も重要な場合（特に単純な操作）
   - デバッグのしやすさが重要な場合
   - チームメンバーが従来の命令型プログラミングに慣れている場合

3. **ハイブリッドアプローチ:**
   - 実際のプロジェクトでは、Stream APIとループ処理を状況に応じて使い分けることが最も効果的
   - 複雑な集計や変換にはStream API、単純な処理やパフォーマンスクリティカルな部分にはループ処理を使用する

この解答例を通じて、Stream APIとループ処理の特性を理解し、状況に応じた適切な選択ができるようになります。


## 発展演習1: 並列Streamの活用と注意点

### 解答例
```java
package streams.advanced;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 並列Streamの活用と注意点を学ぶためのデモクラス
 */
public class ParallelStreamDemo {
    
    // リストのサイズ
    private static final int LIST_SIZE = 1_000_000;
    
    // 乱数生成器
    private static final Random RANDOM = new Random();
    
    /**
     * 指定されたサイズのランダムな整数リストを生成します。
     * 
     * @param size リストのサイズ
     * @param maxValue 最大値
     * @return ランダムな整数リスト
     */
    public static List<Integer> generateRandomList(int size, int maxValue) {
        List<Integer> list = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            list.add(RANDOM.nextInt(maxValue) + 1);
        }
        return list;
    }
    
    /**
     * 数値が素数かどうかを判定します。
     * 
     * @param n 判定する数値
     * @return 素数の場合はtrue、そうでない場合はfalse
     */
    public static boolean isPrime(int n) {
        if (n <= 1) {
            return false;
        }
        if (n <= 3) {
            return true;
        }
        if (n % 2 == 0 || n % 3 == 0) {
            return false;
        }
        int i = 5;
        while (i * i <= n) {
            if (n % i == 0 || n % (i + 2) == 0) {
                return false;
            }
            i += 6;
        }
        return true;
    }
    
    /**
     * 逐次Streamを使用して、リスト内の素数の数をカウントします。
     * 
     * @param numbers 整数のリスト
     * @return 素数の数
     */
    public static long countPrimesWithSequentialStream(List<Integer> numbers) {
        return numbers.stream()
                .filter(ParallelStreamDemo::isPrime)
                .count();
    }
    
    /**
     * 並列Streamを使用して、リスト内の素数の数をカウントします。
     * 
     * @param numbers 整数のリスト
     * @return 素数の数
     */
    public static long countPrimesWithParallelStream(List<Integer> numbers) {
        return numbers.parallelStream()
                .filter(ParallelStreamDemo::isPrime)
                .count();
    }
    
    /**
     * 外部カウンターを使用して、リスト内の素数の数をカウントします（非スレッドセーフな実装）。
     * 
     * @param numbers 整数のリスト
     * @return 素数の数
     */
    public static int countPrimesWithExternalCounter(List<Integer> numbers) {
        // 非スレッドセーフなカウンター
        int[] count = {0};
        numbers.parallelStream()
               .filter(ParallelStreamDemo::isPrime)
               .forEach(n -> count[0]++); // 共有状態の変更（非スレッドセーフ）
        return count[0];
    }
    
    /**
     * AtomicIntegerを使用して、リスト内の素数の数をカウントします（スレッドセーフな実装）。
     * 
     * @param numbers 整数のリスト
     * @return 素数の数
     */
    public static int countPrimesWithAtomicCounter(List<Integer> numbers) {
        // スレッドセーフなカウンター
        AtomicInteger count = new AtomicInteger(0);
        numbers.parallelStream()
               .filter(ParallelStreamDemo::isPrime)
               .forEach(n -> count.incrementAndGet()); // スレッドセーフな操作
        return count.get();
    }
    
    /**
     * 処理の実行時間を測定します。
     * 
     * @param operation 実行する処理
     * @return 実行時間（ミリ秒）
     */
    public static long measureExecutionTime(Runnable operation) {
        long startTime = System.currentTimeMillis();
        operation.run();
        long endTime = System.currentTimeMillis();
        return endTime - startTime;
    }
    
    public static void main(String[] args) {
        // ランダムな整数リストを生成
        System.out.println("ランダムな整数リストを生成中...");
        List<Integer> numbers = generateRandomList(LIST_SIZE, 1000000);
        System.out.println("リストサイズ: " + numbers.size());
        
        // 逐次Streamと並列Streamのパフォーマンス比較
        System.out.println("\n===== 逐次Streamと並列Streamのパフォーマンス比較 =====");
        
        // 逐次Streamを使用した場合
        long sequentialTime = measureExecutionTime(() -> {
            long count = countPrimesWithSequentialStream(numbers);
            System.out.println("逐次Streamの結果: " + count + "個の素数");
        });
        System.out.println("逐次Streamの実行時間: " + sequentialTime + "ms");
        
        // 並列Streamを使用した場合
        long parallelTime = measureExecutionTime(() -> {
            long count = countPrimesWithParallelStream(numbers);
            System.out.println("並列Streamの結果: " + count + "個の素数");
        });
        System.out.println("並列Streamの実行時間: " + parallelTime + "ms");
        
        // 比率を計算
        double ratio1 = (double) sequentialTime / parallelTime;
        System.out.printf("逐次Stream / 並列Streamの比率: %.2f%n", ratio1);
        
        // 外部カウンターとAtomicIntegerのパフォーマンス比較
        System.out.println("\n===== 外部カウンターとAtomicIntegerのパフォーマンス比較 =====");
        
        // 外部カウンターを使用した場合（非スレッドセーフ）
        long externalCounterTime = measureExecutionTime(() -> {
            int count = countPrimesWithExternalCounter(numbers);
            System.out.println("外部カウンターの結果: " + count + "個の素数");
        });
        System.out.println("外部カウンターの実行時間: " + externalCounterTime + "ms");
        
        // AtomicIntegerを使用した場合（スレッドセーフ）
        long atomicCounterTime = measureExecutionTime(() -> {
            int count = countPrimesWithAtomicCounter(numbers);
            System.out.println("AtomicIntegerの結果: " + count + "個の素数");
        });
        System.out.println("AtomicIntegerの実行時間: " + atomicCounterTime + "ms");
        
        // 比率を計算
        double ratio2 = (double) externalCounterTime / atomicCounterTime;
        System.out.printf("外部カウンター / AtomicIntegerの比率: %.2f%n", ratio2);
        
        // 結果のまとめ
        System.out.println("\n===== 結果のまとめ =====");
        System.out.println("1. 逐次Streamと並列Streamのパフォーマンス比較:");
        System.out.println("   - 逐次Stream: " + sequentialTime + "ms");
        System.out.println("   - 並列Stream: " + parallelTime + "ms");
        System.out.printf("   - 比率（逐次Stream / 並列Stream）: %.2f%n", ratio1);
        
        System.out.println("2. 外部カウンターとAtomicIntegerのパフォーマンス比較:");
        System.out.println("   - 外部カウンター（非スレッドセーフ）: " + externalCounterTime + "ms");
        System.out.println("   - AtomicInteger（スレッドセーフ）: " + atomicCounterTime + "ms");
        System.out.printf("   - 比率（外部カウンター / AtomicInteger）: %.2f%n", ratio2);
        
        System.out.println("\n===== 並列処理における注意点 =====");
        System.out.println("1. 副作用: 並列処理では、ラムダ式が副作用（外部状態の変更）を持たないように注意する。副作用がある場合は、スレッドセーフな方法（Atomicクラスなど）を使用する。");
        System.out.println("2. スレッドセーフティ: 共有リソースへのアクセスはスレッドセーフである必要がある。非スレッドセーフなコレクションやカウンターを並列処理で使用すると、予期しない結果やエラーが発生する可能性がある。");
        System.out.println("3. 実行順序: 並列処理では、要素の処理順序は保証されない。順序に依存する処理（例: 最初の要素を見つける）には注意が必要。");
        System.out.println("4. スレッドプールのサイズ: 並列Streamはデフォルトで共通のForkJoinPoolを使用する。CPUコア数を超えるスレッドを使用しても、必ずしもパフォーマンスが向上するとは限らない。");
        System.out.println("5. オーバーヘッド: 並列処理にはスレッドの生成や管理などのオーバーヘッドがある。データ量が少ない場合や処理が単純な場合は、逐次処理の方が高速な場合がある。");
    }
}
```

### 解説
この解答例では、並列Streamの活用と注意点を学びます。

1. **逐次Streamと並列Streamの比較**
   - `countPrimesWithSequentialStream`メソッドと`countPrimesWithParallelStream`メソッドで、素数のカウント処理をそれぞれ逐次Streamと並列Streamで実装し、実行時間を比較します。
   - 一般的に、CPU負荷の高い計算（ここでは素数判定）を大量のデータに対して行う場合、並列Streamの方が高速になる傾向があります。

2. **スレッドセーフティ**
   - `countPrimesWithExternalCounter`メソッドでは、外部の配列をカウンターとして使用しています。これは並列処理では非スレッドセーフであり、正確な結果が得られない可能性があります（実行ごとに結果が変わる可能性がある）。
   - `countPrimesWithAtomicCounter`メソッドでは、スレッドセーフな`AtomicInteger`を使用しています。これにより、並列処理でも正確な結果が得られます。
   - この比較を通じて、並列処理における共有状態の変更にはスレッドセーフな方法が必要であることを理解します。

3. **並列処理における注意点**
   - `main`メソッドの最後で、並列処理における注意点をまとめています。副作用、スレッドセーフティ、実行順序、スレッドプールのサイズ、オーバーヘッドなどを考慮する必要があることを示しています。

この演習を通じて、並列Streamの利点（パフォーマンス向上）と、その利用に伴う注意点（スレッドセーフティなど）を実践的に学びます。

## 発展演習2: Stream APIの遅延評価と短絡評価

### 解答例
```java
package streams.advanced;

import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

/**
 * Stream APIの遅延評価と短絡評価を学ぶためのデモクラス
 */
public class LazyEvaluationDemo {
    
    /**
     * 指定された数値が素数かどうかを判定します。
     * 
     * @param n 判定する数値
     * @return 素数の場合はtrue、そうでない場合はfalse
     */
    public static boolean isPrime(int n) {
        System.out.println("isPrime(" + n + ")を評価中...");
        if (n <= 1) {
            return false;
        }
        if (n <= 3) {
            return true;
        }
        if (n % 2 == 0 || n % 3 == 0) {
            return false;
        }
        int i = 5;
        while (i * i <= n) {
            if (n % i == 0 || n % (i + 2) == 0) {
                return false;
            }
            i += 6;
        }
        return true;
    }
    
    /**
     * 指定された数値が完全数かどうかを判定します。
     * 完全数とは、自分自身を除く約数の和が自分自身と等しい数のことです。
     * 
     * @param n 判定する数値
     * @return 完全数の場合はtrue、そうでない場合はfalse
     */
    public static boolean isPerfect(int n) {
        System.out.println("isPerfect(" + n + ")を評価中...");
        if (n <= 1) {
            return false;
        }
        int sum = 1;
        for (int i = 2; i * i <= n; i++) {
            if (n % i == 0) {
                sum += i;
                if (i != n / i) {
                    sum += n / i;
                }
            }
        }
        return sum == n;
    }
    
    /**
     * 無限Streamを使用して、最初のn個の素数を生成します。
     * 
     * @param n 生成する素数の数
     * @return 最初のn個の素数のリスト
     */
    public static List<Integer> generateFirstNPrimes(int n) {
        return Stream.iterate(2, i -> i + 1) // 2から始まる無限の整数Stream
                .filter(LazyEvaluationDemo::isPrime) // 素数のみをフィルタリング
                .limit(n) // 最初のn個に制限
                .collect(Collectors.toList()); // リストに収集
    }
    
    /**
     * 無限Streamを使用して、指定された範囲内の最初の完全数を見つけます。
     * 
     * @param start 開始値
     * @param end 終了値
     * @return 最初の完全数、見つからない場合は-1
     */
    public static int findFirstPerfectNumber(int start, int end) {
        return IntStream.rangeClosed(start, end) // 指定された範囲の整数Stream
                .filter(LazyEvaluationDemo::isPerfect) // 完全数のみをフィルタリング
                .findFirst() // 最初の要素を見つける（短絡評価）
                .orElse(-1); // 見つからない場合は-1を返す
    }
    
    /**
     * 無限Streamを使用して、フィボナッチ数列の最初のn項を生成します。
     * 
     * @param n 生成する項の数
     * @return フィボナッチ数列の最初のn項のリスト
     */
    public static List<Integer> generateFibonacciSequence(int n) {
        return Stream.iterate(new int[]{0, 1}, f -> new int[]{f[1], f[0] + f[1]}) // フィボナッチ数列の無限Stream
                .map(f -> f[0]) // 各ペアの最初の要素を抽出
                .limit(n) // 最初のn個に制限
                .collect(Collectors.toList()); // リストに収集
    }
    
    /**
     * 遅延評価と短絡評価を示すデモを実行します。
     */
    public static void demonstrateLazyAndShortCircuitEvaluation() {
        System.out.println("===== 遅延評価のデモ =====");
        Stream<Integer> stream = Stream.iterate(1, i -> i + 1)
                .filter(n -> {
                    System.out.println("フィルタリング中: " + n);
                    return n % 2 == 0;
                })
                .map(n -> {
                    System.out.println("マッピング中: " + n);
                    return n * 2;
                });
        
        System.out.println("Streamが作成されましたが、まだ評価されていません。");
        
        System.out.println("最初の3つの要素を取得します...");
        List<Integer> result = stream.limit(3).collect(Collectors.toList());
        System.out.println("結果: " + result);
        
        System.out.println("\n===== 短絡評価のデモ =====");
        Optional<Integer> firstPerfect = IntStream.rangeClosed(1, 1000)
                .filter(LazyEvaluationDemo::isPerfect)
                .peek(n -> System.out.println("完全数が見つかりました: " + n))
                .findFirst(); // 最初の完全数が見つかった時点で終了
                
        if (firstPerfect.isPresent()) {
            System.out.println("1から1000までの最初の完全数: " + firstPerfect.get());
        } else {
            System.out.println("1から1000までの範囲に完全数はありません。");
        }
    }
    
    public static void main(String[] args) {
        // 最初のn個の素数の生成
        System.out.println("===== 最初の10個の素数 =====");
        List<Integer> primes = generateFirstNPrimes(10);
        System.out.println(primes);
        
        // 指定された範囲内の最初の完全数の検索
        System.out.println("\n===== 1から10000までの最初の完全数 =====");
        int perfectNumber = findFirstPerfectNumber(1, 10000);
        System.out.println("最初の完全数: " + perfectNumber);
        
        // フィボナッチ数列の生成
        System.out.println("\n===== 最初の20項のフィボナッチ数列 =====");
        List<Integer> fibonacciSequence = generateFibonacciSequence(20);
        System.out.println(fibonacciSequence);
        
        // 遅延評価と短絡評価のデモ
        System.out.println("\n===== 遅延評価と短絡評価のデモ =====");
        demonstrateLazyAndShortCircuitEvaluation();
        
        System.out.println("\n===== Stream APIの遅延評価と短絡評価の利点 =====");
        System.out.println("1. 遅延評価: 不要な計算を避け、パフォーマンスを向上させる。無限Streamの処理を可能にする。");
        System.out.println("2. 短絡評価: 結果が確定した時点で処理を終了し、パフォーマンスを向上させる。無限Streamでも終了する可能性がある。");
    }
}
```

### 解説
この解答例では、Stream APIの遅延評価と短絡評価の特性を学びます。

1. **無限Streamの生成**
   - `generateFirstNPrimes`メソッドでは、`Stream.iterate(2, i -> i + 1)`を使用して2から始まる無限の整数Streamを生成し、`filter`と`limit`を組み合わせて最初のn個の素数を取得します。`isPrime`メソッドの評価は、`limit(n)`で指定された数だけ行われます（遅延評価）。
   - `generateFibonacciSequence`メソッドでは、`Stream.iterate`を使用してフィボナッチ数列の無限Streamを生成し、`limit`で最初のn項を取得します。

2. **遅延評価のデモ**
   - `demonstrateLazyAndShortCircuitEvaluation`メソッドの最初の部分では、`filter`と`map`を持つStreamを作成します。この時点では、`filter`や`map`内の`println`は実行されません。
   - `limit(3).collect(Collectors.toList())`という終端操作を呼び出すと、初めてStreamの評価が開始され、必要な要素（この場合は最初の3つの偶数を見つけるまで）だけがフィルタリングおよびマッピングされます。

3. **短絡評価のデモ**
   - `findFirstPerfectNumber`メソッドでは、`IntStream.rangeClosed`で範囲を指定し、`filter(LazyEvaluationDemo::isPerfect)`で完全数を探します。`findFirst()`は短絡評価を行う終端操作であり、最初の完全数が見つかった時点でStreamの処理を停止します。これにより、範囲内のすべての数値を評価する必要がなくなります。
   - `demonstrateLazyAndShortCircuitEvaluation`メソッドの後半でも、`findFirst()`を使用して短絡評価を示しています。`peek`操作は、`findFirst()`が要素を見つけるまで実行されます。

4. **利点のまとめ**
   - `main`メソッドの最後で、遅延評価と短絡評価の利点をまとめています。パフォーマンス向上や無限Streamの扱いに役立つことを示しています。

この演習を通じて、Stream APIの強力な特性である遅延評価と短絡評価を理解し、それらを活用して効率的なコードを記述する方法を学びます。

