---
layout: default
title: "Day 3: ラムダ式と匿名クラスの適切な使い分け - 解答例"
---
# Day 3: ラムダ式と匿名クラスの適切な使い分け - 解答例

## 演習1: 匿名クラスの基本

### 解答例
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
        
        // 匿名クラスを使ったRunnableの実装
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("匿名クラスを使ったRunnableの実装です");
            }
        };
        
        // 実行
        runnable.run();
        
        // 2. Comparator<String>インターフェースの実装（匿名クラスを使用）
        System.out.println("\n===== Comparator<String>インターフェースの実装 =====");
        
        // 文字列の配列
        String[] names = {"田中", "佐藤", "鈴木", "高橋", "伊藤"};
        
        System.out.println("ソート前: " + Arrays.toString(names));
        
        // 匿名クラスを使ったComparator<String>の実装
        Comparator<String> comparator = new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                // 文字列の長さでソート
                return Integer.compare(s1.length(), s2.length());
            }
        };
        
        // 配列をソート
        Arrays.sort(names, comparator);
        
        System.out.println("ソート後: " + Arrays.toString(names));
        
        // 3. 独自の関数型インターフェースStringProcessorの実装（匿名クラスを使用）
        System.out.println("\n===== StringProcessorインターフェースの実装 =====");
        
        // 匿名クラスを使ったStringProcessorの実装
        StringProcessor processor = new StringProcessor() {
            @Override
            public String process(String input) {
                return input.toUpperCase();
            }
        };
        
        // StringProcessorを使用して文字列を処理
        String input = "Hello, World!";
        String processed = processor.process(input);
        
        System.out.println("元の文字列: " + input);
        System.out.println("処理後の文字列: " + processed);
    }
}
```

### 解説
この解答例では、3つの異なる関数型インターフェースを匿名クラスを使って実装しています。

1. **Runnableインターフェース**:
   - `run()`メソッドをオーバーライドして、メッセージを表示する処理を実装しています。
   - 匿名クラスの構文 `new Runnable() { ... }` を使用して、インターフェースの実装とインスタンス化を同時に行っています。

2. **Comparator<String>インターフェース**:
   - `compare(String s1, String s2)`メソッドをオーバーライドして、文字列の長さを比較する処理を実装しています。
   - この匿名クラスを使って、文字列の配列を文字列の長さでソートしています。

3. **StringProcessorインターフェース**:
   - `process(String input)`メソッドをオーバーライドして、文字列を大文字に変換する処理を実装しています。
   - この匿名クラスを使って、入力文字列を処理し、結果を表示しています。

匿名クラスは、インターフェースや抽象クラスの実装を一度だけ使用する場合に便利です。ただし、コードが冗長になりがちで、可読性が低下する場合があります。

## 演習2: ラムダ式の基本

### 解答例
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
        
        // ラムダ式を使ったRunnableの実装
        Runnable runnable = () -> {
            System.out.println("ラムダ式を使ったRunnableの実装です");
        };
        
        // 実行
        runnable.run();
        
        // 2. Comparator<String>インターフェースの実装（ラムダ式を使用）
        System.out.println("\n===== Comparator<String>インターフェースの実装 =====");
        
        // 文字列の配列
        String[] names = {"田中", "佐藤", "鈴木", "高橋", "伊藤"};
        
        System.out.println("ソート前: " + Arrays.toString(names));
        
        // ラムダ式を使ったComparator<String>の実装
        Comparator<String> comparator = (s1, s2) -> Integer.compare(s1.length(), s2.length());
        
        // 配列をソート
        Arrays.sort(names, comparator);
        
        System.out.println("ソート後: " + Arrays.toString(names));
        
        // 3. 独自の関数型インターフェースStringProcessorの実装（ラムダ式を使用）
        System.out.println("\n===== StringProcessorインターフェースの実装 =====");
        
        // ラムダ式を使ったStringProcessorの実装
        StringProcessor processor = input -> input.toUpperCase();
        
        // StringProcessorを使用して文字列を処理
        String input = "Hello, World!";
        String processed = processor.process(input);
        
        System.out.println("元の文字列: " + input);
        System.out.println("処理後の文字列: " + processed);
    }
}
```

