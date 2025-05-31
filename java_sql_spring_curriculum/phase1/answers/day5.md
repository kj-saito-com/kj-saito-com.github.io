---
layout: default
title: Day 5: Stream APIとループ処理の比較と選択 - 解答例
---
# Day 5: Stream APIとループ処理の比較と選択 - 解答例

## コア演習1: 基本的なStream操作

### 解答例
```java
package stream.basic;

import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

public class BasicStreamOperations {

    /**
     * 従業員クラス
     */
    static class Employee {
        private String name;
        private String department;
        private int salary;
        private int age;

        public Employee(String name, String department, int salary, int age) {
            this.name = name;
            this.department = department;
            this.salary = salary;
            this.age = age;
        }

        public String getName() { return name; }
        public String getDepartment() { return department; }
        public int getSalary() { return salary; }
        public int getAge() { return age; }

        @Override
        public String toString() {
            return "Employee{" +
                   "name='" + name + '\'' +
                   ", department='" + department + '\'' +
                   ", salary=" + salary +
                   ", age=" + age +
                   '}';
        }
    }

    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Alice", "HR", 50000, 30),
            new Employee("Bob", "Engineering", 80000, 25),
            new Employee("Charlie", "Engineering", 90000, 35),
            new Employee("David", "HR", 60000, 40),
            new Employee("Eve", "Sales", 70000, 28),
            new Employee("Frank", "Sales", 75000, 32)
        );

        // 1. フィルタリング (filter)
        System.out.println("=== 1. フィルタリング (Engineering部門の従業員) ===");
        List<Employee> engineers = employees.stream()
            .filter(e -> "Engineering".equals(e.getDepartment()))
            .collect(Collectors.toList());
        engineers.forEach(System.out::println);

        // 2. マッピング (map)
        System.out.println("\n=== 2. マッピング (従業員の名前リスト) ===");
        List<String> names = employees.stream()
            .map(Employee::getName)
            .collect(Collectors.toList());
        System.out.println(names);

        // 3. ソート (sorted)
        System.out.println("\n=== 3. ソート (給与で降順ソート) ===");
        List<Employee> sortedBySalaryDesc = employees.stream()
            .sorted(Comparator.comparingInt(Employee::getSalary).reversed())
            .collect(Collectors.toList());
        sortedBySalaryDesc.forEach(System.out::println);

        // 4. 集約 (collect - グルーピング)
        System.out.println("\n=== 4. 集約 (部門ごとの従業員リスト) ===");
        Map<String, List<Employee>> employeesByDepartment = employees.stream()
            .collect(Collectors.groupingBy(Employee::getDepartment));
        employeesByDepartment.forEach((dept, emps) -> {
            System.out.println("Department: " + dept);
            emps.forEach(emp -> System.out.println("  " + emp));
        });

        // 5. 集約 (collect - 平均値)
        System.out.println("\n=== 5. 集約 (部門ごとの平均給与) ===");
        Map<String, Double> avgSalaryByDepartment = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingInt(Employee::getSalary)
            ));
        avgSalaryByDepartment.forEach((dept, avgSalary) -> 
            System.out.println("Department: " + dept + ", Average Salary: " + avgSalary));

        // 6. 集約 (reduce - 合計値)
        System.out.println("\n=== 6. 集約 (全従業員の合計給与) ===");
        int totalSalary = employees.stream()
            .mapToInt(Employee::getSalary)
            .sum(); // sum()はreduceの特殊ケース
        System.out.println("Total Salary: " + totalSalary);
        
        // reduceを使った合計計算
        Optional<Integer> totalSalaryOptional = employees.stream()
            .map(Employee::getSalary)
            .reduce((s1, s2) -> s1 + s2);
        totalSalaryOptional.ifPresent(ts -> System.out.println("Total Salary (reduce): " + ts));

        // 7. マッチング (anyMatch, allMatch, noneMatch)
        System.out.println("\n=== 7. マッチング ===");
        boolean hasHighSalaryEmployee = employees.stream()
            .anyMatch(e -> e.getSalary() > 85000);
        System.out.println("給与85000超の従業員は存在するか？ " + hasHighSalaryEmployee);

        boolean allAreAdults = employees.stream()
            .allMatch(e -> e.getAge() >= 20);
        System.out.println("全従業員は20歳以上か？ " + allAreAdults);

        boolean noInterns = employees.stream()
            .noneMatch(e -> "Intern".equals(e.getDepartment()));
        System.out.println("インターンは存在しないか？ " + noInterns);

        // 8. 検索 (findFirst, findAny)
        System.out.println("\n=== 8. 検索 ===");
        Optional<Employee> firstEngineer = employees.stream()
            .filter(e -> "Engineering".equals(e.getDepartment()))
            .findFirst();
        firstEngineer.ifPresent(e -> System.out.println("最初のエンジニア: " + e));

        // 9. 数値ストリーム (IntStream, LongStream, DoubleStream)
        System.out.println("\n=== 9. 数値ストリーム ===");
        IntStream salaryStream = employees.stream()
            .mapToInt(Employee::getSalary);
        System.out.println("最大給与: " + salaryStream.max().orElse(0));
        
        // 再度ストリームを生成する必要がある
        double averageAge = employees.stream()
            .mapToInt(Employee::getAge)
            .average()
            .orElse(0.0);
        System.out.println("平均年齢: " + averageAge);

        // 10. フラットマッピング (flatMap)
        System.out.println("\n=== 10. フラットマッピング ===");
        List<List<String>> listOfLists = Arrays.asList(
            Arrays.asList("a", "b"),
            Arrays.asList("c", "d", "e")
        );
        List<String> flattenedList = listOfLists.stream()
            .flatMap(List::stream)
            .collect(Collectors.toList());
        System.out.println("フラット化されたリスト: " + flattenedList);
        
        // 従業員の名前の文字リスト
        List<Character> nameChars = employees.stream()
            .map(Employee::getName)
            .flatMap(name -> name.chars().mapToObj(c -> (char) c))
            .collect(Collectors.toList());
        System.out.println("全従業員の名前の文字リスト: " + nameChars);

        // 11. リミットとスキップ (limit, skip)
        System.out.println("\n=== 11. リミットとスキップ ===");
        System.out.println("最初の3人の従業員:");
        employees.stream()
            .limit(3)
            .forEach(System.out::println);
            
        System.out.println("\n3人目以降の従業員:");
        employees.stream()
            .skip(2)
            .forEach(System.out::println);

        // 12. 重複排除 (distinct)
        System.out.println("\n=== 12. 重複排除 ===");
        List<String> departments = employees.stream()
            .map(Employee::getDepartment)
            .distinct()
            .collect(Collectors.toList());
        System.out.println("部署リスト（重複なし）: " + departments);
    }
}
```

