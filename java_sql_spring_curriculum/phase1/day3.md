---
layout: default
title: "Day 3: ラムダ式と匿名クラスの適切な使い分け"
---
# Day 3: ラムダ式と匿名クラスの適切な使い分け

## 学習の目的と背景

Javaの進化とともに、より簡潔で読みやすいコードを書くための機能が追加されてきました。Java 8で導入されたラムダ式はその代表的な例です。ラムダ式は匿名クラスの代替として使用でき、コードの可読性と保守性を向上させることができます。

本日の学習では、匿名クラスとラムダ式の基本概念から、それらの違いと適切な使い分けまでを体系的に学びます。実際のコードを書きながら、それぞれの特性と利点を理解し、実務で効果的に活用できるようになることを目指します。

## 前提知識と準備

### 必要な前提知識
- Javaの基本構文（変数、制御構文、クラスなど）
- オブジェクト指向プログラミングの基本概念
- インターフェースの基本的な使い方

### 環境準備
- Eclipse 2023-12 (4.30.0)
- Java 21

### プロジェクト作成手順

1. Eclipseを起動します。
2. 「File」→「New」→「Java Project」を選択します。
3. プロジェクト名を「JavaLambda_Day3」と入力します。
4. 「JRE」の設定で「Use an execution environment JRE:」から「JavaSE-21」を選択します。
5. 「Finish」をクリックしてプロジェクトを作成します。

## 基本概念の解説

### 1. 匿名クラスの基本

匿名クラスは、クラス定義とインスタンス化を同時に行う方法です。主にインターフェースや抽象クラスの実装を一度だけ使用する場合に便利です。

**用語解説: 匿名クラス**  
名前を持たないクラスで、宣言と同時にインスタンス化されます。一度だけ使用するインターフェースや抽象クラスの実装に適しています。

```java
// 匿名クラスの基本的な使い方
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("匿名クラスで実装されたrun()メソッド");
    }
};

// 匿名クラスのインスタンスを使用
runnable.run();
```

### 2. 関数型インターフェース

関数型インターフェースは、単一の抽象メソッドを持つインターフェースです。Java 8以降では、`@FunctionalInterface`アノテーションを使用して明示的に関数型インターフェースであることを示すことができます。

**用語解説: 関数型インターフェース**  
単一の抽象メソッドを持つインターフェースです。ラムダ式の対象となるインターフェースで、Java 8以降では`@FunctionalInterface`アノテーションで明示できます。

```java
// 関数型インターフェースの定義
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

// 関数型インターフェースの実装（匿名クラスを使用）
Calculator addition = new Calculator() {
    @Override
    public int calculate(int a, int b) {
        return a + b;
    }
};

// 関数型インターフェースの使用
int result = addition.calculate(5, 3);  // 結果: 8
```

### 3. ラムダ式の基本

ラムダ式は、関数型インターフェースのインスタンスを作成するための簡潔な方法です。匿名クラスよりも簡潔で読みやすいコードを書くことができます。

**用語解説: ラムダ式**  
関数型インターフェースのインスタンスを作成するための簡潔な表現方法です。`(引数) -> { 処理 }`の形式で記述します。

```java
// ラムダ式の基本的な使い方
Runnable runnable = () -> {
    System.out.println("ラムダ式で実装されたrun()メソッド");
};

// ラムダ式のインスタンスを使用
runnable.run();

// 関数型インターフェースの実装（ラムダ式を使用）
Calculator addition = (a, b) -> a + b;

// ラムダ式の使用
int result = addition.calculate(5, 3);  // 結果: 8
```

### 4. ラムダ式の構文

ラムダ式の構文は、以下のようなバリエーションがあります：

1. **引数なし**：
```java
Runnable r = () -> System.out.println("Hello");
```

2. **引数1つ（型推論可能な場合は括弧を省略可能）**：
```java
Consumer<String> c = s -> System.out.println(s);
```

3. **引数2つ以上**：
```java
BiFunction<Integer, Integer, Integer> f = (a, b) -> a + b;
```

4. **複数の文を含む場合**：
```java
Comparator<String> c = (s1, s2) -> {
    int result = s1.length() - s2.length();
    if (result == 0) {
        result = s1.compareTo(s2);
    }
    return result;
};
```

