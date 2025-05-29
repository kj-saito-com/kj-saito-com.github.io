# Day 3: ラムダ式と匿名クラスの適切な使い分け

## 学習の目的と背景

Java 8から導入されたラムダ式は、コードをより簡潔に、読みやすく、そして関数型プログラミングの考え方を取り入れた形で書くことを可能にしました。一方、匿名クラスは古くからJavaに存在する機能で、インターフェースやクラスをその場で実装するための手段として使われてきました。

これらの機能は、コードの簡潔さと可読性を高めるだけでなく、コレクション操作やイベント処理など、多くの場面で活用されています。特に業務アプリケーションでは、データの変換、フィルタリング、集計などの処理を簡潔に記述するために、ラムダ式が頻繁に使用されます。

本日の学習では、匿名クラスの基本からラムダ式の活用まで、段階的に理解を深めていきます。また、それぞれの特性を理解し、適切な場面で使い分けられるようになることを目指します。

## 前提知識と準備

### 必要な前提知識
- Javaの基本構文（変数、制御構文、クラス、オブジェクト）
- インターフェースの概念と使い方
- Day 2で学習したコレクションフレームワークの基本

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

### 1. インターフェースと実装

Javaでは、インターフェースを使って抽象的な振る舞いを定義し、それを実装するクラスで具体的な処理を記述します。

```java
// インターフェース定義
interface Greeting {
    void sayHello(String name);
}

// 実装クラス
class JapaneseGreeting implements Greeting {
    @Override
    public void sayHello(String name) {
        System.out.println("こんにちは、" + name + "さん！");
    }
}

// 使用例
Greeting greeting = new JapaneseGreeting();
greeting.sayHello("田中"); // 出力: こんにちは、田中さん！
```

### 2. 匿名クラス

匿名クラスは、クラスの宣言と同時にインスタンス化を行う方法です。一度だけ使用するクラスを簡潔に記述できます。

<div class="term-box">
<strong>用語解説: 匿名クラス（Anonymous Class）</strong><br>
名前を持たないクラスで、インターフェースやクラスをその場で実装し、同時にインスタンス化します。一度だけ使用する実装を簡潔に記述するのに適しています。
</div>

```java
// 匿名クラスの例
Greeting englishGreeting = new Greeting() {
    @Override
    public void sayHello(String name) {
        System.out.println("Hello, " + name + "!");
    }
};

englishGreeting.sayHello("John"); // 出力: Hello, John!
```

匿名クラスの特徴：
- クラス名がない（`new インターフェース名() { ... }`の形式）
- その場で実装とインスタンス化を同時に行う
- 一度だけ使用する実装に適している
- 外部スコープの変数にアクセスできるが、その変数は実質的にfinalである必要がある

### 3. 関数型インターフェース

Java 8から導入された関数型インターフェースは、**抽象メソッドを1つだけ持つ**インターフェースです。ラムダ式で実装できるようになっています。

<div class="term-box">
<strong>用語解説: 関数型インターフェース（Functional Interface）</strong><br>
抽象メソッドを1つだけ持つインターフェースです。@FunctionalInterfaceアノテーションで明示的に示すことができます。ラムダ式やメソッド参照で実装できます。
</div>

```java
// 関数型インターフェースの例
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

// 匿名クラスでの実装
Calculator adder = new Calculator() {
    @Override
    public int calculate(int a, int b) {
        return a + b;
    }
};

System.out.println(adder.calculate(5, 3)); // 出力: 8
```

Java標準ライブラリには、よく使われる関数型インターフェースが`java.util.function`パッケージに用意されています：

- `Predicate<T>`: 引数を1つ取り、booleanを返す（例：フィルタリング）
- `Consumer<T>`: 引数を1つ取り、何も返さない（例：出力）
- `Function<T, R>`: 引数を1つ取り、結果を返す（例：変換）
- `Supplier<T>`: 引数を取らず、結果を返す（例：値の生成）
- `BiFunction<T, U, R>`: 引数を2つ取り、結果を返す

### 4. ラムダ式

ラムダ式は、関数型インターフェースのインスタンスを簡潔に作成するための構文です。

<div class="term-box">
<strong>用語解説: ラムダ式（Lambda Expression）</strong><br>
関数型インターフェースを実装するための簡潔な構文です。匿名関数とも呼ばれ、`(引数) -> { 処理 }`の形式で記述します。単一の式の場合は`{ }`を省略できます。
</div>

```java
// ラムダ式の例
Calculator adder = (a, b) -> a + b;
System.out.println(adder.calculate(5, 3)); // 出力: 8

// 複数行の処理を含むラムダ式
Calculator multiplier = (a, b) -> {
    System.out.println("Multiplying " + a + " and " + b);
    return a * b;
};
System.out.println(multiplier.calculate(5, 3)); // 出力: Multiplying 5 and 3 \n 15
```

ラムダ式の特徴：
- 簡潔な構文（`(引数) -> 処理`）
- 型推論により引数の型を省略可能
- 単一の式の場合は`return`キーワードと波括弧を省略可能
- 外部スコープの変数にアクセスできるが、その変数は実質的にfinalである必要がある

### 5. メソッド参照

メソッド参照は、既存のメソッドをラムダ式の代わりに使用する構文です。さらに簡潔にコードを記述できます。

<div class="term-box">
<strong>用語解説: メソッド参照（Method Reference）</strong><br>
既存のメソッドをラムダ式の代わりに使用するための簡潔な構文です。`クラス名::メソッド名`または`インスタンス::メソッド名`の形式で記述します。
</div>

