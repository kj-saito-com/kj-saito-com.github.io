# Day 2: コレクションフレームワークの効果的な活用

## 学習の目的と背景

Javaプログラミングにおいて、複数のデータを効率的に管理・操作するためにコレクションフレームワークは不可欠です。`List`、`Set`、`Map`などのインターフェースとその実装クラスを理解し、それぞれの特性に応じて適切に使い分けることは、パフォーマンスが高く保守性の良いコードを書くための基礎となります。

特に、業務アプリケーションでは大量のデータを扱うことが多く、コレクションの選択が性能に直結する場合があります。また、データの重複を許さない、あるいはキーと値のペアで管理したいといった要件に応じて、適切なコレクションを選択する能力が求められます。

本日の学習では、Javaコレクションフレームワークの主要なインターフェースと実装クラスについて、その特性、使い方、そしてパフォーマンス（計算量）の観点から深く学びます。特に、ユーザーが苦手意識を持つ`Set`と、基本的な`List`（`ArrayList`）との違い、そして`Map`の活用方法に焦点を当てます。

## 前提知識と準備

### 必要な前提知識
- Javaの基本構文（変数、制御構文、クラス、オブジェクト）
- オブジェクト指向の基本概念（継承、インターフェース）
- Day 1で学習した例外処理の基本

### 環境準備
- Eclipse 2023-12 (4.30.0)
- Java 21

### プロジェクト作成手順

1. Eclipseを起動します。
2. 「File」→「New」→「Java Project」を選択します。
3. プロジェクト名を「JavaCollections_Day2」と入力します。
4. 「JRE」の設定で「Use an execution environment JRE:」から「JavaSE-21」を選択します。
5. 「Finish」をクリックしてプロジェクトを作成します。

## 基本概念の解説

### 1. コレクションフレームワーク概要

Javaコレクションフレームワークは、オブジェクトの集まり（コレクション）を表現し、操作するための統一されたアーキテクチャを提供します。主要なインターフェースとして`Collection`があり、その下に`List`、`Set`、`Queue`などが位置づけられます。また、キーと値のペアを扱う`Map`インターフェースも重要な要素です。

![Java Collection Framework](https://example.com/java_collection_framework.png)

### 2. Listインターフェース

`List`は順序付けられた要素のコレクションであり、要素の重複を許します。インデックス（位置）を使って要素にアクセスできます。

#### ArrayList（復習）
- 内部的に配列を使用。
- 要素へのランダムアクセス（インデックス指定での取得）が高速。
- 要素の追加・削除（特に中間）は、配列のコピーが発生するため低速になる場合がある。
- ユーザーが経験済みとのことなので、ここでは基本的な使い方を再確認します。

```java
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.add("Charlie");
names.add("Alice"); // 重複を許す

System.out.println(names.get(1)); // "Bob" (インデックスでアクセス)

for (String name : names) {
    System.out.println(name);
}
```

### 3. Setインターフェース

`Set`は順序を持たない（または特定の順序を持つ）要素のコレクションであり、**要素の重複を許しません**。`List`との最も大きな違いはこの点です。

<div class="term-box">
<strong>List vs Set</strong><br>
- <strong>List</strong>: 順序あり、重複<strong>許可</strong>。インデックスでアクセス可能。
- <strong>Set</strong>: 順序なし（または特定の順序あり）、重複<strong>不許可</strong>。要素の存在確認が高速。
</div>

#### HashSet
- 最も一般的な`Set`の実装。
- ハッシュテーブルに基づいており、要素の追加、削除、検索（`contains`）が非常に高速（平均的にO(1)）。
- 要素の順序は保証されない。
- 要素として格納するオブジェクトは`hashCode()`と`equals()`メソッドを正しく実装する必要がある。

```java
Set<String> uniqueNames = new HashSet<>();
uniqueNames.add("Alice");
uniqueNames.add("Bob");
uniqueNames.add("Alice"); // 重複は無視される

System.out.println(uniqueNames.contains("Bob")); // true
System.out.println(uniqueNames.size()); // 2

for (String name : uniqueNames) {
    System.out.println(name); // 順序は保証されない (例: Bob, Alice)
}
```

#### LinkedHashSet
- `HashSet`と同様にハッシュテーブルを使用するが、要素が追加された順序を保持する。
- 性能は`HashSet`よりわずかに劣るが、順序が必要な場合に有用。

```java
Set<String> orderedUniqueNames = new LinkedHashSet<>();
orderedUniqueNames.add("Charlie");
orderedUniqueNames.add("Alice");
orderedUniqueNames.add("Bob");
orderedUniqueNames.add("Alice");

for (String name : orderedUniqueNames) {
    System.out.println(name); // 追加順 (Charlie, Alice, Bob)
}
```

#### TreeSet
- 要素を自然順序（または指定された`Comparator`による順序）でソートして保持する。
- 赤黒木（Red-Black Tree）に基づいており、要素の追加、削除、検索はO(log n)。
- 要素は`Comparable`インターフェースを実装しているか、`Comparator`が提供される必要がある。

```java
Set<String> sortedNames = new TreeSet<>();
sortedNames.add("Charlie");
sortedNames.add("Alice");
sortedNames.add("Bob");

for (String name : sortedNames) {
    System.out.println(name); // ソート順 (Alice, Bob, Charlie)
}
```

### 4. Mapインターフェース

`Map`はキーと値のペアを格納するコレクションです。キーは一意（重複不可）であり、各キーは最大1つの値に対応します。

#### HashMap
- 最も一般的な`Map`の実装。
- ハッシュテーブルに基づいており、キーによる値の追加、削除、取得が非常に高速（平均的にO(1)）。
- キーと値の順序は保証されない。
- キーとして使用するオブジェクトは`hashCode()`と`equals()`メソッドを正しく実装する必要がある。

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 90);
scores.put("Bob", 85);
scores.put("Charlie", 95);
scores.put("Alice", 92); // キーが重複する場合、値が上書きされる