### 解説
この解答例では、演習1で実装した匿名クラスをラムダ式に書き換えています。

1. **Runnableインターフェース**:
   - 匿名クラス: `new Runnable() { @Override public void run() { ... } }`
   - ラムダ式: `() -> { ... }`
   - 引数がないため、空の括弧 `()` を使用しています。

2. **Comparator<String>インターフェース**:
   - 匿名クラス: `new Comparator<String>() { @Override public int compare(String s1, String s2) { ... } }`
   - ラムダ式: `(s1, s2) -> Integer.compare(s1.length(), s2.length())`
   - 2つの引数 `s1` と `s2` を受け取り、文字列の長さを比較しています。

3. **StringProcessorインターフェース**:
   - 匿名クラス: `new StringProcessor() { @Override public String process(String input) { ... } }`
   - ラムダ式: `input -> input.toUpperCase()`
   - 1つの引数 `input` を受け取り、大文字に変換しています。
   - 処理が1行で完結するため、波括弧 `{}` と `return` キーワードを省略しています。

ラムダ式を使用することで、コードがより簡潔になり、可読性が向上しています。特に、単純な処理を行う場合は、ラムダ式の方が適しています。

## 演習3: ラムダ式と匿名クラスの比較

### 解答例
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
                // thisは匿名クラス自身を参照
                System.out.println("匿名クラス内のthis: " + this);
                
                // 外部クラスのインスタンス変数にアクセスするには、外部クラス名.thisを使用
                System.out.println("外部クラスのインスタンス変数: " + LambdaVsAnonymousDemo.this.instanceVar);
            }
        };
        
        // ラムダ式でのthisの参照
        Runnable lambdaRunnable = () -> {
            // thisは囲むクラス（LambdaVsAnonymousDemo）を参照
            System.out.println("ラムダ式内のthis: " + this);
            
            // 外部クラスのインスタンス変数に直接アクセス可能
            System.out.println("外部クラスのインスタンス変数: " + instanceVar);
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
            // 外部変数と同名の変数を宣言（シャドーイング）
            String var = "匿名クラス内変数";
            
            @Override
            public void run() {
                // varは匿名クラス内の変数を参照
                System.out.println("匿名クラス内のvar: " + var);
                
                // 外部変数にアクセスするには、外部クラス名.thisを使用
                System.out.println("外部のvar: " + LambdaVsAnonymousDemo.this.var);
            }
        };
        
        // ラムダ式での変数のシャドーイング
        Runnable lambdaRunnable = () -> {
            // 外部変数と同名の変数を宣言しようとするとコンパイルエラー
            // String var = "ラムダ式内変数";  // コンパイルエラー
            
            // varは外部変数を参照
            System.out.println("ラムダ式内のvar（外部変数）: " + var);
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
                // 外部変数（配列の要素）を変更
                counter[0]++;
                System.out.println("匿名クラス内でのcounter: " + counter[0]);
            }
        };
        
        // ラムダ式での外部変数の参照
        Runnable lambdaRunnable = () -> {
            // 外部変数（配列の要素）を変更
            counter[0]++;
            System.out.println("ラムダ式内でのcounter: " + counter[0]);
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

### 解説
この解答例では、ラムダ式と匿名クラスの3つの主要な違いを比較しています。

1. **thisの参照の違い**:
   - 匿名クラス内の`this`は匿名クラス自身を参照します。外部クラスのメンバーにアクセスするには、`外部クラス名.this`を使用する必要があります。
   - ラムダ式内の`this`は囲むクラス（この場合は`LambdaVsAnonymousDemo`）を参照します。外部クラスのメンバーに直接アクセスできます。

2. **変数のシャドーイングの違い**:
   - 匿名クラス内では、外部スコープの変数と同名の変数を宣言できます（シャドーイング）。
   - ラムダ式内では、外部スコープの変数と同名の変数を宣言するとコンパイルエラーになります。

3. **外部変数の参照の違い**:
   - 匿名クラスとラムダ式の両方で、外部の変数を参照できますが、その変数は実質的にfinalである必要があります。
   - 配列やコレクションを使用することで、実質的なfinalの制約を回避できます（値自体は変更できませんが、配列やコレクションの要素は変更できます）。

これらの違いを理解することで、ラムダ式と匿名クラスを適切に使い分けることができます。一般的に、単純な処理を行う場合はラムダ式の方が簡潔で可読性が高いですが、複雑な処理や状態を持つ場合は匿名クラスの方が適している場合があります。

## 発展演習1: カスタム関数型インターフェース（難易度：応用）

### 解答例
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
    // 2つの引数を受け取り、1つの結果を返す抽象メソッド
    R calculate(T t, U u);
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
    // 3つの引数を受け取り、結果を返さない抽象メソッド
    void accept(T t, U u, V v);
}

