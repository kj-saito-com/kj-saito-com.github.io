---
layout: default
title: "Day 5: Stream APIとループ処理の比較と選択"
---
# Day 5: Stream APIとループ処理の比較と選択

## 学習の目的と背景

Stream APIはJava 8で導入された機能で、コレクションの要素を宣言的に処理するための強力な手段を提供します。従来のループ処理と比較して、コードの可読性向上や並列処理の容易さなどの利点がありますが、適切な使用場面を理解することが重要です。

本日の学習では、Stream APIの基本から応用までを体系的に学び、従来のループ処理との比較を通じて、それぞれの長所と短所を理解し、適切な選択ができるようになることを目指します。また、パフォーマンスの観点からも両者を比較し、実務での効果的な活用方法を習得します。

## 前提知識と準備

### 必要な前提知識
- Javaの基本構文（変数、制御構文、クラスなど）
- コレクションフレームワークの基本（List, Set, Mapなど）
- ラムダ式の基本構文

### 環境準備
- Eclipse 2023-12 (4.30.0)
- Java 21

### プロジェクト作成手順

1. Eclipseを起動します。
2. 「File」→「New」→「Java Project」を選択します。
3. プロジェクト名を「JavaStreams_Day5」と入力します。
4. 「JRE」の設定で「Use an execution environment JRE:」から「JavaSE-21」を選択します。
5. 「Finish」をクリックしてプロジェクトを作成します。

## 基本概念の解説

### 1. Stream APIの概要

Stream APIは、コレクションの要素を宣言的に処理するためのAPIです。従来のループ処理と異なり、「何を行うか」を記述し、「どのように行うか」の詳細は抽象化されています。

**用語解説: Stream（ストリーム）**  
要素の連続したシーケンスを表す抽象概念です。コレクションとは異なり、要素を保持するデータ構造ではなく、要素を処理するためのパイプラインを提供します。

**用語解説: パイプライン**  
Streamの処理は、パイプラインと呼ばれる一連の操作で構成されます。パイプラインは、ソース（コレクションなど）、中間操作（filter, map, sortなど）、終端操作（collect, forEach, reduceなど）で構成されます。

### 2. Stream APIの基本構造

Stream APIの処理は、以下の3つの部分から構成されます：

1. **ソース（Source）**：
   - Streamを生成するデータソース
   - 例：コレクション、配列、I/Oチャネル、ジェネレータ関数

2. **中間操作（Intermediate Operations）**：
   - Streamを変換する操作
   - 遅延評価される（終端操作が呼び出されるまで実行されない）
   - 例：filter, map, flatMap, distinct, sorted, limit, skip

3. **終端操作（Terminal Operations）**：
   - Streamを消費し、結果を生成する操作
   - パイプラインの実行をトリガーする
   - 例：forEach, collect, reduce, count, min, max, anyMatch, allMatch, noneMatch

### 3. Stream APIの主要な操作

#### 中間操作

**filter**：条件に一致する要素のみを含む新しいStreamを返します。
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
List<String> filteredNames = names.stream()
                                 .filter(name -> name.startsWith("A"))
                                 .collect(Collectors.toList());
// 結果: [Alice]
```

**map**：各要素に関数を適用し、結果を含む新しいStreamを返します。
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
List<Integer> nameLengths = names.stream()
                                .map(String::length)
                                .collect(Collectors.toList());
// 結果: [5, 3, 7]
```

**flatMap**：各要素をStreamに変換し、それらを1つのStreamに連結します。
```java
List<List<Integer>> nestedLists = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);
List<Integer> flattenedList = nestedLists.stream()
                                        .flatMap(Collection::stream)
                                        .collect(Collectors.toList());
// 結果: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**distinct**：重複を除去した新しいStreamを返します。
```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 4, 4, 4);
List<Integer> distinctNumbers = numbers.stream()
                                      .distinct()
                                      .collect(Collectors.toList());
// 結果: [1, 2, 3, 4]
```

**sorted**：要素をソートした新しいStreamを返します。
```java
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
List<String> sortedNames = names.stream()
                               .sorted()
                               .collect(Collectors.toList());
// 結果: [Alice, Bob, Charlie]
```

**limit**：指定された数の要素に制限した新しいStreamを返します。
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> limitedNumbers = numbers.stream()
                                     .limit(3)
                                     .collect(Collectors.toList());
// 結果: [1, 2, 3]
```

**skip**：指定された数の要素をスキップした新しいStreamを返します。
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> skippedNumbers = numbers.stream()
                                     .skip(2)
                                     .collect(Collectors.toList());
// 結果: [3, 4, 5]
```

#### 終端操作

**forEach**：各要素に対してアクションを実行します。
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.stream().forEach(System.out::println);
// 結果: Alice, Bob, Charlie（各要素が別々の行に出力される）
```

**collect**：Streamの要素を集約して、コレクションなどの結果を返します。
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
Set<String> nameSet = names.stream()
                          .collect(Collectors.toSet());
// 結果: [Alice, Bob, Charlie]（Setとして）
```

**reduce**：Streamの要素を結合して、単一の結果を返します。
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream()
                .reduce(0, Integer::sum);
// 結果: 15
```

**count**：Streamの要素数を返します。
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
long count = names.stream().count();
// 結果: 3
```

**min/max**：Streamの最小/最大要素を返します。
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> min = numbers.stream().min(Integer::compareTo);
Optional<Integer> max = numbers.stream().max(Integer::compareTo);
// 結果: min = 1, max = 5
```

**anyMatch/allMatch/noneMatch**：Streamの要素が条件に一致するかどうかを返します。
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
boolean anyEven = numbers.stream().anyMatch(n -> n % 2 == 0);
boolean allEven = numbers.stream().allMatch(n -> n % 2 == 0);
boolean noneEven = numbers.stream().noneMatch(n -> n % 2 == 0);
// 結果: anyEven = true, allEven = false, noneEven = false
```

### 4. Stream APIの特徴

#### 遅延評価（Lazy Evaluation）

Stream APIの中間操作は遅延評価されます。つまり、終端操作が呼び出されるまで実行されません。これにより、不要な計算を避け、パフォーマンスを向上させることができます。

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
Stream<String> stream = names.stream()
                           .filter(name -> {
                               System.out.println("Filtering: " + name);
                               return name.startsWith("A");
                           })
                           .map(name -> {
                               System.out.println("Mapping: " + name);
                               return name.toUpperCase();
                           });
// この時点では何も出力されない

List<String> result = stream.collect(Collectors.toList());
// 出力:
// Filtering: Alice
// Mapping: Alice
// Filtering: Bob
// Filtering: Charlie
// Filtering: David
```

