---
layout: default
title: "Day 2: コレクションフレームワークの効果的な活用"
---
# Day 2: コレクションフレームワークの効果的な活用

## 学習の目的と背景

コレクションフレームワークは、データの格納と操作を効率的に行うためのJavaの重要な機能です。業務アプリケーションでは、リスト、セット、マップなどのコレクションを適切に選択し、効率的に操作することが求められます。特に大量のデータを扱う場合、コレクションの選択と使い方によってパフォーマンスが大きく変わります。

本日の学習では、Javaのコレクションフレームワークの基本から応用までを体系的に学び、実際のコードを書きながら理解を深めていきます。特にArrayListの復習から始め、Set、Mapなどの他のコレクションの特性と使い分けを習得することを目指します。

## 前提知識と準備

### 必要な前提知識
- Javaの基本構文（変数、制御構文、クラスなど）
- オブジェクト指向プログラミングの基本概念
- ArrayListの基本的な使い方

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

### 1. コレクションフレームワークの概要

Javaのコレクションフレームワークは、データの集合を格納し操作するためのクラスとインターフェースの集まりです。主に以下の3つのカテゴリに分類されます：

1. **List**：順序付けられた要素のコレクション。重複を許可します。
   - 例：ArrayList, LinkedList, Vector

2. **Set**：重複を許可しない要素のコレクション。
   - 例：HashSet, LinkedHashSet, TreeSet

3. **Map**：キーと値のペアを格納するコレクション。キーは重複できません。
   - 例：HashMap, LinkedHashMap, TreeMap

**用語解説: コレクションフレームワーク**  
データの集合を扱うためのクラスとインターフェースの枠組みです。Java 1.2から導入され、データ構造の実装と操作を標準化しています。コレクションフレームワークを使用することで、効率的なデータ操作が可能になります。

### 2. Listインターフェースとその実装

#### ArrayList

`ArrayList`は、配列をベースにした`List`インターフェースの実装です。ランダムアクセス（インデックスによる要素へのアクセス）が高速ですが、要素の挿入や削除は比較的遅いという特徴があります。

```java
// ArrayListの基本的な使い方
List<String> names = new ArrayList<>();
names.add("山田");  // 要素の追加
names.add("佐藤");
names.add("鈴木");
System.out.println(names.get(1));  // インデックスによる要素の取得（結果: 佐藤）
names.remove(0);  // インデックスによる要素の削除
names.contains("鈴木");  // 要素の存在確認（結果: true）
```

#### LinkedList

`LinkedList`は、双方向リンクリストをベースにした`List`インターフェースの実装です。要素の挿入や削除が高速ですが、ランダムアクセスは比較的遅いという特徴があります。

```java
// LinkedListの基本的な使い方
List<String> names = new LinkedList<>();
names.add("山田");
names.add("佐藤");
names.add("鈴木");
names.add(1, "田中");  // 指定位置への要素の挿入
```

### 3. Setインターフェースとその実装

#### HashSet

`HashSet`は、ハッシュテーブルをベースにした`Set`インターフェースの実装です。要素の追加、削除、検索が高速（平均的にO(1)の時間計算量）ですが、要素の順序は保証されません。

**用語解説: オーダー記法（Big O表記）**  
アルゴリズムの実行時間や必要なスペースが入力サイズによってどのように変化するかを表す記法です。例えば、O(1)は定数時間、O(n)は入力サイズに比例する時間、O(log n)は対数時間を意味します。

```java
// HashSetの基本的な使い方
Set<String> uniqueNames = new HashSet<>();
uniqueNames.add("山田");  // 要素の追加
uniqueNames.add("佐藤");
uniqueNames.add("山田");  // 重複する要素は追加されない
System.out.println(uniqueNames.size());  // 結果: 2
```

#### LinkedHashSet

`LinkedHashSet`は、`HashSet`と同様にハッシュテーブルをベースにしていますが、要素の挿入順序を保持するという特徴があります。

```java
// LinkedHashSetの基本的な使い方
Set<String> orderedUniqueNames = new LinkedHashSet<>();
orderedUniqueNames.add("山田");
orderedUniqueNames.add("佐藤");
orderedUniqueNames.add("鈴木");
// 要素は追加された順序で取得される
```

#### TreeSet

`TreeSet`は、赤黒木をベースにした`Set`インターフェースの実装です。要素は自然順序（natural ordering）または指定されたComparatorによってソートされます。

```java
// TreeSetの基本的な使い方
Set<String> sortedUniqueNames = new TreeSet<>();
sortedUniqueNames.add("山田");
sortedUniqueNames.add("佐藤");
sortedUniqueNames.add("鈴木");
// 要素はアルファベット順（または指定された順序）で取得される
```

### 4. Mapインターフェースとその実装

#### HashMap

`HashMap`は、ハッシュテーブルをベースにした`Map`インターフェースの実装です。キーと値のペアを格納し、キーによる値の検索が高速（平均的にO(1)の時間計算量）ですが、キーの順序は保証されません。

```java
// HashMapの基本的な使い方
Map<String, Integer> ages = new HashMap<>();
ages.put("山田", 30);  // キーと値のペアを追加
ages.put("佐藤", 25);
ages.put("鈴木", 40);
System.out.println(ages.get("佐藤"));  // キーによる値の取得（結果: 25）
ages.remove("山田");  // キーによるペアの削除
```

#### LinkedHashMap

`LinkedHashMap`は、`HashMap`と同様にハッシュテーブルをベースにしていますが、キーの挿入順序または最近アクセスした順序（アクセス順序）を保持するという特徴があります。

```java
// LinkedHashMapの基本的な使い方
Map<String, Integer> orderedAges = new LinkedHashMap<>();
orderedAges.put("山田", 30);
orderedAges.put("佐藤", 25);
orderedAges.put("鈴木", 40);
// キーと値のペアは追加された順序で取得される
```

#### TreeMap