/**
 * 1つの引数を受け取り、boolean型の結果を返す関数型インターフェース
 * 
 * @param <T> 引数の型
 */
@FunctionalInterface
interface Validator<T> {
    // 1つの引数を受け取り、boolean型の結果を返す抽象メソッド
    boolean validate(T t);
}

/**
 * カスタム関数型インターフェースを使用するデモクラス
 */
public class CustomFunctionalInterfaceDemo {
    
    public static void main(String[] args) {
        // BiCalculatorの使用例
        System.out.println("===== BiCalculatorの使用例 =====");
        
        // 整数の加算を行うBiCalculator
        BiCalculator<Integer, Integer, Integer> add = (a, b) -> a + b;
        System.out.println("BiCalculator (加算): 5 + 3 = " + add.calculate(5, 3));
        
        // 文字列の連結を行うBiCalculator
        BiCalculator<String, String, String> concat = (s1, s2) -> s1 + s2;
        System.out.println("BiCalculator (連結): \"Hello, \" + \"World!\" = " + concat.calculate("Hello, ", "World!"));
        
        // 異なる型の引数と結果を扱うBiCalculator
        BiCalculator<String, Integer, String> repeat = (s, n) -> {
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < n; i++) {
                sb.append(s);
            }
            return sb.toString();
        };
        System.out.println("BiCalculator (繰り返し): \"ABC\" を 3回繰り返す = " + repeat.calculate("ABC", 3));
        
        // TriConsumerの使用例
        System.out.println("\n===== TriConsumerの使用例 =====");
        
        // 3つの引数を受け取り、それらを組み合わせて表示するTriConsumer
        TriConsumer<String, String, Integer> formatPerson = (firstName, lastName, age) -> {
            System.out.println("TriConsumer: " + lastName + " " + firstName + "さん（" + age + "歳）");
        };
        formatPerson.accept("太郎", "山田", 30);
        formatPerson.accept("花子", "佐藤", 25);
        
        // 異なる型の引数を扱うTriConsumer
        TriConsumer<String, Double, Boolean> displayProduct = (name, price, inStock) -> {
            System.out.println("TriConsumer: 商品名: " + name + ", 価格: " + price + "円, 在庫: " + (inStock ? "あり" : "なし"));
        };
        displayProduct.accept("ノートパソコン", 120000.0, true);
        displayProduct.accept("スマートフォン", 80000.0, false);
        
        // Validatorの使用例
        System.out.println("\n===== Validatorの使用例 =====");
        
        // 文字列が空でないことを検証するValidator
        Validator<String> notEmpty = s -> s != null && !s.isEmpty();
        System.out.println("Validator (空でない): \"Hello\" は空でない? " + notEmpty.validate("Hello"));
        System.out.println("Validator (空でない): \"\" は空でない? " + notEmpty.validate(""));
        System.out.println("Validator (空でない): null は空でない? " + notEmpty.validate(null));
        
        // 整数が正の値であることを検証するValidator
        Validator<Integer> isPositive = n -> n != null && n > 0;
        System.out.println("Validator (正の値): 5 は正の値? " + isPositive.validate(5));
        System.out.println("Validator (正の値): 0 は正の値? " + isPositive.validate(0));
        System.out.println("Validator (正の値): -3 は正の値? " + isPositive.validate(-3));
        