## コア演習2: Stream APIとループ処理のパフォーマンス比較

### 解答例
```java
package stream.performance;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.stream.Collectors;

public class StreamVsLoopPerformance {

    static class Data {
        int id;
        double value;

        Data(int id, double value) {
            this.id = id;
            this.value = value;
        }

        public double getValue() {
            return value;
        }
    }

    public static void main(String[] args) {
        int dataSize = 1_000_000; // データサイズ
        List<Data> dataList = generateData(dataSize);

        // --- Stream API --- 
        System.out.println("--- Stream API --- ");
        long startTimeStream = System.nanoTime();

        double sumStream = dataList.stream()
            .filter(d -> d.getValue() > 0.5) // フィルタリング
            .mapToDouble(d -> d.getValue() * 1.1) // マッピング
            .sum(); // 集約

        long endTimeStream = System.nanoTime();
        long durationStream = (endTimeStream - startTimeStream) / 1_000_000; // ミリ秒

        System.out.println("Stream API 合計値: " + sumStream);
        System.out.println("Stream API 処理時間: " + durationStream + " ms");

        // --- 拡張forループ --- 
        System.out.println("\n--- 拡張forループ --- ");
        long startTimeLoop = System.nanoTime();

        double sumLoop = 0.0;
        for (Data data : dataList) {
            if (data.getValue() > 0.5) { // フィルタリング
                double mappedValue = data.getValue() * 1.1; // マッピング
                sumLoop += mappedValue; // 集約
            }
        }

        long endTimeLoop = System.nanoTime();
        long durationLoop = (endTimeLoop - startTimeLoop) / 1_000_000; // ミリ秒

        System.out.println("拡張forループ 合計値: " + sumLoop);
        System.out.println("拡張forループ 処理時間: " + durationLoop + " ms");

        // --- 並列Stream API --- 
        System.out.println("\n--- 並列Stream API --- ");
        long startTimeParallelStream = System.nanoTime();

        double sumParallelStream = dataList.parallelStream()
            .filter(d -> d.getValue() > 0.5) // フィルタリング
            .mapToDouble(d -> d.getValue() * 1.1) // マッピング
            .sum(); // 集約

        long endTimeParallelStream = System.nanoTime();
        long durationParallelStream = (endTimeParallelStream - startTimeParallelStream) / 1_000_000; // ミリ秒

        System.out.println("並列Stream API 合計値: " + sumParallelStream);
        System.out.println("並列Stream API 処理時間: " + durationParallelStream + " ms");

        // --- 結果の比較 --- 
        System.out.println("\n--- 結果の比較 --- ");
        System.out.println("Stream vs Loop 時間比: " + (double) durationStream / durationLoop);
        System.out.println("Parallel Stream vs Loop 時間比: " + (double) durationParallelStream / durationLoop);
        System.out.println("Parallel Stream vs Stream 時間比: " + (double) durationParallelStream / durationStream);
        
        // --- 複雑な処理の比較 --- 
        System.out.println("\n--- 複雑な処理の比較 --- ");
        
        // Stream API
        long startTimeComplexStream = System.nanoTime();
        List<Double> complexResultStream = dataList.stream()
            .filter(d -> d.id % 2 == 0)
            .mapToDouble(Data::getValue)
            .map(v -> Math.sqrt(v * 10))
            .sorted()
            .limit(100)
            .boxed()
            .collect(Collectors.toList());
        long endTimeComplexStream = System.nanoTime();
        long durationComplexStream = (endTimeComplexStream - startTimeComplexStream) / 1_000_000;
        System.out.println("複雑な処理 (Stream): " + durationComplexStream + " ms");
        
        // ループ
        long startTimeComplexLoop = System.nanoTime();
        List<Double> intermediateList = new ArrayList<>();
        for (Data data : dataList) {
            if (data.id % 2 == 0) {
                intermediateList.add(Math.sqrt(data.getValue() * 10));
            }
        }
        intermediateList.sort(Comparator.naturalOrder());
        List<Double> complexResultLoop = new ArrayList<>();
        for (int i = 0; i < Math.min(100, intermediateList.size()); i++) {
            complexResultLoop.add(intermediateList.get(i));
        }
        long endTimeComplexLoop = System.nanoTime();
        long durationComplexLoop = (endTimeComplexLoop - startTimeComplexLoop) / 1_000_000;
        System.out.println("複雑な処理 (Loop): " + durationComplexLoop + " ms");
        System.out.println("複雑な処理 時間比 (Stream/Loop): " + (double) durationComplexStream / durationComplexLoop);
    }

    /**
     * テストデータを生成します。
     */
    private static List<Data> generateData(int size) {
        List<Data> list = new ArrayList<>(size);
        Random random = new Random();
        for (int i = 0; i < size; i++) {
            list.add(new Data(i, random.nextDouble()));
        }
        return list;
    }
}
```