5. **明示的な型指定**：
```java
BiFunction<Integer, Integer, Integer> f = (Integer a, Integer b) -> a + b;
```

### 5. メソッド参照

メソッド参照は、既存のメソッドをラムダ式の代わりに使用する方法です。コードをさらに簡潔にすることができます。

**用語解説: メソッド参照**  
既存のメソッドをラムダ式の代わりに使用する方法です。`クラス名::メソッド名`または`インスタンス::メソッド名`の形式で記述します。

メソッド参照には、以下の4種類があります：

1. **静的メソッド参照**：
```java
Function<String, Integer> f = Integer::parseInt;
```

2. **特定オブジェクトのインスタンスメソッド参照**：
```java
String str = "Hello";
Supplier<Integer> s = str::length;
```

3. **特定の型のインスタンスメソッド参照**：
```java
Function<String, Integer> f = String::length;
```

4. **コンストラクタ参照**：
```java
Supplier<ArrayList<String>> s = ArrayList::new;
```

### 6. ラムダ式と匿名クラスの違い

ラムダ式と匿名クラスには、以下のような違いがあります：

1. **構文の簡潔さ**：
   - ラムダ式は匿名クラスよりも簡潔に記述できます。
   - 例：`Runnable r = () -> System.out.println("Hello");`

2. **スコープ**：
   - 匿名クラスは独自のスコープを持ちます。`this`は匿名クラス自身を参照します。
   - ラムダ式は囲むスコープと同じスコープを持ちます。`this`は囲むクラスを参照します。

3. **シャドーイング**：
   - 匿名クラスでは、外部スコープの変数をシャドーイングできます。
   - ラムダ式では、外部スコープの変数をシャドーイングできません。

4. **機能的制限**：
   - 匿名クラスでは、複数のメソッドを実装できます。
   - ラムダ式は、単一の抽象メソッドを持つインターフェース（関数型インターフェース）にのみ使用できます。

### 7. ラムダ式と匿名クラスの使い分け

ラムダ式と匿名クラスは、以下のような基準で使い分けることができます：

1. **単純な処理**：
   - 単一のメソッドを実装し、処理が簡単な場合は、ラムダ式を使用します。
   - 例：`button.addActionListener(e -> System.out.println("クリックされました"));`

2. **複雑な処理**：
   - 複数のメソッドを実装する必要がある場合や、処理が複雑な場合は、匿名クラスを使用します。
   - 例：複数のリスナーメソッドを持つインターフェースの実装

3. **状態の保持**：
   - 状態を保持する必要がある場合は、匿名クラスを使用します。
   - ラムダ式では、外部の変数を参照できますが、それらは実質的にfinalである必要があります。

4. **`this`の参照**：
   - `this`が匿名クラス自身を参照する必要がある場合は、匿名クラスを使用します。
   - `this`が囲むクラスを参照する必要がある場合は、ラムダ式を使用します。

## Eclipse操作ガイド

### クラスの作成

1. プロジェクト「JavaLambda_Day3」を右クリックします。
2. 「New」→「Class」を選択します。
3. 「Name」に作成するクラス名（例：`LambdaDemo`）を入力します。
4. 「public static void main(String[] args)」にチェックを入れます。
5. 「Finish」をクリックしてクラスを作成します。

### ラムダ式の入力支援

1. 関数型インターフェースの変数を宣言します（例：`Runnable r = `）。
2. `=`の後にスペースを入力し、`Ctrl + Space`を押します。
3. 候補リストから「Lambda Expression」を選択します。
4. Eclipseが自動的にラムダ式の雛形を生成します。

### ラムダ式のデバッグ

1. ラムダ式を含むコードにブレークポイントを設定します（行番号の左側をダブルクリック）。
2. 右クリックして「Debug As」→「Java Application」を選択します。
3. デバッグパースペクティブで変数の値を確認します。
4. 「Step Into」（F5）を使用して、ラムダ式の内部に入ることができます。

### 匿名クラスからラムダ式への変換

1. 匿名クラスを含むコードを選択します。
2. 右クリックして「Refactor」→「Convert Anonymous to Lambda Expression」を選択します。
3. Eclipseが自動的に匿名クラスをラムダ式に変換します。

## コア演習（必須）

### 演習1: 匿名クラスの基本（難易度：基本）

