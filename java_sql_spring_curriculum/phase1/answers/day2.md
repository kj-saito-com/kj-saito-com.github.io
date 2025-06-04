---
layout: default
title: "Day 2: コレクションフレームワークの効果的な活用 - 解答例"
---
# Day 2: コレクションフレームワークの効果的な活用 - 解答例

## コア演習（必須）

### 演習1: リスト操作の基本と応用（難易度：基本）

#### 解答コード

```java
package collections.list;

import java.util.ArrayList;
import java.util.Iterator;
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
        // Listのaddメソッドを使用して社員情報を追加
        this.employees.add(employee);
    }

    /**
     * 名前で社員情報を検索します。
     *
     * @param name 検索する社員の名前（部分一致）
     * @return 見つかった社員情報のリスト（見つからない場合は空のリスト）
     */
    public List<Employee> findEmployeesByName(String name) {
        // 結果を格納するための新しいArrayListを作成
        List<Employee> foundEmployees = new ArrayList<>();
        // employeesリストをループで回し、名前に指定された文字列を含む社員を結果リストに追加
        for (Employee emp : this.employees) {
            if (emp.getName().contains(name)) {
                foundEmployees.add(emp);
            }
        }
        return foundEmployees;
    }

    /**
     * 社員IDで社員情報を検索します。
     *
     * @param id 検索する社員のID
     * @return 見つかった社員情報（見つからない場合はnull）
     */
    public Employee findEmployeeById(int id) {
        // employeesリストをループで回し、IDが一致する社員を返す
        for (Employee emp : this.employees) {
            if (emp.getId() == id) {
                return emp;
            }
        }
        // 見つからない場合はnullを返す
        return null;
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
        // findEmployeeByIdメソッドを使用して社員を検索
        Employee emp = findEmployeeById(id);
        if (emp != null) {
            // 見つかった場合は、nullでない引数の値で社員情報を更新
            if (name != null) {
                emp.setName(name);
            }
            if (department != null) {
                emp.setDepartment(department);
            }
            if (salary > 0) {
                emp.setSalary(salary);
            }
            return true; // 更新成功
        }
        return false; // 更新失敗（社員が見つからない）
    }

    /**
     * 社員情報を削除します。
     *
     * @param id 削除する社員のID
     * @return 削除が成功したかどうか
     */
    public boolean removeEmployee(int id) {
        // Iteratorを使用して安全に削除
        Iterator<Employee> iterator = this.employees.iterator();
        while (iterator.hasNext()) {
            Employee emp = iterator.next();
            if (emp.getId() == id) {
                iterator.remove(); // 見つかった場合は削除
                return true; // 削除成功
            }
        }
        // // findEmployeeByIdメソッドを使用して社員を検索 (Iteratorを使わない場合)
        // Employee emp = findEmployeeById(id);
        // if (emp != null) {
        //     // 見つかった場合は、Listのremoveメソッドで削除
        //     return this.employees.remove(emp);
        // }
        return false; // 削除失敗（社員が見つからない）
    }

    /**
     * 全社員情報を表示します。
     */
    public void displayAllEmployees() {
        // employeesリストをループで回し、各社員情報を表示
        if (employees.isEmpty()) {
            System.out.println("社員情報はありません。");
            return;
        }
        for (Employee emp : this.employees) {
            System.out.println(emp);
        }
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
        if (foundEmployees.isEmpty()) {
            System.out.println("該当する社員は見つかりませんでした。");
        } else {
            for (Employee emp : foundEmployees) {
                System.out.println(emp);
            }
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

#### 解説
1.  **`EmployeeManager`クラス**: 社員情報を管理するためのクラスです。内部で`ArrayList`を使用して社員オブジェクトを保持します。
2.  **`addEmployee`**: `ArrayList`の`add`メソッドを使用して、リストの末尾に社員オブジェクトを追加します。
3.  **`findEmployeesByName`**: 拡張for文でリストを走査し、`String`クラスの`contains`メソッドを使用して名前に指定された文字列が含まれる社員を検索し、新しいリストに追加して返します。
4.  **`findEmployeeById`**: 拡張for文でリストを走査し、IDが一致する最初の社員オブジェクトを返します。見つからない場合は`null`を返します。
5.  **`updateEmployee`**: `findEmployeeById`で社員を検索し、見つかった場合に引数で渡された情報（nullや0でない場合）で更新します。
6.  **`removeEmployee`**: `Iterator`を使用してリストを走査し、IDが一致する社員を見つけて削除します。ループ中にリストから要素を安全に削除するために`Iterator`を使用することが推奨されます。（コメントアウトされたコードは`remove(Object)`を使用する別解です）
7.  **`displayAllEmployees`**: 拡張for文でリストを走査し、各社員オブジェクトの`toString`メソッドの結果を表示します。
8.  **`main`メソッド**: `EmployeeManager`の各メソッドを呼び出して動作を確認します。




### 演習2: セットとマップの活用（難易度：応用）

#### 解答コード

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
        // categoriesはTreeSetで初期化（ソート済みのセット）
        this.categories = new TreeSet<>();
        // productsはHashMapで初期化
        this.products = new HashMap<>();
        // productsByCategoryはHashMapで初期化
        this.productsByCategory = new HashMap<>();
    }
    
    /**
     * 商品を追加します。
     * 
     * @param product 追加する商品
     * @return 追加が成功したかどうか（既に同じ商品コードが存在する場合はfalse）
     */
    public boolean addProduct(Product product) {
        // 既に同じ商品コードが存在する場合はfalseを返す
        if (products.containsKey(product.getCode())) {
            return false;
        }
        
        // 商品を追加する際に、カテゴリをcategoriesに追加
        categories.add(product.getCategory());
        
        // 商品をproductsに追加
        products.put(product.getCode(), product);
        
        // 商品をproductsByCategoryの対応するリストに追加
        String category = product.getCategory();
        if (!productsByCategory.containsKey(category)) {
            productsByCategory.put(category, new ArrayList<>());
        }
        productsByCategory.get(category).add(product);
        
        return true;
    }
    
    /**
     * 商品コードで商品を検索します。
     * 
     * @param code 検索する商品コード
     * @return 見つかった商品（見つからない場合はnull）
     */
    public Product findProductByCode(String code) {
        // Mapのgetメソッドを使用して商品を検索
        return products.get(code);
    }
    
    /**
     * すべての商品カテゴリを取得します。
     * 
     * @return ソート済みの商品カテゴリのセット
     */
    public Set<String> getAllCategories() {
        // categoriesをそのまま返す
        return categories;
    }
    
    /**
     * 指定されたカテゴリの商品一覧を取得します。
     * 
     * @param category 取得するカテゴリ
     * @return カテゴリに属する商品のリスト（カテゴリが存在しない場合は空のリスト）
     */
    public List<Product> getProductsByCategory(String category) {
        // productsByCategoryから指定されたカテゴリの商品リストを取得
        // カテゴリが存在しない場合は空のリストを返す
        return productsByCategory.getOrDefault(category, new ArrayList<>());
    }
    
    /**
     * 指定された価格範囲の商品を検索します。
     * 
     * @param minPrice 最小価格（この価格以上）
     * @param maxPrice 最大価格（この価格以下）
     * @return 価格範囲内の商品のリスト
     */
    public List<Product> findProductsByPriceRange(int minPrice, int maxPrice) {
        // 新しいArrayListを作成して結果を格納
        List<Product> result = new ArrayList<>();
        
        // productsの値（商品）をループで回し、価格範囲内の商品を結果リストに追加
        for (Product product : products.values()) {
            int price = product.getPrice();
            if (price >= minPrice && price <= maxPrice) {
                result.add(product);
            }
        }
        
        return result;
    }
    
    /**
     * すべての商品情報を表示します。
     */
    public void displayAllProducts() {
        // productsの値（商品）をループで回し、各商品情報を表示
        if (products.isEmpty()) {
            System.out.println("商品情報はありません。");
            return;
        }
        
        for (Product product : products.values()) {
            System.out.println(product);
        }
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

#### 解説
1. **`ProductManager`クラス**: 商品情報を管理するためのクラスです。内部で以下のコレクションを使用します：
   - `TreeSet<String> categories`: 商品カテゴリを格納するソート済みセット
   - `HashMap<String, Product> products`: 商品コードをキーとする商品マップ
   - `HashMap<String, List<Product>> productsByCategory`: カテゴリ別の商品リスト

2. **`addProduct`**: 
   - 既に同じ商品コードが存在するかチェックし、存在する場合は`false`を返します。
   - 商品のカテゴリを`categories`に追加します（`TreeSet`は重複を許可しないため、自動的に一意のカテゴリのみが保持されます）。
   - 商品を`products`マップに追加します（商品コードをキーとして）。
   - 商品を`productsByCategory`マップの対応するリストに追加します。カテゴリが存在しない場合は新しいリストを作成します。

3. **`findProductByCode`**: `Map`の`get`メソッドを使用して、指定された商品コードに対応する商品を返します。

4. **`getAllCategories`**: `categories`セットをそのまま返します。`TreeSet`を使用しているため、カテゴリはソートされた状態で返されます。

5. **`getProductsByCategory`**: `productsByCategory`マップから指定されたカテゴリの商品リストを取得します。カテゴリが存在しない場合は空のリストを返します。

6. **`findProductsByPriceRange`**: 新しい`ArrayList`を作成し、`products`マップの値（商品）をループで回して、価格範囲内の商品を結果リストに追加して返します。

7. **`displayAllProducts`**: `products`マップの値（商品）をループで回し、各商品情報を表示します。

8. **`main`メソッド**: `ProductManager`の各メソッドを呼び出して動作を確認します。

この実装では、`Set`と`Map`を効果的に組み合わせて使用しています。`TreeSet`を使用してカテゴリをソートし、`HashMap`を使用して商品コードによる高速な検索を実現しています。また、`productsByCategory`マップを使用してカテゴリ別の商品リストを効率的に管理しています。

### 演習3: コレクション選択の実践（難易度：応用）

#### 解答コード

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
        
        // HashSetを使用してユーザーIDの一意性を保証
        Set<String> userIds = new HashSet<>();
        
        // いくつかのユーザーIDを追加
        System.out.println("ユーザーID 'user001' を追加: " + userIds.add("user001"));
        System.out.println("ユーザーID 'user002' を追加: " + userIds.add("user002"));
        System.out.println("ユーザーID 'user003' を追加: " + userIds.add("user003"));
        
        // 重複するユーザーIDを追加しようとする
        System.out.println("ユーザーID 'user001' を再度追加: " + userIds.add("user001"));
        
        // 現在のユーザーID一覧を表示
        System.out.println("現在のユーザーID一覧: " + userIds);
        System.out.println("ユーザーID数: " + userIds.size());
        
        // 選択理由の説明
        System.out.println("\n【選択理由】");
        System.out.println("HashSetを選択しました。理由は以下の通りです：");
        System.out.println("1. HashSetは重複を許可しないため、ユーザーIDの一意性を自動的に保証します。");
        System.out.println("2. 追加、削除、検索の操作が平均的にO(1)の時間計算量で高速です。");
        System.out.println("3. ユーザーIDの順序が重要でない場合、HashSetは最も効率的な選択肢です。");
        System.out.println("4. もし順序も重要な場合は、LinkedHashSetを使用することで挿入順序を保持できます。");
    }
    
    /**
     * 最近アクセスした10件のドキュメントを履歴として保持するコレクションを実装します。
     */
    public static void recentDocuments() {
        System.out.println("\n===== 最近アクセスした10件のドキュメント =====");
        
        // LinkedHashMapを拡張したLRUキャッシュを実装
        class RecentDocumentsCache extends LinkedHashMap<String, String> {
            private static final int MAX_ENTRIES = 10;
            
            public RecentDocumentsCache() {
                super(16, 0.75f, true); // アクセス順序モードを有効化
            }
            
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, String> eldest) {
                return size() > MAX_ENTRIES; // サイズが10を超えたら最も古いエントリを削除
            }
        }
        
        RecentDocumentsCache recentDocs = new RecentDocumentsCache();
        
        // いくつかのドキュメントにアクセス
        System.out.println("ドキュメント1～12にアクセス:");
        for (int i = 1; i <= 12; i++) {
            String docId = "doc" + i;
            String docName = "ドキュメント" + i;
            recentDocs.put(docId, docName);
            System.out.println("  " + docName + " にアクセスしました");
        }
        
        // 現在の履歴を表示（最新の10件のみ）
        System.out.println("\n現在の履歴（最新の10件）:");
        for (Map.Entry<String, String> entry : recentDocs.entrySet()) {
            System.out.println("  " + entry.getKey() + ": " + entry.getValue());
        }
        
        // doc5に再度アクセスして順序を変更
        System.out.println("\nドキュメント5に再度アクセス:");
        recentDocs.get("doc5");
        
        // 更新された履歴を表示
        System.out.println("\n更新後の履歴（doc5が最新になっているはず）:");
        for (Map.Entry<String, String> entry : recentDocs.entrySet()) {
            System.out.println("  " + entry.getKey() + ": " + entry.getValue());
        }
        
        // 選択理由の説明
        System.out.println("\n【選択理由】");
        System.out.println("LinkedHashMapを選択しました。理由は以下の通りです：");
        System.out.println("1. LinkedHashMapはアクセス順序モードを有効にすることで、最近アクセスした要素を追跡できます。");
        System.out.println("2. removeEldestEntryメソッドをオーバーライドすることで、サイズを制限し最も古いエントリを自動的に削除できます。");
        System.out.println("3. キーによる高速なアクセス（O(1)）が可能です。");
        System.out.println("4. 挿入順序またはアクセス順序を保持できるため、履歴の管理に最適です。");
    }
    
    /**
     * 単語の出現回数をカウントするコレクションを実装します。
     */
    public static void wordFrequencyCounter() {
        System.out.println("\n===== 単語の出現回数をカウント =====");
        
        // サンプルテキスト
        String text = "Java is a programming language. Java is widely used. Programming in Java is fun.";
        System.out.println("テキスト: " + text);
        
        // HashMapを使用して単語の出現回数をカウント
        Map<String, Integer> wordFrequency = new HashMap<>();
        
        // テキストを単語に分割
        String[] words = text.toLowerCase().replaceAll("[^a-zA-Z ]", "").split("\\s+");
        
        // 各単語の出現回数をカウント
        for (String word : words) {
            // getOrDefaultを使用して、既存の値を取得するか、存在しない場合はデフォルト値（0）を使用
            wordFrequency.put(word, wordFrequency.getOrDefault(word, 0) + 1);
        }
        
        // 結果を表示
        System.out.println("\n単語の出現回数:");
        for (Map.Entry<String, Integer> entry : wordFrequency.entrySet()) {
            System.out.println("  " + entry.getKey() + ": " + entry.getValue() + "回");
        }
        
        // 出現回数でソートして表示
        System.out.println("\n出現回数の多い順:");
        wordFrequency.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .forEach(entry -> System.out.println("  " + entry.getKey() + ": " + entry.getValue() + "回"));
        
        // 選択理由の説明
        System.out.println("\n【選択理由】");
        System.out.println("HashMapを選択しました。理由は以下の通りです：");
        System.out.println("1. 単語（キー）と出現回数（値）のペアを効率的に格納できます。");
        System.out.println("2. キーによる検索、更新が平均的にO(1)の時間計算量で高速です。");
        System.out.println("3. getOrDefaultメソッドを使用することで、コードを簡潔に保ちながら出現回数を更新できます。");
        System.out.println("4. 単語の順序が重要でない場合、HashMapは最も効率的な選択肢です。");
        System.out.println("5. 出現回数でソートする場合は、Streamを使用して後処理できます。");
    }
    
    /**
     * 優先度付きのタスクキューを実装します。
     */
    public static void priorityTaskQueue() {
        System.out.println("\n===== 優先度付きのタスクキュー =====");
        
        // タスククラスの定義
        class Task implements Comparable<Task> {
            private String name;
            private int priority; // 優先度（数値が小さいほど優先度が高い）
            
            public Task(String name, int priority) {
                this.name = name;
                this.priority = priority;
            }
            
            public String getName() { return name; }
            public int getPriority() { return priority; }
            
            @Override
            public int compareTo(Task other) {
                // 優先度で比較（数値が小さいほど優先度が高い）
                return Integer.compare(this.priority, other.priority);
            }
            
            @Override
            public String toString() {
                return "Task [name=" + name + ", priority=" + priority + "]";
            }
        }
        
        // PriorityQueueを使用して優先度付きのタスクキューを実装
        Queue<Task> taskQueue = new PriorityQueue<>();
        
        // いくつかのタスクを異なる優先度で追加
        System.out.println("タスクを追加:");
        taskQueue.add(new Task("バグ修正", 1)); // 最高優先度
        System.out.println("  バグ修正（優先度: 1）を追加");
        taskQueue.add(new Task("新機能開発", 3));
        System.out.println("  新機能開発（優先度: 3）を追加");
        taskQueue.add(new Task("ドキュメント作成", 2));
        System.out.println("  ドキュメント作成（優先度: 2）を追加");
        taskQueue.add(new Task("リファクタリング", 4));
        System.out.println("  リファクタリング（優先度: 4）を追加");
        
        // タスクを取り出し、優先度順になっていることを確認
        System.out.println("\nタスクを優先度順に取り出し:");
        while (!taskQueue.isEmpty()) {
            Task task = taskQueue.poll();
            System.out.println("  " + task);
        }
        
        // 選択理由の説明
        System.out.println("\n【選択理由】");
        System.out.println("PriorityQueueを選択しました。理由は以下の通りです：");
        System.out.println("1. PriorityQueueは要素を優先度に基づいて自動的にソートします。");
        System.out.println("2. 最も優先度の高い要素を効率的に取り出すことができます（O(log n)）。");
        System.out.println("3. Comparableインターフェースを実装することで、カスタムの優先度を定義できます。");
        System.out.println("4. タスクスケジューリングや優先度付きの処理キューに最適です。");
    }
    
    public static void main(String[] args) {
        uniqueUserIds();
        recentDocuments();
        wordFrequencyCounter();
        priorityTaskQueue();
    }
}
```