## コア演習3: 可読性と保守性の観点からの選択

### 解答例

**シナリオ1: ユーザーリストから特定の条件（例: アクティブで、かつ登録日が過去1ヶ月以内）を満たすユーザーのメールアドレスリストを取得する**

*   **Stream API:**
    ```java
    import java.time.LocalDate;
    import java.util.List;
    import java.util.stream.Collectors;
    
    class User {
        String email;
        boolean active;
        LocalDate registrationDate;
        // ... getter
    }
    
    public List<String> getRecentActiveUserEmails(List<User> users) {
        LocalDate oneMonthAgo = LocalDate.now().minusMonths(1);
        return users.stream()
            .filter(User::isActive)
            .filter(user -> user.getRegistrationDate().isAfter(oneMonthAgo))
            .map(User::getEmail)
            .collect(Collectors.toList());
    }
    ```
*   **ループ処理:**
    ```java
    import java.time.LocalDate;
    import java.util.ArrayList;
    import java.util.List;
    
    public List<String> getRecentActiveUserEmailsLoop(List<User> users) {
        List<String> emails = new ArrayList<>();
        LocalDate oneMonthAgo = LocalDate.now().minusMonths(1);
        for (User user : users) {
            if (user.isActive()) {
                if (user.getRegistrationDate().isAfter(oneMonthAgo)) {
                    emails.add(user.getEmail());
                }
            }
        }
        return emails;
    }
    ```