#### 短絡評価（Short-Circuit Evaluation）

一部の終端操作（findFirst, findAny, anyMatch, allMatch, noneMatchなど）は短絡評価をサポートしています。つまり、結果が確定した時点で処理を終了します。

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
boolean result = names.stream()
                     .filter(name -> {
                         System.out.println("Filtering: " + name);
                         return name.startsWith("A");
                     })
                     .anyMatch(name -> {
                         System.out.println("Matching: " + name);
                         return name.length() > 3;
                     });
// 出力:
// Filtering: Alice
// Matching: Alice
// （Bobなど他の要素は処理されない）
```

#### 並列処理（Parallel Processing）

Stream APIは並列処理をサポートしています。`parallelStream()`メソッドを使用するか、`parallel()`メソッドを呼び出すことで、並列Streamを作成できます。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
int sum = numbers.parallelStream()
                .filter(n -> n % 2 == 0)
                .mapToInt(n -> n)
                .sum();
// 結果: 30（2 + 4 + 6 + 8 + 10）
```

### 5. Stream APIとループ処理の比較

#### 可読性と表現力

Stream APIは、宣言的なスタイルでコードを記述できるため、「何を行うか」が明確になり、可読性が向上します。一方、従来のループ処理は、「どのように行うか」の詳細を記述するため、コードが冗長になる場合があります。

**Stream API**：
```java
List<String> filteredNames = names.stream()
                                 .filter(name -> name.startsWith("A"))
                                 .map(String::toUpperCase)
                                 .collect(Collectors.toList());
```

**ループ処理**：
```java
List<String> filteredNames = new ArrayList<>();
for (String name : names) {
    if (name.startsWith("A")) {
        filteredNames.add(name.toUpperCase());
    }
}
```

#### パフォーマンス

一般的に、単純な操作では従来のループ処理の方がパフォーマンスが良い場合があります。これは、Stream APIのオーバーヘッド（ラムダ式の生成、Streamの作成など）が原因です。ただし、並列処理が必要な場合や、複雑な操作が必要な場合は、Stream APIの方が効率的な場合があります。

**計算量（オーダー記法）の観点**：
- 基本的な操作（filter, map, forEachなど）：O(n)
- sorted：O(n log n)
- distinct：O(n)（ハッシュベースの場合）

#### 使い分けの指針

以下の場合は、Stream APIの使用を検討してください：
- 複数の変換操作を連鎖させる場合
- 並列処理が必要な場合
- コードの可読性を重視する場合
- 関数型プログラミングのスタイルを採用する場合

以下の場合は、従来のループ処理の使用を検討してください：
- 単純な操作の場合
- パフォーマンスが最も重要な場合
- 副作用（変数の変更など）が必要な場合
- デバッグが容易である必要がある場合

## Eclipse操作ガイド

### クラスの作成

1. プロジェクト「JavaStreams_Day5」を右クリックします。
2. 「New」→「Class」を選択します。
3. 「Name」に作成するクラス名（例：`StreamDemo`）を入力します。
4. 「public static void main(String[] args)」にチェックを入れます。
5. 「Finish」をクリックしてクラスを作成します。

### Stream APIのコード補完

1. Stream APIを使用するコードを記述します。
2. メソッド名の一部を入力し、「Ctrl + Space」を押して、コード補完のメニューを表示します。
3. 適切なメソッドを選択します。

### Stream APIのデバッグ

1. Stream APIを使用するコードにブレークポイントを設定します（行番号の左側をダブルクリック）。
2. 右クリックして「Debug As」→「Java Application」を選択します。
3. デバッグパースペクティブで変数の値を確認します。
4. 「Step Over」（F6）などのボタンで実行を制御します。
5. Stream APIの中間操作は遅延評価されるため、終端操作が呼び出されるまでブレークポイントが有効にならない場合があります。

### ループからStreamへのリファクタリング

1. 従来のループ処理を選択します。
2. 右クリックして「Refactor」→「Convert to Stream」を選択します。
3. Eclipseが自動的にStream APIを使用したコードに変換します。

## コア演習（必須）

### 演習1: 基本的なStream操作（難易度：基本）

#### 課題
以下の操作を行うプログラムを作成してください：
1. 整数のリストから偶数のみをフィルタリングする
2. 文字列のリストから特定の条件に一致する要素を抽出する
3. オブジェクトのリストから特定のフィールドを抽出する