`TreeMap`は、赤黒木をベースにした`Map`インターフェースの実装です。キーは自然順序または指定されたComparatorによってソートされます。

```java
// TreeMapの基本的な使い方
Map<String, Integer> sortedAges = new TreeMap<>();
sortedAges.put("山田", 30);
sortedAges.put("佐藤", 25);
sortedAges.put("鈴木", 40);
// キーと値のペアはキーのアルファベット順（または指定された順序）で取得される
```

### 5. コレクションの選択基準

コレクションを選択する際は、以下の点を考慮することが重要です：

1. **データの特性**：
   - 順序が重要か？
   - 重複を許可するか？
   - キーと値のペアが必要か？

2. **操作の種類と頻度**：
   - 検索が多いか？
   - 挿入/削除が多いか？
   - ランダムアクセスが必要か？

3. **パフォーマンス要件**：
   - 時間計算量（Big O）
   - 空間計算量（メモリ使用量）

#### 主なコレクションの計算量比較

| コレクション | 追加 | 削除 | 検索 | 特徴 |
|------------|------|------|------|------|
| ArrayList | O(1)* | O(n) | O(1)** | ランダムアクセスが速い |
| LinkedList | O(1) | O(1)*** | O(n) | 挿入/削除が速い |
| HashSet | O(1) | O(1) | O(1) | 重複なし、順序なし |
| LinkedHashSet | O(1) | O(1) | O(1) | 重複なし、挿入順序あり |
| TreeSet | O(log n) | O(log n) | O(log n) | 重複なし、ソート済み |
| HashMap | O(1) | O(1) | O(1) | キー重複なし、順序なし |
| LinkedHashMap | O(1) | O(1) | O(1) | キー重複なし、挿入順序あり |
| TreeMap | O(log n) | O(log n) | O(log n) | キー重複なし、ソート済み |

*末尾への追加の場合。途中への挿入はO(n)。
**インデックスによるアクセスの場合。値による検索はO(n)。
***位置が分かっている場合。位置の検索が必要な場合はO(n)。

### 6. hashCode()とequals()の重要性

`HashSet`や`HashMap`などのハッシュベースのコレクションでは、`hashCode()`と`equals()`メソッドが正しく実装されていることが重要です。

**用語解説: hashCode()とequals()**  
`hashCode()`はオブジェクトのハッシュコード（整数値）を返すメソッドで、`equals()`は2つのオブジェクトが等しいかどうかを判定するメソッドです。ハッシュベースのコレクションでは、これらのメソッドを使用して要素の格納位置の決定や等価性の判定を行います。

```java
// hashCode()とequals()の実装例
public class Person {
    private String name;
    private int age;
    
    // コンストラクタ、ゲッター、セッターは省略
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

## Eclipse操作ガイド

### クラスの作成

1. プロジェクト「JavaCollections_Day2」を右クリックします。
2. 「New」→「Class」を選択します。
3. 「Name」に作成するクラス名（例：`CollectionDemo`）を入力します。
4. 「public static void main(String[] args)」にチェックを入れます。
5. 「Finish」をクリックしてクラスを作成します。

### hashCode()とequals()の自動生成

1. クラスのエディタ内で右クリックします。
2. 「Source」→「Generate hashCode() and equals()...」を選択します。
3. ハッシュコードと等価性の判定に使用するフィールドを選択します。
4. 「Generate」をクリックして生成します。

### コレクションの操作とデバッグ

1. コレクションを操作するコードを記述します。
2. ブレークポイントを設定します（行番号の左側をダブルクリック）。
3. 右クリックして「Debug As」→「Java Application」を選択します。
4. デバッグパースペクティブで変数の値を確認します。
5. 「Step Over」（F6）などのボタンで実行を制御します。

## コア演習（必須）

### 演習1: リスト操作の基本と応用（難易度：基本）

#### 課題
社員情報を管理するためのリストを作成し、以下の操作を行うプログラムを作成してください：
1. 社員情報の追加
2. 社員情報の検索（名前による）
3. 社員情報の更新
4. 社員情報の削除
5. 全社員情報の表示

#### 雛形コード
以下のクラスを作成してください：

```java
package collections.list;

import java.util.ArrayList;
import java.util.List;

/**
 * 社員情報を表すクラス
 */
class Employee {
    private int id;
    private String name;
    private String department;
    private int salary;
    
    public Employee(int id, String name, String department, int salary) {
        this.id = id;
        this.name = name;
        this.department = department;
        this.salary = salary;
    }
    
    // ゲッターとセッター
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getDepartment() { return department; }
    public void setDepartment(String department) { this.department = department; }
    public int getSalary() { return salary; }
    public void setSalary(int salary) { this.salary = salary; }
    
    @Override
    public String toString() {
        return "Employee [id=" + id + ", name=" + name + ", department=" + department + ", salary=" + salary + "]";
    }
}

/**
 * 社員情報を管理するクラス
 */
public class EmployeeManager {
    private List<Employee> employees;
    
    public EmployeeManager() {
        this.employees = new ArrayList<>();
    }
    
    /**
     * 社員情報を追加します。
     * 
     * @param employee 追加する社員情報
     */
    public void addEmployee(Employee employee) {
        // TODO: ここに処理を実装
        // ヒント: Listのaddメソッドを使用
    }
    
    /**
     * 名前で社員情報を検索します。
     * 
     * @param name 検索する社員の名前
     * @return 見つかった社員情報のリスト（見つからない場合は空のリスト）
     */
    public List<Employee> findEmployeesByName(String name) {
        // TODO: ここに処理を実装
        // ヒント1: 新しいArrayListを作成して結果を格納
        // ヒント2: employeesリストをループで回し、名前が一致する社員を結果リストに追加
        return null; // 仮の戻り値
    }
    
