# Day 3: ラムダ式と匿名クラスの適切な使い分け - 解答例

## コア演習1: 匿名クラスの基本

### 解答例
```java
package lambda.anonymous;

import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Arrays;
import java.util.Comparator;

public class AnonymousClassBasics {

    /**
     * Runnableインターフェースを実装した匿名クラスを作成し、実行します。
     */
    public void demonstrateRunnableAnonymous() {
        System.out.println("=== Runnableインターフェースの匿名クラス ===");
        
        // 匿名クラスを使用してRunnableを実装
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("匿名クラスのrun()メソッドが実行されました");
            }
        };
        
        // 実行
        runnable.run();
    }
    
    /**
     * ActionListenerインターフェースを実装した匿名クラスを作成し、実行します。
     */
    public void demonstrateActionListenerAnonymous() {
        System.out.println("\n=== ActionListenerインターフェースの匿名クラス ===");
        
        // 匿名クラスを使用してActionListenerを実装
        ActionListener listener = new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("アクションが実行されました: " + e.getActionCommand());
            }
        };
        
        // 実行（実際のイベントをシミュレート）
        ActionEvent event = new ActionEvent(this, ActionEvent.ACTION_PERFORMED, "ボタンクリック");
        listener.actionPerformed(event);
    }
    
    /**
     * Comparatorインターフェースを実装した匿名クラスを作成し、使用します。
     */
    public void demonstrateComparatorAnonymous() {
        System.out.println("\n=== Comparatorインターフェースの匿名クラス ===");
        
        // 文字列の長さでソートするComparator（匿名クラス）
        Comparator<String> lengthComparator = new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return Integer.compare(s1.length(), s2.length());
            }
        };
        
        // 文字列配列を作成
        String[] words = {"apple", "banana", "cherry", "date", "elderberry"};
        
        // ソート前
        System.out.println("ソート前: " + Arrays.toString(words));
        
        // 匿名クラスのComparatorを使用してソート
        Arrays.sort(words, lengthComparator);
        
        // ソート後
        System.out.println("文字列の長さでソート後: " + Arrays.toString(words));
    }
    
    /**
     * 独自インターフェースを実装した匿名クラスを作成し、使用します。
     */
    public void demonstrateCustomInterfaceAnonymous() {
        System.out.println("\n=== 独自インターフェースの匿名クラス ===");
        
        // 匿名クラスを使用してCalculatorを実装
        Calculator adder = new Calculator() {
            @Override
            public double calculate(double a, double b) {
                return a + b;
            }
        };
        
        Calculator multiplier = new Calculator() {
            @Override
            public double calculate(double a, double b) {
                return a * b;
            }
        };
        
        // 実行
        double a = 10.0;
        double b = 5.0;
        System.out.println(a + " + " + b + " = " + adder.calculate(a, b));
        System.out.println(a + " * " + b + " = " + multiplier.calculate(a, b));
    }
    
    /**
     * 抽象クラスを継承した匿名クラスを作成し、使用します。
     */
    public void demonstrateAbstractClassAnonymous() {
        System.out.println("\n=== 抽象クラスの匿名クラス ===");
        
        // 匿名クラスを使用してAbstractShapeを実装
        AbstractShape circle = new AbstractShape() {
            @Override
            public double area() {
                double radius = 5.0;
                return Math.PI * radius * radius;
            }
            
            @Override
            public String getType() {
                return "円";
            }
        };
        
        AbstractShape rectangle = new AbstractShape() {
            @Override
            public double area() {
                double width = 4.0;
                double height = 6.0;
                return width * height;
            }
            
            @Override
            public String getType() {
                return "長方形";
            }
        };
        
        // 実行
        System.out.println(circle.getType() + "の面積: " + circle.area());
        System.out.println(rectangle.getType() + "の面積: " + rectangle.area());
    }
    
    // 独自インターフェース
    interface Calculator {
        double calculate(double a, double b);
    }
    
    // 抽象クラス
    abstract class AbstractShape {
        public abstract double area();
        public abstract String getType();
    }

    public static void main(String[] args) {
        AnonymousClassBasics demo = new AnonymousClassBasics();
        
        demo.demonstrateRunnableAnonymous();
        demo.demonstrateActionListenerAnonymous();
        demo.demonstrateComparatorAnonymous();
        demo.demonstrateCustomInterfaceAnonymous();
        demo.demonstrateAbstractClassAnonymous();
    }
}
```

## コア演習2: ラムダ式の基本