#### 雛形コード
以下のクラスを作成してください：

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
        // TODO: ここにStream APIを使用したコードを実装
        // ヒント1: streamメソッドを使用してStreamを作成
        // ヒント2: filterメソッドを使用して偶数のみをフィルタリング
        // ヒント3: collectメソッドを使用して結果をリストに変換
        
        return null; // 仮の戻り値
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
        // TODO: ここにStream APIを使用したコードを実装
        // ヒント1: streamメソッドを使用してStreamを作成
        // ヒント2: filterメソッドを使用して条件に一致する文字列のみをフィルタリング
        // ヒント3: collectメソッドを使用して結果をリストに変換
        
        return null; // 仮の戻り値
    }
    
    /**
     * 人物のリストから、指定された年齢以上の人物の名前のみを抽出します。
     * 
     * @param people 人物のリスト
     * @param minAge 最小の年齢
     * @return 条件に一致する人物の名前のみを含むリスト
     */
    public static List<String> extractNames(List<Person> people, int minAge) {
        // TODO: ここにStream APIを使用したコードを実装
        // ヒント1: streamメソッドを使用してStreamを作成
        // ヒント2: filterメソッドを使用して条件に一致する人物のみをフィルタリング
        // ヒント3: mapメソッドを使用して人物から名前を抽出
        // ヒント4: collectメソッドを使用して結果をリストに変換
        
        return null; // 仮の戻り値
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

#### 実装ヒント
1. `filterEvenNumbers`メソッドでは、`stream()`メソッドを使用してStreamを作成し、`filter(n -> n % 2 == 0)`を使用して偶数のみをフィルタリングします。最後に`collect(Collectors.toList())`を使用して結果をリストに変換します。
2. `filterStrings`メソッドでは、`stream()`メソッドを使用してStreamを作成し、`filter(s -> s.startsWith(String.valueOf(startChar)) && s.length() >= minLength)`を使用して条件に一致する文字列のみをフィルタリングします。
3. `extractNames`メソッドでは、`stream()`メソッドを使用してStreamを作成し、`filter(p -> p.getAge() >= minAge)`を使用して条件に一致する人物のみをフィルタリングし、`map(Person::getName)`を使用して人物から名前を抽出します。

### 演習2: Stream APIとループ処理のパフォーマンス比較（難易度：応用）

#### 課題
Stream APIと従来のループ処理のパフォーマンスを比較するプログラムを作成してください。以下の操作について、両者の実行時間を測定し、比較してください：
1. 大きな整数リストからの偶数のフィルタリングと合計計算
2. 大きな整数リストのソートと上位N個の要素の抽出
3. 大きな整数リストの並列処理と逐次処理の比較

#### 雛形コード
以下のクラスを作成してください：

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
        // TODO: ここにStream APIを使用したコードを実装
        // ヒント1: streamメソッドを使用してStreamを作成
        // ヒント2: filterメソッドを使用して偶数のみをフィルタリング
        // ヒント3: mapToIntメソッドを使用してIntStreamに変換
        // ヒント4: sumメソッドを使用して合計を計算
        
        return 0; // 仮の戻り値
    }
    
    /**
     * ループ処理を使用して、リストから偶数のみをフィルタリングし、合計を計算します。
     * 
     * @param numbers 整数のリスト
     * @return 偶数の合計
     */
    public static int sumEvenNumbersWithLoop(List<Integer> numbers) {
        // TODO: ここにループ処理を使用したコードを実装
        // ヒント1: forループを使用してリストを走査
        // ヒント2: if文を使用して偶数のみをフィルタリング
        // ヒント3: 合計を計算
        
        return 0; // 仮の戻り値
    }
    
    /**
     * Stream APIを使用して、リストをソートし、上位N個の要素を抽出します。
     * 
     * @param numbers 整数のリスト
     * @param n 抽出する要素の数
     * @return 上位N個の要素を含むリスト
     */
    public static List<Integer> topNWithStream(List<Integer> numbers, int n) {
        // TODO: ここにStream APIを使用したコードを実装
        // ヒント1: streamメソッドを使用してStreamを作成
        // ヒント2: sortedメソッドを使用してソート
        // ヒント3: limitメソッドを使用して上位N個の要素を抽出
        // ヒント4: collectメソッドを使用して結果をリストに変換
        
        return null; // 仮の戻り値
    }
    
    /**
     * ループ処理を使用して、リストをソートし、上位N個の要素を抽出します。
     * 
     * @param numbers 整数のリスト
     * @param n 抽出する要素の数
     * @return 上位N個の要素を含むリスト
     */
    public static List<Integer> topNWithLoop(List<Integer> numbers, int n) {
        // TODO: ここにループ処理を使用したコードを実装
        // ヒント1: リストのコピーを作成
        // ヒント2: Collectionsクラスのsortメソッドを使用してソート
        // ヒント3: サブリストを作成して上位N個の要素を抽出
        
        return null; // 仮の戻り値
    }
    
    /**
     * 並列Stream APIを使用して、リストから偶数のみをフィルタリングし、合計を計算します。
     * 
     * @param numbers 整数のリスト
     * @return 偶数の合計
     */
    public static int sumEvenNumbersWithParallelStream(List<Integer> numbers) {
        // TODO: ここに並列Stream APIを使用したコードを実装
        // ヒント1: parallelStreamメソッドを使用して並列Streamを作成
        // ヒント2: filterメソッドを使用して偶数のみをフィルタリング
        // ヒント3: mapToIntメソッドを使用してIntStreamに変換
        // ヒント4: sumメソッドを使用して合計を計算
        
        return 0; // 仮の戻り値
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
        List<Integer> numbers = generateRandomList(LIST_SIZE);
        System.out.println("リストサイズ: " + numbers.size());
        
        // 偶数の合計計算のパフォーマンス比較
        System.out.println("\n===== 偶数の合計計算のパフォーマンス比較 =====");
        
        // Stream APIを使用した場合
        long streamTime = measureExecutionTime(() -> {
            int sum = sumEvenNumbersWithStream(numbers);
            System.out.println("Stream APIの結果: " + sum);
        });
        System.out.println("Stream APIの実行時間: " + streamTime + "ms");
        
        // ループ処理を使用した場合
        long loopTime = measureExecutionTime(() -> {
            int sum = sumEvenNumbersWithLoop(numbers);
            System.out.println("ループ処理の結果: " + sum);
        });
        System.out.println("ループ処理の実行時間: " + loopTime + "ms");
        
        // 比率を計算
        double ratio1 = (double) streamTime / loopTime;
        System.out.printf("Stream API / ループ処理の比率: %.2f%n", ratio1);
        
        // ソートと上位N個の要素抽出のパフォーマンス比較
        System.out.println("\n===== ソートと上位N個の要素抽出のパフォーマンス比較 =====");
        final int n = 10;
        
        // Stream APIを使用した場合
        long streamSortTime = measureExecutionTime(() -> {
            List<Integer> topN = topNWithStream(numbers, n);
            System.out.println("Stream APIの結果（先頭3つ）: " + topN.subList(0, Math.min(3, topN.size())));
        });
        System.out.println("Stream APIの実行時間: " + streamSortTime + "ms");
        
        // ループ処理を使用した場合
        long loopSortTime = measureExecutionTime(() -> {
            List<Integer> topN = topNWithLoop(numbers, n);
            System.out.println("ループ処理の結果（先頭3つ）: " + topN.subList(0, Math.min(3, topN.size())));
        });
        System.out.println("ループ処理の実行時間: " + loopSortTime + "ms");
        
        // 比率を計算
        double ratio2 = (double) streamSortTime / loopSortTime;
        System.out.printf("Stream API / ループ処理の比率: %.2f%n", ratio2);
        
        // 並列処理と逐次処理のパフォーマンス比較
        System.out.println("\n===== 並列処理と逐次処理のパフォーマンス比較 =====");
        
        // 逐次Stream APIを使用した場合
        long sequentialStreamTime = measureExecutionTime(() -> {
            int sum = sumEvenNumbersWithStream(numbers);
            System.out.println("逐次Stream APIの結果: " + sum);
        });
        System.out.println("逐次Stream APIの実行時間: " + sequentialStreamTime + "ms");
        
        // 並列Stream APIを使用した場合
        long parallelStreamTime = measureExecutionTime(() -> {
            int sum = sumEvenNumbersWithParallelStream(numbers);
            System.out.println("並列Stream APIの結果: " + sum);
        });
        System.out.println("並列Stream APIの実行時間: " + parallelStreamTime + "ms");
        
        // 比率を計算
        double ratio3 = (double) sequentialStreamTime / parallelStreamTime;
        System.out.printf("逐次Stream API / 並列Stream APIの比率: %.2f%n", ratio3);
        
        // 結果のまとめ
        System.out.println("\n===== 結果のまとめ =====");
        System.out.println("1. 偶数の合計計算:");
        System.out.println("   - Stream API: " + streamTime + "ms");
        System.out.println("   - ループ処理: " + loopTime + "ms");
        System.out.printf("   - 比率（Stream API / ループ処理）: %.2f%n", ratio1);
        
        System.out.println("2. ソートと上位N個の要素抽出:");
        System.out.println("   - Stream API: " + streamSortTime + "ms");
        System.out.println("   - ループ処理: " + loopSortTime + "ms");
        System.out.printf("   - 比率（Stream API / ループ処理）: %.2f%n", ratio2);
        
        System.out.println("3. 並列処理と逐次処理:");
        System.out.println("   - 逐次Stream API: " + sequentialStreamTime + "ms");
        System.out.println("   - 並列Stream API: " + parallelStreamTime + "ms");
        System.out.printf("   - 比率（逐次Stream API / 並列Stream API）: %.2f%n", ratio3);
        
        System.out.println("\n===== 考察 =====");
        // TODO: ここに測定結果の考察を記述
    }
}
```

#### 実装ヒント
1. `sumEvenNumbersWithStream`メソッドでは、`stream()`メソッドを使用してStreamを作成し、`filter(n -> n % 2 == 0)`を使用して偶数のみをフィルタリングし、`mapToInt(n -> n)`を使用してIntStreamに変換し、`sum()`を使用して合計を計算します。
2. `sumEvenNumbersWithLoop`メソッドでは、forループを使用してリストを走査し、if文を使用して偶数のみをフィルタリングし、合計を計算します。
3. `topNWithStream`メソッドでは、`stream()`メソッドを使用してStreamを作成し、`sorted()`を使用してソートし、`limit(n)`を使用して上位N個の要素を抽出し、`collect(Collectors.toList())`を使用して結果をリストに変換します。
4. `topNWithLoop`メソッドでは、リストのコピーを作成し、`Collections.sort()`を使用してソートし、`subList(0, Math.min(n, list.size()))`を使用して上位N個の要素を抽出します。
5. `sumEvenNumbersWithParallelStream`メソッドでは、`parallelStream()`メソッドを使用して並列Streamを作成し、それ以外は`sumEvenNumbersWithStream`メソッドと同様に実装します。

### 演習3: 可読性と保守性の観点からの選択（難易度：応用）

#### 課題
以下のシナリオに対して、Stream APIとループ処理のどちらが適切かを判断し、その理由を説明してください。また、選択した方法で実装してください：
1. ユーザーのリストから特定の条件に一致するユーザーを検索し、その情報を表示する
2. 大量のログファイルから特定のパターンに一致する行を抽出し、集計する
3. 複数のデータソースから取得したデータを結合し、変換して出力する

#### 雛形コード
以下のクラスを作成してください：

```java
package streams.readability;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

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
            return "User [id=" + id + ", name=" + name + ", age=" + age + ", department=" + department + ", salary=" + salary + "]";
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
     * 製品を表すクラス
     */
    static class Product {
        private String id;
        private String name;
        private double price;
        private String category;
        
        public Product(String id, String name, double price, String category) {
            this.id = id;
            this.name = name;
            this.price = price;
            this.category = category;
        }
        
        public String getId() { return id; }
        public String getName() { return name; }
        public double getPrice() { return price; }
        public String getCategory() { return category; }
        
        @Override
        public String toString() {
            return "Product [id=" + id + ", name=" + name + ", price=" + price + ", category=" + category + "]";
        }
    }
    
    /**
     * 注文を表すクラス
     */
    static class Order {
        private String id;
        private String userId;
        private String productId;
        private int quantity;
        
        public Order(String id, String userId, String productId, int quantity) {
            this.id = id;
            this.userId = userId;
            this.productId = productId;
            this.quantity = quantity;
        }
        
        public String getId() { return id; }
        public String getUserId() { return userId; }
        public String getProductId() { return productId; }
        public int getQuantity() { return quantity; }
        
        @Override
        public String toString() {
            return "Order [id=" + id + ", userId=" + userId + ", productId=" + productId + ", quantity=" + quantity + "]";
        }
    }
    
    /**
     * 注文詳細を表すクラス
     */
    static class OrderDetail {
        private String orderId;
        private String productName;
        private String userName;
        private int quantity;
        private double totalPrice;
        
        public OrderDetail(String orderId, String productName, String userName, int quantity, double totalPrice) {
            this.orderId = orderId;
            this.productName = productName;
            this.userName = userName;
            this.quantity = quantity;
            this.totalPrice = totalPrice;
        }
        
        @Override
        public String toString() {
            return "OrderDetail [orderId=" + orderId + ", productName=" + productName + ", userName=" + userName + ", quantity=" + quantity + ", totalPrice=" + totalPrice + "]";
        }
    }
    
    /**
     * シナリオ1: ユーザーのリストから特定の条件に一致するユーザーを検索し、その情報を表示します。
     * 条件: 30歳以上で、開発部門に所属し、給与が50万円以上のユーザー
     * 
     * @param users ユーザーのリスト
     * @return 条件に一致するユーザーのリスト
     */
    public static List<User> findUsersScenario1(List<User> users) {
        // TODO: ここにStream APIまたはループ処理を使用したコードを実装
        // ヒント1: 条件に一致するユーザーをフィルタリング
        // ヒント2: 可読性と保守性を考慮して実装
        
        return null; // 仮の戻り値
    }
    
    /**
     * シナリオ2: 大量のログファイルから特定のパターンに一致する行を抽出し、集計します。
     * 条件: ERRORレベルのログエントリを抽出し、メッセージごとに出現回数を集計
     * 
     * @param logEntries ログエントリのリスト
     * @return メッセージごとの出現回数を含むマップ
     */
    public static Map<String, Long> analyzeLogsScenario2(List<LogEntry> logEntries) {
        // TODO: ここにStream APIまたはループ処理を使用したコードを実装
        // ヒント1: ERRORレベルのログエントリをフィルタリング
        // ヒント2: メッセージごとに出現回数を集計
        // ヒント3: 可読性と保守性を考慮して実装
        
        return null; // 仮の戻り値
    }
    
    /**
     * シナリオ3: 複数のデータソースから取得したデータを結合し、変換して出力します。
     * 条件: 注文情報と製品情報とユーザー情報を結合し、注文詳細を作成
     * 
     * @param orders 注文のリスト
     * @param products 製品のリスト
     * @param users ユーザーのリスト
     * @return 注文詳細のリスト
     */
    public static List<OrderDetail> processOrdersScenario3(List<Order> orders, List<Product> products, List<User> users) {
        // TODO: ここにStream APIまたはループ処理を使用したコードを実装
        // ヒント1: 注文情報と製品情報とユーザー情報を結合
        // ヒント2: 注文詳細を作成
        // ヒント3: 可読性と保守性を考慮して実装
        
        return null; // 仮の戻り値
    }
    
    public static void main(String[] args) {
        // ユーザーのリスト
        List<User> users = Arrays.asList(
            new User("U001", "山田太郎", 35, "開発", 600000),
            new User("U002", "佐藤花子", 28, "営業", 450000),
            new User("U003", "鈴木一郎", 42, "開発", 800000),
            new User("U004", "田中美咲", 31, "人事", 500000),
            new User("U005", "伊藤健太", 25, "開発", 400000)
        );
        
        // ログエントリのリスト
        List<LogEntry> logEntries = Arrays.asList(
            new LogEntry("2023-01-01 10:00:00", "INFO", "アプリケーションが起動しました"),
            new LogEntry("2023-01-01 10:05:00", "ERROR", "データベース接続エラー"),
            new LogEntry("2023-01-01 10:10:00", "WARN", "メモリ使用率が高くなっています"),
            new LogEntry("2023-01-01 10:15:00", "ERROR", "データベース接続エラー"),
            new LogEntry("2023-01-01 10:20:00", "INFO", "バックアップが完了しました"),
            new LogEntry("2023-01-01 10:25:00", "ERROR", "ファイルが見つかりません"),
            new LogEntry("2023-01-01 10:30:00", "ERROR", "データベース接続エラー")
        );
        
        // 製品のリスト
        List<Product> products = Arrays.asList(
            new Product("P001", "ノートパソコン", 150000, "電子機器"),
            new Product("P002", "スマートフォン", 100000, "電子機器"),
            new Product("P003", "デスクトップPC", 200000, "電子機器"),
            new Product("P004", "プリンター", 50000, "電子機器"),
            new Product("P005", "オフィスチェア", 30000, "家具")
        );
        
        // 注文のリスト
        List<Order> orders = Arrays.asList(
            new Order("O001", "U001", "P001", 1),
            new Order("O002", "U002", "P002", 2),
            new Order("O003", "U003", "P003", 1),
            new Order("O004", "U001", "P004", 3),
            new Order("O005", "U004", "P005", 2)
        );
        
        // シナリオ1: ユーザーのリストから特定の条件に一致するユーザーを検索し、その情報を表示する
        System.out.println("===== シナリオ1: ユーザー検索 =====");
        List<User> filteredUsers = findUsersScenario1(users);
        System.out.println("条件に一致するユーザー:");
        for (User user : filteredUsers) {
            System.out.println(user);
        }
        
        // シナリオ2: 大量のログファイルから特定のパターンに一致する行を抽出し、集計する
        System.out.println("\n===== シナリオ2: ログ分析 =====");
        Map<String, Long> logStats = analyzeLogsScenario2(logEntries);
        System.out.println("ERRORレベルのログメッセージの出現回数:");
        for (Map.Entry<String, Long> entry : logStats.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue() + "回");
        }
        
        // シナリオ3: 複数のデータソースから取得したデータを結合し、変換して出力する
        System.out.println("\n===== シナリオ3: データ結合と変換 =====");
        List<OrderDetail> orderDetails = processOrdersScenario3(orders, products, users);
        System.out.println("注文詳細:");
        for (OrderDetail detail : orderDetails) {
            System.out.println(detail);
        }
        
        // 選択理由の説明
        System.out.println("\n===== 選択理由の説明 =====");
        System.out.println("シナリオ1（ユーザー検索）:");
        // TODO: ここにシナリオ1の選択理由を記述
        
        System.out.println("\nシナリオ2（ログ分析）:");
        // TODO: ここにシナリオ2の選択理由を記述
        
        System.out.println("\nシナリオ3（データ結合と変換）:");
        // TODO: ここにシナリオ3の選択理由を記述
    }
}
```

#### 実装ヒント
1. `findUsersScenario1`メソッドでは、Stream APIを使用して、`filter(u -> u.getAge() >= 30 && u.getDepartment().equals("開発") && u.getSalary() >= 500000)`のように条件に一致するユーザーをフィルタリングします。
2. `analyzeLogsScenario2`メソッドでは、Stream APIを使用して、`filter(log -> log.getLevel().equals("ERROR"))`でERRORレベルのログエントリをフィルタリングし、`collect(Collectors.groupingBy(LogEntry::getMessage, Collectors.counting()))`でメッセージごとに出現回数を集計します。
3. `processOrdersScenario3`メソッドでは、ループ処理を使用して、注文情報と製品情報とユーザー情報を結合し、注文詳細を作成します。この場合、複数のデータソースを結合する処理は、ループ処理の方が理解しやすい場合があります。

## 発展演習（オプション）

### 発展演習1: 並列Streamの活用と注意点（難易度：発展）

#### 課題
並列Streamを活用して、大量のデータを効率的に処理するプログラムを作成してください。また、並列処理における注意点を理解し、適切に対処してください。以下の要件を満たす必要があります：

1. 大量の整数リストに対して、並列Streamを使用して素数の数をカウントする
2. 並列処理と逐次処理のパフォーマンスを比較する
3. 並列処理における注意点（副作用、スレッドセーフティなど）を考慮した実装を行う

#### 雛形コード
以下のクラスを作成してください：

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
        // TODO: ここに逐次Streamを使用したコードを実装
        // ヒント1: streamメソッドを使用してStreamを作成
        // ヒント2: filterメソッドを使用して素数のみをフィルタリング
        // ヒント3: countメソッドを使用して素数の数をカウント
        
        return 0; // 仮の戻り値
    }
    
    /**
     * 並列Streamを使用して、リスト内の素数の数をカウントします。
     * 
     * @param numbers 整数のリスト
     * @return 素数の数
     */
    public static long countPrimesWithParallelStream(List<Integer> numbers) {
        // TODO: ここに並列Streamを使用したコードを実装
        // ヒント1: parallelStreamメソッドを使用して並列Streamを作成
        // ヒント2: filterメソッドを使用して素数のみをフィルタリング
        // ヒント3: countメソッドを使用して素数の数をカウント
        
        return 0; // 仮の戻り値
    }
    
    /**
     * 外部カウンターを使用して、リスト内の素数の数をカウントします（非スレッドセーフな実装）。
     * 
     * @param numbers 整数のリスト
     * @return 素数の数
     */
    public static int countPrimesWithExternalCounter(List<Integer> numbers) {
        // TODO: ここに外部カウンターを使用したコードを実装
        // ヒント1: 外部カウンターを使用（非スレッドセーフ）
        // ヒント2: parallelStreamメソッドを使用して並列Streamを作成
        // ヒント3: forEachメソッドを使用して各要素を処理
        
        return 0; // 仮の戻り値
    }
    
    /**
     * AtomicIntegerを使用して、リスト内の素数の数をカウントします（スレッドセーフな実装）。
     * 
     * @param numbers 整数のリスト
     * @return 素数の数
     */
    public static int countPrimesWithAtomicCounter(List<Integer> numbers) {
        // TODO: ここにAtomicIntegerを使用したコードを実装
        // ヒント1: AtomicIntegerを使用（スレッドセーフ）
        // ヒント2: parallelStreamメソッドを使用して並列Streamを作成
        // ヒント3: forEachメソッドを使用して各要素を処理
        
        return 0; // 仮の戻り値
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
        // TODO: ここに並列処理における注意点を記述
    }
}
```