    /**
     * 社員IDで社員情報を検索します。
     * 
     * @param id 検索する社員のID
     * @return 見つかった社員情報（見つからない場合はnull）
     */
    public Employee findEmployeeById(int id) {
        // TODO: ここに処理を実装
        // ヒント: employeesリストをループで回し、IDが一致する社員を返す
        return null; // 仮の戻り値
    }
    
    /**
     * 社員情報を更新します。
     * 
     * @param id 更新する社員のID
     * @param name 新しい名前（nullの場合は更新しない）
     * @param department 新しい部署（nullの場合は更新しない）
     * @param salary 新しい給与（0以下の場合は更新しない）
     * @return 更新が成功したかどうか
     */
    public boolean updateEmployee(int id, String name, String department, int salary) {
        // TODO: ここに処理を実装
        // ヒント1: findEmployeeByIdメソッドを使用して社員を検索
        // ヒント2: 見つかった場合は、nullでない引数の値で社員情報を更新
        return false; // 仮の戻り値
    }
    
    /**
     * 社員情報を削除します。
     * 
     * @param id 削除する社員のID
     * @return 削除が成功したかどうか
     */
    public boolean removeEmployee(int id) {
        // TODO: ここに処理を実装
        // ヒント1: findEmployeeByIdメソッドを使用して社員を検索
        // ヒント2: 見つかった場合は、Listのremoveメソッドで削除
        return false; // 仮の戻り値
    }
    
    /**
     * 全社員情報を表示します。
     */
    public void displayAllEmployees() {
        // TODO: ここに処理を実装
        // ヒント: employeesリストをループで回し、各社員情報を表示
    }
    
    public static void main(String[] args) {
        EmployeeManager manager = new EmployeeManager();
        
        // 社員情報の追加
        manager.addEmployee(new Employee(1, "山田太郎", "営業部", 350000));
        manager.addEmployee(new Employee(2, "佐藤花子", "総務部", 320000));
        manager.addEmployee(new Employee(3, "鈴木一郎", "開発部", 400000));
        manager.addEmployee(new Employee(4, "田中美咲", "営業部", 380000));
        manager.addEmployee(new Employee(5, "伊藤健太", "開発部", 420000));
        
        // 全社員情報の表示
        System.out.println("===== 全社員情報 =====");
        manager.displayAllEmployees();
        
        // 名前による検索
        System.out.println("\n===== 名前に「田」を含む社員 =====");
        List<Employee> foundEmployees = manager.findEmployeesByName("田");
        for (Employee emp : foundEmployees) {
            System.out.println(emp);
        }
        
        // 社員情報の更新
        System.out.println("\n===== 社員ID 3の給与を更新 =====");
        boolean updated = manager.updateEmployee(3, null, null, 450000);
        System.out.println("更新結果: " + updated);
        
        // 更新後の社員情報を表示
        Employee updatedEmployee = manager.findEmployeeById(3);
        if (updatedEmployee != null) {
            System.out.println(updatedEmployee);
        }
        
        // 社員情報の削除
        System.out.println("\n===== 社員ID 2を削除 =====");
        boolean removed = manager.removeEmployee(2);
        System.out.println("削除結果: " + removed);
        
        // 削除後の全社員情報を表示
        System.out.println("\n===== 削除後の全社員情報 =====");
        manager.displayAllEmployees();
    }
}
```

#### 実装ヒント
1. `addEmployee`メソッドでは、`List`の`add`メソッドを使用して社員情報を追加します。
2. `findEmployeesByName`メソッドでは、新しい`ArrayList`を作成し、名前に指定された文字列を含む社員を追加して返します。
3. `findEmployeeById`メソッドでは、`for`ループで社員リストを走査し、IDが一致する社員を返します。
4. `updateEmployee`メソッドでは、`findEmployeeById`メソッドで社員を検索し、見つかった場合は情報を更新します。
5. `removeEmployee`メソッドでは、`findEmployeeById`メソッドで社員を検索し、見つかった場合は`List`の`remove`メソッドで削除します。
6. `displayAllEmployees`メソッドでは、`for`ループで社員リストを走査し、各社員情報を表示します。

### 演習2: セットとマップの活用（難易度：応用）

#### 課題
商品情報を管理するためのシステムを作成してください。以下の機能を実装してください：
1. 商品カテゴリの管理（重複なし、ソート済み）
2. 商品情報の管理（商品コードをキーとする）
3. カテゴリ別の商品一覧の取得
4. 価格帯による商品の検索

#### 雛形コード
以下のクラスを作成してください：

```java
package collections.setmap;

import java.util.*;

/**
 * 商品情報を表すクラス
 */
class Product {
    private String code;
    private String name;
    private String category;
    private int price;
    
    public Product(String code, String name, String category, int price) {
        this.code = code;
        this.name = name;
        this.category = category;
        this.price = price;
    }
    
    // ゲッター
    public String getCode() { return code; }
    public String getName() { return name; }
    public String getCategory() { return category; }
    public int getPrice() { return price; }
    
    @Override
    public String toString() {
        return "Product [code=" + code + ", name=" + name + ", category=" + category + ", price=" + price + "]";
    }
}

/**
 * 商品情報を管理するクラス
 */
public class ProductManager {
    private Set<String> categories;  // 商品カテゴリのセット
    private Map<String, Product> products;  // 商品コードをキーとする商品マップ
    private Map<String, List<Product>> productsByCategory;  // カテゴリ別の商品リスト
    
    public ProductManager() {
        // TODO: ここに初期化処理を実装
        // ヒント1: categoriesはTreeSetで初期化（ソート済みのセット）
        // ヒント2: productsはHashMapで初期化
        // ヒント3: productsByCategoryはHashMapで初期化
    }
    