### 解答例
```java
package lambda.basics;

import java.util.Arrays;
import java.util.List;
import java.util.function.BiFunction;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

public class LambdaBasics {

    /**
     * 基本的なラムダ式の構文を示します。
     */
    public void demonstrateLambdaSyntax() {
        System.out.println("=== 基本的なラムダ式の構文 ===");
        
        // 引数なし、戻り値なし
        Runnable noArgsNoReturn = () -> System.out.println("引数なし、戻り値なしのラムダ式");
        
        // 引数1つ、戻り値なし
        Consumer<String> oneArgNoReturn = s -> System.out.println("引数: " + s);
        
        // 引数1つ、戻り値あり
        Function<String, Integer> oneArgWithReturn = s -> s.length();
        
        // 引数2つ、戻り値あり
        BiFunction<String, String, String> twoArgsWithReturn = (s1, s2) -> s1 + s2;
        
        // 引数なし、戻り値あり
        Supplier<String> noArgsWithReturn = () -> "戻り値のみのラムダ式";
        
        // 複数行のラムダ式
        BiFunction<Integer, Integer, Integer> multiLineLambda = (a, b) -> {
            int result = a + b;
            System.out.println("計算中: " + a + " + " + b);
            return result;
        };
        
        // 実行
        noArgsNoReturn.run();
        oneArgNoReturn.accept("こんにちは");
        System.out.println("文字列の長さ: " + oneArgWithReturn.apply("こんにちは"));
        System.out.println("文字列の連結: " + twoArgsWithReturn.apply("Hello, ", "World!"));
        System.out.println("生成された文字列: " + noArgsWithReturn.get());
        System.out.println("複数行の計算結果: " + multiLineLambda.apply(5, 3));
    }
    
    /**
     * 標準関数型インターフェースを使用したラムダ式の例を示します。
     */
    public void demonstrateFunctionalInterfaces() {
        System.out.println("\n=== 標準関数型インターフェース ===");
        
        // Predicate<T>: T -> boolean
        Predicate<String> isLongString = s -> s.length() > 5;
        System.out.println("「こんにちは」は長い文字列か: " + isLongString.test("こんにちは"));
        System.out.println("「Hello」は長い文字列か: " + isLongString.test("Hello"));
        
        // Consumer<T>: T -> void
        Consumer<String> printer = s -> System.out.println("出力: " + s);
        printer.accept("こんにちは、世界！");
        
        // Function<T, R>: T -> R
        Function<Integer, String> intToString = i -> "数値: " + i;
        System.out.println(intToString.apply(42));
        
        // Supplier<T>: () -> T
        Supplier<Double> randomSupplier = () -> Math.random();
        System.out.println("ランダム値: " + randomSupplier.get());
        
        // BiFunction<T, U, R>: (T, U) -> R
        BiFunction<Integer, Integer, Integer> adder = (a, b) -> a + b;
        System.out.println("5 + 3 = " + adder.apply(5, 3));
    }
    
    /**
     * ラムダ式を使用したリスト操作の例を示します。
     */
    public void demonstrateListOperations() {
        System.out.println("\n=== ラムダ式を使用したリスト操作 ===");
        
        List<String> fruits = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        System.out.println("すべての果物:");
        fruits.forEach(fruit -> System.out.println("- " + fruit));
        
        System.out.println("\n5文字以上の果物:");
        fruits.stream()
              .filter(fruit -> fruit.length() >= 5)
              .forEach(fruit -> System.out.println("- " + fruit));
        
        System.out.println("\n大文字に変換した果物:");
        fruits.stream()
              .map(fruit -> fruit.toUpperCase())
              .forEach(fruit -> System.out.println("- " + fruit));
        
        boolean anyStartWithA = fruits.stream()
                                      .anyMatch(fruit -> fruit.startsWith("a"));
        System.out.println("\n'a'で始まる果物はありますか？ " + anyStartWithA);
        
        boolean allLongerThan3 = fruits.stream()
                                       .allMatch(fruit -> fruit.length() > 3);
        System.out.println("すべての果物の長さは3より大きいですか？ " + allLongerThan3);
    }
    
    /**
     * ラムダ式でのローカル変数キャプチャの例を示します。
     */
    public void demonstrateVariableCapture() {
        System.out.println("\n=== ラムダ式でのローカル変数キャプチャ ===");
        
        // インスタンス変数のキャプチャ
        String instanceVar = "インスタンス変数";
        Runnable r1 = () -> System.out.println("キャプチャした " + instanceVar);
        r1.run();
        
        // ローカル変数のキャプチャ（実質的にfinal）
        String localVar = "ローカル変数";
        Runnable r2 = () -> System.out.println("キャプチャした " + localVar);
        r2.run();
        
        // ローカル変数のキャプチャ（明示的にfinal）
        final String finalLocalVar = "finalローカル変数";
        Runnable r3 = () -> System.out.println("キャプチャした " + finalLocalVar);
        r3.run();
        
        // 以下はコンパイルエラーになる（コメントアウト）
        // String mutableVar = "変更可能な変数";
        // Runnable r4 = () -> System.out.println(mutableVar);
        // mutableVar = "変更後"; // ラムダ式でキャプチャされた変数は変更できない
    }

    public static void main(String[] args) {
        LambdaBasics demo = new LambdaBasics();
        
        demo.demonstrateLambdaSyntax();
        demo.demonstrateFunctionalInterfaces();
        demo.demonstrateListOperations();
        demo.demonstrateVariableCapture();
    }
}
```

## コア演習3: コレクションとラムダ式