System.out.println(scores.get("Bob")); // 85
System.out.println(scores.containsKey("David")); // false

for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue()); // 順序は保証されない
}
```

#### LinkedHashMap
- `HashMap`と同様にハッシュテーブルを使用するが、キーと値のペアが追加された（またはアクセスされた）順序を保持する。
- 性能は`HashMap`よりわずかに劣るが、順序が必要な場合に有用。

```java
Map<String, Integer> orderedScores = new LinkedHashMap<>();
orderedScores.put("Charlie", 95);
orderedScores.put("Alice", 90);
orderedScores.put("Bob", 85);

for (Map.Entry<String, Integer> entry : orderedScores.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue()); // 追加順 (Charlie, Alice, Bob)
}
```

#### TreeMap
- キーを自然順序（または指定された`Comparator`による順序）でソートして保持する。
- 赤黒木に基づいており、キーによる値の追加、削除、取得はO(log n)。
- キーは`Comparable`インターフェースを実装しているか、`Comparator`が提供される必要がある。

```java
Map<String, Integer> sortedScores = new TreeMap<>();
sortedScores.put("Charlie", 95);
sortedScores.put("Alice", 90);
sortedScores.put("Bob", 85);

for (Map.Entry<String, Integer> entry : sortedScores.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue()); // キーのソート順 (Alice, Bob, Charlie)
}
```

### 5. 計算量（オーダー記法）とコレクションの性能

<div class="term-box">
<strong>用語解説: オーダー記法（Big O表記）</strong><br>
アルゴリズムの実行時間やメモリ使用量が、入力データサイズ（n）の増加に対してどのように増加するかを示す指標です。主な表記法：
<ul>
<li><strong>O(1) - 定数時間</strong>: データサイズに関係なく、常に一定時間で完了します。非常に高速です。（例: HashMap/HashSetの平均的な追加・検索）</li>
<li><strong>O(log n) - 対数時間</strong>: データサイズが倍になっても、時間はわずかしか増加しません。高速です。（例: TreeSet/TreeMapの追加・検索、二分探索）</li>
<li><strong>O(n) - 線形時間</strong>: データサイズに比例して時間が増加します。一般的です。（例: ArrayListの検索、配列の全要素走査）</li>
<li><strong>O(n log n) - 線形対数時間</strong>: O(n)より遅いですが、効率的なソートアルゴリズムなどで見られます。（例: マージソート、クイックソートの平均）</li>
<li><strong>O(n<sup>2</sup>) - 二乗時間</strong>: データサイズが増えると急激に時間が増加します。遅いです。（例: バブルソート、単純な二重ループ）</li>
</ul>
計算量を理解することは、データ構造やアルゴリズムを選択する上で非常に重要です。
</div>

#### 主要コレクション操作の計算量比較

| 操作        | ArrayList      | HashSet (平均) | LinkedHashSet (平均) | TreeSet (保証) | HashMap (平均) | LinkedHashMap (平均) | TreeMap (保証) |
|-------------|----------------|----------------|----------------------|----------------|----------------|------------------------|----------------|
| `add`       | O(1) (末尾) / O(n) (中間) | O(1)           | O(1)                 | O(log n)       | O(1)           | O(1)                   | O(log n)       |
| `remove`    | O(n)           | O(1)           | O(1)                 | O(log n)       | O(1)           | O(1)                   | O(log n)       |
| `contains`/`get` | O(n)           | O(1)           | O(1)                 | O(log n)       | O(1)           | O(1)                   | O(log n)       |
| イテレーション | O(n)           | O(n)           | O(n)                 | O(n)           | O(n)           | O(n)                   | O(n)           |

**注意:** `HashMap`/`HashSet`系のO(1)は平均的なケースであり、ハッシュ衝突が頻発する最悪ケースではO(n)になる可能性があります。`TreeMap`/`TreeSet`のO(log n)は保証された性能です。

**選択のポイント:**
- **頻繁な検索が必要で、順序が不要なら**: `HashSet` / `HashMap`
- **追加順を保持したいなら**: `LinkedHashSet` / `LinkedHashMap`
- **要素をソートして保持したいなら**: `TreeSet` / `TreeMap`
- **インデックスアクセスが必要なら**: `ArrayList`

### 6. Generics（ジェネリクス）

コレクションフレームワークでは、格納する要素の型をコンパイル時に指定するためにジェネリクスが使われます。これにより、型安全性が向上し、キャストが不要になります。

```java
// ジェネリクス未使用（非推奨）
List namesOld = new ArrayList();
namesOld.add("Alice");
String name = (String) namesOld.get(0); // キャストが必要
namesOld.add(123); // コンパイルエラーにならないが、実行時に問題発生の可能性