#### 課題
以下の関数型インターフェースを匿名クラスを使って実装してください：
1. `Runnable`インターフェース
2. `Comparator<String>`インターフェース
3. 独自の関数型インターフェース`StringProcessor`

#### 雛形コード
以下のクラスを作成してください：

```java
package lambda.basic;

import java.util.Arrays;
import java.util.Comparator;

/**
 * 文字列を処理する関数型インターフェース
 */
@FunctionalInterface
interface StringProcessor {
    String process(String input);
}

/**
 * 匿名クラスの基本を学ぶためのデモクラス
 */
public class AnonymousClassDemo {
    
    public static void main(String[] args) {
        // 1. Runnableインターフェースの実装（匿名クラスを使用）
        System.out.println("===== Runnableインターフェースの実装 =====");
        
        // TODO: ここに匿名クラスを使ったRunnableの実装を追加
        // ヒント: new Runnable() { ... } の形式で実装
        
        // 2. Comparator<String>インターフェースの実装（匿名クラスを使用）
        System.out.println("\n===== Comparator<String>インターフェースの実装 =====");
        
        // 文字列の配列
        String[] names = {"田中", "佐藤", "鈴木", "高橋", "伊藤"};
        
        System.out.println("ソート前: " + Arrays.toString(names));
        
        // TODO: ここに匿名クラスを使ったComparator<String>の実装を追加
        // ヒント1: new Comparator<String>() { ... } の形式で実装
        // ヒント2: 文字列の長さでソートする比較関数を実装
        
        // 配列をソート
        // Arrays.sort(names, comparator);
        
        System.out.println("ソート後: " + Arrays.toString(names));
        
        // 3. 独自の関数型インターフェースStringProcessorの実装（匿名クラスを使用）
        System.out.println("\n===== StringProcessorインターフェースの実装 =====");
        
        // TODO: ここに匿名クラスを使ったStringProcessorの実装を追加
        // ヒント1: new StringProcessor() { ... } の形式で実装
        // ヒント2: 文字列を大文字に変換する処理を実装
        
        // StringProcessorを使用して文字列を処理
        String input = "Hello, World!";
        // String processed = processor.process(input);
        
        // System.out.println("元の文字列: " + input);
        // System.out.println("処理後の文字列: " + processed);
    }
}
```

#### 実装ヒント
1. `Runnable`インターフェースの実装では、`run()`メソッドをオーバーライドし、簡単なメッセージを表示します。
2. `Comparator<String>`インターフェースの実装では、`compare(String s1, String s2)`メソッドをオーバーライドし、文字列の長さを比較します。
3. `StringProcessor`インターフェースの実装では、`process(String input)`メソッドをオーバーライドし、文字列を大文字に変換します。

### 演習2: ラムダ式の基本（難易度：基本）

#### 課題
演習1で実装した匿名クラスをラムダ式に書き換えてください：
1. `Runnable`インターフェース
2. `Comparator<String>`インターフェース
3. 独自の関数型インターフェース`StringProcessor`

#### 雛形コード
以下のクラスを作成してください：

```java
package lambda.basic;

import java.util.Arrays;
import java.util.Comparator;

/**
 * ラムダ式の基本を学ぶためのデモクラス
 */
public class LambdaExpressionDemo {
    
    public static void main(String[] args) {
        // 1. Runnableインターフェースの実装（ラムダ式を使用）
        System.out.println("===== Runnableインターフェースの実装 =====");
        
        // TODO: ここにラムダ式を使ったRunnableの実装を追加
        // ヒント: () -> { ... } の形式で実装
        
        // 2. Comparator<String>インターフェースの実装（ラムダ式を使用）
        System.out.println("\n===== Comparator<String>インターフェースの実装 =====");
        
        // 文字列の配列
        String[] names = {"田中", "佐藤", "鈴木", "高橋", "伊藤"};
        
        System.out.println("ソート前: " + Arrays.toString(names));
        
        // TODO: ここにラムダ式を使ったComparator<String>の実装を追加
        // ヒント: (s1, s2) -> ... の形式で実装
        
        // 配列をソート
        // Arrays.sort(names, comparator);
        
        System.out.println("ソート後: " + Arrays.toString(names));
        
        // 3. 独自の関数型インターフェースStringProcessorの実装（ラムダ式を使用）
        System.out.println("\n===== StringProcessorインターフェースの実装 =====");
        
        // TODO: ここにラムダ式を使ったStringProcessorの実装を追加
        // ヒント: input -> ... の形式で実装
        
        // StringProcessorを使用して文字列を処理
        String input = "Hello, World!";
        // String processed = processor.process(input);
        
        // System.out.println("元の文字列: " + input);
        // System.out.println("処理後の文字列: " + processed);
    }
}
```