*   **選択理由:** Stream APIの方が適している。
    *   **可読性:** `filter`と`map`により、処理の意図（フィルタリングして、メールアドレスを抽出する）が明確に表現されている。
    *   **保守性:** 条件の追加や変更が容易。例えば、「特定の部署のユーザー」という条件を追加する場合、`filter`を追加するだけで済む。
    *   **簡潔性:** ループ処理に比べてコードが短く、ネストも浅い。

**シナリオ2: 注文リスト内の各注文項目について、在庫を確認し、在庫があれば価格を計算して合計金額を算出し、在庫がなければエラーリストに追加する**

*   **Stream API:**
    ```java
    import java.util.ArrayList;
    import java.util.List;
    import java.util.concurrent.atomic.AtomicReference;
    
    class OrderItem {
        String productId;
        int quantity;
        // ... getter
    }
    
    class InventoryService {
        boolean hasStock(String productId, int quantity) { /* 在庫確認 */ return Math.random() > 0.2; }
    }
    
    class PricingService {
        double getPrice(String productId) { /* 価格取得 */ return Math.random() * 100; }
    }
    
    public double calculateTotalAmount(List<OrderItem> items, InventoryService inventory, PricingService pricing, List<String> errors) {
        AtomicReference<Double> totalAmount = new AtomicReference<>(0.0);
        
        items.forEach(item -> {
            if (inventory.hasStock(item.getProductId(), item.getQuantity())) {
                double price = pricing.getPrice(item.getProductId());
                totalAmount.updateAndGet(current -> current + (price * item.getQuantity()));
            } else {
                errors.add("在庫不足: " + item.getProductId());
            }
        });
        
        return totalAmount.get();
        // 注意: このStream APIの使い方は副作用（errorsリストの変更、AtomicReferenceの使用）があり、
        // Streamの思想からは少し外れる。ループの方が自然かもしれない。
    }
    ```
*   **ループ処理:**
    ```java
    import java.util.ArrayList;
    import java.util.List;
    
    public double calculateTotalAmountLoop(List<OrderItem> items, InventoryService inventory, PricingService pricing, List<String> errors) {
        double totalAmount = 0.0;
        for (OrderItem item : items) {
            if (inventory.hasStock(item.getProductId(), item.getQuantity())) {
                double price = pricing.getPrice(item.getProductId());
                totalAmount += price * item.getQuantity();
            } else {
                errors.add("在庫不足: " + item.getProductId());
            }
        }
        return totalAmount;
    }
    ```
*   **選択理由:** ループ処理の方が適している。
    *   **可読性:** 各要素に対して複数の処理（在庫確認、価格計算、合計加算、エラーリスト追加）を行う場合、ループの方が処理の流れを追いやすい。
    *   **副作用の扱い:** Stream APIで副作用（`errors`リストの変更）を扱うのは少し不自然。ループの方が状態の変更（`totalAmount`の更新、`errors`への追加）を素直に記述できる。
    *   **デバッグ:** ループ内の各ステップで変数の状態を確認しやすく、デバッグが容易。