// ジェネリクス使用
List<String> namesNew = new ArrayList<>();
namesNew.add("Alice");
String nameNew = namesNew.get(0); // キャスト不要
// namesNew.add(123); // コンパイルエラーになる
```

## Eclipse操作ガイド

### ジェネリクスの利用
Eclipseはジェネリクスの型推論をサポートしており、コード補完も適切に行われます。

```java
// Eclipseで List< と入力すると、型パラメータの候補が表示される
List<String> list = new ArrayList<>(); // <> (ダイヤモンド演算子) で型推論が働く
```

### コレクションのデバッグ
デバッグ時に「Variables」ビューでコレクションの内容を確認できます。要素を展開して個々の値を見ることができます。

![Eclipseコレクションデバッグ](https://example.com/eclipse_collection_debug.png)

## コア演習（必須）

### 演習1: 基本的なコレクション操作（難易度：基本）

#### 課題
`ArrayList`, `HashSet`, `HashMap`を使用して、以下の操作を行うプログラムを作成してください。
1. `ArrayList`にいくつかの文字列を追加し、特定の要素を検索・削除する。
2. `HashSet`にいくつかの文字列を追加し、重複がどのように扱われるか確認する。
3. `HashMap`にキー（Integer）と値（String）のペアをいくつか追加し、特定のキーに対応する値を取得・更新する。

#### 雛形コード
```java
package collections.basic;

import java.util.*;

/**
 * List, Set, Mapの基本的な操作を練習するクラス
 */
public class BasicOperations {