#### 実装ヒント
1. `Runnable`インターフェースの実装では、`() -> { System.out.println("..."); }`の形式でラムダ式を記述します。
2. `Comparator<String>`インターフェースの実装では、`(s1, s2) -> s1.length() - s2.length()`の形式でラムダ式を記述します。
3. `StringProcessor`インターフェースの実装では、`input -> input.toUpperCase()`の形式でラムダ式を記述します。

### 演習3: ラムダ式と匿名クラスの比較（難易度：応用）

#### 課題
ラムダ式と匿名クラスの違いを理解するために、以下の点を比較するプログラムを作成してください：
1. `this`の参照の違い
2. 変数のシャドーイングの違い
3. 外部変数の参照の違い

#### 雛形コード
以下のクラスを作成してください：

```java
package lambda.comparison;

/**
 * ラムダ式と匿名クラスの違いを比較するデモクラス
 */
public class LambdaVsAnonymousDemo {
    
    private String instanceVar = "インスタンス変数";
    
    /**
     * thisの参照の違いを比較します。
     */
    public void compareThis() {
        System.out.println("===== thisの参照の違い =====");
        
        // 匿名クラスでのthisの参照
        Runnable anonymousRunnable = new Runnable() {
            @Override
            public void run() {
                // TODO: ここにthisを使った処理を追加
                // ヒント1: thisが何を参照しているか確認
                // ヒント2: 外部クラスのインスタンス変数にアクセスする方法を確認
            }
        };
        
        // ラムダ式でのthisの参照
        Runnable lambdaRunnable = () -> {
            // TODO: ここにthisを使った処理を追加
            // ヒント: thisが何を参照しているか確認
        };
        
        System.out.println("匿名クラスでのthisの参照:");
        anonymousRunnable.run();
        
        System.out.println("\nラムダ式でのthisの参照:");
        lambdaRunnable.run();
    }
    
    /**
     * 変数のシャドーイングの違いを比較します。
     */
    public void compareShadowing() {
        System.out.println("\n===== 変数のシャドーイングの違い =====");
        
        String var = "外部変数";
        
        // 匿名クラスでの変数のシャドーイング
        Runnable anonymousRunnable = new Runnable() {
            // TODO: ここに外部変数と同名の変数を宣言
            // String var = "匿名クラス内変数";
            
            @Override
            public void run() {
                // TODO: ここにvarを使った処理を追加
                // ヒント: varが何を参照しているか確認
            }
        };
        
        // ラムダ式での変数のシャドーイング
        Runnable lambdaRunnable = () -> {
            // TODO: ここに外部変数と同名の変数を宣言しようとしてみる
            // String var = "ラムダ式内変数";  // コンパイルエラーになるはず
            
            // TODO: ここにvarを使った処理を追加
            // ヒント: varが何を参照しているか確認
        };
        
        System.out.println("匿名クラスでの変数のシャドーイング:");
        anonymousRunnable.run();
        
        System.out.println("\nラムダ式での変数のシャドーイング:");
        lambdaRunnable.run();
    }
    
    /**
     * 外部変数の参照の違いを比較します。
     */
    public void compareExternalVarReference() {
        System.out.println("\n===== 外部変数の参照の違い =====");
        
        // 外部変数
        int[] counter = {0};  // 配列を使って実質的なfinalを回避
        
        // 匿名クラスでの外部変数の参照
        Runnable anonymousRunnable = new Runnable() {
            @Override
            public void run() {
                // TODO: ここにcounterを使った処理を追加
                // ヒント: counterの値を増やして表示
            }
        };
        
        // ラムダ式での外部変数の参照
        Runnable lambdaRunnable = () -> {
            // TODO: ここにcounterを使った処理を追加
            // ヒント: counterの値を増やして表示
        };
        
        System.out.println("匿名クラスでの外部変数の参照:");
        anonymousRunnable.run();
        System.out.println("counter = " + counter[0]);
        
        System.out.println("\nラムダ式での外部変数の参照:");
        lambdaRunnable.run();
        System.out.println("counter = " + counter[0]);
    }
    
    public static void main(String[] args) {
        LambdaVsAnonymousDemo demo = new LambdaVsAnonymousDemo();
        demo.compareThis();
        demo.compareShadowing();
        demo.compareExternalVarReference();
    }
}
```

