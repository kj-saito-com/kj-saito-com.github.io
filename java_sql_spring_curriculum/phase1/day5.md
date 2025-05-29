# Day 5: Stream APIとループ処理の比較と選択

## 学習の目的と背景

Java 8で導入されたStream APIは、コレクションや配列などのデータ集合に対する操作を、宣言的かつ関数的なスタイルで記述するための強力なツールです。従来のループ処理（for文、拡張for文、while文など）と比較して、コードの可読性や簡潔性を向上させ、並列処理を容易にするなどの利点があります。

しかし、Stream APIが常にループ処理よりも優れているわけではありません。処理の内容やデータの特性、パフォーマンス要件によっては、従来のループ処理の方が適切な場合もあります。本日の学習では、Stream APIの基本的な使い方を理解し、ループ処理との違いを明確にした上で、それぞれのメリット・デメリットを比較検討し、状況に応じて適切な処理方法を選択できるようになることを目指します。

## 前提知識と準備

### 必要な前提知識
- Javaの基本構文（変数、制御構文、クラス、オブジェクト）
- Day 2で学習したコレクションフレームワーク（List, Set, Map）
- Day 3で学習したラムダ式と関数型インターフェース

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

Stream APIは、データソース（コレクション、配列など）から取得した要素のシーケンス（Stream）に対して、一連の操作（中間操作、終端操作）をパイプライン形式で適用する仕組みです。

<div class="term-box">
<strong>用語解説: Stream</strong><br>
データソースから取得された要素のシーケンスです。Stream自体はデータを保持せず、データソースへの参照と操作のパイプラインを保持します。一度消費されると再利用できません。
</div>

<div class="term-box">
<strong>用語解説: パイプライン（Pipeline）</strong><br>
Streamに対する一連の操作（中間操作と終端操作）の連鎖です。データソース → 0個以上の中間操作 → 1個の終端操作、という構成になります。
</div>

#### Streamの主な特徴
- **宣言的な処理**: 何をしたいか（What）を記述し、どのように行うか（How）はライブラリに任せる
- **遅延評価**: 中間操作は終端操作が実行されるまで実行されない
- **内部イテレーション**: 要素の反復処理はStream API内部で行われる
- **並列処理の容易さ**: `parallelStream()`を使うことで、簡単に並列処理に切り替えられる
- **非破壊的**: 元のデータソースを変更しない
- **一度きり**: Streamは一度消費されると再利用できない

#### Streamの生成方法
- コレクションから: `collection.stream()`, `collection.parallelStream()`
- 配列から: `Arrays.stream(array)`
- 特定の値から: `Stream.of(value1, value2, ...)`
- 数値範囲から: `IntStream.range(start, end)`, `IntStream.rangeClosed(start, end)`
- ファイルから: `Files.lines(path)`
- 無限ストリーム: `Stream.iterate()`, `Stream.generate()`

### 2. 中間操作（Intermediate Operations）

中間操作は、Streamを受け取り、新しいStreamを返す操作です。複数の中間操作を連結できます。

<div class="term-box">
<strong>用語解説: 中間操作（Intermediate Operation）</strong><br>
Streamを受け取り、新しいStreamを返す操作です。遅延評価され、終端操作が実行されるまで実際の処理は行われません。例：`filter`, `map`, `sorted`, `distinct`。
</div>

#### 主な中間操作
- `filter(Predicate<T> predicate)`: 条件に一致する要素のみを抽出
- `map(Function<T, R> mapper)`: 各要素を別の値に変換
- `flatMap(Function<T, Stream<R>> mapper)`: 各要素をStreamに変換し、それらを連結して1つのStreamにする
- `sorted()`: 要素を自然順序でソート
- `sorted(Comparator<T> comparator)`: 指定されたComparatorでソート
- `distinct()`: 重複する要素を削除
- `limit(long maxSize)`: 指定された数までの要素に制限
- `skip(long n)`: 先頭から指定された数の要素をスキップ
- `peek(Consumer<T> action)`: 各要素に対して指定されたアクションを実行（デバッグ用）

### 3. 終端操作（Terminal Operations）

終端操作は、Streamを受け取り、結果（非Stream値、コレクション、voidなど）を生成する操作です。終端操作が実行されると、パイプライン全体の処理が開始されます。

<div class="term-box">
<strong>用語解説: 終端操作（Terminal Operation）</strong><br>
Streamを受け取り、最終的な結果を生成する操作です。終端操作が実行されると、パイプライン全体の処理が開始され、Streamは消費されます。例：`forEach`, `collect`, `count`, `reduce`。
</div>