#### 実装ヒント
1. `countPrimesWithSequentialStream`メソッドでは、`stream()`メソッドを使用してStreamを作成し、`filter(ParallelStreamDemo::isPrime)`を使用して素数のみをフィルタリングし、`count()`を使用して素数の数をカウントします。
2. `countPrimesWithParallelStream`メソッドでは、`parallelStream()`メソッドを使用して並列Streamを作成し、それ以外は`countPrimesWithSequentialStream`メソッドと同様に実装します。
3. `countPrimesWithExternalCounter`メソッドでは、外部カウンターを使用して素数の数をカウントします。ただし、この実装は非スレッドセーフであるため、並列処理では正確な結果が得られない可能性があります。
4. `countPrimesWithAtomicCounter`メソッドでは、`AtomicInteger`を使用して素数の数をカウントします。この実装はスレッドセーフであるため、並列処理でも正確な結果が得られます。
5. 並列処理における注意点として、副作用（外部状態の変更）、スレッドセーフティ、実行順序の非決定性、スレッドプールのサイズなどを考慮する必要があります。

### 発展演習2: Stream APIの遅延評価と短絡評価（難易度：発展）

#### 課題
Stream APIの遅延評価と短絡評価の特性を理解し、それらを活用したプログラムを作成してください。以下の要件を満たす必要があります：