        // 複合条件を持つValidator
        Validator<String> isValidPassword = password -> 
            password != null && 
            password.length() >= 8 && 
            password.matches(".*[A-Z].*") && 
            password.matches(".*[a-z].*") && 
            password.matches(".*[0-9].*");
        
        System.out.println("Validator (有効なパスワード): \"Password123\" は有効? " + isValidPassword.validate("Password123"));
        System.out.println("Validator (有効なパスワード): \"password\" は有効? " + isValidPassword.validate("password"));
        System.out.println("Validator (有効なパスワード): \"12345678\" は有効? " + isValidPassword.validate("12345678"));
        
        // 標準関数型インターフェースとの比較
        System.out.println("\n===== 標準関数型インターフェースとの比較 =====");
        
        // BiFunction（標準関数型インターフェース）の使用例
        BiFunction<Integer, Integer, Integer> addStd = (a, b) -> a + b;
        System.out.println("BiFunction: 5 + 3 = " + addStd.apply(5, 3));
        
        // Consumer（標準関数型インターフェース）の使用例
        Consumer<String> print = s -> System.out.println("Consumer: " + s);
        print.accept("Hello, World!");
        
        // Predicate（標準関数型インターフェース）の使用例
        Predicate<Integer> isPositiveStd = n -> n > 0;
        System.out.println("Predicate: 5 is positive? " + isPositiveStd.test(5));
        System.out.println("Predicate: -3 is positive? " + isPositiveStd.test(-3));
        