#### 実装ヒント
1. `compareThis`メソッドでは、匿名クラス内の`this`は匿名クラス自身を参照し、ラムダ式内の`this`は囲むクラス（`LambdaVsAnonymousDemo`）を参照することを確認します。
2. `compareShadowing`メソッドでは、匿名クラス内で外部変数と同名の変数を宣言できますが、ラムダ式内では外部変数と同名の変数を宣言するとコンパイルエラーになることを確認します。
3. `compareExternalVarReference`メソッドでは、匿名クラスとラムダ式の両方で外部変数を参照できますが、その変数は実質的にfinalである必要があることを確認します。配列を使って実質的なfinalを回避する方法も確認します。

## 発展演習（オプション）

### 発展演習1: カスタム関数型インターフェース（難易度：応用）

#### 課題
以下の要件を満たすカスタム関数型インターフェースを作成し、それをラムダ式で実装してください：

1. 2つの引数を受け取り、1つの結果を返す関数型インターフェース`BiCalculator<T, U, R>`
2. 3つの引数を受け取り、結果を返さない関数型インターフェース`TriConsumer<T, U, V>`
3. 1つの引数を受け取り、boolean型の結果を返す関数型インターフェース`Validator<T>`

#### 雛形コード
以下のクラスを作成してください：

```java
package lambda.advanced;

import java.util.function.BiFunction;
import java.util.function.Consumer;
import java.util.function.Predicate;

/**
 * 2つの引数を受け取り、1つの結果を返す関数型インターフェース
 * 
 * @param <T> 第1引数の型
 * @param <U> 第2引数の型
 * @param <R> 結果の型
 */
@FunctionalInterface
interface BiCalculator<T, U, R> {
    // TODO: ここに抽象メソッドを定義
    // ヒント: 2つの引数を受け取り、1つの結果を返すメソッドを定義
}

/**
 * 3つの引数を受け取り、結果を返さない関数型インターフェース
 * 
 * @param <T> 第1引数の型
 * @param <U> 第2引数の型
 * @param <V> 第3引数の型
 */
@FunctionalInterface
interface TriConsumer<T, U, V> {
    // TODO: ここに抽象メソッドを定義
    // ヒント: 3つの引数を受け取り、結果を返さないメソッドを定義
}

/**
 * 1つの引数を受け取り、boolean型の結果を返す関数型インターフェース
 * 
 * @param <T> 引数の型
 */
@FunctionalInterface
interface Validator<T> {
    // TODO: ここに抽象メソッドを定義
    // ヒント: 1つの引数を受け取り、boolean型の結果を返すメソッドを定義
}

/**
 * カスタム関数型インターフェースを使用するデモクラス
 */
public class CustomFunctionalInterfaceDemo {
    
    public static void main(String[] args) {
        // BiCalculatorの使用例
        System.out.println("===== BiCalculatorの使用例 =====");
        
        // TODO: ここにBiCalculatorのラムダ式実装を追加
        // ヒント1: 整数の加算を行うBiCalculatorを実装
        // ヒント2: 文字列の連結を行うBiCalculatorを実装
        
        // TriConsumerの使用例
        System.out.println("\n===== TriConsumerの使用例 =====");
        
        // TODO: ここにTriConsumerのラムダ式実装を追加
        // ヒント: 3つの引数を受け取り、それらを組み合わせて表示するTriConsumerを実装
        
        // Validatorの使用例
        System.out.println("\n===== Validatorの使用例 =====");
        
        // TODO: ここにValidatorのラムダ式実装を追加
        // ヒント1: 文字列が空でないことを検証するValidatorを実装
        // ヒント2: 整数が正の値であることを検証するValidatorを実装
        
        // 標準関数型インターフェースとの比較
        System.out.println("\n===== 標準関数型インターフェースとの比較 =====");
        
        // BiFunction（標準関数型インターフェース）の使用例
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        System.out.println("BiFunction: 5 + 3 = " + add.apply(5, 3));
        
        // Consumer（標準関数型インターフェース）の使用例
        Consumer<String> print = s -> System.out.println("Consumer: " + s);
        print.accept("Hello, World!");
        
        // Predicate（標準関数型インターフェース）の使用例
        Predicate<Integer> isPositive = n -> n > 0;
        System.out.println("Predicate: 5 is positive? " + isPositive.test(5));
        System.out.println("Predicate: -3 is positive? " + isPositive.test(-3));
    }
}
```