    /**
     * Listの基本操作をデモンストレーションします。
     */
    public void demonstrateList() {
        System.out.println("--- List Demonstration ---");
        List<String> fruits = new ArrayList<>();
        
        // TODO: 1. "Apple", "Banana", "Orange", "Apple" をリストに追加
        
        // TODO: 2. リストの内容をコンソールに出力
        
        // TODO: 3. "Banana" がリストに含まれているか確認し、結果を出力
        
        // TODO: 4. 最初の "Apple" を削除し、リストの内容を再度出力
        
        // TODO: 5. インデックス1の要素を取得して出力
    }

    /**
     * Setの基本操作をデモンストレーションします。
     */
    public void demonstrateSet() {
        System.out.println("\n--- Set Demonstration ---");
        Set<String> colors = new HashSet<>();
        
        // TODO: 1. "Red", "Green", "Blue", "Red" をセットに追加
        
        // TODO: 2. セットの内容をコンソールに出力（順序は問わない）
        
        // TODO: 3. セットのサイズを出力
        
        // TODO: 4. "Green" がセットに含まれているか確認し、結果を出力
        
        // TODO: 5. "Blue" を削除し、セットの内容を再度出力
    }

    /**
     * Mapの基本操作をデモンストレーションします。
     */
    public void demonstrateMap() {
        System.out.println("\n--- Map Demonstration ---");
        Map<Integer, String> studentMap = new HashMap<>();
        
        // TODO: 1. 以下のキーと値のペアをマップに追加
        //    キー: 101, 値: "Alice"
        //    キー: 102, 値: "Bob"
        //    キー: 103, 値: "Charlie"
        
        // TODO: 2. マップの内容をコンソールに出力（キーと値のペア）
        
        // TODO: 3. キー 102 に対応する値を取得して出力
        
        // TODO: 4. キー 101 に対応する値を "Alicia" に更新
        
        // TODO: 5. キー 104 がマップに含まれているか確認し、結果を出力
        
        // TODO: 6. キー 103 のペアを削除し、マップの内容を再度出力
    }

    public static void main(String[] args) {
        BasicOperations demo = new BasicOperations();
        demo.demonstrateList();
        demo.demonstrateSet();
        demo.demonstrateMap();
    }
}
```

#### 実装ヒント
- `List`: `add()`, `contains()`, `remove()`, `get()`, `size()`
- `Set`: `add()`, `contains()`, `remove()`, `size()`
- `Map`: `put()`, `get()`, `containsKey()`, `remove()`, `entrySet()` (for iteration)

### 演習2: 適切なコレクションの選択（難易度：応用）

#### 課題
以下の各シナリオに対して、最も適切なコレクション（`ArrayList`, `HashSet`, `LinkedHashSet`, `TreeSet`, `HashMap`, `LinkedHashMap`, `TreeMap`）を選択し、その理由（特に性能面や機能面でのメリット）を説明してください。簡単なコード例も示してください。

1. **ユーザーIDのリストを管理し、重複なく、追加された順序で保持したい。**
2. **商品コード（String）とその価格（Double）を管理し、商品コードで高速に価格を検索したい。順序は問わない。**
3. **訪問したWebページのURLを記録し、訪問回数をカウントしたい。URLで高速に検索でき、かつURLをアルファベット順で表示したい。**
4. **タスクリストを管理し、タスクの追加・削除・完了状態の変更を頻繁に行う。タスクは実行順（追加順）に保持したい。**
5. **あるクラスの生徒名簿（String）を管理し、特定の生徒が名簿にいるか高速に確認したい。重複はなく、順序は問わない。**

#### 雛形コード
```java
package collections.choice;

import java.util.*;

/**
 * シナリオに応じて適切なコレクションを選択し、理由を説明するクラス
 */
public class CollectionChooser {

    public void scenario1() {
        System.out.println("--- Scenario 1: ユーザーIDリスト（重複なし、追加順保持） ---");
        // TODO: 適切なコレクションを選択し、インスタンス化
        // Collection<Long> userIds = new ...<>();
        
        // TODO: 簡単な操作例（追加、表示など）
        
        System.out.println("選択したコレクション: [コレクション名]");
        System.out.println("理由: [重複を許さず、追加順を保持するため、XXXが適している。検索性能は...]\n");
    }