メソッド参照の種類：
1. 静的メソッド参照: `クラス名::静的メソッド名`
2. インスタンスメソッド参照: `インスタンス::メソッド名`
3. 特定型のインスタンスメソッド参照: `クラス名::インスタンスメソッド名`
4. コンストラクタ参照: `クラス名::new`

```java
// 静的メソッド参照
Function<String, Integer> parseInt = Integer::parseInt;
System.out.println(parseInt.apply("123")); // 出力: 123

// インスタンスメソッド参照
String str = "Hello";
Supplier<Integer> length = str::length;
System.out.println(length.get()); // 出力: 5

// 特定型のインスタンスメソッド参照
Function<String, String> toUpperCase = String::toUpperCase;
System.out.println(toUpperCase.apply("hello")); // 出力: HELLO

// コンストラクタ参照
Supplier<ArrayList<String>> listCreator = ArrayList::new;
ArrayList<String> list = listCreator.get(); // 新しいArrayListを作成
```

### 6. ラムダ式と匿名クラスの違い

ラムダ式と匿名クラスは似ていますが、いくつかの重要な違いがあります：

1. **構文の簡潔さ**：ラムダ式は匿名クラスよりも簡潔です。
2. **スコープ**：匿名クラスは独自のスコープを持ちますが、ラムダ式は囲むスコープと同じです。
   - 匿名クラス内の`this`はそのクラス自身を参照します。
   - ラムダ式内の`this`は囲むクラスを参照します。
3. **シャドーイング**：匿名クラスでは外部スコープの変数をシャドーイングできますが、ラムダ式ではできません。
4. **機能的制限**：ラムダ式は関数型インターフェース（抽象メソッドが1つだけ）にのみ使用できます。
5. **実装方法**：内部的な実装が異なります（ラムダ式はより効率的）。

```java
// スコープの違い
class ScopeExample {
    private String field = "Class Field";
    
    public void demonstrate() {
        String localVar = "Local Variable";
        
        // 匿名クラスの場合
        Runnable anonymousClass = new Runnable() {
            private String field = "Anonymous Class Field";
            
            @Override
            public void run() {
                System.out.println(this.field); // "Anonymous Class Field"
                System.out.println(ScopeExample.this.field); // "Class Field"
                System.out.println(localVar); // "Local Variable"
            }
        };
        
        // ラムダ式の場合
        Runnable lambda = () -> {
            // private String field = "Lambda Field"; // コンパイルエラー
            System.out.println(this.field); // "Class Field"
            System.out.println(localVar); // "Local Variable"
        };
        
        anonymousClass.run();
        lambda.run();
    }
}
```

### 7. 使い分けの指針

ラムダ式と匿名クラスの使い分けについて、以下の指針が役立ちます：

1. **関数型インターフェースの実装**：
   - 単純な処理の場合は**ラムダ式**を使用（簡潔で読みやすい）
   - 既存のメソッドを使用する場合は**メソッド参照**を検討

2. **複数のメソッドを持つインターフェースの実装**：
   - **匿名クラス**を使用（ラムダ式は使用不可）

3. **状態を持つ必要がある場合**：
   - **匿名クラス**を使用（フィールドを定義できる）

4. **this参照が必要な場合**：
   - 囲むクラスのthisが必要なら**ラムダ式**
   - 実装自体のthisが必要なら**匿名クラス**

5. **例外処理**：
   - チェック例外をスローする必要がある場合、対応する関数型インターフェースがない場合は**匿名クラス**を検討

## Eclipse操作ガイド

### ラムダ式の入力支援

Eclipseはラムダ式の入力をサポートしています：

1. 関数型インターフェースの変数を宣言し、初期化部分で`Ctrl + Space`を押すと、ラムダ式のテンプレートが提案されます。
2. ラムダ式の矢印演算子（`->`）は、`->`と入力するか、`Alt + -`（ハイフン）を押すことで入力できます。

### ラムダ式のデバッグ

ラムダ式のデバッグは通常のコードと同様に行えますが、いくつかの注意点があります：

1. ブレークポイントをラムダ式の中に設定できます。
2. 変数ビューでは、ラムダ式がキャプチャした変数を確認できます。
3. ステップ実行（Step Into, Step Over, Step Return）も通常通り機能します。