#### 実装ヒント
1. `BiCalculator<T, U, R>`インターフェースでは、`R calculate(T t, U u)`のような抽象メソッドを定義します。
2. `TriConsumer<T, U, V>`インターフェースでは、`void accept(T t, U u, V v)`のような抽象メソッドを定義します。
3. `Validator<T>`インターフェースでは、`boolean validate(T t)`のような抽象メソッドを定義します。
4. 各インターフェースのラムダ式実装では、具体的な処理を記述します。例えば、整数の加算、文字列の連結、値の検証などです。

### 発展演習2: メソッド参照の活用（難易度：発展）

#### 課題
メソッド参照を活用して、以下の処理を実装してください：

1. 静的メソッド参照を使用した文字列の変換
2. インスタンスメソッド参照を使用したリストのフィルタリング
3. コンストラクタ参照を使用したオブジェクトの生成

#### 雛形コード
以下のクラスを作成してください：

```java
package lambda.advanced;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

/**
 * 商品情報を表すクラス
 */
class Product {
    private String name;
    private int price;
    private String category;
    
    public Product(String name, int price, String category) {
        this.name = name;
        this.price = price;
        this.category = category;
    }
    
    // ゲッター
    public String getName() { return name; }
    public int getPrice() { return price; }
    public String getCategory() { return category; }
    
    // 静的メソッド: 商品名を大文字に変換
    public static String toUpperCase(String name) {
        return name.toUpperCase();
    }
    
    // インスタンスメソッド: 商品が指定された価格以上かどうかを判定
    public boolean isPriceGreaterThanOrEqual(int threshold) {
        return price >= threshold;
    }
    
    @Override
    public String toString() {
        return "Product [name=" + name + ", price=" + price + ", category=" + category + "]";
    }
}

/**
 * メソッド参照を活用するデモクラス
 */
public class MethodReferenceDemo {
    
    public static void main(String[] args) {
        // 商品リストの作成
        List<Product> products = Arrays.asList(
            new Product("ノートパソコン", 120000, "電化製品"),
            new Product("スマートフォン", 80000, "電化製品"),
            new Product("コーヒーメーカー", 15000, "キッチン"),
            new Product("電子レンジ", 30000, "キッチン"),
            new Product("テレビ", 200000, "電化製品")
        );
        
        // 1. 静的メソッド参照を使用した文字列の変換
        System.out.println("===== 静的メソッド参照を使用した文字列の変換 =====");
        
        // TODO: ここに静的メソッド参照を使用したコードを追加
        // ヒント1: Product.toUpperCaseメソッドを参照するFunctionを作成
        // ヒント2: 商品名のリストを作成し、各商品名を大文字に変換
        
        // 2. インスタンスメソッド参照を使用したリストのフィルタリング
        System.out.println("\n===== インスタンスメソッド参照を使用したリストのフィルタリング =====");
        
        // TODO: ここにインスタンスメソッド参照を使用したコードを追加
        // ヒント1: Product::isPriceGreaterThanOrEqualメソッドを参照するPredicateを作成
        // ヒント2: 商品リストから価格が50000円以上の商品をフィルタリング
        
        // 3. コンストラクタ参照を使用したオブジェクトの生成
        System.out.println("\n===== コンストラクタ参照を使用したオブジェクトの生成 =====");
        
        // TODO: ここにコンストラクタ参照を使用したコードを追加
        // ヒント1: ArrayList::newを参照するSupplierを作成
        // ヒント2: Supplierを使用して新しいArrayListを生成
    }
    
    /**
     * 条件に一致する要素だけを含む新しいリストを返します。
     * 
     * @param <T> リストの要素の型
     * @param list フィルタリング対象のリスト
     * @param predicate フィルタリング条件
     * @return 条件に一致する要素だけを含む新しいリスト
     */
    public static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
        List<T> result = new ArrayList<>();
        for (T item : list) {
            if (predicate.test(item)) {
                result.add(item);
            }
        }
        return result;
    }
    
    /**
     * リストの各要素に関数を適用した結果を含む新しいリストを返します。
     * 
     * @param <T> 入力リストの要素の型
     * @param <R> 出力リストの要素の型
     * @param list 変換対象のリスト
     * @param function 変換関数
     * @return 変換結果を含む新しいリスト
     */
    public static <T, R> List<R> map(List<T> list, Function<T, R> function) {
        List<R> result = new ArrayList<>();
        for (T item : list) {
            result.add(function.apply(item));
        }
        return result;
    }
}
```