**シナリオ3: 整数リスト内の各要素を2倍し、その結果が10より大きいものだけを抽出し、最初の5件を取得する**

*   **Stream API:**
    ```java
    import java.util.List;
    import java.util.stream.Collectors;
    
    public List<Integer> processNumbers(List<Integer> numbers) {
        return numbers.stream()
            .map(n -> n * 2)
            .filter(n -> n > 10)
            .limit(5)
            .collect(Collectors.toList());
    }
    ```
*   **ループ処理:**
    ```java
    import java.util.ArrayList;
    import java.util.List;
    
    public List<Integer> processNumbersLoop(List<Integer> numbers) {
        List<Integer> result = new ArrayList<>();
        for (Integer number : numbers) {
            int doubled = number * 2;
            if (doubled > 10) {
                result.add(doubled);
                if (result.size() == 5) {
                    break; // 5件取得したらループを抜ける
                }
            }
        }
        return result;
    }
    ```
*   **選択理由:** Stream APIの方が適している。
    *   **可読性:** `map`, `filter`, `limit`という一連の操作が明確に表現されており、処理の意図が理解しやすい。
    *   **簡潔性:** ループ処理では結果リストの管理や`break`文による早期終了の制御が必要だが、Stream APIではそれらが不要。
    *   **保守性:** 条件の変更（例: 3倍する、20より大きいもの）や取得件数の変更が容易。

## 発展演習1: 並列Streamの活用と注意点