    public void scenario2() {
        System.out.println("--- Scenario 2: 商品コードと価格（高速検索、順序不問） ---");
        // TODO: 適切なコレクションを選択し、インスタンス化
        // Map<String, Double> productPrices = new ...<>();
        
        // TODO: 簡単な操作例（追加、検索など）
        
        System.out.println("選択したコレクション: [コレクション名]");
        System.out.println("理由: [キーによる高速な検索が求められ、順序は問わないため、XXXが適している。計算量は...]\n");
    }

    public void scenario3() {
        System.out.println("--- Scenario 3: URL訪問回数（高速検索、キーソート） ---");
        // TODO: 適切なコレクションを選択し、インスタンス化
        // Map<String, Integer> urlVisitCounts = new ...<>();
        
        // TODO: 簡単な操作例（追加、インクリメント、表示など）
        
        System.out.println("選択したコレクション: [コレクション名]");
        System.out.println("理由: [キーで高速に検索でき、かつキーをソート順で保持する必要があるため、XXXが適している。計算量は...]\n");
    }

    public void scenario4() {
        System.out.println("--- Scenario 4: タスクリスト（追加順保持、頻繁な変更） ---");
        // TODO: 適切なコレクションを選択し、インスタンス化
        // List<String> taskList = new ...<>(); // または他のコレクション？
        
        // TODO: 簡単な操作例（追加、削除、表示など）
        
        System.out.println("選択したコレクション: [コレクション名]");
        System.out.println("理由: [追加順を保持し、要素の変更（完了状態など）も考慮するとXXXが適している。ただし、中間への挿入/削除が多い場合は...]\n");
    }

    public void scenario5() {
        System.out.println("--- Scenario 5: 生徒名簿（高速存在確認、重複なし、順序不問） ---");
        // TODO: 適切なコレクションを選択し、インスタンス化
        // Set<String> studentRoster = new ...<>();
        
        // TODO: 簡単な操作例（追加、存在確認など）
        
        System.out.println("選択したコレクション: [コレクション名]");
        System.out.println("理由: [重複なく、特定の要素の存在確認を高速に行いたいため、XXXが適している。計算量は...]\n");
    }

    public static void main(String[] args) {
        CollectionChooser chooser = new CollectionChooser();
        chooser.scenario1();
        chooser.scenario2();
        chooser.scenario3();
        chooser.scenario4();
        chooser.scenario5();
    }
}
```

#### 実装ヒント
- 各シナリオの要件（順序、重複、検索速度、キー/値）を整理する。
- 前述の「主要コレクション操作の計算量比較」表を参考にする。
- なぜ他のコレクションでは不適切なのかも考えると理解が深まる。

### 演習3: コレクションのパフォーマンス比較（難易度：応用）

#### 課題
`ArrayList`, `LinkedList`, `HashSet`, `TreeSet`に対して、大量の要素を追加する操作と、特定の要素を検索（`contains`）する操作の実行時間を計測し、比較するプログラムを作成してください。

#### 雛形コード
```java
package collections.performance;

import java.util.*;

/**
 * 各種コレクションのパフォーマンスを比較するクラス
 */
public class PerformanceComparator {

    private static final int NUM_ELEMENTS = 1_000_000; // 要素数
    private static final int NUM_SEARCHES = 100_000;  // 検索回数

    /**
     * 指定されたコレクションに要素を追加する時間を計測します。
     * 
     * @param collection 対象のコレクション
     * @param elementsToAdd 追加する要素数
     * @return 計測時間（ミリ秒）
     */
    public long measureAddTime(Collection<Integer> collection, int elementsToAdd) {
        long startTime = System.nanoTime();
        // TODO: 0からelementsToAdd-1までの整数をコレクションに追加
        long endTime = System.nanoTime();
        return (endTime - startTime) / 1_000_000; // ミリ秒に変換
    }

    /**
     * 指定されたコレクションで要素を検索する時間を計測します。
     * 
     * @param collection 対象のコレクション
     * @param searchesToPerform 検索回数
     * @return 計測時間（ミリ秒）
     */
    public long measureContainsTime(Collection<Integer> collection, int searchesToPerform) {
        Random random = new Random();
        long startTime = System.nanoTime();
        // TODO: searchesToPerform回、ランダムな整数（0からNUM_ELEMENTS*2の範囲）を検索
        //       collection.contains() を使用
        long endTime = System.nanoTime();
        return (endTime - startTime) / 1_000_000; // ミリ秒に変換
    }