1. 無限Streamを使用して、特定の条件を満たす要素を生成する
2. 遅延評価を活用して、必要な要素だけを計算する
3. 短絡評価を活用して、条件を満たす最初の要素を見つける

#### 雛形コード
以下のクラスを作成してください：

```java
package streams.advanced;

import java.util.List;
import java.util.stream.Collectors;
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
        // TODO: ここに無限Streamを使用したコードを実装
        // ヒント1: Stream.iterateメソッドを使用して無限Streamを作成
        // ヒント2: filterメソッドを使用して素数のみをフィルタリング
        // ヒント3: limitメソッドを使用して最初のn個の要素を取得
        // ヒント4: collectメソッドを使用して結果をリストに変換
        
        return null; // 仮の戻り値
    }
    
    /**
     * 無限Streamを使用して、指定された範囲内の最初の完全数を見つけます。
     * 
     * @param start 開始値
     * @param end 終了値
     * @return 最初の完全数、見つからない場合は-1
     */
    public static int findFirstPerfectNumber(int start, int end) {
        // TODO: ここに無限Streamを使用したコードを実装
        // ヒント1: IntStreamのrangeメソッドを使用して範囲内の数値のStreamを作成
        // ヒント2: filterメソッドを使用して完全数のみをフィルタリング
        // ヒント3: findFirstメソッドを使用して最初の要素を取得
        // ヒント4: orElseメソッドを使用して見つからない場合のデフォルト値を設定
        
        return -1; // 仮の戻り値
    }
    
    /**
     * 無限Streamを使用して、フィボナッチ数列の最初のn項を生成します。
     * 
     * @param n 生成する項の数
     * @return フィボナッチ数列の最初のn項のリスト
     */
    public static List<Integer> generateFibonacciSequence(int n) {
        // TODO: ここに無限Streamを使用したコードを実装
        // ヒント1: Stream.iterateメソッドを使用して無限Streamを作成
        // ヒント2: limitメソッドを使用して最初のn個の要素を取得
        // ヒント3: collectメソッドを使用して結果をリストに変換
        
        return null; // 仮の戻り値
    }
    
    /**
     * 遅延評価と短絡評価を示すデモを実行します。
     */
    public static void demonstrateLazyAndShortCircuitEvaluation() {
        System.out.println("===== 遅延評価のデモ =====");
        
        // TODO: ここに遅延評価を示すコードを実装
        // ヒント1: 中間操作（filter, map, peekなど）を連鎖させる
        // ヒント2: 終端操作を呼び出さない場合、中間操作は実行されない
        // ヒント3: 終端操作を呼び出した場合、中間操作が実行される
        
        System.out.println("\n===== 短絡評価のデモ =====");
        
        // TODO: ここに短絡評価を示すコードを実装
        // ヒント1: findFirst, findAny, anyMatch, allMatch, noneMatchなどの短絡終端操作を使用
        // ヒント2: 結果が確定した時点で処理が終了することを示す
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
        // TODO: ここにStream APIの遅延評価と短絡評価の利点を記述
    }
}
```