#### 主な終端操作
- `forEach(Consumer<T> action)`: 各要素に対して指定されたアクションを実行
- `collect(Collector<T, A, R> collector)`: Streamの要素を集約してコレクションや単一の値にする（`Collectors`クラスのメソッドと組み合わせて使用）
  - `Collectors.toList()`: Listに集約
  - `Collectors.toSet()`: Setに集約
  - `Collectors.toMap(keyMapper, valueMapper)`: Mapに集約
  - `Collectors.joining(delimiter)`: 文字列に連結
  - `Collectors.groupingBy(classifier)`: グループ化
  - `Collectors.counting()`: 要素数をカウント
  - `Collectors.summingInt(mapper)`: 合計値を計算
  - `Collectors.averagingInt(mapper)`: 平均値を計算
- `count()`: 要素数をカウント
- `reduce(BinaryOperator<T> accumulator)`: 要素を結合して単一の結果を生成
- `reduce(T identity, BinaryOperator<T> accumulator)`: 初期値を持つ`reduce`
- `min(Comparator<T> comparator)`: 最小要素を取得（`Optional<T>`）
- `max(Comparator<T> comparator)`: 最大要素を取得（`Optional<T>`）
- `anyMatch(Predicate<T> predicate)`: いずれかの要素が条件に一致するかどうか
- `allMatch(Predicate<T> predicate)`: すべての要素が条件に一致するかどうか
- `noneMatch(Predicate<T> predicate)`: いずれの要素も条件に一致しないかどうか
- `findFirst()`: 最初の要素を取得（`Optional<T>`）
- `findAny()`: いずれかの要素を取得（並列処理で効率的）（`Optional<T>`）

### 4. Stream API vs ループ処理

| 特徴             | Stream API                                  | ループ処理 (for, while)                     |
|------------------|---------------------------------------------|---------------------------------------------|
| **スタイル**     | 宣言的 (何をしたいか)                       | 命令的 (どのように行うか)                   |
| **可読性**       | 複雑な処理も比較的簡潔に書ける              | 単純な処理は直感的、複雑になると冗長になりがち |
| **内部/外部反復**| 内部イテレーション (ライブラリが反復を管理) | 外部イテレーション (開発者が反復を制御)     |
| **並列処理**     | `parallelStream()`で容易に実現可能          | 手動での実装が必要（複雑）                  |
| **状態管理**     | ステートレス（副作用を避ける設計）          | 状態変数の管理が必要                        |
| **遅延評価**     | あり（中間操作）                            | なし（即時実行）                            |
| **再利用性**     | Streamは一度消費すると再利用不可            | ループ変数は再利用可能                      |
| **デバッグ**     | `peek`やデバッガの機能が必要                | ステップ実行で比較的容易                    |
| **パフォーマンス** | 単純な処理ではループより遅い場合がある      | 単純な処理では高速な場合が多い              |
|                  | 並列処理や複雑な処理では有利な場合がある    | 並列化は手動で行う必要があり、難しい        |

### 5. Stream APIを使うべき場合

- **複雑なデータ処理パイプライン**: 複数のフィルタリング、マッピング、集約操作を組み合わせる場合
- **可読性の向上**: 処理の流れを明確にしたい場合
- **並列処理**: 大量のデータを並列で処理したい場合（ただし、並列化のオーバーヘッドに注意）
- **関数型プログラミング**: 関数型スタイルでコードを書きたい場合
- **コレクション操作**: `filter`, `map`, `reduce`などの一般的なコレクション操作を行う場合

### 6. ループ処理を使うべき場合

- **単純な反復処理**: 特定の回数だけ処理を繰り返す、単純な要素アクセスなど
- **パフォーマンス重視**: 処理速度が最優先で、Stream APIのオーバーヘッドが無視できない場合
- **状態の変更**: ループ内で外部の状態変数を頻繁に変更する必要がある場合（Stream APIでは副作用を避けるべき）
- **細かい制御**: ループの途中で`break`や`continue`を使いたい場合（Stream APIでは限定的）
- **インデックスアクセス**: 要素のインデックスが必要な場合（Stream APIでは一手間かかる）
- **可読性**: チームメンバーがStream APIに慣れていない場合や、ループの方が直感的に理解できる単純な処理の場合

## Eclipse操作ガイド