    /**
     * 商品を追加します。
     * 
     * @param product 追加する商品
     * @return 追加が成功したかどうか（既に同じ商品コードが存在する場合はfalse）
     */
    public boolean addProduct(Product product) {
        // TODO: ここに処理を実装
        // ヒント1: 既に同じ商品コードが存在する場合はfalseを返す
        // ヒント2: 商品を追加する際に、カテゴリをcategoriesに追加
        // ヒント3: 商品をproductsに追加
        // ヒント4: 商品をproductsByCategoryの対応するリストに追加
        return false; // 仮の戻り値
    }
    
    /**
     * 商品コードで商品を検索します。
     * 
     * @param code 検索する商品コード
     * @return 見つかった商品（見つからない場合はnull）
     */
    public Product findProductByCode(String code) {
        // TODO: ここに処理を実装
        // ヒント: Mapのgetメソッドを使用
        return null; // 仮の戻り値
    }
    
    /**
     * すべての商品カテゴリを取得します。
     * 
     * @return ソート済みの商品カテゴリのセット
     */
    public Set<String> getAllCategories() {
        // TODO: ここに処理を実装
        // ヒント: categoriesをそのまま返す
        return null; // 仮の戻り値
    }
    
    /**
     * 指定されたカテゴリの商品一覧を取得します。
     * 
     * @param category 取得するカテゴリ
     * @return カテゴリに属する商品のリスト（カテゴリが存在しない場合は空のリスト）
     */
    public List<Product> getProductsByCategory(String category) {
        // TODO: ここに処理を実装
        // ヒント1: productsByCategoryから指定されたカテゴリの商品リストを取得
        // ヒント2: カテゴリが存在しない場合は空のリストを返す
        return null; // 仮の戻り値
    }
    
    /**
     * 指定された価格範囲の商品を検索します。
     * 
     * @param minPrice 最小価格（この価格以上）
     * @param maxPrice 最大価格（この価格以下）
     * @return 価格範囲内の商品のリスト
     */
    public List<Product> findProductsByPriceRange(int minPrice, int maxPrice) {
        // TODO: ここに処理を実装
        // ヒント1: 新しいArrayListを作成して結果を格納
        // ヒント2: productsの値（商品）をループで回し、価格範囲内の商品を結果リストに追加
        return null; // 仮の戻り値
    }
    
    /**
     * すべての商品情報を表示します。
     */
    public void displayAllProducts() {
        // TODO: ここに処理を実装
        // ヒント: productsの値（商品）をループで回し、各商品情報を表示
    }
    
    public static void main(String[] args) {
        ProductManager manager = new ProductManager();
        
        // 商品の追加
        manager.addProduct(new Product("P001", "ノートパソコン", "電化製品", 120000));
        manager.addProduct(new Product("P002", "デスクトップPC", "電化製品", 150000));
        manager.addProduct(new Product("P003", "スマートフォン", "電化製品", 80000));
        manager.addProduct(new Product("P004", "コーヒーメーカー", "キッチン", 15000));
        manager.addProduct(new Product("P005", "電子レンジ", "キッチン", 30000));
        manager.addProduct(new Product("P006", "テレビ", "電化製品", 200000));
        manager.addProduct(new Product("P007", "冷蔵庫", "キッチン", 180000));
        manager.addProduct(new Product("P008", "洗濯機", "家電", 100000));
        manager.addProduct(new Product("P009", "掃除機", "家電", 50000));
        
        // すべての商品情報を表示
        System.out.println("===== すべての商品情報 =====");
        manager.displayAllProducts();
        
        // すべてのカテゴリを表示
        System.out.println("\n===== すべてのカテゴリ =====");
        Set<String> categories = manager.getAllCategories();
        for (String category : categories) {
            System.out.println(category);
        }
        
        // カテゴリ別の商品一覧を表示
        System.out.println("\n===== カテゴリ別の商品一覧 =====");
        for (String category : categories) {
            System.out.println("【" + category + "】");
            List<Product> categoryProducts = manager.getProductsByCategory(category);
            for (Product product : categoryProducts) {
                System.out.println("  " + product);
            }
        }
        
        // 価格帯による商品検索
        System.out.println("\n===== 価格帯による商品検索（50,000円～150,000円） =====");
        List<Product> priceRangeProducts = manager.findProductsByPriceRange(50000, 150000);
        for (Product product : priceRangeProducts) {
            System.out.println(product);
        }
        
        // 商品コードによる検索
        System.out.println("\n===== 商品コードによる検索（P005） =====");
        Product foundProduct = manager.findProductByCode("P005");
        if (foundProduct != null) {
            System.out.println(foundProduct);
        } else {
            System.out.println("商品が見つかりませんでした。");
        }
    }
}
```

#### 実装ヒント
1. コンストラクタでは、`categories`を`TreeSet`で初期化し、`products`と`productsByCategory`を`HashMap`で初期化します。
2. `addProduct`メソッドでは、商品コードが既に存在するかチェックし、存在しない場合のみ追加します。また、カテゴリを`categories`に追加し、商品を`products`と`productsByCategory`に追加します。
3. `findProductByCode`メソッドでは、`Map`の`get`メソッドを使用して商品を検索します。
4. `getAllCategories`メソッドでは、`categories`をそのまま返します。
5. `getProductsByCategory`メソッドでは、`productsByCategory`から指定されたカテゴリの商品リストを取得します。カテゴリが存在しない場合は空のリストを返します。
6. `findProductsByPriceRange`メソッドでは、新しい`ArrayList`を作成し、価格範囲内の商品を追加して返します。
7. `displayAllProducts`メソッドでは、`products`の値（商品）をループで回し、各商品情報を表示します。

### 演習3: コレクション選択の実践（難易度：応用）

#### 課題
以下のシナリオに対して、最適なコレクションを選択し、その理由を説明してください。また、選択したコレクションを使用して簡単な実装を行ってください。

1. ユーザーIDの一意性を保証する必要がある場合
2. 最近アクセスした10件のドキュメントを履歴として保持する場合
3. 単語の出現回数をカウントする場合
4. 優先度付きのタスクキューを実装する場合

#### 雛形コード
以下のクラスを作成してください：

```java
package collections.selection;