### 解答例
```java
package lambda.collections;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.function.Function;
import java.util.stream.Collectors;

public class LambdaWithCollections {

    /**
     * 従業員クラス
     */
    static class Employee {
        private String id;
        private String name;
        private String department;
        private double salary;
        
        public Employee(String id, String name, String department, double salary) {
            this.id = id;
            this.name = name;
            this.department = department;
            this.salary = salary;
        }
        
        public String getId() { return id; }
        public String getName() { return name; }
        public String getDepartment() { return department; }
        public double getSalary() { return salary; }
        
        @Override
        public String toString() {
            return "Employee{id='" + id + "', name='" + name + "', department='" + department + "', salary=" + salary + "}";
        }
    }

    /**
     * リストのフィルタリングと変換の例を示します。
     */
    public void demonstrateFilteringAndMapping() {
        System.out.println("=== リストのフィルタリングと変換 ===");
        
        // 従業員リストの作成
        List<Employee> employees = Arrays.asList(
            new Employee("E001", "山田太郎", "営業部", 350000),
            new Employee("E002", "鈴木花子", "開発部", 400000),
            new Employee("E003", "佐藤次郎", "営業部", 320000),
            new Employee("E004", "田中美咲", "人事部", 380000),
            new Employee("E005", "伊藤健太", "開発部", 450000)
        );
        
        // 開発部の従業員のみをフィルタリング
        List<Employee> devEmployees = employees.stream()
                                              .filter(emp -> emp.getDepartment().equals("開発部"))
                                              .collect(Collectors.toList());
        
        System.out.println("開発部の従業員:");
        devEmployees.forEach(emp -> System.out.println("- " + emp.getName()));
        
        // 給与が400000以上の従業員のみをフィルタリング
        List<Employee> highPaidEmployees = employees.stream()
                                                  .filter(emp -> emp.getSalary() >= 400000)
                                                  .collect(Collectors.toList());
        
        System.out.println("\n給与が400,000以上の従業員:");
        highPaidEmployees.forEach(emp -> System.out.println("- " + emp.getName() + ": " + emp.getSalary()));
        
        // 従業員の名前のみを抽出
        List<String> employeeNames = employees.stream()
                                            .map(Employee::getName)
                                            .collect(Collectors.toList());
        
        System.out.println("\n従業員の名前一覧:");
        employeeNames.forEach(name -> System.out.println("- " + name));
        
        // 部署ごとの従業員数をカウント
        Map<String, Long> departmentCounts = employees.stream()
                                                    .collect(Collectors.groupingBy(
                                                        Employee::getDepartment,
                                                        Collectors.counting()
                                                    ));
        
        System.out.println("\n部署ごとの従業員数:");
        departmentCounts.forEach((dept, count) -> System.out.println("- " + dept + ": " + count + "人"));
    }
    
    /**
     * リストのソートの例を示します。
     */
    public void demonstrateSorting() {
        System.out.println("\n=== リストのソート ===");
        
        // 従業員リストの作成
        List<Employee> employees = Arrays.asList(
            new Employee("E001", "山田太郎", "営業部", 350000),
            new Employee("E002", "鈴木花子", "開発部", 400000),
            new Employee("E003", "佐藤次郎", "営業部", 320000),
            new Employee("E004", "田中美咲", "人事部", 380000),
            new Employee("E005", "伊藤健太", "開発部", 450000)
        );
        
        // 名前でソート
        List<Employee> sortedByName = new ArrayList<>(employees);
        sortedByName.sort(Comparator.comparing(Employee::getName));
        
        System.out.println("名前でソート:");
        sortedByName.forEach(emp -> System.out.println("- " + emp.getName()));
        
        // 給与の降順でソート
        List<Employee> sortedBySalaryDesc = new ArrayList<>(employees);
        sortedBySalaryDesc.sort(Comparator.comparing(Employee::getSalary).reversed());
        
        System.out.println("\n給与の降順でソート:");
        sortedBySalaryDesc.forEach(emp -> System.out.println("- " + emp.getName() + ": " + emp.getSalary()));
        
        // 部署でソートし、同じ部署内では給与の昇順でソート
        List<Employee> sortedByDeptAndSalary = new ArrayList<>(employees);
        sortedByDeptAndSalary.sort(
            Comparator.comparing(Employee::getDepartment)
                      .thenComparing(Employee::getSalary)
        );
        
        System.out.println("\n部署と給与でソート:");
        sortedByDeptAndSalary.forEach(emp -> 
            System.out.println("- " + emp.getDepartment() + ": " + emp.getName() + " (" + emp.getSalary() + ")")
        );
    }
    
    /**
     * リストの集計操作の例を示します。
     */
    public void demonstrateAggregation() {
        System.out.println("\n=== リストの集計操作 ===");
        
        // 従業員リストの作成
        List<Employee> employees = Arrays.asList(
            new Employee("E001", "山田太郎", "営業部", 350000),
            new Employee("E002", "鈴木花子", "開発部", 400000),
            new Employee("E003", "佐藤次郎", "営業部", 320000),
            new Employee("E004", "田中美咲", "人事部", 380000),
            new Employee("E005", "伊藤健太", "開発部", 450000)
        );
        
        // 給与の合計
        double totalSalary = employees.stream()
                                    .mapToDouble(Employee::getSalary)
                                    .sum();
        
        System.out.println("給与の合計: " + totalSalary);
        
        // 給与の平均
        double averageSalary = employees.stream()
                                      .mapToDouble(Employee::getSalary)
                                      .average()
                                      .orElse(0.0);
        
        System.out.println("給与の平均: " + averageSalary);
        
        // 最高給与の従業員
        Optional<Employee> highestPaidEmployee = employees.stream()
                                                        .max(Comparator.comparing(Employee::getSalary));
        
        highestPaidEmployee.ifPresent(emp -> 
            System.out.println("最高給与の従業員: " + emp.getName() + " (" + emp.getSalary() + ")")
        );
        
        // 最低給与の従業員
        Optional<Employee> lowestPaidEmployee = employees.stream()
                                                       .min(Comparator.comparing(Employee::getSalary));
        
        lowestPaidEmployee.ifPresent(emp -> 
            System.out.println("最低給与の従業員: " + emp.getName() + " (" + emp.getSalary() + ")")
        );
        
        // 部署ごとの給与合計
        Map<String, Double> totalSalaryByDept = employees.stream()
                                                       .collect(Collectors.groupingBy(
                                                           Employee::getDepartment,
                                                           Collectors.summingDouble(Employee::getSalary)
                                                       ));
        
        System.out.println("\n部署ごとの給与合計:");
        totalSalaryByDept.forEach((dept, total) -> System.out.println("- " + dept + ": " + total));
        
        // 部署ごとの給与平均
        Map<String, Double> avgSalaryByDept = employees.stream()
                                                     .collect(Collectors.groupingBy(
                                                         Employee::getDepartment,
                                                         Collectors.averagingDouble(Employee::getSalary)
                                                     ));
        
        System.out.println("\n部署ごとの給与平均:");
        avgSalaryByDept.forEach((dept, avg) -> System.out.println("- " + dept + ": " + avg));
    }
    
    /**
     * マップ操作の例を示します。
     */
    public void demonstrateMapOperations() {
        System.out.println("\n=== マップ操作 ===");
        
        // 従業員マップの作成（キー: ID, 値: 従業員オブジェクト）
        Map<String, Employee> employeeMap = new HashMap<>();
        employeeMap.put("E001", new Employee("E001", "山田太郎", "営業部", 350000));
        employeeMap.put("E002", new Employee("E002", "鈴木花子", "開発部", 400000));
        employeeMap.put("E003", new Employee("E003", "佐藤次郎", "営業部", 320000));
        employeeMap.put("E004", new Employee("E004", "田中美咲", "人事部", 380000));
        employeeMap.put("E005", new Employee("E005", "伊藤健太", "開発部", 450000));
        
        // forEach を使用してマップの内容を表示
        System.out.println("マップの内容:");
        employeeMap.forEach((id, emp) -> 
            System.out.println("- " + id + ": " + emp.getName() + " (" + emp.getDepartment() + ")")
        );
        
        // computeIfAbsent を使用して、キーが存在しない場合に値を計算して追加
        Employee newEmployee = employeeMap.computeIfAbsent("E006", id -> 
            new Employee(id, "新入社員", "総務部", 300000)
        );
        
        System.out.println("\ncomputeIfAbsent で追加された従業員: " + newEmployee);
        
        // computeIfPresent を使用して、キーが存在する場合に値を更新
        employeeMap.computeIfPresent("E001", (id, emp) -> 
            new Employee(id, emp.getName(), emp.getDepartment(), emp.getSalary() * 1.1)
        );
        
        System.out.println("computeIfPresent で更新された従業員: " + employeeMap.get("E001"));
        
        // getOrDefault を使用して、キーが存在しない場合にデフォルト値を取得
        Employee defaultEmployee = employeeMap.getOrDefault("E999", 
            new Employee("E999", "存在しない従業員", "不明", 0)
        );
        
        System.out.println("\ngetOrDefault で取得された従業員: " + defaultEmployee);
        
        // マップのエントリをフィルタリングして新しいマップを作成
        Map<String, Employee> highPaidEmployees = employeeMap.entrySet().stream()
                                                           .filter(entry -> entry.getValue().getSalary() >= 400000)
                                                           .collect(Collectors.toMap(
                                                               Map.Entry::getKey,
                                                               Map.Entry::getValue
                                                           ));
        
        System.out.println("\n給与が400,000以上の従業員マップ:");
        highPaidEmployees.forEach((id, emp) -> 
            System.out.println("- " + id + ": " + emp.getName() + " (" + emp.getSalary() + ")")
        );
        
        // マップの値を変換して新しいマップを作成
        Map<String, String> employeeNames = employeeMap.entrySet().stream()
                                                     .collect(Collectors.toMap(
                                                         Map.Entry::getKey,
                                                         entry -> entry.getValue().getName()
                                                     ));
        
        System.out.println("\n従業員IDと名前のマップ:");
        employeeNames.forEach((id, name) -> System.out.println("- " + id + ": " + name));
        
        // IDから従業員名を取得する関数を作成
        Function<String, String> getEmployeeName = id -> {
            Employee emp = employeeMap.get(id);
            return emp != null ? emp.getName() : "不明";
        };
        
        System.out.println("\n関数を使用して従業員名を取得:");
        System.out.println("- E003: " + getEmployeeName.apply("E003"));
        System.out.println("- E999: " + getEmployeeName.apply("E999"));
    }

    public static void main(String[] args) {
        LambdaWithCollections demo = new LambdaWithCollections();
        
        demo.demonstrateFilteringAndMapping();
        demo.demonstrateSorting();
        demo.demonstrateAggregation();
        demo.demonstrateMapOperations();
    }
}
```