#### 解説
この演習では、異なるシナリオに対して最適なコレクションを選択し、その理由を説明しています。

1. **ユーザーIDの一意性を保証する場合**:
   - `HashSet`を選択しています。
   - `HashSet`は重複を許可せず、追加・検索・削除の操作が平均的にO(1)の時間計算量で高速です。
   - ユーザーIDの一意性を自動的に保証するため、このユースケースに最適です。

2. **最近アクセスした10件のドキュメントを履歴として保持する場合**:
   - `LinkedHashMap`を拡張したカスタムクラスを実装しています。
   - アクセス順序モードを有効にし、`removeEldestEntry`メソッドをオーバーライドしてサイズを制限しています。
   - これにより、最近アクセスしたドキュメントを効率的に管理でき、最も古いエントリが自動的に削除されます。

3. **単語の出現回数をカウントする場合**:
   - `HashMap`を選択しています。
   - 単語をキー、出現回数を値として格納し、`getOrDefault`メソッドを使用して効率的にカウントを更新しています。
   - キーによる高速な検索・更新が可能で、単語の出現回数のカウントに最適です。

4. **優先度付きのタスクキューを実装する場合**:
   - `PriorityQueue`を選択しています。
   - `Comparable`インターフェースを実装したタスククラスを定義し、優先度に基づいて自動的にソートされるようにしています。
   - 最も優先度の高いタスクを効率的に取り出すことができ、タスクスケジューリングに最適です。