        // カスタムインターフェースと標準インターフェースの使い分け
        System.out.println("\n===== カスタムインターフェースと標準インターフェースの使い分け =====");
        System.out.println("1. 標準インターフェースで十分な場合は標準インターフェースを使用する");
        System.out.println("2. 特殊な引数や戻り値の型が必要な場合はカスタムインターフェースを作成する");
        System.out.println("3. 意味のある名前を付けることでコードの可読性を向上させる場合はカスタムインターフェースを検討する");
    }
}
```

### 解説
この解答例では、3つのカスタム関数型インターフェースを作成し、それぞれをラムダ式で実装しています。

1. **BiCalculator<T, U, R>インターフェース**:
   - 2つの引数を受け取り、1つの結果を返す関数型インターフェースです。
   - `calculate(T t, U u)`メソッドを定義しています。
   - 標準の`BiFunction`インターフェースと似ていますが、メソッド名が異なります。
   - 整数の加算、文字列の連結、文字列の繰り返しなど、様々な用途に使用できます。

2. **TriConsumer<T, U, V>インターフェース**:
   - 3つの引数を受け取り、結果を返さない関数型インターフェースです。
   - `accept(T t, U u, V v)`メソッドを定義しています。
   - 標準の`Consumer`インターフェースを3つの引数に拡張したものです。
   - 人物情報の表示、商品情報の表示など、複数の引数を組み合わせて処理する場合に便利です。

3. **Validator<T>インターフェース**:
   - 1つの引数を受け取り、boolean型の結果を返す関数型インターフェースです。
   - `validate(T t)`メソッドを定義しています。
   - 標準の`Predicate`インターフェースと似ていますが、メソッド名が異なります。
   - 文字列が空でないことの検証、整数が正の値であることの検証、パスワードの複合条件の検証など、様々な検証に使用できます。

また、標準の関数型インターフェース（`BiFunction`、`Consumer`、`Predicate`）との比較も行っています。一般的に、標準インターフェースで十分な場合は標準インターフェースを使用し、特殊な引数や戻り値の型が必要な場合や、意味のある名前を付けることでコードの可読性を向上させたい場合にカスタムインターフェースを作成することが推奨されます。

## 発展演習2: メソッド参照の活用（難易度：発展）

### 解答例
```java
package lambda.advanced;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;
import java.util.stream.Collectors;

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
        
        // 商品名のリストを作成
        List<String> productNames = products.stream()
                                           .map(Product::getName)
                                           .collect(Collectors.toList());
        
        System.out.println("商品名リスト: " + productNames);
        
        // Product.toUpperCaseメソッドを参照するFunctionを作成
        Function<String, String> toUpperCaseFunction = Product::toUpperCase;
        
        // 商品名を大文字に変換
        List<String> upperCaseNames = productNames.stream()
                                                 .map(toUpperCaseFunction)
                                                 .collect(Collectors.toList());
        
        System.out.println("大文字に変換した商品名リスト: " + upperCaseNames);
        
        // 別の方法: 直接メソッド参照を使用
        List<String> upperCaseNames2 = productNames.stream()
                                                  .map(Product::toUpperCase)
                                                  .collect(Collectors.toList());
        
        System.out.println("大文字に変換した商品名リスト（直接メソッド参照）: " + upperCaseNames2);
        
        // 2. インスタンスメソッド参照を使用したリストのフィルタリング
        System.out.println("\n===== インスタンスメソッド参照を使用したリストのフィルタリング =====");
        
        // 価格のしきい値
        int priceThreshold = 50000;
        
        // ラムダ式を使用したフィルタリング
        List<Product> expensiveProducts1 = products.stream()
                                                  .filter(p -> p.isPriceGreaterThanOrEqual(priceThreshold))
                                                  .collect(Collectors.toList());
        
        System.out.println("高価な商品（ラムダ式）:");
        expensiveProducts1.forEach(System.out::println);
        
        // インスタンスメソッド参照を使用したフィルタリング（特定のインスタンスのメソッド）
        // 注: この場合、各商品に対して同じしきい値でフィルタリングするため、
        // 各商品のisPriceGreaterThanOrEqualメソッドを直接参照することはできません。
        // 代わりに、以下のようにPredicateを作成します。
        
        // 特定のインスタンスメソッドを参照するPredicateを作成
        List<Predicate<Integer>> priceCheckers = new ArrayList<>();
        for (Product product : products) {
            // 各商品のisPriceGreaterThanOrEqualメソッドを参照
            priceCheckers.add(product::isPriceGreaterThanOrEqual);
        }
        
        System.out.println("\n各商品の価格チェック（" + priceThreshold + "円以上）:");
        for (int i = 0; i < products.size(); i++) {
            System.out.println(products.get(i).getName() + ": " + 
                              priceCheckers.get(i).test(priceThreshold));
        }
        
        // 任意のオブジェクトのインスタンスメソッド参照
        // String::lengthのような形式（クラス名::インスタンスメソッド名）
        System.out.println("\n商品名の長さでソート（インスタンスメソッド参照）:");
        productNames.stream()
                   .sorted((s1, s2) -> s1.length() - s2.length())  // ラムダ式
                   .forEach(System.out::println);
        
        System.out.println("\n商品名の長さでソート（インスタンスメソッド参照、別の書き方）:");
        productNames.stream()
                   .sorted(Comparator.comparingInt(String::length))  // インスタンスメソッド参照
                   .forEach(System.out::println);
        
        // 3. コンストラクタ参照を使用したオブジェクトの生成
        System.out.println("\n===== コンストラクタ参照を使用したオブジェクトの生成 =====");
        
        // 商品名、価格、カテゴリのリスト
        List<String> names = Arrays.asList("冷蔵庫", "洗濯機", "掃除機");
        List<Integer> prices = Arrays.asList(150000, 100000, 50000);
        List<String> categories = Arrays.asList("キッチン", "家電", "家電");
        
        // ラムダ式を使用したオブジェクト生成
        List<Product> newProducts1 = new ArrayList<>();
        for (int i = 0; i < names.size(); i++) {
            newProducts1.add(new Product(names.get(i), prices.get(i), categories.get(i)));
        }
        
        System.out.println("新しい商品リスト（通常の方法）:");
        newProducts1.forEach(System.out::println);
        
        // コンストラクタ参照を使用したオブジェクト生成
        // まず、商品を生成するSupplierのリストを作成
        List<Supplier<Product>> productSuppliers = Arrays.asList(
            () -> new Product("冷蔵庫", 150000, "キッチン"),
            () -> new Product("洗濯機", 100000, "家電"),
            () -> new Product("掃除機", 50000, "家電")
        );
        
        // Supplierを使用して商品を生成
        List<Product> newProducts2 = productSuppliers.stream()
                                                    .map(Supplier::get)
                                                    .collect(Collectors.toList());
        
        System.out.println("\n新しい商品リスト（Supplier使用）:");
        newProducts2.forEach(System.out::println);
        
        // コンストラクタ参照を使用した別の例
        // 引数なしのコンストラクタを持つクラスの場合
        class SimpleProduct {
            private String name;
            
            public SimpleProduct() {
                this.name = "未設定";
            }
            
            public void setName(String name) {
                this.name = name;
            }
            
            @Override
            public String toString() {
                return "SimpleProduct [name=" + name + "]";
            }
        }
        
        // コンストラクタ参照を使用してSupplierを作成
        Supplier<SimpleProduct> simpleProductSupplier = SimpleProduct::new;
        
        // Supplierを使用してオブジェクトを生成
        SimpleProduct simpleProduct = simpleProductSupplier.get();
        simpleProduct.setName("シンプル商品");
        
        System.out.println("\nシンプル商品（コンストラクタ参照）: " + simpleProduct);
        
        // メソッド参照の種類のまとめ
        System.out.println("\n===== メソッド参照の種類のまとめ =====");
        System.out.println("1. 静的メソッド参照: クラス名::静的メソッド名");
        System.out.println("   例: Product::toUpperCase");
        System.out.println("2. 特定のインスタンスのメソッド参照: インスタンス::メソッド名");
        System.out.println("   例: product::isPriceGreaterThanOrEqual");
        System.out.println("3. 任意のオブジェクトのインスタンスメソッド参照: クラス名::インスタンスメソッド名");
        System.out.println("   例: String::length");
        System.out.println("4. コンストラクタ参照: クラス名::new");
        System.out.println("   例: SimpleProduct::new");
    }
}
```

### 解説
この解答例では、メソッド参照の4つの主要な形式を活用しています。

1. **静的メソッド参照（クラス名::静的メソッド名）**:
   - `Product::toUpperCase`のように、クラスの静的メソッドを参照します。
   - 商品名のリストを大文字に変換する例を示しています。
   - `Function<String, String> toUpperCaseFunction = Product::toUpperCase;`のように変数に格納することも、
   - `productNames.stream().map(Product::toUpperCase)`のように直接使用することもできます。

2. **特定のインスタンスのメソッド参照（インスタンス::メソッド名）**:
   - `product::isPriceGreaterThanOrEqual`のように、特定のインスタンスのメソッドを参照します。
   - 各商品の価格が閾値以上かどうかを判定する例を示しています。
   - `priceCheckers.add(product::isPriceGreaterThanOrEqual);`のように、各インスタンスのメソッドを参照するPredicateをリストに追加しています。

3. **任意のオブジェクトのインスタンスメソッド参照（クラス名::インスタンスメソッド名）**:
   - `String::length`のように、クラスのインスタンスメソッドを参照します。
   - 商品名の長さでソートする例を示しています。
   - `productNames.stream().sorted(Comparator.comparingInt(String::length))`のように使用します。

4. **コンストラクタ参照（クラス名::new）**:
   - `SimpleProduct::new`のように、クラスのコンストラクタを参照します。
   - 引数なしのコンストラクタを持つクラスのインスタンスを生成する例を示しています。
   - `Supplier<SimpleProduct> simpleProductSupplier = SimpleProduct::new;`のように使用します。

メソッド参照を使用することで、ラムダ式よりもさらに簡潔で読みやすいコードを書くことができます。特に、既存のメソッドをそのまま使用する場合や、単純なオブジェクト生成を行う場合に有効です。ただし、複雑なロジックや複数の処理を行う場合は、ラムダ式の方が適している場合もあります。

メソッド参照の種類を適切に使い分けることで、コードの可読性と保守性を向上させることができます。