#### 実装ヒント
1. 静的メソッド参照では、`Product::toUpperCase`を使用して、商品名を大文字に変換します。
2. インスタンスメソッド参照では、`Product::isPriceGreaterThanOrEqual`を使用して、価格が指定された閾値以上の商品をフィルタリングします。
3. コンストラクタ参照では、`ArrayList::new`を使用して、新しい`ArrayList`インスタンスを生成します。

## 解説と補足

### ラムダ式と匿名クラスの使い分け

ラムダ式と匿名クラスは、どちらも関数型インターフェースの実装を提供する方法ですが、それぞれに適した使用シーンがあります。

#### ラムダ式が適している場合

1. **単純な処理**：
   - 処理が単純で、1行または数行で記述できる場合
   - 例：`button.addActionListener(e -> System.out.println("クリックされました"));`

2. **関数型インターフェースの実装**：
   - 単一の抽象メソッドを持つインターフェースの実装
   - 例：`Runnable`, `Comparator`, `Consumer`, `Function`など

3. **コードの可読性を重視する場合**：
   - 簡潔で読みやすいコードを書きたい場合
   - 例：`list.forEach(item -> System.out.println(item));`

4. **囲むクラスのコンテキストにアクセスする場合**：
   - `this`が囲むクラスを参照する必要がある場合
   - 例：インスタンス変数やメソッドにアクセスする場合

#### 匿名クラスが適している場合

1. **複雑な処理**：
   - 処理が複雑で、複数のメソッドや変数が必要な場合
   - 例：複数のコールバックメソッドを持つリスナーの実装

2. **複数のメソッドを持つインターフェースの実装**：
   - 関数型インターフェースではないインターフェースの実装
   - 例：複数のメソッドを持つリスナーインターフェース

3. **独自のスコープが必要な場合**：
   - 独自の変数やメソッドを持つ必要がある場合
   - 例：状態を保持する必要がある場合

4. **`this`が実装自身を参照する必要がある場合**：
   - 匿名クラス内で`this`を使用して自身を参照する必要がある場合
   - 例：自身のメソッドを呼び出す場合

### ラムダ式のパフォーマンス

ラムダ式は、匿名クラスと比較して以下のようなパフォーマンス上の特徴があります：

1. **メモリ使用量**：
   - ラムダ式は、匿名クラスよりも少ないメモリを使用します。
   - 匿名クラスは、クラスファイルごとに新しいクラスが生成されますが、ラムダ式はそうではありません。

2. **初期化時間**：
   - ラムダ式は、匿名クラスよりも初期化が高速です。
   - 匿名クラスは、クラスのロードと初期化が必要ですが、ラムダ式はそうではありません。

3. **実行時間**：
   - 単純な処理の場合、ラムダ式と匿名クラスの実行時間に大きな違いはありません。
   - 複雑な処理や頻繁に呼び出される処理の場合、ラムダ式の方が若干高速な場合があります。

### メソッド参照の活用

メソッド参照は、既存のメソッドをラムダ式の代わりに使用する方法です。以下のような場合に活用できます：

1. **コードの簡潔さ**：
   - メソッド参照は、ラムダ式よりもさらに簡潔に記述できます。
   - 例：`list.forEach(System.out::println);`

2. **既存のメソッドの再利用**：
   - 既に実装されているメソッドを再利用できます。
   - 例：`strings.stream().map(String::toUpperCase);`