## 発展演習1: メソッド参照

### 解答例
```java
package lambda.methodref;

import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.function.BiFunction;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;
import java.util.stream.Collectors;

public class MethodReferenceDemo {

    /**
     * 商品クラス
     */
    static class Product {
        private String id;
        private String name;
        private String category;
        private double price;
        
        public Product(String id, String name, String category, double price) {
            this.id = id;
            this.name = name;
            this.category = category;
            this.price = price;
        }
        
        public String getId() { return id; }
        public String getName() { return name; }
        public String getCategory() { return category; }
        public double getPrice() { return price; }
        
        // インスタンスメソッド
        public boolean isExpensive() {
            return price >= 5000;
        }
        
        public String getDisplayName() {
            return name + " (" + category + ")";
        }
        
        public void printDetails() {
            System.out.println("商品ID: " + id);
            System.out.println("商品名: " + name);
            System.out.println("カテゴリ: " + category);
            System.out.println("価格: " + price);
        }
        
        // 静的メソッド
        public static int compareByPrice(Product p1, Product p2) {
            return Double.compare(p1.getPrice(), p2.getPrice());
        }
        
        public static Product createDefaultProduct() {
            return new Product("DEFAULT", "デフォルト商品", "その他", 1000);
        }
        
        @Override
        public String toString() {
            return "Product{id='" + id + "', name='" + name + "', category='" + category + "', price=" + price + "}";
        }
    }
    
    /**
     * 静的メソッド参照の例を示します。
     */
    public void demonstrateStaticMethodReference() {
        System.out.println("=== 静的メソッド参照 ===");
        
        // 商品リストの作成
        List<Product> products = Arrays.asList(
            new Product("P001", "ノートPC", "電子機器", 80000),
            new Product("P002", "スマートフォン", "電子機器", 60000),
            new Product("P003", "コーヒーメーカー", "家電", 15000),
            new Product("P004", "書籍", "書籍", 3000),
            new Product("P005", "デスク", "家具", 25000)
        );
        
        // 静的メソッド参照を使用して商品を価格でソート
        // ラムダ式: (p1, p2) -> Product.compareByPrice(p1, p2)
        // メソッド参照: Product::compareByPrice
        products.sort(Product::compareByPrice);
        
        System.out.println("価格でソートされた商品:");
        products.forEach(p -> System.out.println("- " + p.getName() + ": " + p.getPrice()));
        
        // 静的メソッド参照を使用してデフォルト商品を作成
        // ラムダ式: () -> Product.createDefaultProduct()
        // メソッド参照: Product::createDefaultProduct
        Supplier<Product> defaultProductSupplier = Product::createDefaultProduct;
        Product defaultProduct = defaultProductSupplier.get();
        
        System.out.println("\nデフォルト商品: " + defaultProduct);
        
        // 静的メソッド参照を使用して文字列を整数に変換
        // ラムダ式: s -> Integer.parseInt(s)
        // メソッド参照: Integer::parseInt
        Function<String, Integer> parser = Integer::parseInt;
        
        String[] numberStrings = {"10", "20", "30", "40", "50"};
        System.out.println("\n文字列から整数への変換:");
        for (String numberString : numberStrings) {
            System.out.println("- " + numberString + " -> " + parser.apply(numberString));
        }
        
        // 静的メソッド参照を使用して最大値を取得
        // ラムダ式: (a, b) -> Math.max(a, b)
        // メソッド参照: Math::max
        BiFunction<Integer, Integer, Integer> maxFinder = Math::max;
        
        System.out.println("\n最大値の取得:");
        System.out.println("- max(10, 20): " + maxFinder.apply(10, 20));
        System.out.println("- max(30, 15): " + maxFinder.apply(30, 15));
    }
    
    /**
     * インスタンスメソッド参照の例を示します。
     */
    public void demonstrateInstanceMethodReference() {
        System.out.println("\n=== インスタンスメソッド参照 ===");
        
        // 商品リストの作成
        List<Product> products = Arrays.asList(
            new Product("P001", "ノートPC", "電子機器", 80000),
            new Product("P002", "スマートフォン", "電子機器", 60000),
            new Product("P003", "コーヒーメーカー", "家電", 15000),
            new Product("P004", "書籍", "書籍", 3000),
            new Product("P005", "デスク", "家具", 25000)
        );
        
        // インスタンスメソッド参照を使用して高価な商品をフィルタリング
        // ラムダ式: p -> p.isExpensive()
        // メソッド参照: Product::isExpensive
        List<Product> expensiveProducts = products.stream()
                                                .filter(Product::isExpensive)
                                                .collect(Collectors.toList());
        
        System.out.println("高価な商品:");
        expensiveProducts.forEach(p -> System.out.println("- " + p.getName() + ": " + p.getPrice()));
        
        // インスタンスメソッド参照を使用して商品の表示名を取得
        // ラムダ式: p -> p.getDisplayName()
        // メソッド参照: Product::getDisplayName
        List<String> productDisplayNames = products.stream()
                                                 .map(Product::getDisplayName)
                                                 .collect(Collectors.toList());
        
        System.out.println("\n商品の表示名:");
        productDisplayNames.forEach(name -> System.out.println("- " + name));
        
        // インスタンスメソッド参照を使用して商品の詳細を表示
        // ラムダ式: p -> p.printDetails()
        // メソッド参照: Product::printDetails
        System.out.println("\n最初の商品の詳細:");
        Consumer<Product> detailsPrinter = Product::printDetails;
        detailsPrinter.accept(products.get(0));
        
        // 特定のインスタンスのメソッド参照
        System.out.println("\n特定の商品の詳細:");
        Product specificProduct = products.get(1);
        Runnable detailsPrinter2 = specificProduct::printDetails;
        detailsPrinter2.run();
    }
    
    /**
     * 任意のオブジェクトのインスタンスメソッド参照の例を示します。
     */
    public void demonstrateArbitraryObjectMethodReference() {
        System.out.println("\n=== 任意のオブジェクトのインスタンスメソッド参照 ===");
        
        // 文字列リストの作成
        List<String> strings = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        // 任意のオブジェクトのインスタンスメソッド参照を使用して文字列の長さでソート
        // ラムダ式: (s1, s2) -> s1.compareTo(s2)
        // メソッド参照: String::compareTo
        strings.sort(String::compareTo);
        
        System.out.println("アルファベット順にソートされた文字列:");
        strings.forEach(s -> System.out.println("- " + s));
        
        // 任意のオブジェクトのインスタンスメソッド参照を使用して文字列の長さでソート
        // ラムダ式: (s1, s2) -> Integer.compare(s1.length(), s2.length())
        // メソッド参照: Comparator.comparing(String::length)
        strings.sort(Comparator.comparing(String::length));
        
        System.out.println("\n長さでソートされた文字列:");
        strings.forEach(s -> System.out.println("- " + s + " (長さ: " + s.length() + ")"));
        
        // 任意のオブジェクトのインスタンスメソッド参照を使用して文字列を大文字に変換
        // ラムダ式: s -> s.toUpperCase()
        // メソッド参照: String::toUpperCase
        List<String> upperCaseStrings = strings.stream()
                                             .map(String::toUpperCase)
                                             .collect(Collectors.toList());
        
        System.out.println("\n大文字に変換された文字列:");
        upperCaseStrings.forEach(s -> System.out.println("- " + s));
        
        // 任意のオブジェクトのインスタンスメソッド参照を使用して文字列が特定の文字で始まるかをチェック
        // ラムダ式: s -> s.startsWith("a")
        // メソッド参照: 直接的な方法はないため、ラムダ式を使用
        Predicate<String> startsWithA = s -> s.startsWith("a");
        boolean anyStartWithA = strings.stream().anyMatch(startsWithA);
        
        System.out.println("\n'a'で始まる文字列はありますか？ " + anyStartWithA);
    }
    
    /**
     * コンストラクタ参照の例を示します。
     */
    public void demonstrateConstructorReference() {
        System.out.println("\n=== コンストラクタ参照 ===");
        
        // コンストラクタ参照を使用して新しい商品を作成
        // ラムダ式: (id, name, category, price) -> new Product(id, name, category, price)
        // コンストラクタ参照: Product::new
        ProductFactory factory = Product::new;
        
        Product newProduct = factory.create("P006", "テレビ", "家電", 50000);
        System.out.println("新しく作成された商品: " + newProduct);
        
        // コンストラクタ参照を使用して文字列から整数のリストを作成
        // ラムダ式: size -> new ArrayList<>(size)
        // コンストラクタ参照: ArrayList::new
        Function<Integer, List<Integer>> listFactory = ArrayList::new;
        
        List<Integer> newList = listFactory.apply(10);
        for (int i = 1; i <= 5; i++) {
            newList.add(i * 10);
        }
        
        System.out.println("\n新しく作成されたリスト: " + newList);
        
        // 商品IDから商品を作成するマップを作成
        Map<String, Product> productMap = Arrays.asList(
            new Product("P001", "ノートPC", "電子機器", 80000),
            new Product("P002", "スマートフォン", "電子機器", 60000),
            new Product("P003", "コーヒーメーカー", "家電", 15000)
        ).stream().collect(Collectors.toMap(
            Product::getId,  // キーマッパー
            Function.identity()  // 値マッパー（商品オブジェクトそのもの）
        ));
        
        System.out.println("\n商品マップ:");
        productMap.forEach((id, product) -> System.out.println("- " + id + ": " + product.getName()));
    }
    
    // 商品ファクトリーインターフェース
    @FunctionalInterface
    interface ProductFactory {
        Product create(String id, String name, String category, double price);
    }
    
    // ArrayListのカスタム実装（デモ用）
    static class ArrayList<T> implements List<T> {
        private final java.util.ArrayList<T> delegate;
        
        public ArrayList() {
            this.delegate = new java.util.ArrayList<>();
        }
        
        public ArrayList(int initialCapacity) {
            this.delegate = new java.util.ArrayList<>(initialCapacity);
            System.out.println("初期容量 " + initialCapacity + " のArrayListを作成しました");
        }
        
        // List<T>インターフェースのメソッドを委譲
        @Override
        public int size() { return delegate.size(); }
        
        @Override
        public boolean isEmpty() { return delegate.isEmpty(); }
        
        @Override
        public boolean contains(Object o) { return delegate.contains(o); }
        
        @Override
        public java.util.Iterator<T> iterator() { return delegate.iterator(); }
        
        @Override
        public Object[] toArray() { return delegate.toArray(); }
        
        @Override
        public <T1> T1[] toArray(T1[] a) { return delegate.toArray(a); }
        
        @Override
        public boolean add(T t) { return delegate.add(t); }
        
        @Override
        public boolean remove(Object o) { return delegate.remove(o); }
        
        @Override
        public boolean containsAll(java.util.Collection<?> c) { return delegate.containsAll(c); }
        
        @Override
        public boolean addAll(java.util.Collection<? extends T> c) { return delegate.addAll(c); }
        
        @Override
        public boolean addAll(int index, java.util.Collection<? extends T> c) { return delegate.addAll(index, c); }
        
        @Override
        public boolean removeAll(java.util.Collection<?> c) { return delegate.removeAll(c); }
        
        @Override
        public boolean retainAll(java.util.Collection<?> c) { return delegate.retainAll(c); }
        
        @Override
        public void clear() { delegate.clear(); }
        
        @Override
        public T get(int index) { return delegate.get(index); }
        
        @Override
        public T set(int index, T element) { return delegate.set(index, element); }
        
        @Override
        public void add(int index, T element) { delegate.add(index, element); }
        
        @Override
        public T remove(int index) { return delegate.remove(index); }
        
        @Override
        public int indexOf(Object o) { return delegate.indexOf(o); }
        
        @Override
        public int lastIndexOf(Object o) { return delegate.lastIndexOf(o); }
        
        @Override
        public java.util.ListIterator<T> listIterator() { return delegate.listIterator(); }
        
        @Override
        public java.util.ListIterator<T> listIterator(int index) { return delegate.listIterator(index); }
        
        @Override
        public java.util.List<T> subList(int fromIndex, int toIndex) { return delegate.subList(fromIndex, toIndex); }
        
        @Override
        public String toString() { return delegate.toString(); }
    }

    public static void main(String[] args) {
        MethodReferenceDemo demo = new MethodReferenceDemo();
        
        demo.demonstrateStaticMethodReference();
        demo.demonstrateInstanceMethodReference();
        demo.demonstrateArbitraryObjectMethodReference();
        demo.demonstrateConstructorReference();
    }
}
```