### Stream APIのコード補完

EclipseはStream APIのメソッドチェーンに対しても強力なコード補完を提供します。

1. Streamを生成するコード（例：`list.stream()`）を入力します。
2. `.`を入力すると、利用可能な中間操作や終端操作の候補が表示されます。
3. ラムダ式を入力する際も、引数の型などが補完されます。

![Eclipse Stream補完](https://example.com/eclipse_stream_completion.png)

### Stream APIのデバッグ

Streamパイプラインのデバッグは、従来のループ処理よりも難しい場合がありますが、Eclipseの機能や`peek`操作を活用できます。

1. **ブレークポイント**: ラムダ式内にブレークポイントを設定できます。デバッグモードで実行すると、各要素がそのラムダ式を通過する際に停止します。
2. **`peek`操作**: パイプラインの途中で要素の状態を確認したい場合に`peek`を使います。
   ```java
   List<String> result = names.stream()
       .filter(s -> s.startsWith("A"))
       .peek(s -> System.out.println("Filtered: " + s)) // 途中経過を出力
       .map(String::toUpperCase)
       .peek(s -> System.out.println("Mapped: " + s))   // 途中経過を出力
       .collect(Collectors.toList());
   ```
3. **インスペクション**: デバッグ中にStreamオブジェクトやラムダ式内の変数をインスペクション（調査）できます。

![Eclipse Streamデバッグ](https://example.com/eclipse_stream_debug.png)

### Stream APIへのリファクタリング

Eclipseには、従来のループ処理をStream APIに変換するリファクタリング機能があります（限定的ですが）。

1. 変換したいループ処理を選択します。
2. 右クリックして「Refactor」→「Convert Loop to Stream」のようなオプションを探します（利用可能な場合）。

## コア演習（必須）

### 演習1: Stream APIの基本操作（難易度：基本）

#### 課題
与えられた文字列リストに対して、Stream APIを使用して以下の処理を行うプログラムを作成してください。
1. 長さが5文字以上の文字列のみを抽出する
2. 抽出した文字列をすべて大文字に変換する
3. 変換後の文字列をアルファベット順にソートする
4. 結果をリストとして収集する

#### 雛形コード
```java
package stream.basic;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

/**
 * Stream APIの基本操作を練習するクラス
 */
public class BasicStreamOperations {

    /**
     * 文字列リストに対してStream操作を行います。
     * 
     * @param inputList 入力となる文字列リスト
     * @return 処理後の文字列リスト
     */
    public List<String> processStrings(List<String> inputList) {
        // TODO: Stream APIを使用して以下の処理を実装
        // 1. 長さが5文字以上の文字列のみを抽出 (filter)
        // 2. 抽出した文字列をすべて大文字に変換 (map)
        // 3. 変換後の文字列をアルファベット順にソート (sorted)
        // 4. 結果をリストとして収集 (collect)
        
        return null; // 仮の戻り値
    }

    public static void main(String[] args) {
        BasicStreamOperations demo = new BasicStreamOperations();
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve", "Frank", "Grace");
        
        System.out.println("元のリスト: " + names);
        
        List<String> result = demo.processStrings(names);
        
        System.out.println("処理後のリスト: " + result);
        // 期待される出力例: [ALICE, CHARLIE, DAVID, FRANK, GRACE]
    }
}
```

#### 実装ヒント
- `inputList.stream()`でStreamを生成します。
- `filter`メソッドで文字列の長さをチェックします (`s.length() >= 5`)。
- `map`メソッドで大文字に変換します (`String::toUpperCase`)。
- `sorted`メソッドでソートします。
- `collect(Collectors.toList())`で結果をリストに収集します。

### 演習2: Stream API vs ループ処理（パフォーマンス比較）（難易度：応用）

#### 課題
数値リストに対して、以下の処理をStream APIと従来のループ処理の両方で実装し、それぞれの実行時間を測定・比較してください。
1. 偶数のみを抽出する
2. 抽出した数値を2倍にする
3. 2倍にした数値の合計値を求める

#### 雛形コード
```java
package stream.comparison;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.stream.Collectors;

/**
 * Stream APIとループ処理のパフォーマンスを比較するクラス
 */
public class StreamVsLoopPerformance {

    private static final int LIST_SIZE = 1_000_000; // リストのサイズ
    private static final int MAX_VALUE = 1000;      // 数値の最大値

    /**
     * Stream APIを使用して処理を行います。
     * 
     * @param numbers 数値リスト
     * @return 処理結果（合計値）
     */
    public long processWithStream(List<Integer> numbers) {
        // TODO: Stream APIを使用して以下の処理を実装
        // 1. 偶数のみを抽出 (filter)
        // 2. 抽出した数値を2倍にする (map)
        // 3. 2倍にした数値の合計値を求める (sum)
        
        return 0L; // 仮の戻り値
    }

    /**
     * 従来のループ処理を使用して処理を行います。
     * 
     * @param numbers 数値リスト
     * @return 処理結果（合計値）
     */
    public long processWithLoop(List<Integer> numbers) {
        long sum = 0;
        // TODO: 従来のループ処理（拡張for文など）を使用して以下の処理を実装
        // 1. 各数値をチェックし、偶数かどうか判定
        // 2. 偶数であれば2倍にする
        // 3. 合計値に加算する
        
        return sum;
    }

    /**
     * パフォーマンスを測定します。
     * 
     * @param taskName タスク名
     * @param task 測定対象のタスク (Runnable)
     * @return 実行時間（ミリ秒）
     */
    public long measurePerformance(String taskName, Runnable task) {
        System.out.println("測定開始: " + taskName);
        long startTime = System.nanoTime();
        task.run();
        long endTime = System.nanoTime();
        long duration = (endTime - startTime) / 1_000_000; // ミリ秒に変換
        System.out.println("測定終了: " + taskName + " - 実行時間: " + duration + "ms");
        return duration;
    }

    public static void main(String[] args) {
        StreamVsLoopPerformance demo = new StreamVsLoopPerformance();
        
        // 大量の数値リストを生成
        List<Integer> numbers = new ArrayList<>(LIST_SIZE);
        Random random = new Random();
        for (int i = 0; i < LIST_SIZE; i++) {
            numbers.add(random.nextInt(MAX_VALUE));
        }
        
        // Stream APIでの処理時間を測定
        long streamResult = 0;
        long streamTime = demo.measurePerformance("Stream API", () -> {
            // 複数回実行して平均を取る方が望ましいが、ここでは簡略化
            long result = demo.processWithStream(numbers);
            // 結果を変数に格納しないと最適化で処理がスキップされる可能性があるため注意
            // この例ではstreamResultに代入しているが、実際には戻り値を使うなどする
            // non-local variable streamResult cannot be assigned to from within lambda expression
            // なので、このままだとコンパイルエラー。実際には戻り値を使うか、AtomicLongなどを使う。
            // ここでは簡単のため、測定メソッド内で結果を返すように修正する前提とする。
            // （雛形コードのmeasurePerformanceはRunnableを受け取るため、戻り値がない）
            // 修正案：measurePerformanceがSupplier<Long>を受け取るようにする
            // Supplier<Long> task = () -> demo.processWithStream(numbers);
        });
        // streamResult = demo.processWithStream(numbers); // 実際の結果取得
        
        // ループ処理での処理時間を測定
        long loopResult = 0;
        long loopTime = demo.measurePerformance("ループ処理", () -> {
            // Supplier<Long> task = () -> demo.processWithLoop(numbers);
        });
        // loopResult = demo.processWithLoop(numbers); // 実際の結果取得
        
        // 結果の検証（StreamとLoopの結果が一致するか）
        // System.out.println("Stream APIの結果: " + streamResult);
        // System.out.println("ループ処理の結果: " + loopResult);
        // if (streamResult == loopResult) {
        //     System.out.println("結果は一致しました。");
        // } else {
        //     System.err.println("警告: 結果が一致しません！");
        // }
        
        System.out.println("\n=== パフォーマンス比較 ===");
        System.out.println("Stream API: " + streamTime + "ms");
        System.out.println("ループ処理: " + loopTime + "ms");
        if (loopTime > 0) {
            System.out.printf("Stream APIはループ処理の %.2f 倍の速度%n", (double) loopTime / streamTime);
        }
    }
}
```

#### 実装ヒント
- `processWithStream`メソッド:
  - `numbers.stream()`でStreamを生成します。
  - `filter(n -> n % 2 == 0)`で偶数を抽出します。
  - `mapToLong(n -> (long)n * 2)`で2倍にして`LongStream`に変換します（`sum()`を使うため）。`mapToInt`でも良いですが、合計値が`int`の範囲を超える可能性を考慮して`long`にします。
  - `sum()`で合計値を求めます。
- `processWithLoop`メソッド:
  - 拡張for文 (`for (int number : numbers)`) を使用します。
  - `if (number % 2 == 0)`で偶数を判定します。
  - 偶数であれば `sum += (long)number * 2;` のように合計値に加算します。
- パフォーマンス測定:
  - `measurePerformance`メソッドは、与えられた`Runnable`を実行し、その時間を計測します。ただし、この雛形では`Runnable`が結果を返せないため、実際には`Supplier<Long>`など結果を返せるインターフェースを使うか、メソッド内で結果を取得・検証するように修正が必要です。
  - JVMのウォームアップや複数回測定の平均を取ることで、より正確な測定が可能です（この演習では簡略化）。

### 演習3: Stream APIの選択（可読性と保守性）（難易度：応用）

#### 課題
ユーザーオブジェクトのリストがあり、以下の条件を満たすユーザーの名前（文字列のリスト）を取得する処理を、Stream APIと従来のループ処理の両方で実装してください。どちらの実装がより可読性が高く、保守しやすいかを考察してください。
1. アクティブ（`isActive`が`true`）なユーザー
2. 年齢（`age`）が30歳以上
3. 居住地（`city`）が"Tokyo"または"Osaka"
4. 結果はユーザー名（`name`）のリストとし、名前のアルファベット順でソートする

#### 雛形コード
```java
package stream.selection;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;

/**
 * Stream APIとループ処理の可読性・保守性を比較するクラス
 */
public class StreamVsLoopReadability {

    /**
     * ユーザークラス
     */
    public static class User {
        private String name;
        private int age;
        private String city;
        private boolean isActive;

        public User(String name, int age, String city, boolean isActive) {
            this.name = name;
            this.age = age;
            this.city = city;
            this.isActive = isActive;
        }

        public String getName() { return name; }
        public int getAge() { return age; }
        public String getCity() { return city; }
        public boolean isActive() { return isActive; }

        @Override
        public String toString() {
            return "User{" + "name=\'" + name + "\', age=" + age + ", city=\'" + city + "\', isActive=" + isActive + "}";
        }
    }

    /**
     * Stream APIを使用して条件に合うユーザー名を取得します。
     * 
     * @param users ユーザーリスト
     * @return 条件に合うユーザー名のリスト（ソート済み）
     */
    public List<String> getUserNamesWithStream(List<User> users) {
        // TODO: Stream APIを使用して以下の処理を実装
        // 1. アクティブなユーザーを抽出 (filter)
        // 2. 年齢が30歳以上のユーザーを抽出 (filter)
        // 3. 居住地が"Tokyo"または"Osaka"のユーザーを抽出 (filter)
        // 4. ユーザー名を抽出 (map)
        // 5. 名前でソート (sorted)
        // 6. 結果をリストとして収集 (collect)
        
        return null; // 仮の戻り値
    }

    /**
     * 従来のループ処理を使用して条件に合うユーザー名を取得します。
     * 
     * @param users ユーザーリスト
     * @return 条件に合うユーザー名のリスト（ソート済み）
     */
    public List<String> getUserNamesWithLoop(List<User> users) {
        List<String> resultNames = new ArrayList<>();
        // TODO: 従来のループ処理を使用して以下の処理を実装
        // 1. 各ユーザーをループで処理
        // 2. 条件（アクティブ、年齢、居住地）をif文でチェック
        // 3. 条件に合致すればユーザー名をリストに追加
        // 4. ループ終了後、リストをソート
        
        return resultNames;
    }

    public static void main(String[] args) {
        StreamVsLoopReadability demo = new StreamVsLoopReadability();
        List<User> users = List.of(
            new User("Alice", 25, "Tokyo", true),
            new User("Bob", 35, "Osaka", true),
            new User("Charlie", 40, "Tokyo", false),
            new User("David", 32, "Kyoto", true),
            new User("Eve", 45, "Tokyo", true),
            new User("Frank", 28, "Osaka", true),
            new User("Grace", 38, "Osaka", true)
        );

        System.out.println("=== Stream APIによる実装 ===");
        List<String> streamResult = demo.getUserNamesWithStream(users);
        System.out.println(streamResult);
        // 期待される出力例: [Bob, Eve, Grace]

        System.out.println("\n=== ループ処理による実装 ===");
        List<String> loopResult = demo.getUserNamesWithLoop(users);
        System.out.println(loopResult);
        // 期待される出力例: [Bob, Eve, Grace]
        
        System.out.println("\n=== 考察 ===