![Eclipseラムダデバッグ](https://example.com/eclipse_lambda_debug.png)

### 匿名クラスからラムダ式への変換

Eclipseには、適格な匿名クラスをラムダ式に変換する機能があります：

1. 匿名クラスのコードを選択します。
2. 右クリックして「Refactor」→「Convert Anonymous to Lambda Expression」を選択します。

## コア演習（必須）

### 演習1: 匿名クラスの基本（難易度：基本）

#### 課題
以下のインターフェースを使用して、匿名クラスで実装してください：
1. `Comparator<String>`を使用して、文字列を長さの昇順でソートする匿名クラスを作成
2. `Runnable`を使用して、「Hello from Anonymous Class!」と出力する匿名クラスを作成
3. 独自の関数型インターフェース`StringProcessor`を定義し、文字列を大文字に変換する匿名クラスを作成

#### 雛形コード
```java
package lambda.basic;

import java.util.*;

/**
 * 匿名クラスの基本を練習するクラス
 */
public class AnonymousClassDemo {

    // 独自の関数型インターフェース
    @FunctionalInterface
    interface StringProcessor {
        String process(String input);
    }

    /**
     * Comparator<String>を使用して、文字列を長さの昇順でソートする匿名クラスを作成します。
     */
    public void demonstrateComparator() {
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        // TODO: Comparator<String>の匿名クラスを作成し、文字列の長さで昇順ソート
        Comparator<String> lengthComparator = null; // ここを実装
        
        Collections.sort(words, lengthComparator);
        System.out.println("Sorted by length: " + words);
    }

    /**
     * Runnableを使用して、メッセージを出力する匿名クラスを作成します。
     */
    public void demonstrateRunnable() {
        // TODO: Runnableの匿名クラスを作成し、"Hello from Anonymous Class!"と出力
        Runnable helloRunnable = null; // ここを実装
        
        Thread thread = new Thread(helloRunnable);
        thread.start();
    }

    /**
     * StringProcessorを使用して、文字列を大文字に変換する匿名クラスを作成します。
     */
    public void demonstrateStringProcessor() {
        // TODO: StringProcessorの匿名クラスを作成し、文字列を大文字に変換
        StringProcessor upperCaseProcessor = null; // ここを実装
        
        String result = upperCaseProcessor.process("Hello, World!");
        System.out.println("Processed: " + result);
    }

    public static void main(String[] args) {
        AnonymousClassDemo demo = new AnonymousClassDemo();
        demo.demonstrateComparator();
        demo.demonstrateRunnable();
        demo.demonstrateStringProcessor();
    }
}
```

#### 実装ヒント
- `Comparator<String>`の匿名クラスでは、`compare`メソッドをオーバーライドします。文字列の長さを比較するには`String.length()`を使用します。
- `Runnable`の匿名クラスでは、`run`メソッドをオーバーライドします。
- `StringProcessor`の匿名クラスでは、`process`メソッドをオーバーライドします。文字列を大文字に変換するには`String.toUpperCase()`を使用します。

### 演習2: ラムダ式の基本（難易度：基本）

#### 課題
演習1で作成した匿名クラスをラムダ式に書き換えてください。また、Java標準の関数型インターフェース（`Predicate`, `Consumer`, `Function`, `Supplier`）を使用した例も作成してください。

#### 雛形コード
```java
package lambda.basic;

import java.util.*;
import java.util.function.*;

/**
 * ラムダ式の基本を練習するクラス
 */
public class LambdaExpressionDemo {

    // 独自の関数型インターフェース
    @FunctionalInterface
    interface StringProcessor {
        String process(String input);
    }

    /**
     * Comparator<String>をラムダ式で実装します。
     */
    public void demonstrateComparatorLambda() {
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        // TODO: Comparator<String>をラムダ式で実装し、文字列の長さで昇順ソート
        Comparator<String> lengthComparator = null; // ここを実装
        
        Collections.sort(words, lengthComparator);
        System.out.println("Sorted by length (lambda): " + words);
    }

    /**
     * Runnableをラムダ式で実装します。
     */
    public void demonstrateRunnableLambda() {
        // TODO: Runnableをラムダ式で実装し、"Hello from Lambda!"と出力
        Runnable helloRunnable = null; // ここを実装
        
        Thread thread = new Thread(helloRunnable);
        thread.start();
    }

    /**
     * StringProcessorをラムダ式で実装します。
     */
    public void demonstrateStringProcessorLambda() {
        // TODO: StringProcessorをラムダ式で実装し、文字列を大文字に変換
        StringProcessor upperCaseProcessor = null; // ここを実装
        
        String result = upperCaseProcessor.process("Hello, World!");
        System.out.println("Processed (lambda): " + result);
    }

    /**
     * 標準関数型インターフェースを使用します。
     */
    public void demonstrateStandardFunctionalInterfaces() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave", "Eve");
        
        // TODO: Predicateを使用して、長さが4文字以上の名前をフィルタリング
        Predicate<String> lengthFilter = null; // ここを実装
        
        // TODO: Consumerを使用して、名前を出力
        Consumer<String> printer = null; // ここを実装
        
        // TODO: Functionを使用して、名前を小文字に変換
        Function<String, String> toLowerCase = null; // ここを実装
        
        // TODO: Supplierを使用して、現在時刻を文字列で返す
        Supplier<String> currentTime = null; // ここを実装
        
        System.out.println("Names with length >= 4:");
        for (String name : names) {
            if (lengthFilter.test(name)) {
                printer.accept(toLowerCase.apply(name));
            }
        }
        
        System.out.println("Current time: " + currentTime.get());
    }

    public static void main(String[] args) {
        LambdaExpressionDemo demo = new LambdaExpressionDemo();
        demo.demonstrateComparatorLambda();
        demo.demonstrateRunnableLambda();
        demo.demonstrateStringProcessorLambda();
        demo.demonstrateStandardFunctionalInterfaces();
    }
}
```

#### 実装ヒント
- ラムダ式の基本構文は `(引数) -> { 処理 }` です。
- 単一の式の場合は、波括弧と`return`キーワードを省略できます。例：`(a, b) -> a + b`
- `Predicate<T>`は`boolean test(T t)`メソッドを持ち、条件判定に使用します。
- `Consumer<T>`は`void accept(T t)`メソッドを持ち、値を消費（使用）するだけで何も返しません。
- `Function<T, R>`は`R apply(T t)`メソッドを持ち、入力を変換して出力します。
- `Supplier<T>`は`T get()`メソッドを持ち、値を供給（生成）します。

### 演習3: コレクションとラムダ式（難易度：応用）

#### 課題
コレクションフレームワークとラムダ式を組み合わせて、以下の操作を行うプログラムを作成してください：
1. 整数のリストから偶数のみをフィルタリングし、それらを2倍にして、結果を出力する
2. 文字列のリストから特定の条件（長さが5以上など）を満たす要素だけを抽出し、大文字に変換して、アルファベット順にソートする
3. `Person`オブジェクト（フィールド: `name`, `age`）のリストから、年齢が特定の値以上の人だけを抽出し、名前のリストを作成する

#### 雛形コード
```java
package lambda.collections;

import java.util.*;
import java.util.function.*;
import java.util.stream.*;

/**
 * コレクションとラムダ式を組み合わせて使用するクラス
 */
public class CollectionLambdaDemo {

    /**
     * 整数のリストから偶数のみをフィルタリングし、それらを2倍にして、結果を出力します。
     */
    public void processNumbers() {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // TODO: 偶数のみをフィルタリングし、それらを2倍にして、結果を出力
        // ヒント: filter()とmap()メソッドを使用
        
        System.out.println("Original numbers: " + numbers);
        System.out.println("Processed numbers: "); // 結果を出力
    }

    /**
     * 文字列のリストから特定の条件を満たす要素だけを抽出し、変換して、ソートします。
     */
    public void processStrings() {
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry", "fig", "grape");
        
        // TODO: 長さが5以上の単語だけを抽出し、大文字に変換して、アルファベット順にソート
        // ヒント: filter(), map(), sorted()メソッドを使用
        
        System.out.println("Original words: " + words);
        System.out.println("Processed words: "); // 結果を出力
    }

    /**
     * Personオブジェクトのリストから、条件を満たす要素だけを抽出し、変換します。
     */
    public void processPersons() {
        List<Person> persons = Arrays.asList(
            new Person("Alice", 25),
            new Person("Bob", 30),
            new Person("Charlie", 35),
            new Person("Dave", 40),
            new Person("Eve", 45)
        );
        
        // TODO: 年齢が30以上の人だけを抽出し、名前のリストを作成
        // ヒント: filter()とmap()メソッドを使用し、collect()でリストに変換
        
        System.out.println("All persons: " + persons);
        System.out.println("Names of persons aged 30 or older: "); // 結果を出力
    }

    // Personクラス
    static class Person {
        private String name;
        private int age;
        
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public String getName() { return name; }
        public int getAge() { return age; }
        
        @Override
        public String toString() {
            return name + " (" + age + ")";
        }
    }

    public static void main(String[] args) {
        CollectionLambdaDemo demo = new CollectionLambdaDemo();
        demo.processNumbers();
        demo.processStrings();
        demo.processPersons();
    }
}
```

#### 実装ヒント
- Java 8のStream APIを使用すると、コレクションの処理が簡潔に書けます。
- `stream()`メソッドでコレクションをストリームに変換します。
- `filter(Predicate)`メソッドで条件に合う要素だけを抽出します。
- `map(Function)`メソッドで各要素を変換します。
- `sorted()`または`sorted(Comparator)`メソッドで要素をソートします。
- `collect(Collectors.toList())`でストリームをリストに変換します。
- `forEach(Consumer)`メソッドで各要素に対して処理を実行します。

例：
```java
List<Integer> evenDoubled = numbers.stream()
    .filter(n -> n % 2 == 0)    // 偶数のみをフィルタリング
    .map(n -> n * 2)            // 各要素を2倍に
    .collect(Collectors.toList()); // リストに変換
```

## 発展演習（オプション）

### 発展演習1: メソッド参照の活用（難易度：発展）

#### 課題
ラムダ式の代わりにメソッド参照を使用して、以下の操作を行うプログラムを作成してください：
1. 文字列のリストを長さでソートする（静的メソッド参照）
2. 整数のリストの各要素を文字列に変換する（静的メソッド参照）
3. 文字列のリストの各要素を大文字に変換する（インスタンスメソッド参照）
4. `Person`オブジェクトのリストを名前でソートする（インスタンスメソッド参照）
5. 文字列のリストから新しい`Person`オブジェクトのリストを作成する（コンストラクタ参照）

#### 雛形コード
```java
package lambda.advanced;

import java.util.*;
import java.util.function.*;
import java.util.stream.*;

/**
 * メソッド参照を活用するクラス
 */
public class MethodReferenceDemo {

    /**
     * 静的メソッド参照を使用して、文字列のリストを長さでソートします。
     */
    public void sortStringsByLength() {
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        // TODO: 静的メソッド参照を使用して、文字列を長さでソート
        // ヒント: Comparator.comparingInt()とString::lengthを使用
        
        System.out.println("Sorted by length: " + words);
    }

    /**
     * 静的メソッド参照を使用して、整数のリストの各要素を文字列に変換します。
     */
    public void convertIntegersToStrings() {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        
        // TODO: 静的メソッド参照を使用して、整数を文字列に変換
        // ヒント: Integer::toStringを使用
        
        System.out.println("Converted to strings: "); // 結果を出力
    }

    /**
     * インスタンスメソッド参照を使用して、文字列のリストの各要素を大文字に変換します。
     */
    public void convertStringsToUpperCase() {
        List<String> words = Arrays.asList("apple", "banana", "cherry");
        
        // TODO: インスタンスメソッド参照を使用して、文字列を大文字に変換
        // ヒント: String::toUpperCaseを使用
        
        System.out.println("Converted to upper case: "); // 結果を出力
    }

    /**
     * インスタンスメソッド参照を使用して、Personオブジェクトのリストを名前でソートします。
     */
    public void sortPersonsByName() {
        List<Person> persons = Arrays.asList(
            new Person("Charlie", 35),
            new Person("Alice", 25),
            new Person("Bob", 30)
        );
        
        // TODO: インスタンスメソッド参照を使用して、Personを名前でソート
        // ヒント: Comparator.comparing()とPerson::getNameを使用
        
        System.out.println("Sorted by name: " + persons);
    }

    /**
     * コンストラクタ参照を使用して、文字列のリストから新しいPersonオブジェクトのリストを作成します。
     */
    public void createPersonsFromStrings() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        // TODO: コンストラクタ参照を使用して、文字列からPersonオブジェクトを作成
        // ヒント: Person::newを使用（Personに適切なコンストラクタが必要）
        
        System.out.println("Created persons: "); // 結果を出力
    }

    // Personクラス
    static class Person {
        private String name;
        private int age;
        
        public Person(String name) {
            this(name, 0); // 年齢不明の場合は0とする
        }
        
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public String getName() { return name; }
        public int getAge() { return age; }
        
        @Override
        public String toString() {
            return name + " (" + age + ")";
        }
    }

    public static void main(String[] args) {
        MethodReferenceDemo demo = new MethodReferenceDemo();
        demo.sortStringsByLength();
        demo.convertIntegersToStrings();
        demo.convertStringsToUpperCase();
        demo.sortPersonsByName();
        demo.createPersonsFromStrings();
    }
}
```

#### 実装ヒント
- 静的メソッド参照: `クラス名::静的メソッド名`
  例: `Integer::parseInt`
- インスタンスメソッド参照: `インスタンス::メソッド名` または `クラス名::インスタンスメソッド名`
  例: `String::toUpperCase`
- コンストラクタ参照: `クラス名::new`
  例: `ArrayList::new`
- `Comparator.comparing()`や`Comparator.comparingInt()`などのユーティリティメソッドを使うと、メソッド参照を使ったソートが簡単に書けます。

### 発展演習2: カスタム関数型インターフェースの設計（難易度：発展）

#### 課題
以下の要件を満たすカスタム関数型インターフェースを設計し、それを使用するプログラムを作成してください：
1. 3つの引数を取り、1つの結果を返す関数型インターフェース`TriFunction<T, U, V, R>`
2. 2つの引数を取り、例外をスローする可能性のある関数型インターフェース`ThrowingBiFunction<T, U, R, E extends Exception>`
3. 上記のインターフェースを使用して、実際の処理を実装する

#### 雛形コード
```java
package lambda.advanced;

import java.io.*;
import java.util.*;
import java.util.function.*;

/**
 * カスタム関数型インターフェースを設計・使用するクラス
 */
public class CustomFunctionalInterfaceDemo {

    /**
     * 3つの引数を取り、1つの結果を返す関数型インターフェース
     */
    @FunctionalInterface
    interface TriFunction<T, U, V, R> {
        // TODO: 適切な抽象メソッドを定義
    }

    /**
     * 2つの引数を取り、例外をスローする可能性のある関数型インターフェース
     */
    @FunctionalInterface
    interface ThrowingBiFunction<T, U, R, E extends Exception> {
        // TODO: 適切な抽象メソッドを定義
        
        /**
         * 例外をキャッチして実行時例外に変換するデフォルトメソッド
         */
        default BiFunction<T, U, R> unchecked() {
            return (t, u) -> {
                try {
                    return apply(t, u);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            };
        }
    }

    /**
     * TriFunctionを使用して、3つの数値の加重平均を計算します。
     */
    public void demonstrateTriFunction() {
        // TODO: TriFunctionを使用して、3つの数値の加重平均を計算
        // 加重平均 = (value1 * weight1 + value2 * weight2 + value3 * weight3) / (weight1 + weight2 + weight3)
        
        double result = 0.0; // 計算結果
        System.out.println("Weighted average: " + result);
    }

    /**
     * ThrowingBiFunctionを使用して、ファイルの特定の行を読み込みます。
     */
    public void demonstrateThrowingBiFunction() {
        // TODO: ThrowingBiFunctionを使用して、ファイルの特定の行を読み込む
        // 引数1: ファイルパス、引数2: 行番号、戻り値: その行の内容
        
        try {
            // テスト用のファイルを作成
            createTestFile("test.txt", Arrays.asList("Line 1", "Line 2", "Line 3", "Line 4", "Line 5"));
            
            // ThrowingBiFunctionを使用してファイルの行を読み込む
            
            System.out.println("Line 3 content: "); // 結果を出力
        } catch (Exception e) {
            System.err.println("Error: " + e.getMessage());
        }
    }

    /**
     * テスト用のファイルを作成します。
     */
    private void createTestFile(String filename, List<String> lines) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            for (String line : lines) {
                writer.write(line);
                writer.newLine();
            }
        }
    }

    public static void main(String[] args) {
        CustomFunctionalInterfaceDemo demo = new CustomFunctionalInterfaceDemo();
        demo.demonstrateTriFunction();
        demo.demonstrateThrowingBiFunction();
    }
}
```

#### 実装ヒント
- `TriFunction<T, U, V, R>`の抽象メソッドは、3つの引数を取り、1つの結果を返すメソッドです。例：`R apply(T t, U u, V v);`
- `ThrowingBiFunction<T, U, R, E extends Exception>`の抽象メソッドは、2つの引数を取り、1つの結果を返し、例外をスローする可能性があるメソッドです。例：`R apply(T t, U u) throws E;`
- `unchecked()`メソッドは、チェック例外を実行時例外に変換するためのユーティリティメソッドです。これにより、例外をスローするメソッドを標準の関数型インターフェースで使用できるようになります。
- ファイルの特定の行を読み込むには、`BufferedReader`と`FileReader`を使用します。指定された行番号まで読み進め、その行の内容を返します。

## 解答例

### 演習1: 匿名クラスの基本の解答例

```java
package lambda.basic;

import java.util.*;

/**
 * 匿名クラスの基本を練習するクラス
 */
public class AnonymousClassDemo {

    // 独自の関数型インターフェース
    @FunctionalInterface
    interface StringProcessor {
        String process(String input);
    }

    /**
     * Comparator<String>を使用して、文字列を長さの昇順でソートする匿名クラスを作成します。
     */
    public void demonstrateComparator() {
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        // Comparator<String>の匿名クラスを作成し、文字列の長さで昇順ソート
        Comparator<String> lengthComparator = new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return Integer.compare(s1.length(), s2.length());
            }
        };
        
        Collections.sort(words, lengthComparator);
        System.out.println("Sorted by length: " + words);
    }

    /**
     * Runnableを使用して、メッセージを出力する匿名クラスを作成します。
     */
    public void demonstrateRunnable() {
        // Runnableの匿名クラスを作成し、"Hello from Anonymous Class!"と出力
        Runnable helloRunnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello from Anonymous Class!");
            }
        };
        
        Thread thread = new Thread(helloRunnable);
        thread.start();
    }

    /**
     * StringProcessorを使用して、文字列を大文字に変換する匿名クラスを作成します。
     */
    public void demonstrateStringProcessor() {
        // StringProcessorの匿名クラスを作成し、文字列を大文字に変換
        StringProcessor upperCaseProcessor = new StringProcessor() {
            @Override
            public String process(String input) {
                return input.toUpperCase();
            }
        };
        
        String result = upperCaseProcessor.process("Hello, World!");
        System.out.println("Processed: " + result);
    }

    public static void main(String[] args) {
        AnonymousClassDemo demo = new AnonymousClassDemo();
        demo.demonstrateComparator();
        demo.demonstrateRunnable();
        demo.demonstrateStringProcessor();
    }
}
```

### 演習2: ラムダ式の基本の解答例

```java
package lambda.basic;

import java.util.*;
import java.util.function.*;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;

/**
 * ラムダ式の基本を練習するクラス
 */
public class LambdaExpressionDemo {

    // 独自の関数型インターフェース
    @FunctionalInterface
    interface StringProcessor {
        String process(String input);
    }

    /**
     * Comparator<String>をラムダ式で実装します。
     */
    public void demonstrateComparatorLambda() {
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        // Comparator<String>をラムダ式で実装し、文字列の長さで昇順ソート
        Comparator<String> lengthComparator = (s1, s2) -> Integer.compare(s1.length(), s2.length());
        
        Collections.sort(words, lengthComparator);
        System.out.println("Sorted by length (lambda): " + words);
    }

    /**
     * Runnableをラムダ式で実装します。
     */
    public void demonstrateRunnableLambda() {
        // Runnableをラムダ式で実装し、"Hello from Lambda!"と出力
        Runnable helloRunnable = () -> System.out.println("Hello from Lambda!");
        
        Thread thread = new Thread(helloRunnable);
        thread.start();
    }

    /**
     * StringProcessorをラムダ式で実装します。
     */
    public void demonstrateStringProcessorLambda() {
        // StringProcessorをラムダ式で実装し、文字列を大文字に変換
        StringProcessor upperCaseProcessor = input -> input.toUpperCase();
        
        String result = upperCaseProcessor.process("Hello, World!");
        System.out.println("Processed (lambda): " + result);
    }

    /**
     * 標準関数型インターフェースを使用します。
     */
    public void demonstrateStandardFunctionalInterfaces() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave", "Eve");
        
        // Predicateを使用して、長さが4文字以上の名前をフィルタリング
        Predicate<String> lengthFilter = name -> name.length() >= 4;
        
        // Consumerを使用して、名前を出力
        Consumer<String> printer = name -> System.out.println("Name: " + name);
        
        // Functionを使用して、名前を小文字に変換
        Function<String, String> toLowerCase = name -> name.toLowerCase();
        
        // Supplierを使用して、現在時刻を文字列で返す
        Supplier<String> currentTime = () -> LocalTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss"));
        
        System.out.println("Names with length >= 4:");
        for (String name : names) {
            if (lengthFilter.test(name)) {
                printer.accept(toLowerCase.apply(name));
            }
        }
        
        System.out.println("Current time: " + currentTime.get());
    }

    public static void main(String[] args) {
        LambdaExpressionDemo demo = new LambdaExpressionDemo();
        demo.demonstrateComparatorLambda();
        demo.demonstrateRunnableLambda();
        demo.demonstrateStringProcessorLambda();
        demo.demonstrateStandardFunctionalInterfaces();
    }
}
```

### 演習3: コレクションとラムダ式の解答例

```java
package lambda.collections;

import java.util.*;
import java.util.function.*;
import java.util.stream.*;

/**
 * コレクションとラムダ式を組み合わせて使用するクラス
 */
public class CollectionLambdaDemo {

    /**
     * 整数のリストから偶数のみをフィルタリングし、それらを2倍にして、結果を出力します。
     */
    public void processNumbers() {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // 偶数のみをフィルタリングし、それらを2倍にして、結果を出力
        List<Integer> evenDoubled = numbers.stream()
                .filter(n -> n % 2 == 0)    // 偶数のみをフィルタリング
                .map(n -> n * 2)            // 各要素を2倍に
                .collect(Collectors.toList()); // リストに変換
        
        System.out.println("Original numbers: " + numbers);
        System.out.println("Processed numbers: " + evenDoubled);
    }

    /**
     * 文字列のリストから特定の条件を満たす要素だけを抽出し、変換して、ソートします。
     */
    public void processStrings() {
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry", "fig", "grape");
        
        // 長さが5以上の単語だけを抽出し、大文字に変換して、アルファベット順にソート
        List<String> processedWords = words.stream()
                .filter(word -> word.length() >= 5)  // 長さが5以上の単語をフィルタリング
                .map(String::toUpperCase)            // 大文字に変換
                .sorted()                            // アルファベット順にソート
                .collect(Collectors.toList());       // リストに変換
        
        System.out.println("Original words: " + words);
        System.out.println("Processed words: " + processedWords);
    }

    /**
     * Personオブジェクトのリストから、条件を満たす要素だけを抽出し、変換します。
     */
    public void processPersons() {
        List<Person> persons = Arrays.asList(
            new Person("Alice", 25),
            new Person("Bob", 30),
            new Person("Charlie", 35),
            new Person("Dave", 40),
            new Person("Eve", 45)
        );
        
        // 年齢が30以上の人だけを抽出し、名前のリストを作成
        List<String> names = persons.stream()
                .filter(person -> person.getAge() >= 30)  // 年齢が30以上の人をフィルタリング
                .map(Person::getName)                     // 名前を抽出
                .collect(Collectors.toList());            // リストに変換
        
        System.out.println("All persons: " + persons);
        System.out.println("Names of persons aged 30 or older: " + names);
    }

    // Personクラス
    static class Person {
        private String name;
        private int age;
        
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public String getName() { return name; }
        public int getAge() { return age; }
        
        @Override
        public String toString() {
            return name + " (" + age + ")";
        }
    }

    public static void main(String[] args) {
        CollectionLambdaDemo demo = new CollectionLambdaDemo();
        demo.processNumbers();
        demo.processStrings();
        demo.processPersons();
    }
}
```

## 解説と補足

### ラムダ式と匿名クラスの内部実装の違い

ラムダ式と匿名クラスは、見た目は似ていますが、内部的な実装は大きく異なります：

1. **匿名クラス**：
   - コンパイル時に新しいクラスファイル（`.class`）が生成される
   - 各匿名クラスは独自のクラスローダーエントリを持つ
   - メモリ使用量が比較的多い

2. **ラムダ式**：
   - Java 8以降では、`invokedynamic`命令を使用して実装される
   - 新しいクラスファイルは生成されない
   - メモリ使用量が少ない
   - JVMの最適化対象になりやすい

この違いにより、多数のインスタンスを作成する場合、ラムダ式の方がメモリ効率が良く、パフォーマンスも向上する可能性があります。

### 関数型プログラミングの考え方

ラムダ式の導入により、Javaでも関数型プログラミングの考え方を取り入れることができるようになりました。関数型プログラミングの主な特徴は以下の通りです：

1. **不変性（Immutability）**：
   - データは変更せず、新しい値を生成する
   - 副作用を減らし、予測可能性を高める

2. **高階関数（Higher-order Functions）**：
   - 関数を引数として受け取ったり、結果として返したりする
   - 例：`map`, `filter`, `reduce`などの操作

3. **宣言的プログラミング（Declarative Programming）**：
   - 「何を」行うかを記述し、「どのように」行うかは抽象化する
   - 例：命令型の`for`ループではなく、`stream().filter().map()`のような宣言的な記述

4. **純粋関数（Pure Functions）**：
   - 同じ入力に対して常に同じ出力を返す
   - 副作用がない（外部の状態を変更しない）

Javaは純粋な関数型言語ではありませんが、ラムダ式とStream APIを使うことで、関数型プログラミングのスタイルを取り入れることができます。

### メソッド参照の種類と使い分け

メソッド参照は、既存のメソッドをラムダ式の代わりに使用する簡潔な構文です。4種類のメソッド参照があり、それぞれ異なる場面で使用します：

1. **静的メソッド参照**：`クラス名::静的メソッド名`
   - 例：`Integer::parseInt`
   - 使用場面：静的メソッドを呼び出す場合

2. **特定オブジェクトのインスタンスメソッド参照**：`インスタンス::メソッド名`
   - 例：`System.out::println`
   - 使用場面：特定のオブジェクトのメソッドを呼び出す場合

3. **特定型のインスタンスメソッド参照**：`クラス名::インスタンスメソッド名`
   - 例：`String::toUpperCase`
   - 使用場面：ラムダ式の引数がメソッドのレシーバーになる場合

4. **コンストラクタ参照**：`クラス名::new`
   - 例：`ArrayList::new`
   - 使用場面：新しいオブジェクトを作成する場合

メソッド参照は、既存のメソッドをそのまま使用する場合に、ラムダ式よりも簡潔で読みやすいコードになります。

### Stream APIの基本

Stream APIは、コレクションの要素を一連の処理ステップで変換するためのAPIです。主な特徴は以下の通りです：

1. **パイプライン処理**：
   - 複数の操作を連鎖させて、データを変換・処理できる
   - 例：`stream().filter().map().collect()`

2. **遅延評価**：
   - 終端操作（`collect`, `forEach`など）が呼び出されるまで、中間操作は実行されない
   - 必要な要素だけを処理できる（ショートサーキット評価）

3. **並列処理**：
   - `parallelStream()`を使用することで、マルチコアプロセッサを活用した並列処理が可能
   - 大量のデータを処理する場合に効果的

主な操作：
- **中間操作**（結果として新しいStreamを返す）：
  - `filter`: 条件に合う要素だけを抽出
  - `map`: 各要素を変換
  - `flatMap`: 各要素を複数の要素に変換し、平坦化
  - `sorted`: 要素をソート
  - `distinct`: 重複を除去
  - `limit`: 先頭から指定した数の要素だけを取得
  - `skip`: 先頭から指定した数の要素をスキップ

- **終端操作**（Streamの処理を実行し、結果を返す）：
  - `collect`: 要素を集約（リスト、セット、マップなどに変換）
  - `forEach`: 各要素に対して処理を実行
  - `reduce`: 要素を集約して単一の結果を返す
  - `count`: 要素の数を返す
  - `anyMatch`, `allMatch`, `noneMatch`: 条件に合う要素があるか、すべてが合うか、すべてが合わないかを判定
  - `findFirst`, `findAny`: 最初の要素、または任意の要素を返す

## 実務での活用シーン

### 1. イベント処理

GUIアプリケーションやWebアプリケーションでは、ボタンクリックなどのイベントに対する処理を定義する必要があります。ラムダ式を使うと、イベントハンドラを簡潔に記述できます。

```java
// 従来の方法（匿名クラス）
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("Button clicked!");
    }
});

// ラムダ式を使用
button.addActionListener(e -> System.out.println("Button clicked!"));
```

### 2. コレクション操作

業務アプリケーションでは、データのフィルタリング、変換、集計などの操作が頻繁に必要になります。Stream APIとラムダ式を使うと、これらの操作を簡潔かつ宣言的に記述できます。

```java
// 従来の方法
List<Order> highValueOrders = new ArrayList<>();
for (Order order : orders) {
    if (order.getAmount() > 1000) {
        highValueOrders.add(order);
    }
}
Collections.sort(highValueOrders, new Comparator<Order>() {
    @Override
    public int compare(Order o1, Order o2) {
        return o2.getAmount().compareTo(o1.getAmount());
    }
});

// Stream APIとラムダ式を使用
List<Order> highValueOrders = orders.stream()
    .filter(order -> order.getAmount() > 1000)
    .sorted((o1, o2) -> o2.getAmount().compareTo(o1.getAmount()))
    .collect(Collectors.toList());
```

### 3. 非同期処理

マルチスレッドプログラミングやバックグラウンド処理を行う場合、ラムダ式を使うとコードが簡潔になります。

```java
// 従来の方法
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Running in background");
    }
}).start();

// ラムダ式を使用
new Thread(() -> System.out.println("Running in background")).start();

// CompletableFutureを使用した非同期処理
CompletableFuture.supplyAsync(() -> fetchDataFromDatabase())
    .thenApply(data -> processData(data))
    .thenAccept(result -> saveResult(result))
    .exceptionally(ex -> {
        handleError(ex);
        return null;
    });
```

### 4. 依存性の注入とファクトリパターン

依存性の注入やファクトリパターンなど、オブジェクト生成のロジックを抽象化する場合にもラムダ式が役立ちます。

```java
// 従来の方法
interface Factory<T> {
    T create();
}

class UserFactory implements Factory<User> {
    @Override
    public User create() {
        return new User();
    }
}

// ラムダ式を使用
Factory<User> userFactory = () -> new User();
// または
Supplier<User> userSupplier = User::new;
```

### 5. バリデーションとビジネスルール

バリデーションやビジネスルールを定義する場合、ラムダ式を使うと柔軟性が高まります。

```java
// バリデーションルールの定義
Map<String, Predicate<String>> validationRules = new HashMap<>();
validationRules.put("username", s -> s != null && s.length() >= 3 && s.length() <= 20);
validationRules.put("email", s -> s != null && s.matches("^[A-Za-z0-9+_.-]+@(.+)$"));
validationRules.put("password", s -> s != null && s.length() >= 8 && s.matches(".*[A-Z].*") && s.matches(".*[0-9].*"));

// バリデーションの実行
public List<String> validate(Map<String, String> formData) {
    List<String> errors = new ArrayList<>();
    for (Map.Entry<String, Predicate<String>> rule : validationRules.entrySet()) {
        String field = rule.getKey();
        Predicate<String> validator = rule.getValue();
        if (!validator.test(formData.get(field))) {
            errors.add(field + " is invalid");
        }
    }
    return errors;
}
```

## 参考資料

1. Java公式チュートリアル: [Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
2. Java公式ドキュメント: [Package java.util.function](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html)
3. Java公式ドキュメント: [Stream (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html)
4. 書籍: 「Effective Java 第3版」（Joshua Bloch著） - 特に項目42, 43, 44（ラムダとStream）
5. 書籍: 「Java SE 8 for the Really Impatient」（Cay S. Horstmann著）
6. Baeldung: [Lambda Expressions and Functional Interfaces](https://www.baeldung.com/java-8-lambda-expressions-tips)
7. Baeldung: [Method References in Java](https://www.baeldung.com/java-method-references)