## 発展演習2: カスタム関数型インターフェース

### 解答例
```java
package lambda.custom;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;
import java.util.function.Predicate;

public class CustomFunctionalInterfaceDemo {

    /**
     * 3項演算を行う関数型インターフェース
     */
    @FunctionalInterface
    interface TriFunction<T, U, V, R> {
        R apply(T t, U u, V v);
    }
    
    /**
     * 例外をスローできる関数型インターフェース
     */
    @FunctionalInterface
    interface ThrowingFunction<T, R, E extends Exception> {
        R apply(T t) throws E;
        
        /**
         * 例外をキャッチして標準の関数に変換するデフォルトメソッド
         */
        default Function<T, R> unchecked() {
            return t -> {
                try {
                    return apply(t);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            };
        }
    }
    
    /**
     * 2つの条件を組み合わせる述語インターフェース
     */
    @FunctionalInterface
    interface BiPredicate<T> {
        boolean test(T t1, T t2);
        
        /**
         * AND演算子
         */
        default BiPredicate<T> and(BiPredicate<T> other) {
            return (t1, t2) -> this.test(t1, t2) && other.test(t1, t2);
        }
        
        /**
         * OR演算子
         */
        default BiPredicate<T> or(BiPredicate<T> other) {
            return (t1, t2) -> this.test(t1, t2) || other.test(t1, t2);
        }
        
        /**
         * NOT演算子
         */
        default BiPredicate<T> negate() {
            return (t1, t2) -> !this.test(t1, t2);
        }
    }
    
    /**
     * 商品クラス
     */
    static class Product {
        private String id;
        private String name;
        private String category;
        private double price;
        
        public Product(String id, String name, String category, double price) {
            this.id = id;
            this.name = name;
            this.category = category;
            this.price = price;
        }
        
        public String getId() { return id; }
        public String getName() { return name; }
        public String getCategory() { return category; }
        public double getPrice() { return price; }
        
        @Override
        public String toString() {
            return "Product{id='" + id + "', name='" + name + "', category='" + category + "', price=" + price + "}";
        }
    }
    
    /**
     * TriFunctionの使用例を示します。
     */
    public void demonstrateTriFunction() {
        System.out.println("=== TriFunction の使用例 ===");
        
        // 3つの数値の加重平均を計算するTriFunction
        TriFunction<Double, Double, Double, Double> weightedAverage = (a, b, c) -> (a * 0.3 + b * 0.3 + c * 0.4);
        
        double result = weightedAverage.apply(80.0, 90.0, 75.0);
        System.out.println("加重平均: " + result);
        
        // 3つの文字列を結合するTriFunction
        TriFunction<String, String, String, String> concatenate = (s1, s2, s3) -> s1 + s2 + s3;
        
        String concatenated = concatenate.apply("Hello, ", "functional ", "programming!");
        System.out.println("結合結果: " + concatenated);
        
        // 商品情報から表示文字列を生成するTriFunction
        TriFunction<String, String, Double, String> formatProduct = (name, category, price) -> 
            name + " (" + category + "): " + price + "円";
        
        String formattedProduct = formatProduct.apply("ノートPC", "電子機器", 80000.0);
        System.out.println("商品情報: " + formattedProduct);
        
        // 3つの条件に基づいて商品をフィルタリングするTriFunction
        TriFunction<List<Product>, String, Double, List<Product>> filterProducts = (products, categoryFilter, minPrice) -> {
            List<Product> filtered = new ArrayList<>();
            for (Product product : products) {
                if (product.getCategory().equals(categoryFilter) && product.getPrice() >= minPrice) {
                    filtered.add(product);
                }
            }
            return filtered;
        };
        
        List<Product> products = Arrays.asList(
            new Product("P001", "ノートPC", "電子機器", 80000),
            new Product("P002", "スマートフォン", "電子機器", 60000),
            new Product("P003", "コーヒーメーカー", "家電", 15000),
            new Product("P004", "タブレット", "電子機器", 40000),
            new Product("P005", "デスク", "家具", 25000)
        );
        
        List<Product> filteredProducts = filterProducts.apply(products, "電子機器", 50000.0);
        
        System.out.println("\n電子機器カテゴリで50,000円以上の商品:");
        for (Product product : filteredProducts) {
            System.out.println("- " + product.getName() + ": " + product.getPrice() + "円");
        }
    }
    
    /**
     * ThrowingFunctionの使用例を示します。
     */
    public void demonstrateThrowingFunction() {
        System.out.println("\n=== ThrowingFunction の使用例 ===");
        
        // 文字列を整数に変換するThrowingFunction
        ThrowingFunction<String, Integer, NumberFormatException> parseInteger = Integer::parseInt;
        
        // 例外をキャッチして標準の関数に変換
        Function<String, Integer> safeParseInteger = parseInteger.unchecked();
        
        // 正常なケース
        try {
            Integer result1 = safeParseInteger.apply("123");
            System.out.println("変換結果: " + result1);
        } catch (RuntimeException e) {
            System.out.println("変換エラー: " + e.getMessage());
        }
        
        // 例外が発生するケース
        try {
            Integer result2 = safeParseInteger.apply("abc");
            System.out.println("変換結果: " + result2);
        } catch (RuntimeException e) {
            System.out.println("変換エラー: " + e.getCause().getMessage());
        }
        
        // ファイルの内容を読み込むThrowingFunction
        ThrowingFunction<String, String, java.io.IOException> readFile = path -> {
            try (java.io.BufferedReader reader = new java.io.BufferedReader(new java.io.FileReader(path))) {
                StringBuilder content = new StringBuilder();
                String line;
                while ((line = reader.readLine()) != null) {
                    content.append(line).append("\n");
                }
                return content.toString();
            }
        };
        
        // 例外をキャッチして標準の関数に変換
        Function<String, String> safeReadFile = readFile.unchecked();
        
        // 存在しないファイルを読み込もうとする
        try {
            String content = safeReadFile.apply("non_existent_file.txt");
            System.out.println("ファイルの内容: " + content);
        } catch (RuntimeException e) {
            System.out.println("ファイル読み込みエラー: " + e.getCause().getMessage());
        }
    }
    
    /**
     * BiPredicateの使用例を示します。
     */
    public void demonstrateBiPredicate() {
        System.out.println("\n=== BiPredicate の使用例 ===");
        
        // 2つの数値が等しいかをチェックするBiPredicate
        BiPredicate<Integer> equals = (a, b) -> a.equals(b);
        
        // 2つの数値の和が100より大きいかをチェックするBiPredicate
        BiPredicate<Integer> sumGreaterThan100 = (a, b) -> (a + b) > 100;
        
        // 2つの数値の積が1000より大きいかをチェックするBiPredicate
        BiPredicate<Integer> productGreaterThan1000 = (a, b) -> (a * b) > 1000;
        
        // BiPredicateを組み合わせる
        BiPredicate<Integer> complexCondition1 = equals.or(sumGreaterThan100);
        BiPredicate<Integer> complexCondition2 = sumGreaterThan100.and(productGreaterThan1000);
        BiPredicate<Integer> complexCondition3 = equals.negate().and(sumGreaterThan100);
        
        // テスト
        int a = 50;
        int b = 60;
        
        System.out.println(a + "と" + b + "は等しいか、または和が100より大きい: " + complexCondition1.test(a, b));
        System.out.println(a + "と" + b + "の和が100より大きく、かつ積が1000より大きい: " + complexCondition2.test(a, b));
        System.out.println(a + "と" + b + "は等しくなく、かつ和が100より大きい: " + complexCondition3.test(a, b));
        
        // 商品の価格を比較するBiPredicate
        BiPredicate<Product> priceEquals = (p1, p2) -> p1.getPrice() == p2.getPrice();
        BiPredicate<Product> sameCategoryDifferentPrice = (p1, p2) -> 
            p1.getCategory().equals(p2.getCategory()) && p1.getPrice() != p2.getPrice();
        
        Product product1 = new Product("P001", "ノートPC", "電子機器", 80000);
        Product product2 = new Product("P002", "スマートフォン", "電子機器", 60000);
        Product product3 = new Product("P003", "タブレット", "電子機器", 60000);
        
        System.out.println("\n商品の比較:");
        System.out.println("商品1と商品2の価格は等しい: " + priceEquals.test(product1, product2));
        System.out.println("商品2と商品3の価格は等しい: " + priceEquals.test(product2, product3));
        System.out.println("商品1と商品2は同じカテゴリで価格が異なる: " + sameCategoryDifferentPrice.test(product1, product2));
        System.out.println("商品2と商品3は同じカテゴリで価格が異なる: " + sameCategoryDifferentPrice.test(product2, product3));
    }
    
    /**
     * 複数の関数型インターフェースを組み合わせた例を示します。
     */
    public void demonstrateCombination() {
        System.out.println("\n=== 関数型インターフェースの組み合わせ ===");
        
        // 商品リストの作成
        List<Product> products = Arrays.asList(
            new Product("P001", "ノートPC", "電子機器", 80000),
            new Product("P002", "スマートフォン", "電子機器", 60000),
            new Product("P003", "コーヒーメーカー", "家電", 15000),
            new Product("P004", "タブレット", "電子機器", 40000),
            new Product("P005", "デスク", "家具", 25000)
        );
        
        // 商品をフィルタリングする関数
        TriFunction<List<Product>, Predicate<Product>, Function<Product, String>, List<String>> processProducts = 
            (productList, filter, mapper) -> {
                List<String> result = new ArrayList<>();
                for (Product product : productList) {
                    if (filter.test(product)) {
                        result.add(mapper.apply(product));
                    }
                }
                return result;
            };
        
        // 電子機器カテゴリの商品をフィルタリング
        Predicate<Product> isElectronic = product -> product.getCategory().equals("電子機器");
        
        // 商品名と価格をフォーマットするマッパー
        Function<Product, String> formatNameAndPrice = product -> 
            product.getName() + ": " + product.getPrice() + "円";
        
        // 処理を実行
        List<String> formattedElectronics = processProducts.apply(products, isElectronic, formatNameAndPrice);
        
        System.out.println("電子機器カテゴリの商品:");
        for (String formatted : formattedElectronics) {
            System.out.println("- " + formatted);
        }
        
        // 高価な商品（50,000円以上）をフィルタリング
        Predicate<Product> isExpensive = product -> product.getPrice() >= 50000;
        
        // カテゴリと商品名をフォーマットするマッパー
        Function<Product, String> formatCategoryAndName = product -> 
            product.getCategory() + ": " + product.getName();
        
        // 処理を実行
        List<String> formattedExpensiveProducts = processProducts.apply(products, isExpensive, formatCategoryAndName);
        
        System.out.println("\n高価な商品（50,000円以上）:");
        for (String formatted : formattedExpensiveProducts) {
            System.out.println("- " + formatted);
        }
        
        // 複合条件：電子機器カテゴリかつ高価な商品
        Predicate<Product> isExpensiveElectronic = isElectronic.and(isExpensive);
        
        // 商品の詳細情報をフォーマットするマッパー
        Function<Product, String> formatDetails = product -> 
            product.getName() + " (" + product.getCategory() + "): " + product.getPrice() + "円";
        
        // 処理を実行
        List<String> formattedExpensiveElectronics = processProducts.apply(products, isExpensiveElectronic, formatDetails);
        
        System.out.println("\n高価な電子機器（電子機器カテゴリかつ50,000円以上）:");
        for (String formatted : formattedExpensiveElectronics) {
            System.out.println("- " + formatted);
        }
    }

    public static void main(String[] args) {
        CustomFunctionalInterfaceDemo demo = new CustomFunctionalInterfaceDemo();
        
        demo.demonstrateTriFunction();
        demo.demonstrateThrowingFunction();
        demo.demonstrateBiPredicate();
        demo.demonstrateCombination();
    }
}
```