#### 実装ヒント
1. `generateFirstNPrimes`メソッドでは、`Stream.iterate(2, n -> n + 1)`を使用して無限Streamを作成し、`filter(LazyEvaluationDemo::isPrime)`を使用して素数のみをフィルタリングし、`limit(n)`を使用して最初のn個の要素を取得し、`collect(Collectors.toList())`を使用して結果をリストに変換します。
2. `findFirstPerfectNumber`メソッドでは、`IntStream.range(start, end + 1)`を使用して範囲内の数値のStreamを作成し、`filter(LazyEvaluationDemo::isPerfect)`を使用して完全数のみをフィルタリングし、`findFirst()`を使用して最初の要素を取得し、`orElse(-1)`を使用して見つからない場合のデフォルト値を設定します。
3. `generateFibonacciSequence`メソッドでは、`Stream.iterate(new int[]{0, 1}, f -> new int[]{f[1], f[0] + f[1]})`を使用して無限Streamを作成し、`map(f -> f[0])`を使用して最初の要素を抽出し、`limit(n)`を使用して最初のn個の要素を取得し、`collect(Collectors.toList())`を使用して結果をリストに変換します。
4. `demonstrateLazyAndShortCircuitEvaluation`メソッドでは、遅延評価と短絡評価を示すデモを実行します。遅延評価では、中間操作（filter, map, peekなど）を連鎖させ、終端操作を呼び出さない場合は中間操作が実行されないことを示します。短絡評価では、findFirst, findAny, anyMatch, allMatch, noneMatchなどの短絡終端操作を使用し、結果が確定した時点で処理が終了することを示します。