import java.util.*;

/**
 * 様々なシナリオに対して最適なコレクションを選択するデモクラス
 */
public class CollectionSelectionDemo {
    
    /**
     * ユーザーIDの一意性を保証するためのコレクションを実装します。
     */
    public static void uniqueUserIds() {
        System.out.println("===== ユーザーIDの一意性を保証する =====");
        
        // TODO: ここに処理を実装
        // ヒント1: 一意性を保証するためのコレクションを選択（例: HashSet）
        // ヒント2: いくつかのユーザーIDを追加し、重複を試みる
        // ヒント3: 結果を表示し、一意性が保証されていることを確認
        
        // 選択理由の説明
        System.out.println("\n【選択理由】");
        System.out.println("HashSetを選択しました。理由は...");
        // TODO: ここに選択理由を記述
    }
    
    /**
     * 最近アクセスした10件のドキュメントを履歴として保持するコレクションを実装します。
     */
    public static void recentDocuments() {
        System.out.println("\n===== 最近アクセスした10件のドキュメント =====");
        
        // TODO: ここに処理を実装
        // ヒント1: 順序を保持し、サイズを制限できるコレクションを選択または実装
        // ヒント2: いくつかのドキュメントにアクセスし、履歴を更新
        // ヒント3: 結果を表示し、最新の10件のみが保持されていることを確認
        
        // 選択理由の説明
        System.out.println("\n【選択理由】");
        System.out.println("LinkedHashMapを選択しました。理由は...");
        // TODO: ここに選択理由を記述
    }
    
    /**
     * 単語の出現回数をカウントするコレクションを実装します。
     */
    public static void wordFrequencyCounter() {
        System.out.println("\n===== 単語の出現回数をカウント =====");
        
        // サンプルテキスト
        String text = "Java is a programming language. Java is widely used. Programming in Java is fun.";
        
        // TODO: ここに処理を実装
        // ヒント1: キーと値のペアを格納できるコレクションを選択（例: HashMap）
        // ヒント2: テキストを単語に分割し、各単語の出現回数をカウント
        // ヒント3: 結果を表示
        
        // 選択理由の説明
        System.out.println("\n【選択理由】");
        System.out.println("HashMapを選択しました。理由は...");
        // TODO: ここに選択理由を記述
    }
    
    /**
     * 優先度付きのタスクキューを実装します。
     */
    public static void priorityTaskQueue() {
        System.out.println("\n===== 優先度付きのタスクキュー =====");
        
        // TODO: ここに処理を実装
        // ヒント1: 優先度に基づいて要素を取り出せるコレクションを選択（例: PriorityQueue）
        // ヒント2: いくつかのタスクを異なる優先度で追加
        // ヒント3: タスクを取り出し、優先度順になっていることを確認
        
        // 選択理由の説明
        System.out.println("\n【選択理由】");
        System.out.println("PriorityQueueを選択しました。理由は...");
        // TODO: ここに選択理由を記述
    }
    
    public static void main(String[] args) {
        uniqueUserIds();
        recentDocuments();
        wordFrequencyCounter();
        priorityTaskQueue();
    }
}
```

#### 実装ヒント
1. `uniqueUserIds`メソッドでは、`HashSet`を使用してユーザーIDの一意性を保証します。`HashSet`は重複を許可せず、追加・検索が高速（O(1)）であるため、一意性を保証する用途に適しています。
2. `recentDocuments`メソッドでは、`LinkedHashMap`を使用して最近アクセスしたドキュメントを保持します。アクセス順序モードを有効にし、サイズが10を超えたら最も古いエントリを削除するようにします。
3. `wordFrequencyCounter`メソッドでは、`HashMap`を使用して単語の出現回数をカウントします。単語をキー、出現回数を値として格納します。
4. `priorityTaskQueue`メソッドでは、`PriorityQueue`を使用して優先度付きのタスクキューを実装します。タスクの優先度に基づいてタスクを取り出せるようにします。

## 発展演習（オプション）

### 発展演習1: パフォーマンス比較（難易度：発展）

#### 課題
異なるコレクション（ArrayList, LinkedList, HashSet, TreeSet）のパフォーマンスを比較するプログラムを作成してください。以下の操作について、各コレクションの実行時間を計測し、結果を比較してください：

1. 要素の追加（10万件）
2. 要素の検索（存在する要素と存在しない要素）
3. 要素の削除（先頭、中間、末尾）
4. 要素の走査

#### 雛形コード
以下のクラスを作成してください：

```java
package collections.performance;

import java.util.*;

/**
 * 異なるコレクションのパフォーマンスを比較するクラス
 */
public class CollectionPerformanceComparison {
    
    private static final int NUM_ELEMENTS = 100000;
    
    /**
     * 要素の追加のパフォーマンスを計測します。
     */
    public static void testAddPerformance() {
        System.out.println("===== 要素の追加のパフォーマンス比較 =====");
        
        // ArrayList
        long startTime = System.currentTimeMillis();
        List<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            arrayList.add(i);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("ArrayList: " + (endTime - startTime) + "ms");
        
        // TODO: LinkedList, HashSet, TreeSetについても同様に計測
        // ヒント: 上記のArrayListと同様のコードを各コレクションに対して実装
    }
    
    /**
     * 要素の検索のパフォーマンスを計測します。
     */
    public static void testSearchPerformance() {
        System.out.println("\n===== 要素の検索のパフォーマンス比較 =====");
        
        // コレクションの準備
        List<Integer> arrayList = new ArrayList<>();
        List<Integer> linkedList = new LinkedList<>();
        Set<Integer> hashSet = new HashSet<>();
        Set<Integer> treeSet = new TreeSet<>();
        
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            arrayList.add(i);
            linkedList.add(i);
            hashSet.add(i);
            treeSet.add(i);
        }
        
        // 存在する要素の検索
        System.out.println("--- 存在する要素の検索 ---");
        