    public static void main(String[] args) {
        PerformanceComparator comparator = new PerformanceComparator();

        List<Integer> arrayList = new ArrayList<>();
        List<Integer> linkedList = new LinkedList<>(); // LinkedListも比較対象に追加
        Set<Integer> hashSet = new HashSet<>();
        Set<Integer> treeSet = new TreeSet<>();

        System.out.println("要素数: " + NUM_ELEMENTS);
        System.out.println("検索回数: " + NUM_SEARCHES);
        System.out.println("-------------------------------------------");
        System.out.println("コレクション | 追加時間(ms) | 検索時間(ms)");
        System.out.println("-------------------------------------------");

        // ArrayList
        long addTime = comparator.measureAddTime(arrayList, NUM_ELEMENTS);
        long containsTime = comparator.measureContainsTime(arrayList, NUM_SEARCHES);
        System.out.printf("ArrayList    | %12d | %12d\n", addTime, containsTime);

        // LinkedList
        addTime = comparator.measureAddTime(linkedList, NUM_ELEMENTS);
        containsTime = comparator.measureContainsTime(linkedList, NUM_SEARCHES);
        System.out.printf("LinkedList   | %12d | %12d\n", addTime, containsTime);

        // HashSet
        addTime = comparator.measureAddTime(hashSet, NUM_ELEMENTS);
        containsTime = comparator.measureContainsTime(hashSet, NUM_SEARCHES);
        System.out.printf("HashSet      | %12d | %12d\n", addTime, containsTime);

        // TreeSet
        addTime = comparator.measureAddTime(treeSet, NUM_ELEMENTS);
        containsTime = comparator.measureContainsTime(treeSet, NUM_SEARCHES);
        System.out.printf("TreeSet      | %12d | %12d\n", addTime, containsTime);
        System.out.println("-------------------------------------------");
    }
}
```

#### 実装ヒント
- `System.nanoTime()` を使用して、処理の開始時刻と終了時刻を取得し、差分を計算します。
- `measureAddTime` では、単純なループで要素を追加します。
- `measureContainsTime` では、`Random`クラスを使って検索対象の値をランダムに生成し、`contains()`メソッドを呼び出します。検索対象が存在する場合としない場合の両方が含まれるように、検索値の範囲を要素数より広く設定します（例: `0` から `NUM_ELEMENTS * 2`）。
- `LinkedList`も比較対象に加えてみましょう。`ArrayList`との違いが明確になります。
- 実行結果から、各コレクションの計算量の違いが実際の性能差として現れることを確認します。

## 発展演習（オプション）

### 発展演習1: カスタムオブジェクトとhashCode/equals（難易度：発展）

#### 課題
`Person`クラス（フィールド: `id`, `name`）を作成し、`HashSet`と`HashMap`で`Person`オブジェクトを正しく扱えるように、`hashCode()`と`equals()`メソッドを適切にオーバーライドしてください。オーバーライドしない場合と、した場合で動作がどう変わるか確認してください。

#### 雛形コード
```java
package collections.custom;

import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

class Person {
    private int id;
    private String name;

    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() { return id; }
    public String getName() { return name; }

    @Override
    public String toString() {
        return "Person{id=" + id + ", name='" + name + "'}";
    }

    // TODO: equalsメソッドをオーバーライド
    // ヒント: idが同じであれば同じPersonとみなす
    @Override
    public boolean equals(Object o) {
        // 実装
        return false; // 仮
    }

    // TODO: hashCodeメソッドをオーバーライド
    // ヒント: equalsで使うフィールド(id)を元にハッシュコードを生成する
    //       Objects.hash(id) を使うと簡単
    @Override
    public int hashCode() {
        // 実装
        return 0; // 仮
    }
}