## 解説と補足

### Stream APIの遅延評価

Stream APIの中間操作は遅延評価されます。つまり、終端操作が呼び出されるまで実行されません。これにより、以下のような利点があります：

1. **不要な計算の回避**：
   - 必要な要素だけを計算することで、パフォーマンスを向上させることができます。
   - 例えば、`limit(n)`を使用すると、最初のn個の要素だけが処理されます。

2. **無限Streamの処理**：
   - 遅延評価により、無限Streamを扱うことができます。
   - 例えば、`Stream.iterate(0, n -> n + 1)`は無限の整数Streamを生成しますが、`limit(n)`を使用することで有限の結果を得ることができます。

3. **最適化の機会**：
   - 遅延評価により、実行時に最適化を行うことができます。
   - 例えば、`filter`と`map`の順序を入れ替えることで、処理する要素の数を減らすことができます。

### Stream APIの短絡評価

一部の終端操作（findFirst, findAny, anyMatch, allMatch, noneMatchなど）は短絡評価をサポートしています。つまり、結果が確定した時点で処理を終了します。これにより、以下のような利点があります：

1. **早期終了**：
   - 条件を満たす要素が見つかった時点で処理を終了することで、パフォーマンスを向上させることができます。
   - 例えば、`anyMatch`は条件を満たす要素が1つでも見つかれば`true`を返し、処理を終了します。

2. **無限Streamの処理**：
   - 短絡評価により、無限Streamでも終了する可能性があります。
   - 例えば、`Stream.iterate(0, n -> n + 1).anyMatch(n -> n > 100)`は`true`を返し、処理を終了します。

3. **不要な計算の回避**：
   - 結果が確定した時点で処理を終了することで、不要な計算を回避することができます。
   - 例えば、`allMatch`は条件を満たさない要素が1つでも見つかれば`false`を返し、処理を終了します。

### Stream APIと並列処理

Stream APIは並列処理をサポートしています。`parallelStream()`メソッドを使用するか、`parallel()`メソッドを呼び出すことで、並列Streamを作成できます。並列処理には以下のような特徴があります：

1. **マルチコアの活用**：
   - 並列処理により、マルチコアプロセッサの性能を活用することができます。
   - 特に、大量のデータを処理する場合に効果的です。

2. **スレッドセーフティの考慮**：
   - 並列処理では、スレッドセーフティを考慮する必要があります。
   - 外部状態を変更する場合は、`AtomicInteger`などのスレッドセーフなクラスを使用する必要があります。

3. **実行順序の非決定性**：
   - 並列処理では、実行順序が非決定的になります。
   - 順序に依存する処理を行う場合は、注意が必要です。