### 解答例
```java
package stream.parallel;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Vector;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class ParallelStreamPitfalls {

    public static void main(String[] args) {
        int size = 10000;
        List<Integer> numbers = IntStream.range(0, size).boxed().collect(Collectors.toList());

        // --- 1. 共有された可変状態 (Stateful Operation) --- 
        System.out.println("--- 1. 共有された可変状態 --- ");
        
        // 問題のある例 (ArrayListはスレッドセーフではない)
        List<Integer> problematicList = new ArrayList<>();
        try {
            numbers.parallelStream()
                .filter(n -> n % 2 == 0)
                .forEach(problematicList::add); // 競合状態が発生する可能性
            System.out.println("問題のあるリストのサイズ (期待値: " + size / 2 + "): " + problematicList.size());
        } catch (ArrayIndexOutOfBoundsException e) {
            System.err.println("問題のあるリストで競合が発生しました: " + e.getMessage());
        }
        
        // 解決策1: スレッドセーフなコレクションを使用 (例: Vector, CopyOnWriteArrayList)
        List<Integer> vectorList = new Vector<>();
        numbers.parallelStream()
            .filter(n -> n % 2 == 0)
            .forEach(vectorList::add);
        System.out.println("Vectorリストのサイズ: " + vectorList.size());
        
        List<Integer> cowList = new CopyOnWriteArrayList<>();
        numbers.parallelStream()
            .filter(n -> n % 2 == 0)
            .forEach(cowList::add);
        System.out.println("CopyOnWriteArrayListのサイズ: " + cowList.size());
        
        // 解決策2: collectを使用 (推奨)
        List<Integer> collectedList = numbers.parallelStream()
            .filter(n -> n % 2 == 0)
            .collect(Collectors.toList());
        System.out.println("collectを使用したリストのサイズ: " + collectedList.size());
        
        // ConcurrentHashMapを使った例
        ConcurrentHashMap<Integer, Integer> concurrentMap = new ConcurrentHashMap<>();
        numbers.parallelStream()
            .forEach(n -> concurrentMap.put(n, n * n));
        System.out.println("ConcurrentHashMapのサイズ: " + concurrentMap.size());

        // --- 2. 順序依存の操作 (Order-Dependent Operations) --- 
        System.out.println("\n--- 2. 順序依存の操作 --- ");
        
        // findFirst vs findAny
        long startTimeFindFirst = System.nanoTime();
        int firstEven = numbers.parallelStream()
            .filter(n -> n % 2 == 0)
            .findFirst()
            .orElse(-1);
        long durationFindFirst = (System.nanoTime() - startTimeFindFirst) / 1_000;
        System.out.println("findFirstの結果: " + firstEven + " (時間: " + durationFindFirst + " us)");
        
        long startTimeFindAny = System.nanoTime();
        int anyEven = numbers.parallelStream()
            .filter(n -> n % 2 == 0)
            .findAny()
            .orElse(-1);
        long durationFindAny = (System.nanoTime() - startTimeFindAny) / 1_000;
        System.out.println("findAnyの結果: " + anyEven + " (時間: " + durationFindAny + " us)"); // 結果は実行ごとに異なる可能性
        
        // limit
        List<Integer> limitedList = numbers.parallelStream()
            .limit(5)
            .collect(Collectors.toList());
        System.out.println("parallelStream().limit(5)の結果: " + limitedList); // 順序は保証されない場合がある
        
        List<Integer> sequentialLimitedList = numbers.stream()
            .limit(5)
            .collect(Collectors.toList());
        System.out.println("stream().limit(5)の結果: " + sequentialLimitedList); // 順序は保証される
        
        // forEach vs forEachOrdered
        System.out.println("parallelStream().forEachの結果 (順序不定): ");
        numbers.parallelStream()
            .limit(5)
            .forEach(n -> System.out.print(n + " "));
        System.out.println();
        
        System.out.println("parallelStream().forEachOrderedの結果 (順序維持): ");
        numbers.parallelStream()
            .limit(5)
            .forEachOrdered(n -> System.out.print(n + " "));
        System.out.println();

        // --- 3. ブロッキング操作 (Blocking Operations) --- 
        System.out.println("\n--- 3. ブロッキング操作 --- ");
        
        // 時間のかかる処理をシミュレート
        long startTimeBlocking = System.nanoTime();
        List<Integer> blockingResult = numbers.parallelStream()
            .map(n -> {
                try {
                    Thread.sleep(1); // ブロッキング操作をシミュレート
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                return n * 2;
            })
            .collect(Collectors.toList());
        long durationBlocking = (System.nanoTime() - startTimeBlocking) / 1_000_000;
        System.out.println("ブロッキング操作ありの処理時間: " + durationBlocking + " ms");
        
        // 逐次ストリームでの処理時間
        long startTimeSequentialBlocking = System.nanoTime();
        List<Integer> sequentialBlockingResult = numbers.stream()
            .map(n -> {
                try {
                    Thread.sleep(1); // ブロッキング操作をシミュレート
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                return n * 2;
            })
            .collect(Collectors.toList());
        long durationSequentialBlocking = (System.nanoTime() - startTimeSequentialBlocking) / 1_000_000;
        System.out.println("逐次ストリームでのブロッキング操作ありの処理時間: " + durationSequentialBlocking + " ms");
        // 並列化しても、ブロッキング操作がボトルネックとなり、期待するほどの高速化が得られない場合がある

        // --- 4. パフォーマンスのオーバーヘッド --- 
        System.out.println("\n--- 4. パフォーマンスのオーバーヘッド --- ");
        int smallSize = 100;
        List<Integer> smallNumbers = IntStream.range(0, smallSize).boxed().collect(Collectors.toList());
        
        // 小規模データでの逐次ストリーム
        long startTimeSmallSequential = System.nanoTime();
        smallNumbers.stream().map(n -> n * 2).collect(Collectors.toList());
        long durationSmallSequential = (System.nanoTime() - startTimeSmallSequential) / 1_000;
        System.out.println("小規模データ (逐次): " + durationSmallSequential + " us");
        
        // 小規模データでの並列ストリーム
        long startTimeSmallParallel = System.nanoTime();
        smallNumbers.parallelStream().map(n -> n * 2).collect(Collectors.toList());
        long durationSmallParallel = (System.nanoTime() - startTimeSmallParallel) / 1_000;
        System.out.println("小規模データ (並列): " + durationSmallParallel + " us");
        // データ量が少ない場合、並列化のオーバーヘッド（タスク分割、結果結合）の方が大きくなり、
        // 逐次処理よりも遅くなることがある。
    }
}
```