各シナリオで、コレクションの選択理由を詳細に説明し、実際の動作を確認するためのコードを実装しています。これにより、適切なコレクションを選択することの重要性と、各コレクションの特性を理解することができます。

## 発展演習（オプション）

### 発展演習1: パフォーマンス比較（難易度：発展）

#### 解答コード

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
        
        // LinkedList
        startTime = System.currentTimeMillis();
        List<Integer> linkedList = new LinkedList<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            linkedList.add(i);
        }
        endTime = System.currentTimeMillis();
        System.out.println("LinkedList: " + (endTime - startTime) + "ms");
        
        // HashSet
        startTime = System.currentTimeMillis();
        Set<Integer> hashSet = new HashSet<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            hashSet.add(i);
        }
        endTime = System.currentTimeMillis();
        System.out.println("HashSet: " + (endTime - startTime) + "ms");
        
        // TreeSet
        startTime = System.currentTimeMillis();
        Set<Integer> treeSet = new TreeSet<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            treeSet.add(i);
        }
        endTime = System.currentTimeMillis();
        System.out.println("TreeSet: " + (endTime - startTime) + "ms");
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
        
        // LinkedList
        startTime = System.currentTimeMillis();
        found = linkedList.contains(NUM_ELEMENTS / 2);
        endTime = System.currentTimeMillis();
        System.out.println("LinkedList: " + (endTime - startTime) + "ms, 結果: " + found);
        
        // HashSet
        startTime = System.currentTimeMillis();
        found = hashSet.contains(NUM_ELEMENTS / 2);
        endTime = System.currentTimeMillis();
        System.out.println("HashSet: " + (endTime - startTime) + "ms, 結果: " + found);
        
        // TreeSet
        startTime = System.currentTimeMillis();
        found = treeSet.contains(NUM_ELEMENTS / 2);
        endTime = System.currentTimeMillis();
        System.out.println("TreeSet: " + (endTime - startTime) + "ms, 結果: " + found);
        
        // 存在しない要素の検索
        System.out.println("--- 存在しない要素の検索 ---");
        
        // ArrayList
        startTime = System.currentTimeMillis();
        found = arrayList.contains(NUM_ELEMENTS + 1);
        endTime = System.currentTimeMillis();
        System.out.println("ArrayList: " + (endTime - startTime) + "ms, 結果: " + found);
        
        // LinkedList
        startTime = System.currentTimeMillis();
        found = linkedList.contains(NUM_ELEMENTS + 1);
        endTime = System.currentTimeMillis();
        System.out.println("LinkedList: " + (endTime - startTime) + "ms, 結果: " + found);
        
        // HashSet
        startTime = System.currentTimeMillis();
        found = hashSet.contains(NUM_ELEMENTS + 1);
        endTime = System.currentTimeMillis();
        System.out.println("HashSet: " + (endTime - startTime) + "ms, 結果: " + found);
        
        // TreeSet
        startTime = System.currentTimeMillis();
        found = treeSet.contains(NUM_ELEMENTS + 1);
        endTime = System.currentTimeMillis();
        System.out.println("TreeSet: " + (endTime - startTime) + "ms, 結果: " + found);
    }
    
    /**
     * 要素の削除のパフォーマンスを計測します。
     */
    public static void testRemovePerformance() {
        System.out.println("\n===== 要素の削除のパフォーマンス比較 =====");
        
        // 先頭要素の削除
        System.out.println("--- 先頭要素の削除 ---");
        
        // ArrayList
        List<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            arrayList.add(i);
        }
        long startTime = System.currentTimeMillis();
        arrayList.remove(0);
        long endTime = System.currentTimeMillis();
        System.out.println("ArrayList: " + (endTime - startTime) + "ms");
        
        // LinkedList
        List<Integer> linkedList = new LinkedList<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            linkedList.add(i);
        }
        startTime = System.currentTimeMillis();
        linkedList.remove(0);
        endTime = System.currentTimeMillis();
        System.out.println("LinkedList: " + (endTime - startTime) + "ms");
        
        // 中間要素の削除
        System.out.println("--- 中間要素の削除 ---");
        
        // ArrayList
        arrayList = new ArrayList<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            arrayList.add(i);
        }
        startTime = System.currentTimeMillis();
        arrayList.remove(NUM_ELEMENTS / 2);
        endTime = System.currentTimeMillis();
        System.out.println("ArrayList: " + (endTime - startTime) + "ms");
        
        // LinkedList
        linkedList = new LinkedList<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            linkedList.add(i);
        }
        startTime = System.currentTimeMillis();
        linkedList.remove(NUM_ELEMENTS / 2);
        endTime = System.currentTimeMillis();
        System.out.println("LinkedList: " + (endTime - startTime) + "ms");
        
        // 末尾要素の削除
        System.out.println("--- 末尾要素の削除 ---");
        
        // ArrayList
        arrayList = new ArrayList<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            arrayList.add(i);
        }
        startTime = System.currentTimeMillis();
        arrayList.remove(arrayList.size() - 1);
        endTime = System.currentTimeMillis();
        System.out.println("ArrayList: " + (endTime - startTime) + "ms");
        
        // LinkedList
        linkedList = new LinkedList<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            linkedList.add(i);
        }
        startTime = System.currentTimeMillis();
        linkedList.remove(linkedList.size() - 1);
        endTime = System.currentTimeMillis();
        System.out.println("LinkedList: " + (endTime - startTime) + "ms");
        
        // 値による削除
        System.out.println("--- 値による削除 ---");
        
        // HashSet
        Set<Integer> hashSet = new HashSet<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            hashSet.add(i);
        }
        startTime = System.currentTimeMillis();
        hashSet.remove(NUM_ELEMENTS / 2);
        endTime = System.currentTimeMillis();
        System.out.println("HashSet: " + (endTime - startTime) + "ms");
        
        // TreeSet
        Set<Integer> treeSet = new TreeSet<>();
        for (int i = 0; i < NUM_ELEMENTS; i++) {
            treeSet.add(i);
        }
        startTime = System.currentTimeMillis();
        treeSet.remove(NUM_ELEMENTS / 2);
        endTime = System.currentTimeMillis();
        System.out.println("TreeSet: " + (endTime - startTime) + "ms");
    }
    
    /**
     * 要素の走査のパフォーマンスを計測します。
     */
    public static void testIterationPerformance() {
        System.out.println("\n===== 要素の走査のパフォーマンス比較 =====");
        
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
        
        // ArrayList
        long startTime = System.currentTimeMillis();
        int sum = 0;
        for (Integer value : arrayList) {
            sum += value;
        }
        long endTime = System.currentTimeMillis();
        System.out.println("ArrayList: " + (endTime - startTime) + "ms, 合計: " + sum);
        
        // LinkedList
        startTime = System.currentTimeMillis();
        sum = 0;
        for (Integer value : linkedList) {
            sum += value;
        }
        endTime = System.currentTimeMillis();
        System.out.println("LinkedList: " + (endTime - startTime) + "ms, 合計: " + sum);
        
        // HashSet
        startTime = System.currentTimeMillis();
        sum = 0;
        for (Integer value : hashSet) {
            sum += value;
        }
        endTime = System.currentTimeMillis();
        System.out.println("HashSet: " + (endTime - startTime) + "ms, 合計: " + sum);
        
        // TreeSet
        startTime = System.currentTimeMillis();
        sum = 0;
        for (Integer value : treeSet) {
            sum += value;
        }
        endTime = System.currentTimeMillis();
        System.out.println("TreeSet: " + (endTime - startTime) + "ms, 合計: " + sum);
    }
    
    public static void main(String[] args) {
        testAddPerformance();
        testSearchPerformance();
        testRemovePerformance();
        testIterationPerformance();
        
        System.out.println("\n===== 考察 =====");
        System.out.println("1. 要素の追加:");
        System.out.println("   - ArrayListは末尾への追加が高速（平均的にO(1)）ですが、先頭や中間への挿入は遅い（O(n)）です。");
        System.out.println("   - LinkedListは先頭や中間への挿入が高速（位置が分かっている場合はO(1)）ですが、位置を探すのに時間がかかります（O(n)）。");
        System.out.println("   - HashSetは追加が高速（平均的にO(1)）ですが、TreeSetは要素をソートするため遅い（O(log n)）です。");
        System.out.println();
        System.out.println("2. 要素の検索:");
        System.out.println("   - ArrayListはインデックスによるアクセスが高速（O(1)）ですが、値による検索は遅い（O(n)）です。");
        System.out.println("   - LinkedListは値による検索が遅い（O(n)）です。");
        System.out.println("   - HashSetは検索が高速（平均的にO(1)）です。");
        System.out.println("   - TreeSetは検索が比較的高速（O(log n)）です。");
        System.out.println();
        System.out.println("3. 要素の削除:");
        System.out.println("   - ArrayListは末尾からの削除が高速（O(1)）ですが、先頭や中間からの削除は遅い（O(n)）です。");
        System.out.println("   - LinkedListは位置が分かっている場合の削除が高速（O(1)）ですが、位置を探すのに時間がかかります（O(n)）。");
        System.out.println("   - HashSetは削除が高速（平均的にO(1)）です。");
        System.out.println("   - TreeSetは削除が比較的高速（O(log n)）です。");
        System.out.println();
        System.out.println("4. 要素の走査:");
        System.out.println("   - ArrayListは連続したメモリ領域を使用するため、走査が高速です。");
        System.out.println("   - LinkedListはノード間のポインタを辿るため、走査が比較的遅いです。");
        System.out.println("   - HashSetは順序が保証されないため、走査の順序は予測できません。");
        System.out.println("   - TreeSetは要素がソートされているため、走査は順序付けられています。");
        System.out.println();
        System.out.println("5. 総合的な選択基準:");
        System.out.println("   - ランダムアクセスが多い場合はArrayListが適しています。");
        System.out.println("   - 挿入/削除が多い場合はLinkedListが適しています（特に位置が分かっている場合）。");
        System.out.println("   - 重複を許可せず、検索が多い場合はHashSetが適しています。");
        System.out.println("   - 要素をソートする必要がある場合はTreeSetが適しています。");
    }
}
```

#### 解説
この発展演習では、異なるコレクション（ArrayList, LinkedList, HashSet, TreeSet）のパフォーマンスを比較しています。以下の操作について、各コレクションの実行時間を計測し、結果を比較しています：

1. **要素の追加（10万件）**:
   - `ArrayList`は末尾への追加が高速（平均的にO(1)）ですが、先頭や中間への挿入は遅い（O(n)）です。
   - `LinkedList`は先頭や中間への挿入が高速（位置が分かっている場合はO(1)）ですが、位置を探すのに時間がかかります（O(n)）。
   - `HashSet`は追加が高速（平均的にO(1)）ですが、`TreeSet`は要素をソートするため遅い（O(log n)）です。

2. **要素の検索（存在する要素と存在しない要素）**:
   - `ArrayList`はインデックスによるアクセスが高速（O(1)）ですが、値による検索は遅い（O(n)）です。
   - `LinkedList`は値による検索が遅い（O(n)）です。
   - `HashSet`は検索が高速（平均的にO(1)）です。
   - `TreeSet`は検索が比較的高速（O(log n)）です。

3. **要素の削除（先頭、中間、末尾）**:
   - `ArrayList`は末尾からの削除が高速（O(1)）ですが、先頭や中間からの削除は遅い（O(n)）です。
   - `LinkedList`は位置が分かっている場合の削除が高速（O(1)）ですが、位置を探すのに時間がかかります（O(n)）。
   - `HashSet`は削除が高速（平均的にO(1)）です。
   - `TreeSet`は削除が比較的高速（O(log n)）です。

4. **要素の走査**:
   - `ArrayList`は連続したメモリ領域を使用するため、走査が高速です。
   - `LinkedList`はノード間のポインタを辿るため、走査が比較的遅いです。
   - `HashSet`は順序が保証されないため、走査の順序は予測できません。
   - `TreeSet`は要素がソートされているため、走査は順序付けられています。

この演習を通じて、各コレクションの特性と、どのような状況でどのコレクションを選択すべきかを理解することができます。適切なコレクションを選択することで、アプリケーションのパフォーマンスを大幅に向上させることができます。

### 発展演習2: LRUキャッシュの実装（難易度：発展）

#### 解答コード

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
    private LinkedHashMap<K, V> cache;  // キャッシュの実体
    
    /**
     * コンストラクタ
     * 
     * @param capacity キャッシュの最大容量
     */
    public LRUCache(int capacity) {
        this.capacity = capacity;
        // LinkedHashMapを使用し、アクセス順序モードを有効にする
        // 第3引数のtrueはアクセス順序モードを有効にすることを意味する
        this.cache = new LinkedHashMap<K, V>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                // キャッシュが最大容量に達した場合、最も長い間使われていない項目を削除
                return size() > LRUCache.this.capacity;
            }
        };
    }
    
    /**
     * キーに対応する値を取得します。
     * 
     * @param key キー
     * @return 値（キーが存在しない場合はnull）
     */
    public V get(K key) {
        // キャッシュからキーに対応する値を取得
        // getメソッドを呼び出すと、そのキーが最近使用されたことになる
        return cache.get(key);
    }
    
    /**
     * キーと値のペアをキャッシュに追加します。
     * 
     * @param key キー
     * @param value 値
     */
    public void put(K key, V value) {
        // キャッシュにキーと値のペアを追加
        // キャッシュが最大容量に達した場合、最も長い間使われていない項目が自動的に削除される
        cache.put(key, value);
    }
    
    /**
     * キャッシュの現在のサイズを取得します。
     * 
     * @return キャッシュのサイズ
     */
    public int size() {
        // キャッシュのサイズを返す
        return cache.size();
    }
    
    /**
     * キャッシュの内容を表示します。
     */
    public void display() {
        // キャッシュの内容を表示（キーと値のペア）
        if (cache.isEmpty()) {
            System.out.println("キャッシュは空です。");
            return;
        }
        
        for (Map.Entry<K, V> entry : cache.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
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

#### 解説
この発展演習では、LRU（Least Recently Used）キャッシュを実装しています。LRUキャッシュは、最も長い間使われていない項目が削除される仕組みのキャッシュです。

1. **`LRUCache`クラス**:
   - `LinkedHashMap`を拡張して実装しています。
   - コンストラクタで`LinkedHashMap`のアクセス順序モードを有効にしています（第3引数に`true`を指定）。
   - `removeEldestEntry`メソッドをオーバーライドして、キャッシュが最大容量に達した場合に最も古い項目を削除するようにしています。

2. **`get`メソッド**:
   - キャッシュからキーに対応する値を取得します。
   - `LinkedHashMap`の`get`メソッドを呼び出すと、そのキーが最近使用されたことになり、アクセス順序が更新されます。

3. **`put`メソッド**:
   - キャッシュにキーと値のペアを追加します。
   - キャッシュが最大容量に達した場合、`removeEldestEntry`メソッドにより最も長い間使われていない項目が自動的に削除されます。

4. **`size`メソッド**:
   - キャッシュの現在のサイズを返します。

5. **`display`メソッド**:
   - キャッシュの内容（キーと値のペア）を表示します。
   - `LinkedHashMap`はアクセス順序モードが有効な場合、イテレーションの順序は最近使用された順になります。

6. **`main`メソッド**:
   - LRUキャッシュのテストを行います。
   - 初期状態、キーへのアクセス、新しい項目の追加、存在しないキーへのアクセス、既存のキーの更新などのシナリオをテストしています。

この実装では、`LinkedHashMap`のアクセス順序モードと`removeEldestEntry`メソッドを活用することで、シンプルかつ効率的なLRUキャッシュを実現しています。LRUキャッシュは、データベースのクエリ結果、ウェブページ、画像などの頻繁にアクセスされるデータをキャッシュするのに適しています。