4. **オーバーヘッド**：
   - 並列処理にはオーバーヘッドがあります。
   - 小さなデータセットや単純な操作では、逐次処理の方が効率的な場合があります。

### Stream APIの使い分け

Stream APIと従来のループ処理は、それぞれ異なる状況で適しています。以下に、それぞれの使い分けの指針を示します。

#### Stream APIが適している場合

1. **宣言的なコード**：
   - 「何を行うか」を明確に表現したい場合
   - コードの可読性を重視する場合

2. **関数型プログラミング**：
   - 関数型プログラミングのスタイルを採用している場合
   - 副作用を最小限に抑えたい場合

3. **複雑な変換**：
   - 複数の変換操作を連鎖させる場合
   - フィルタリング、マッピング、集約などの操作を組み合わせる場合

4. **並列処理**：
   - マルチコアプロセッサの性能を活用したい場合
   - 大量のデータを処理する場合

#### 従来のループ処理が適している場合

1. **命令的なコード**：
   - 「どのように行うか」を明確に制御したい場合
   - 処理の詳細を明示的に記述したい場合

2. **単純な操作**：
   - 単純なループ処理で十分な場合
   - オーバーヘッドを最小限に抑えたい場合

3. **副作用**：
   - 外部状態を変更する必要がある場合
   - 複雑な条件分岐や制御フローが必要な場合

4. **デバッグ**：
   - デバッグが容易である必要がある場合
   - 処理の各ステップを追跡したい場合

### Stream APIのパフォーマンス

Stream APIのパフォーマンスは、使用する操作や処理するデータの特性によって異なります。以下に、パフォーマンスに関する考慮事項を示します。

1. **オーバーヘッド**：
   - Stream APIには、ラムダ式の生成、Streamの作成などのオーバーヘッドがあります。
   - 小さなデータセットや単純な操作では、従来のループ処理の方がパフォーマンスが良い場合があります。

2. **並列処理**：
   - 大量のデータを処理する場合、並列処理によりパフォーマンスを向上させることができます。
   - ただし、並列処理にはオーバーヘッドがあるため、データセットのサイズや操作の複雑さによっては、逐次処理の方が効率的な場合があります。

3. **遅延評価**：
   - 遅延評価により、不要な計算を回避することができます。
   - 特に、`limit`や`findFirst`などの操作と組み合わせると、効果的です。

4. **短絡評価**：
   - 短絡評価により、結果が確定した時点で処理を終了することができます。
   - 特に、`anyMatch`や`findFirst`などの操作で効果的です。

5. **ボックス化とアンボックス化**：
   - プリミティブ型のボックス化とアンボックス化のオーバーヘッドを避けるために、`IntStream`、`LongStream`、`DoubleStream`などのプリミティブ型のStreamを使用することを検討してください。

## 実務での活用シーン

### 1. データ変換パイプライン

Stream APIは、データの変換パイプラインを構築するのに適しています。複数の変換操作を連鎖させることで、データを段階的に処理することができます。

```java
// ユーザーのリストから、30歳以上のユーザーの名前を取得し、アルファベット順にソートする
List<String> sortedNames = users.stream()
                               .filter(user -> user.getAge() >= 30)
                               .map(User::getName)
                               .sorted()
                               .collect(Collectors.toList());
```

このようなデータ変換パイプラインは、ETL（Extract, Transform, Load）処理や、データの前処理、クレンジング、正規化などのタスクで活用できます。

### 2. 集計処理

Stream APIは、データの集計処理に適しています。`collect`メソッドと`Collectors`クラスを組み合わせることで、様々な集計操作を行うことができます。

```java
// 部門ごとの平均給与を計算する
Map<String, Double> averageSalaryByDepartment = employees.stream()
                                                      .collect(Collectors.groupingBy(
                                                          Employee::getDepartment,
                                                          Collectors.averagingDouble(Employee::getSalary)
                                                      ));

// 給与の合計、平均、最小、最大を計算する
DoubleSummaryStatistics salaryStats = employees.stream()
                                            .collect(Collectors.summarizingDouble(Employee::getSalary));
System.out.println("合計: " + salaryStats.getSum());
System.out.println("平均: " + salaryStats.getAverage());
System.out.println("最小: " + salaryStats.getMin());
System.out.println("最大: " + salaryStats.getMax());
System.out.println("件数: " + salaryStats.getCount());
```

このような集計処理は、レポート生成、データ分析、ダッシュボード表示などのタスクで活用できます。

### 3. 並列処理

Stream APIの並列処理機能は、大量のデータを効率的に処理するのに適しています。特に、CPUバウンドな処理（計算量の多い処理）で効果的です。

```java
// 大量の数値の合計を並列処理で計算する
long sum = numbers.parallelStream()
                 .mapToLong(Long::valueOf)
                 .sum();

// 大量のファイルを並列処理で処理する
Files.list(Paths.get("/path/to/directory"))
     .parallel()
     .filter(Files::isRegularFile)
     .forEach(file -> processFile(file));
```

このような並列処理は、バッチ処理、データマイニング、シミュレーション、画像処理などのタスクで活用できます。

### 4. 条件検索と抽出

Stream APIは、条件に一致する要素を検索し、抽出するのに適しています。`filter`メソッドと`findFirst`、`findAny`、`anyMatch`などのメソッドを組み合わせることで、効率的に検索を行うことができます。

```java
// 特定の条件に一致する最初のユーザーを検索する
Optional<User> user = users.stream()
                          .filter(u -> u.getName().equals("Alice"))
                          .findFirst();

// 特定の条件に一致するユーザーが存在するかどうかを確認する
boolean exists = users.stream()
                     .anyMatch(u -> u.getAge() > 30 && u.getDepartment().equals("開発"));
```

このような条件検索と抽出は、検索機能、フィルタリング機能、バリデーション機能などのタスクで活用できます。

これらの実務での活用シーンを参考に、Stream APIを効果的に活用してください。Stream APIは、コードの可読性、保守性、効率性を向上させる強力なツールです。

## 解答例

<a href="answers/day5">解答例はこちら</a>