## 発展演習2: Stream APIの遅延評価と短絡評価

### 解答例
```java
package stream.evaluation;

import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class LazyAndShortCircuitEvaluation {

    public static void main(String[] args) {
        List<String> data = Arrays.asList("apple", "banana", "apricot", "blueberry", "cherry", "avocado");

        // --- 遅延評価 (Lazy Evaluation) --- 
        System.out.println("--- 遅延評価 --- ");
        
        Stream<String> stream = data.stream()
            .filter(s -> {
                System.out.println("フィルタリング中: " + s);
                return s.startsWith("a");
            })
            .map(s -> {
                System.out.println("マッピング中: " + s);
                return s.toUpperCase();
            });
            
        System.out.println("Stream定義完了。まだ処理は実行されていない。");
        
        // 終端操作を実行して初めて処理が開始される
        System.out.println("\n終端操作 (collect) を実行:");
        List<String> result = stream.collect(Collectors.toList());
        System.out.println("結果: " + result);
        
        // --- 短絡評価 (Short-Circuiting Evaluation) --- 
        System.out.println("\n--- 短絡評価 --- ");
        
        // findFirst
        System.out.println("\nfindFirstの例:");
        Optional<String> firstA = data.stream()
            .filter(s -> {
                System.out.println("フィルタリング中 (findFirst): " + s);
                return s.startsWith("a");
            })
            .map(s -> {
                System.out.println("マッピング中 (findFirst): " + s);
                return s.toUpperCase();
            })
            .findFirst(); // 条件に合う最初の要素が見つかった時点で処理を停止
            
        firstA.ifPresent(s -> System.out.println("最初に見つかった'a'で始まる要素: " + s));
        
        // anyMatch
        System.out.println("\nanyMatchの例:");
        boolean hasBlueberry = data.stream()
            .peek(s -> System.out.println("チェック中 (anyMatch): " + s)) // peekで中間状態を確認
            .anyMatch(s -> s.equals("blueberry")); // 条件に合う要素が見つかった時点でtrueを返し、処理を停止
            
        System.out.println("'blueberry'は存在するか？ " + hasBlueberry);
        
        // allMatch
        System.out.println("\nallMatchの例:");
        boolean allStartWithB = data.stream()
            .peek(s -> System.out.println("チェック中 (allMatch): " + s))
            .allMatch(s -> s.startsWith("b")); // 条件に合わない要素が見つかった時点でfalseを返し、処理を停止
            
        System.out.println("全ての要素が'b'で始まるか？ " + allStartWithB);
        
        // limit
        System.out.println("\nlimitの例:");
        List<String> limited = data.stream()
            .filter(s -> {
                System.out.println("フィルタリング中 (limit): " + s);
                return s.length() > 5;
            })
            .limit(2) // 指定された数の要素が集まった時点で処理を停止
            .collect(Collectors.toList());
            
        System.out.println("長さ5超の最初の2要素: " + limited);
        
        // --- 無限ストリームと短絡評価 --- 
        System.out.println("\n--- 無限ストリームと短絡評価 --- ");
        
        // 無限ストリームを生成し、limitで停止させる
        List<Integer> firstFiveEvens = Stream.iterate(0, n -> n + 2) // 0, 2, 4, ... の無限ストリーム
            .peek(n -> System.out.println("生成中: " + n))
            .limit(5)
            .collect(Collectors.toList());
            
        System.out.println("最初の5つの偶数: " + firstFiveEvens);
        
        // 無限ストリームで条件に合う最初の要素を見つける
        Optional<Integer> firstMultipleOf7 = Stream.iterate(1, n -> n + 1) // 1, 2, 3, ... の無限ストリーム
            .filter(n -> {
                System.out.println("チェック中 (無限): " + n);
                return n % 7 == 0;
            })
            .findFirst(); // 7が見つかった時点で停止
            
        firstMultipleOf7.ifPresent(n -> System.out.println("最初に見つかった7の倍数: " + n));
    }
}
```