3. **コードの意図の明確化**：
   - メソッド名が処理の意図を明確に表現している場合、メソッド参照を使用することで、コードの意図がより明確になります。
   - 例：`list.stream().filter(Objects::nonNull);`

### 関数型プログラミングの考え方

ラムダ式と関数型インターフェースの導入により、Javaでも関数型プログラミングのスタイルを取り入れることができるようになりました。関数型プログラミングの主な特徴は以下の通りです：

1. **不変性（Immutability）**：
   - データは変更されず、新しいデータが生成されます。
   - 例：`String`クラスは不変であり、文字列操作は新しい`String`インスタンスを返します。

2. **副作用のない関数（Pure Functions）**：
   - 関数は入力に対して常に同じ出力を返し、外部の状態を変更しません。
   - 例：`Math.abs(-5)`は常に`5`を返し、外部の状態を変更しません。

3. **高階関数（Higher-Order Functions）**：
   - 関数を引数として受け取ったり、関数を戻り値として返したりする関数です。
   - 例：`Stream.map()`は関数を引数として受け取ります。

4. **遅延評価（Lazy Evaluation）**：
   - 式の評価は、結果が必要になるまで遅延されます。
   - 例：`Stream`のパイプラインは、終端操作が呼び出されるまで評価されません。

## 実務での活用シーン

### 1. イベント処理

ラムダ式は、GUIアプリケーションやWebアプリケーションでのイベント処理に適しています。

```java
// ボタンクリックイベントの処理（匿名クラスを使用）
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("ボタンがクリックされました");
    }
});

// ボタンクリックイベントの処理（ラムダ式を使用）
button.addActionListener(e -> System.out.println("ボタンがクリックされました"));
```

### 2. コレクション操作

ラムダ式とStream APIを組み合わせることで、コレクションの操作を簡潔に記述できます。

```java
// 商品リストから価格が10000円以上の商品を抽出し、名前のリストを作成（従来の方法）
List<String> expensiveProductNames = new ArrayList<>();
for (Product product : products) {
    if (product.getPrice() >= 10000) {
        expensiveProductNames.add(product.getName());
    }
}

// 商品リストから価格が10000円以上の商品を抽出し、名前のリストを作成（ラムダ式とStream APIを使用）
List<String> expensiveProductNames = products.stream()
    .filter(product -> product.getPrice() >= 10000)
    .map(Product::getName)
    .collect(Collectors.toList());
```

### 3. 非同期処理

ラムダ式は、非同期処理やマルチスレッド処理にも適しています。

```java
// 非同期タスクの実行（匿名クラスを使用）
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        return "非同期タスクの結果";
    }
});

// 非同期タスクの実行（ラムダ式を使用）
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.submit(() -> "非同期タスクの結果");
```

### 4. 依存性の注入とファクトリパターン

ラムダ式は、依存性の注入やファクトリパターンの実装にも活用できます。

```java
// 商品ファクトリー（匿名クラスを使用）
ProductFactory laptopFactory = new ProductFactory() {
    @Override
    public Product createProduct() {
        return new Product("ノートパソコン", 120000, "電化製品");
    }
};

// 商品ファクトリー（ラムダ式を使用）
ProductFactory laptopFactory = () -> new Product("ノートパソコン", 120000, "電化製品");
```

### 5. バリデーションとビジネスルール

ラムダ式は、バリデーションやビジネスルールの実装にも適しています。

```java
// バリデーションルール（匿名クラスを使用）
Validator<String> notEmptyValidator = new Validator<String>() {
    @Override
    public boolean validate(String value) {
        return value != null && !value.isEmpty();
    }
};

// バリデーションルール（ラムダ式を使用）
Validator<String> notEmptyValidator = value -> value != null && !value.isEmpty();

// 複数のバリデーションルールの組み合わせ
Validator<String> emailValidator = value -> value.matches("^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,6}$");
Validator<String> passwordValidator = value -> value.length() >= 8 && value.matches(".*[A-Z].*") && value.matches(".*[0-9].*");
```

これらの実務での活用シーンを参考に、ラムダ式と匿名クラスの適切な使い分けを理解し、効果的に活用してください。

## 解答例

<a href="answers/day3">解答例はこちら</a>