        // ArrayList
        long startTime = System.currentTimeMillis();
        boolean found = arrayList.contains(NUM_ELEMENTS / 2);
        long endTime = System.currentTimeMillis();
        System.out.println("ArrayList: " + (endTime - startTime) + "ms, 結果: " + found);
        
        // TODO: LinkedList, HashSet, TreeSetについても同様に計測
        // ヒント: 上記のArrayListと同様のコードを各コレクションに対して実装
        
        // 存在しない要素の検索
        System.out.println("--- 存在しない要素の検索 ---");
        
        // TODO: 各コレクションで存在しない要素の検索を計測
        // ヒント: containsメソッドを使用して、NUM_ELEMENTS + 1などの存在しない値を検索
    }
    
    /**
     * 要素の削除のパフォーマンスを計測します。
     */
    public static void testRemovePerformance() {
        System.out.println("\n===== 要素の削除のパフォーマンス比較 =====");
        
        // TODO: ここに処理を実装
        // ヒント1: 各コレクションに要素を追加
        // ヒント2: 先頭、中間、末尾の要素を削除し、実行時間を計測
        // ヒント3: 結果を表示
    }
    
    /**
     * 要素の走査のパフォーマンスを計測します。
     */
    public static void testIterationPerformance() {
        System.out.println("\n===== 要素の走査のパフォーマンス比較 =====");
        
        // TODO: ここに処理を実装
        // ヒント1: 各コレクションに要素を追加
        // ヒント2: for-each文を使用して全要素を走査し、実行時間を計測
        // ヒント3: 結果を表示
    }
    
    public static void main(String[] args) {
        testAddPerformance();
        testSearchPerformance();
        testRemovePerformance();
        testIterationPerformance();
        
        System.out.println("\n===== 考察 =====");
        // TODO: ここに結果の考察を記述
        // ヒント: 各コレクションの特性と計測結果を関連付けて考察
    }
}
```

#### 実装ヒント
1. `testAddPerformance`メソッドでは、各コレクションに対して同じ数の要素を追加し、実行時間を計測します。
2. `testSearchPerformance`メソッドでは、各コレクションに対して存在する要素と存在しない要素の検索を行い、実行時間を計測します。
3. `testRemovePerformance`メソッドでは、各コレクションに対して先頭、中間、末尾の要素を削除し、実行時間を計測します。
4. `testIterationPerformance`メソッドでは、各コレクションに対して全要素を走査し、実行時間を計測します。
5. 最後に、計測結果を考察し、各コレクションの特性と結果の関連性を説明します。

### 発展演習2: LRUキャッシュの実装（難易度：発展）

#### 課題
LRU（Least Recently Used）キャッシュを実装してください。LRUキャッシュは、最も長い間使われていない項目が削除される仕組みのキャッシュです。以下の機能を持つLRUキャッシュを実装してください：

1. キャッシュの最大サイズを指定できる
2. キーによる値の取得（get）
3. キーと値のペアの追加（put）
4. キャッシュが最大サイズに達した場合、最も長い間使われていない項目を削除

#### 雛形コード
以下のクラスを作成してください：

```java
package collections.cache;

import java.util.*;

/**
 * LRU（Least Recently Used）キャッシュの実装
 * 
 * @param <K> キーの型
 * @param <V> 値の型
 */
public class LRUCache<K, V> {
    
    private final int capacity;  // キャッシュの最大容量
    private Map<K, V> cache;     // キャッシュの実体
    
    /**
     * コンストラクタ
     * 
     * @param capacity キャッシュの最大容量
     */
    public LRUCache(int capacity) {
        this.capacity = capacity;
        // TODO: ここにキャッシュの初期化処理を実装
        // ヒント: LinkedHashMapを使用し、アクセス順序モードを有効にする
    }
    
    /**
     * キーに対応する値を取得します。
     * 
     * @param key キー
     * @return 値（キーが存在しない場合はnull）
     */
    public V get(K key) {
        // TODO: ここに処理を実装
        // ヒント: キャッシュからキーに対応する値を取得
        // 注意: getメソッドを呼び出すと、そのキーが最近使用されたことになる
        return null; // 仮の戻り値
    }
    
    /**
     * キーと値のペアをキャッシュに追加します。
     * 
     * @param key キー
     * @param value 値
     */
    public void put(K key, V value) {
        // TODO: ここに処理を実装
        // ヒント1: キャッシュにキーと値のペアを追加
        // ヒント2: キャッシュが最大容量に達した場合、最も長い間使われていない項目が自動的に削除される
    }
    
    /**
     * キャッシュの現在のサイズを取得します。
     * 
     * @return キャッシュのサイズ
     */
    public int size() {
        // TODO: ここに処理を実装
        // ヒント: キャッシュのサイズを返す
        return 0; // 仮の戻り値
    }
    
    /**
     * キャッシュの内容を表示します。
     */
    public void display() {
        // TODO: ここに処理を実装
        // ヒント: キャッシュの内容を表示（キーと値のペア）
    }
    