public class CustomObjectDemo {
    public static void main(String[] args) {
        Person p1 = new Person(1, "Alice");
        Person p2 = new Person(2, "Bob");
        Person p3 = new Person(1, "Alicia"); // IDはp1と同じだが名前が違う
        Person p4 = new Person(1, "Alice");  // p1と全く同じ

        Set<Person> personSet = new HashSet<>();
        personSet.add(p1);
        personSet.add(p2);
        personSet.add(p3);
        personSet.add(p4);

        System.out.println("Set size: " + personSet.size()); // hashCode/equals未実装だと？ 実装後だと？
        System.out.println("Set contains p1 (id=1, name=Alice)? " + personSet.contains(p1));
        System.out.println("Set contains p3 (id=1, name=Alicia)? " + personSet.contains(p3)); // 実装後だとどうなる？
        System.out.println("Set contains new Person(1, \"Alice\")? " + personSet.contains(new Person(1, "Alice"))); // 実装後だとどうなる？

        System.out.println("\nSet elements:");
        for (Person p : personSet) {
            System.out.println(p);
        }
    }
}
```

#### 実装ヒント
- `equals()`: `Object`クラスの`equals`をオーバーライドします。引数が`null`でないか、同じクラスかを確認し、フィールド（ここでは`id`）が等しいか比較します。
- `hashCode()`: `Object`クラスの`hashCode`をオーバーライドします。`equals`で使用するフィールド（ここでは`id`）に基づいてハッシュコードを生成します。`Objects.hash()`ユーティリティメソッドを使うと簡単に実装できます。
- **重要なルール**: `a.equals(b)`が`true`ならば、`a.hashCode() == b.hashCode()`でなければなりません。
- Eclipseには`hashCode()`と`equals()`を自動生成する機能があります（Source -> Generate hashCode() and equals()...）。

### 発展演習2: LRUキャッシュの実装（難易度：発展）

#### 課題
`LinkedHashMap`の機能を利用して、Least Recently Used（LRU）アルゴリズムに基づくシンプルなキャッシュ（最大サイズ固定）を実装してください。キャッシュがいっぱいになった場合、最も長い間アクセスされていない要素が自動的に削除されるようにします。

#### 雛形コード
```java
package collections.cache;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * LRUキャッシュを実装するクラス
 * LinkedHashMapを継承して実装します。
 * 
 * @param <K> キーの型
 * @param <V> 値の型
 */
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    /**
     * コンストラクタ
     * 
     * @param capacity キャッシュの最大容量
     */
    public LRUCache(int capacity) {
        // TODO: LinkedHashMapの適切なコンストラクタを呼び出す
        // ヒント: 初期容量、負荷係数、アクセス順序(accessOrder=true)を指定する
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    /**
     * 最も古いエントリを削除すべきかどうかを判断するメソッドをオーバーライドします。
     * Mapのサイズが容量を超えた場合にtrueを返します。
     */
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // TODO: キャッシュサイズが容量を超えたらtrueを返すように実装
        return size() > capacity;
    }

    public static void main(String[] args) {
        LRUCache<Integer, String> cache = new LRUCache<>(3); // 容量3のキャッシュ

        cache.put(1, "A");
        cache.put(2, "B");
        cache.put(3, "C");
        System.out.println("Initial cache: " + cache); // {1=A, 2=B, 3=C}

        cache.get(1); // 1にアクセス
        System.out.println("After accessing 1: " + cache); // {2=B, 3=C, 1=A} (アクセス順)

        cache.put(4, "D"); // 新しい要素を追加（容量オーバー）
        System.out.println("After adding 4: " + cache); // {3=C, 1=A, 4=D} (最も古い2=Bが削除される)

        cache.put(5, "E"); // さらに追加
        System.out.println("After adding 5: " + cache); // {1=A, 4=D, 5=E} (最も古い3=Cが削除される)
    }
}
```

#### 実装ヒント
- `LinkedHashMap`のコンストラクタ`LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder)`を呼び出します。`accessOrder`を`true`に設定すると、アクセス順（`get`メソッド呼び出しも含む）で要素が並び替えられます。
- `removeEldestEntry(Map.Entry<K, V> eldest)`メソッドをオーバーライドします。このメソッドは`put`や`putAll`の後に呼び出され、`true`を返すと最も古いエントリ（`eldest`）がマップから削除されます。現在のサイズが`capacity`を超えているかどうかを判定して`true`/`false`を返します。

## 解答例

（各演習の解答例コードをここに記載。長くなるため省略しますが、雛形コードのTODO部分を埋める形で作成します。）

## 解説と補足

### hashCode()とequals()の重要性

`HashSet`や`HashMap`などのハッシュベースのコレクションは、オブジェクトを効率的に格納・検索するために`hashCode()`と`equals()`メソッドを利用します。これらのメソッドが正しく実装されていないと、コレクションは期待通りに動作しません（例：同じはずのオブジェクトが重複して格納される、`contains`や`get`で見つからない）。

**契約（Contract）:**
1. `obj.equals(obj)` は常に `true`。
2. `a.equals(b)` が `true` ならば `b.equals(a)` も `true`。
3. `a.equals(b)` が `true` かつ `b.equals(c)` が `true` ならば `a.equals(c)` も `true`。
4. `a.equals(b)` の結果は、オブジェクトが変更されない限り常に同じ。
5. `a.equals(null)` は常に `false`。
6. **`a.equals(b)` が `true` ならば `a.hashCode() == b.hashCode()` でなければならない。**
7. `a.equals(b)` が `false` でも `a.hashCode() == b.hashCode()` となることは許容される（ハッシュ衝突）。ただし、衝突が少ないほど性能が良い。

カスタムオブジェクトをハッシュベースのコレクションで使用する場合は、必ずこれらの契約を満たすように`hashCode()`と`equals()`をオーバーライドしてください。

### Genericsのメリット

- **型安全性**: コンパイル時に型チェックが行われ、意図しない型のオブジェクトがコレクションに追加されるのを防ぎます。
- **キャスト不要**: コレクションから要素を取り出す際にキャストが不要になり、コードが簡潔になります。
- **可読性向上**: コレクションがどの型の要素を格納するかが明確になります。

### Concurrent Collections

複数のスレッドから同時にアクセスされる可能性がある場合は、スレッドセーフなコレクションを使用する必要があります。`java.util.concurrent`パッケージには、`ConcurrentHashMap`, `CopyOnWriteArrayList`, `CopyOnWriteArraySet`などのスレッドセーフなコレクションクラスが提供されています。（本カリキュラムでは深く扱いませんが、存在を知っておくことは重要です。）

## 実務での活用シーン

### 1. データの一時保管と処理
- DBから取得した大量のレコードを`ArrayList`に格納して処理する。
- 処理結果を`HashMap`に格納し、後でキーを使って参照する。
- ユーザーセッション情報を`HashMap`で管理する。

### 2. 重複データの排除
- ユーザー入力や外部データから重複する項目を除去するために`HashSet`を使用する。
- ユニークなIDの一覧を管理するために`HashSet`を使用する。

### 3. 設定値やマスタデータの管理
- アプリケーションの設定値をキーと値のペアで`HashMap`にロードして保持する。
- コード体系（例：都道府県コードと都道府県名）を`HashMap`や`TreeMap`で管理し、コードから名称を引けるようにする。

### 4. キャッシュの実装
- 頻繁にアクセスされるが生成コストが高いデータを`LinkedHashMap`（LRUキャッシュ）や`ConcurrentHashMap`でキャッシュし、応答性能を向上させる。

### 5. データのグルーピングと集計
- 大量のトランザクションデータを特定のキー（例：顧客ID、商品カテゴリ）でグルーピングするために`HashMap<Key, List<Data>>`のような構造を使用する。
- ログデータを解析し、IPアドレスごとのアクセス数を`HashMap<String, Integer>`で集計する。

## 参考資料

1. Java公式チュートリアル: [Collections Trail](https://docs.oracle.com/javase/tutorial/collections/index.html)
2. Java公式ドキュメント: [Collection (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collection.html)
3. Java公式ドキュメント: [Map (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Map.html)
4. 書籍: 「Effective Java 第3版」（Joshua Bloch著） - 特に項目10, 11 (equals, hashCode)
5. Baeldung: [Guide to Java Collections](https://www.baeldung.com/java-collections)
6. Baeldung: [A Guide to Big O Notation](https://www.baeldung.com/java-big-o-notation)