    public static void main(String[] args) {
        // LRUキャッシュのテスト（容量3）
        LRUCache<String, String> cache = new LRUCache<>(3);
        
        // キャッシュにデータを追加
        cache.put("key1", "value1");
        cache.put("key2", "value2");
        cache.put("key3", "value3");
        
        System.out.println("===== 初期状態 =====");
        cache.display();
        
        // key1にアクセス（最近使用した項目になる）
        System.out.println("\n===== key1にアクセス =====");
        System.out.println("key1の値: " + cache.get("key1"));
        cache.display();
        
        // 新しい項目を追加（最大容量を超えるため、最も長い間使われていない項目が削除される）
        System.out.println("\n===== 新しい項目を追加 =====");
        cache.put("key4", "value4");
        cache.display();
        
        // 存在しないキーにアクセス
        System.out.println("\n===== 存在しないキーにアクセス =====");
        System.out.println("key2の値: " + cache.get("key2"));  // key2は削除されているはず
        cache.display();
        
        // 既存のキーに新しい値を設定
        System.out.println("\n===== 既存のキーに新しい値を設定 =====");
        cache.put("key1", "new-value1");
        cache.display();
    }
}
```

#### 実装ヒント
1. `LinkedHashMap`を使用して、アクセス順序モードを有効にします。これにより、最も長い間使われていない項目を簡単に特定できます。
2. `LinkedHashMap`のコンストラクタで、第3引数に`true`を指定することでアクセス順序モードを有効にします。
3. `LinkedHashMap`のサブクラスを作成し、`removeEldestEntry`メソッドをオーバーライドして、キャッシュが最大容量に達した場合に最も古い項目を削除するようにします。
4. `get`メソッドでは、キャッシュからキーに対応する値を取得します。このメソッドを呼び出すと、そのキーが最近使用されたことになります。
5. `put`メソッドでは、キャッシュにキーと値のペアを追加します。キャッシュが最大容量に達した場合、最も長い間使われていない項目が自動的に削除されます。

## 解説と補足

### コレクション選択の重要性

適切なコレクションを選択することは、アプリケーションのパフォーマンスと可読性に大きな影響を与えます。以下の点を考慮して、最適なコレクションを選択することが重要です：

1. **データの特性**：
   - 順序が重要か？
   - 重複を許可するか？
   - キーと値のペアが必要か？
   - 要素の挿入/削除が頻繁に行われるか？

2. **操作の種類と頻度**：
   - 検索が多いか？
   - 挿入/削除が多いか？
   - ランダムアクセスが必要か？
   - イテレーション（走査）が頻繁に行われるか？

3. **パフォーマンス要件**：
   - 時間計算量（Big O）
   - 空間計算量（メモリ使用量）
   - スケーラビリティ（データ量の増加に対する対応）

### コレクションの使い分け

#### Listの使い分け

- **ArrayList**：
  - ランダムアクセスが多い場合
  - 要素の追加/削除が主に末尾で行われる場合
  - メモリ効率が重要な場合

- **LinkedList**：
  - 要素の挿入/削除が頻繁に行われる場合（特に先頭や中間）
  - キューやスタックとして使用する場合
  - リストの先頭と末尾の操作が多い場合

#### Setの使い分け

- **HashSet**：
  - 要素の追加/削除/検索が高速である必要がある場合
  - 要素の順序が重要でない場合
  - 重複を許可しない場合

- **LinkedHashSet**：
  - 要素の挿入順序を保持する必要がある場合
  - 要素の追加/削除/検索が高速である必要がある場合
  - 重複を許可しない場合

- **TreeSet**：
  - 要素をソートする必要がある場合
  - 範囲検索が必要な場合
  - 重複を許可しない場合

#### Mapの使い分け

- **HashMap**：
  - キーによる値の検索が高速である必要がある場合
  - キーの順序が重要でない場合
  - キーの重複を許可しない場合

- **LinkedHashMap**：
  - キーの挿入順序またはアクセス順序を保持する必要がある場合
  - キーによる値の検索が高速である必要がある場合
  - キーの重複を許可しない場合

- **TreeMap**：
  - キーをソートする必要がある場合
  - 範囲検索が必要な場合
  - キーの重複を許可しない場合

### パフォーマンスに関する考察

コレクションのパフォーマンスは、データ量や操作の種類によって大きく変わります。以下の点に注意することで、パフォーマンスを向上させることができます：

1. **適切な初期容量の設定**：
   - `ArrayList`や`HashMap`などのコレクションでは、初期容量を適切に設定することで、再割り当てのオーバーヘッドを減らすことができます。
   - 例：`new ArrayList<>(10000)`、`new HashMap<>(10000, 0.75f)`

2. **不要なボクシング/アンボクシングの回避**：
   - プリミティブ型の値をコレクションに格納する場合、ボクシング/アンボクシングのオーバーヘッドが発生します。
   - 大量のデータを扱う場合は、専用のコレクション（例：Trove、Eclipse Collections）の使用を検討してください。

3. **イテレータの再利用**：
   - コレクションを繰り返し走査する場合、イテレータを再利用することでオブジェクトの生成を減らすことができます。

4. **並列処理の活用**：
   - 大量のデータを処理する場合、`parallelStream()`などを使用して並列処理を行うことで、パフォーマンスを向上させることができます。
   - ただし、小さなデータセットでは、並列処理のオーバーヘッドが大きくなる場合があります。

### hashCode()とequals()の実装

`HashSet`や`HashMap`などのハッシュベースのコレクションでは、`hashCode()`と`equals()`メソッドが正しく実装されていることが重要です。以下の点に注意して実装してください：

1. **equals()メソッドのルール**：
   - 反射性：`x.equals(x)`は`true`
   - 対称性：`x.equals(y)`が`true`なら`y.equals(x)`も`true`
   - 推移性：`x.equals(y)`と`y.equals(z)`が`true`なら`x.equals(z)`も`true`
   - 一貫性：オブジェクトが変更されない限り、`equals()`の結果は一貫している
   - null参照：`x.equals(null)`は`false`

2. **hashCode()メソッドのルール**：
   - 一貫性：オブジェクトが変更されない限り、`hashCode()`の結果は一貫している
   - equals()との整合性：`x.equals(y)`が`true`なら`x.hashCode() == y.hashCode()`
   - 分散性：異なるオブジェクトのハッシュコードはできるだけ均等に分散されるべき

3. **実装例**：
```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;
    Person person = (Person) obj;
    return age == person.age && Objects.equals(name, person.name);
}

@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

## 実務での活用シーン

### 1. データの一時保管と処理

業務アプリケーションでは、データベースから取得したデータを一時的にメモリ上に保管し、処理することがよくあります。この場合、適切なコレクションを選択することで、処理効率を向上させることができます。

```java
// 顧客データの一時保管と処理
public List<Customer> processCustomerData(List<Customer> customers) {
    // 処理結果を格納するリスト
    List<Customer> processedCustomers = new ArrayList<>(customers.size());
    
    // 顧客IDをキー、顧客データを値とするマップ
    Map<String, Customer> customerMap = new HashMap<>();
    
    // 顧客データをマップに格納
    for (Customer customer : customers) {
        customerMap.put(customer.getId(), customer);
    }
    
    // 特定の条件に一致する顧客を処理
    for (Customer customer : customers) {
        if (customer.getStatus().equals("ACTIVE") && customer.getAge() >= 20) {
            // 顧客データを処理
            customer.setProcessed(true);
            customer.setLastProcessedDate(new Date());
            
            // 処理結果を追加
            processedCustomers.add(customer);
        }
    }
    
    return processedCustomers;
}
```

### 2. 重複データの排除

業務データには重複が含まれることがあります。このような場合、`Set`を使用して重複を排除することができます。

```java
// メールアドレスの重複を排除
public Set<String> removeDuplicateEmails(List<String> emails) {
    // HashSetを使用して重複を排除
    Set<String> uniqueEmails = new HashSet<>(emails);
    
    // 重複を排除した結果を返す
    return uniqueEmails;
}

// 顧客データの重複を排除（カスタムクラスの場合）
public Set<Customer> removeDuplicateCustomers(List<Customer> customers) {
    // HashSetを使用して重複を排除（hashCode()とequals()が正しく実装されている必要がある）
    Set<Customer> uniqueCustomers = new HashSet<>(customers);
    
    // 重複を排除した結果を返す
    return uniqueCustomers;
}
```

### 3. 設定値やマスタデータの管理

アプリケーションの設定値やマスタデータを管理する場合、`Map`を使用することで、キーによる高速な検索が可能になります。

```java
// 都道府県コードと都道府県名のマッピング
public Map<String, String> getPrefectureMaster() {
    Map<String, String> prefectures = new HashMap<>();
    
    // 都道府県コードと都道府県名を追加
    prefectures.put("01", "北海道");
    prefectures.put("02", "青森県");
    prefectures.put("03", "岩手県");
    // ... 他の都道府県も追加
    
    return prefectures;
}

// 設定値の管理
public class ConfigManager {
    private Map<String, String> configs = new HashMap<>();
    
    public void loadConfigs() {
        // 設定ファイルから設定値を読み込む
        // ...
        
        // 設定値をマップに格納
        configs.put("app.name", "SampleApp");
        configs.put("app.version", "1.0.0");
        configs.put("db.url", "jdbc:mysql://localhost:3306/mydb");
        configs.put("db.user", "user");
        // ...
    }
    
    public String getConfig(String key) {
        return configs.get(key);
    }
    
    public String getConfig(String key, String defaultValue) {
        return configs.getOrDefault(key, defaultValue);
    }
}
```

### 4. キャッシュの実装

頻繁にアクセスされるデータをキャッシュすることで、パフォーマンスを向上させることができます。`LinkedHashMap`を使用して、LRU（Least Recently Used）キャッシュを実装することができます。

```java
// LRUキャッシュの実装
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);  // アクセス順序モードを有効化
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;  // キャッシュが最大容量を超えた場合、最も古いエントリを削除
    }
}

// LRUキャッシュの使用例
public class ProductService {
    private LRUCache<String, Product> productCache = new LRUCache<>(100);  // 最大100件のキャッシュ
    
    public Product getProduct(String productId) {
        // キャッシュから商品を取得
        Product product = productCache.get(productId);
        
        if (product == null) {
            // キャッシュにない場合はデータベースから取得
            product = productRepository.findById(productId);
            
            if (product != null) {
                // キャッシュに追加
                productCache.put(productId, product);
            }
        }
        
        return product;
    }
}
```

### 5. データのグルーピングと集計

業務データをグループ化して集計する場合、`Map`を使用することで効率的に処理することができます。

```java
// 部署別の社員数を集計
public Map<String, Integer> countEmployeesByDepartment(List<Employee> employees) {
    Map<String, Integer> departmentCounts = new HashMap<>();
    
    for (Employee employee : employees) {
        String department = employee.getDepartment();
        
        // 部署ごとにカウントを増やす
        departmentCounts.put(department, departmentCounts.getOrDefault(department, 0) + 1);
    }
    
    return departmentCounts;
}

// 部署別の平均給与を計算
public Map<String, Double> calculateAverageSalaryByDepartment(List<Employee> employees) {
    // 部署別の給与合計と社員数を管理するマップ
    Map<String, Integer> departmentTotalSalaries = new HashMap<>();
    Map<String, Integer> departmentEmployeeCounts = new HashMap<>();
    
    for (Employee employee : employees) {
        String department = employee.getDepartment();
        int salary = employee.getSalary();
        
        // 部署ごとに給与合計を更新
        departmentTotalSalaries.put(department, departmentTotalSalaries.getOrDefault(department, 0) + salary);
        
        // 部署ごとに社員数を更新
        departmentEmployeeCounts.put(department, departmentEmployeeCounts.getOrDefault(department, 0) + 1);
    }
    
    // 部署別の平均給与を計算
    Map<String, Double> departmentAverageSalaries = new HashMap<>();
    
    for (String department : departmentTotalSalaries.keySet()) {
        int totalSalary = departmentTotalSalaries.get(department);
        int employeeCount = departmentEmployeeCounts.get(department);
        
        double averageSalary = (double) totalSalary / employeeCount;
        departmentAverageSalaries.put(department, averageSalary);
    }
    
    return departmentAverageSalaries;
}
```

これらの実務での活用シーンを参考に、コレクションフレームワークの重要性と効果的な使い方を理解してください。

## 解答例

<a href="answers/day2">解答例はこちら</a>