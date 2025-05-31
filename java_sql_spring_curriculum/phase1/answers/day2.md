---
layout: default
title: Day 2: コレクションフレームワークの効果的な活用 - 解答例
---
# Day 2: コレクションフレームワークの効果的な活用 - 解答例

## コア演習1: コレクション操作の基本

### 解答例
```java
package collection.basic;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class BasicCollectionOperations {

    /**
     * 指定された文字列リストから、重複を除去して返します。
     * 
     * @param inputList 入力文字列リスト
     * @return 重複が除去された文字列リスト
     */
    public List<String> removeDuplicates(List<String> inputList) {
        // HashSetを使用して重複を除去
        Set<String> uniqueSet = new HashSet<>(inputList);
        
        // 結果をArrayListに変換して返す
        return new ArrayList<>(uniqueSet);
    }

    /**
     * 2つの文字列リストの共通要素を返します。
     * 
     * @param list1 1つ目の文字列リスト
     * @param list2 2つ目の文字列リスト
     * @return 共通要素を含む文字列リスト
     */
    public List<String> findCommonElements(List<String> list1, List<String> list2) {
        // 1つ目のリストからSetを作成
        Set<String> set1 = new HashSet<>(list1);
        
        // 2つ目のリストの要素のうち、set1に含まれるものだけを結果に追加
        Set<String> commonSet = new HashSet<>();
        for (String item : list2) {
            if (set1.contains(item)) {
                commonSet.add(item);
            }
        }
        
        // 結果をArrayListに変換して返す
        return new ArrayList<>(commonSet);
    }

    /**
     * 指定された文字列リストから、特定の文字で始まる要素のみを抽出します。
     * 
     * @param inputList 入力文字列リスト
     * @param prefix 検索する接頭辞
     * @return 条件に一致する文字列リスト
     */
    public List<String> filterByPrefix(List<String> inputList, String prefix) {
        List<String> result = new ArrayList<>();
        
        for (String item : inputList) {
            if (item != null && item.startsWith(prefix)) {
                result.add(item);
            }
        }
        
        return result;
    }

    public static void main(String[] args) {
        BasicCollectionOperations demo = new BasicCollectionOperations();
        
        // 重複除去のテスト
        List<String> duplicatesList = List.of("apple", "banana", "apple", "orange", "banana", "grape");
        List<String> uniqueList = demo.removeDuplicates(duplicatesList);
        System.out.println("元のリスト: " + duplicatesList);
        System.out.println("重複除去後: " + uniqueList);
        
        // 共通要素のテスト
        List<String> list1 = List.of("apple", "banana", "orange", "grape", "melon");
        List<String> list2 = List.of("banana", "kiwi", "orange", "pear");
        List<String> commonElements = demo.findCommonElements(list1, list2);
        System.out.println("\nリスト1: " + list1);
        System.out.println("リスト2: " + list2);
        System.out.println("共通要素: " + commonElements);
        
        // 接頭辞フィルタリングのテスト
        List<String> words = List.of("apple", "application", "banana", "apartment", "book", "air");
        String prefix = "ap";
        List<String> filteredWords = demo.filterByPrefix(words, prefix);
        System.out.println("\n元のリスト: " + words);
        System.out.println("接頭辞 '" + prefix + "' で始まる要素: " + filteredWords);
    }
}
```

## コア演習2: 適切なコレクション選択

### 解答例
```java
package collection.selection;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.LinkedHashMap;
import java.util.LinkedList;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;
import java.util.TreeSet;

public class CollectionSelectionDemo {

    /**
     * 指定された文字列リストから、一意の単語を抽出し、アルファベット順にソートします。
     * 
     * @param sentences 文章のリスト
     * @return ソートされた一意の単語リスト
     */
    public List<String> extractUniqueWordsSorted(List<String> sentences) {
        // TreeSetを使用して、一意性の保証とソートを同時に行う
        Set<String> uniqueWords = new TreeSet<>();
        
        for (String sentence : sentences) {
            if (sentence != null) {
                // 文章を単語に分割
                String[] words = sentence.toLowerCase().split("\\s+");
                
                // 各単語をSetに追加（重複は自動的に除去される）
                for (String word : words) {
                    // 空白や記号のみの場合はスキップ
                    if (!word.isEmpty() && word.matches(".*[a-zA-Z0-9].*")) {
                        // 句読点などを除去
                        word = word.replaceAll("[^a-zA-Z0-9]", "");
                        uniqueWords.add(word);
                    }
                }
            }
        }
        
        // TreeSetの内容をArrayListに変換して返す
        return new ArrayList<>(uniqueWords);
    }

    /**
     * 指定された文字列リストから、各単語の出現回数をカウントします。
     * 
     * @param sentences 文章のリスト
     * @return 単語と出現回数のマップ
     */
    public Map<String, Integer> countWordFrequency(List<String> sentences) {
        // HashMapを使用して単語の出現回数をカウント
        Map<String, Integer> wordFrequency = new HashMap<>();
        
        for (String sentence : sentences) {
            if (sentence != null) {
                // 文章を単語に分割
                String[] words = sentence.toLowerCase().split("\\s+");
                
                // 各単語の出現回数をカウント
                for (String word : words) {
                    // 空白や記号のみの場合はスキップ
                    if (!word.isEmpty() && word.matches(".*[a-zA-Z0-9].*")) {
                        // 句読点などを除去
                        word = word.replaceAll("[^a-zA-Z0-9]", "");
                        
                        // 単語の出現回数を更新
                        wordFrequency.put(word, wordFrequency.getOrDefault(word, 0) + 1);
                    }
                }
            }
        }
        
        return wordFrequency;
    }

    /**
     * 指定された単語の出現回数マップから、出現回数の多い順にソートして返します。
     * 
     * @param wordFrequency 単語と出現回数のマップ
     * @param limit 上位何件を取得するか
     * @return 出現回数の多い順にソートされた単語と出現回数のマップ
     */
    public Map<String, Integer> getTopWords(Map<String, Integer> wordFrequency, int limit) {
        // 出現回数でソートするために、エントリのリストを作成
        List<Map.Entry<String, Integer>> entries = new ArrayList<>(wordFrequency.entrySet());
        
        // 出現回数の降順でソート
        entries.sort((e1, e2) -> e2.getValue().compareTo(e1.getValue()));
        
        // 上位N件を取得してLinkedHashMapに格納（挿入順序を保持）
        Map<String, Integer> topWords = new LinkedHashMap<>();
        int count = 0;
        for (Map.Entry<String, Integer> entry : entries) {
            if (count >= limit) {
                break;
            }
            topWords.put(entry.getKey(), entry.getValue());
            count++;
        }
        
        return topWords;
    }

    /**
     * 指定された文字列リストから、各単語の初出位置を記録します。
     * 
     * @param sentences 文章のリスト
     * @return 単語と初出位置のマップ
     */
    public Map<String, Integer> recordFirstOccurrence(List<String> sentences) {
        // LinkedHashMapを使用して、単語の初出位置を記録
        Map<String, Integer> firstOccurrence = new HashMap<>();
        
        int position = 0;
        for (String sentence : sentences) {
            if (sentence != null) {
                // 文章を単語に分割
                String[] words = sentence.toLowerCase().split("\\s+");
                
                // 各単語の初出位置を記録
                for (String word : words) {
                    // 空白や記号のみの場合はスキップ
                    if (!word.isEmpty() && word.matches(".*[a-zA-Z0-9].*")) {
                        // 句読点などを除去
                        word = word.replaceAll("[^a-zA-Z0-9]", "");
                        
                        // まだ記録されていない単語のみ追加
                        if (!firstOccurrence.containsKey(word)) {
                            firstOccurrence.put(word, position);
                        }
                    }
                    position++;
                }
            }
        }
        
        return firstOccurrence;
    }

    public static void main(String[] args) {
        CollectionSelectionDemo demo = new CollectionSelectionDemo();
        
        // サンプルデータ
        List<String> sentences = List.of(
            "Java collections framework is powerful and flexible.",
            "Collections in Java include List, Set, and Map interfaces.",
            "ArrayList is a resizable array implementation of List.",
            "HashSet is a Set implementation that uses a hash table.",
            "HashMap is a Map implementation based on a hash table."
        );
        
        // 一意の単語をソートして取得
        List<String> uniqueWords = demo.extractUniqueWordsSorted(sentences);
        System.out.println("一意の単語（アルファベット順）:");
        for (String word : uniqueWords) {
            System.out.println("- " + word);
        }
        
        // 単語の出現回数をカウント
        Map<String, Integer> wordFrequency = demo.countWordFrequency(sentences);
        System.out.println("\n単語の出現回数:");
        for (Map.Entry<String, Integer> entry : wordFrequency.entrySet()) {
            System.out.println("- " + entry.getKey() + ": " + entry.getValue());
        }
        
        // 出現回数の多い上位5単語を取得
        Map<String, Integer> topWords = demo.getTopWords(wordFrequency, 5);
        System.out.println("\n出現回数の多い上位5単語:");
        int rank = 1;
        for (Map.Entry<String, Integer> entry : topWords.entrySet()) {
            System.out.println(rank + ". " + entry.getKey() + ": " + entry.getValue());
            rank++;
        }
        
        // 単語の初出位置を記録
        Map<String, Integer> firstOccurrence = demo.recordFirstOccurrence(sentences);
        System.out.println("\n単語の初出位置:");
        // 初出位置でソート
        List<Map.Entry<String, Integer>> sortedByPosition = new ArrayList<>(firstOccurrence.entrySet());
        sortedByPosition.sort(Map.Entry.comparingByValue());
        for (Map.Entry<String, Integer> entry : sortedByPosition) {
            System.out.println("- " + entry.getKey() + ": 位置 " + entry.getValue());
        }
        
        // 異なるコレクション実装の比較
        System.out.println("\n異なるコレクション実装の比較:");
        
        // リスト実装の比較
        List<String> arrayList = new ArrayList<>();
        List<String> linkedList = new LinkedList<>();
        System.out.println("ArrayList vs LinkedList:");
        System.out.println("- ArrayList: 高速なランダムアクセス、末尾への追加が効率的");
        System.out.println("- LinkedList: 先頭/末尾への追加/削除が効率的、ランダムアクセスは遅い");
        
        // セット実装の比較
        Set<String> hashSet = new HashSet<>();
        Set<String> treeSet = new TreeSet<>();
        System.out.println("\nHashSet vs TreeSet:");
        System.out.println("- HashSet: 高速な追加/検索/削除（O(1)）、順序なし");
        System.out.println("- TreeSet: ソート順を維持、追加/検索/削除はO(log n)");
        
        // マップ実装の比較
        Map<String, Integer> hashMap = new HashMap<>();
        Map<String, Integer> treeMap = new TreeMap<>();
        Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
        System.out.println("\nHashMap vs TreeMap vs LinkedHashMap:");
        System.out.println("- HashMap: 高速な追加/検索/削除（O(1)）、順序なし");
        System.out.println("- TreeMap: キーでソート、追加/検索/削除はO(log n)");
        System.out.println("- LinkedHashMap: 挿入順序を維持、HashMap並みのパフォーマンス");
    }
}
```

## コア演習3: パフォーマンス比較

### 解答例
```java
package collection.performance;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.LinkedList;
import java.util.List;
import java.util.Map;
import java.util.Random;
import java.util.Set;
import java.util.TreeMap;
import java.util.TreeSet;

public class CollectionPerformanceComparison {

    private static final int DATA_SIZE = 100000;
    private static final int SEARCH_SIZE = 10000;
    private static final Random random = new Random();

    /**
     * パフォーマンスを測定します。
     * 
     * @param description 測定の説明
     * @param task 測定するタスク
     * @return 実行時間（ミリ秒）
     */
    public static long measurePerformance(String description, Runnable task) {
        System.out.println("測定: " + description);
        
        // ウォームアップ（JITコンパイラの最適化のため）
        for (int i = 0; i < 3; i++) {
            task.run();
        }
        
        // 実際の測定
        long startTime = System.nanoTime();
        task.run();
        long endTime = System.nanoTime();
        
        long duration = (endTime - startTime) / 1_000_000; // ミリ秒に変換
        System.out.println("  実行時間: " + duration + " ms");
        
        return duration;
    }

    /**
     * リスト実装のパフォーマンスを比較します。
     */
    public static void compareListPerformance() {
        System.out.println("\n===== リスト実装のパフォーマンス比較 =====");
        
        // テストデータの準備
        List<Integer> data = new ArrayList<>();
        for (int i = 0; i < DATA_SIZE; i++) {
            data.add(random.nextInt(1000000));
        }
        
        // ArrayList vs LinkedList: 追加操作
        List<Integer> arrayList = new ArrayList<>();
        List<Integer> linkedList = new LinkedList<>();
        
        // 末尾への追加
        measurePerformance("ArrayList - 末尾への追加", () -> {
            List<Integer> list = new ArrayList<>();
            for (int i = 0; i < DATA_SIZE; i++) {
                list.add(data.get(i));
            }
        });
        
        measurePerformance("LinkedList - 末尾への追加", () -> {
            List<Integer> list = new LinkedList<>();
            for (int i = 0; i < DATA_SIZE; i++) {
                list.add(data.get(i));
            }
        });
        
        // 先頭への追加
        measurePerformance("ArrayList - 先頭への追加", () -> {
            List<Integer> list = new ArrayList<>();
            for (int i = 0; i < DATA_SIZE / 10; i++) { // サイズを小さくして実行時間を短縮
                list.add(0, data.get(i));
            }
        });
        
        measurePerformance("LinkedList - 先頭への追加", () -> {
            List<Integer> list = new LinkedList<>();
            for (int i = 0; i < DATA_SIZE / 10; i++) { // サイズを小さくして実行時間を短縮
                list.add(0, data.get(i));
            }
        });
        
        // リストの作成
        arrayList = new ArrayList<>(data);
        linkedList = new LinkedList<>(data);
        
        // ランダムアクセス
        measurePerformance("ArrayList - ランダムアクセス", () -> {
            int sum = 0;
            for (int i = 0; i < SEARCH_SIZE; i++) {
                int index = random.nextInt(DATA_SIZE);
                sum += arrayList.get(index);
            }
        });
        
        measurePerformance("LinkedList - ランダムアクセス", () -> {
            int sum = 0;
            for (int i = 0; i < SEARCH_SIZE; i++) {
                int index = random.nextInt(DATA_SIZE);
                sum += linkedList.get(index);
            }
        });
        
        // 反復処理
        measurePerformance("ArrayList - 反復処理", () -> {
            int sum = 0;
            for (Integer value : arrayList) {
                sum += value;
            }
        });
        
        measurePerformance("LinkedList - 反復処理", () -> {
            int sum = 0;
            for (Integer value : linkedList) {
                sum += value;
            }
        });
    }

    /**
     * セット実装のパフォーマンスを比較します。
     */
    public static void compareSetPerformance() {
        System.out.println("\n===== セット実装のパフォーマンス比較 =====");
        
        // テストデータの準備
        List<Integer> data = new ArrayList<>();
        for (int i = 0; i < DATA_SIZE; i++) {
            data.add(random.nextInt(1000000));
        }
        
        // 検索用データ
        List<Integer> searchData = new ArrayList<>();
        for (int i = 0; i < SEARCH_SIZE; i++) {
            // 半分は存在するデータ、半分は存在しないデータ
            if (i % 2 == 0) {
                searchData.add(data.get(random.nextInt(DATA_SIZE)));
            } else {
                searchData.add(random.nextInt(2000000) + 1000000);
            }
        }
        
        // HashSet vs TreeSet: 追加操作
        measurePerformance("HashSet - 追加", () -> {
            Set<Integer> set = new HashSet<>();
            for (Integer value : data) {
                set.add(value);
            }
        });
        
        measurePerformance("TreeSet - 追加", () -> {
            Set<Integer> set = new TreeSet<>();
            for (Integer value : data) {
                set.add(value);
            }
        });
        
        // セットの作成
        Set<Integer> hashSet = new HashSet<>(data);
        Set<Integer> treeSet = new TreeSet<>(data);
        
        // 検索操作
        measurePerformance("HashSet - 検索", () -> {
            int count = 0;
            for (Integer value : searchData) {
                if (hashSet.contains(value)) {
                    count++;
                }
            }
        });
        
        measurePerformance("TreeSet - 検索", () -> {
            int count = 0;
            for (Integer value : searchData) {
                if (treeSet.contains(value)) {
                    count++;
                }
            }
        });
        
        // 反復処理
        measurePerformance("HashSet - 反復処理", () -> {
            int sum = 0;
            for (Integer value : hashSet) {
                sum += value;
            }
        });
        
        measurePerformance("TreeSet - 反復処理", () -> {
            int sum = 0;
            for (Integer value : treeSet) {
                sum += value;
            }
        });
    }

    /**
     * マップ実装のパフォーマンスを比較します。
     */
    public static void compareMapPerformance() {
        System.out.println("\n===== マップ実装のパフォーマンス比較 =====");
        
        // テストデータの準備
        List<Integer> keys = new ArrayList<>();
        List<String> values = new ArrayList<>();
        for (int i = 0; i < DATA_SIZE; i++) {
            keys.add(random.nextInt(1000000));
            values.add("Value-" + i);
        }
        
        // 検索用データ
        List<Integer> searchKeys = new ArrayList<>();
        for (int i = 0; i < SEARCH_SIZE; i++) {
            // 半分は存在するキー、半分は存在しないキー
            if (i % 2 == 0) {
                searchKeys.add(keys.get(random.nextInt(DATA_SIZE)));
            } else {
                searchKeys.add(random.nextInt(2000000) + 1000000);
            }
        }
        
        // HashMap vs TreeMap: 追加操作
        measurePerformance("HashMap - 追加", () -> {
            Map<Integer, String> map = new HashMap<>();
            for (int i = 0; i < DATA_SIZE; i++) {
                map.put(keys.get(i), values.get(i));
            }
        });
        
        measurePerformance("TreeMap - 追加", () -> {
            Map<Integer, String> map = new TreeMap<>();
            for (int i = 0; i < DATA_SIZE; i++) {
                map.put(keys.get(i), values.get(i));
            }
        });
        
        // マップの作成
        Map<Integer, String> hashMap = new HashMap<>();
        Map<Integer, String> treeMap = new TreeMap<>();
        for (int i = 0; i < DATA_SIZE; i++) {
            hashMap.put(keys.get(i), values.get(i));
            treeMap.put(keys.get(i), values.get(i));
        }
        
        // 検索操作
        measurePerformance("HashMap - 検索", () -> {
            int count = 0;
            for (Integer key : searchKeys) {
                if (hashMap.containsKey(key)) {
                    count++;
                }
            }
        });
        
        measurePerformance("TreeMap - 検索", () -> {
            int count = 0;
            for (Integer key : searchKeys) {
                if (treeMap.containsKey(key)) {
                    count++;
                }
            }
        });
        
        // 値の取得
        measurePerformance("HashMap - 値の取得", () -> {
            int count = 0;
            for (Integer key : searchKeys) {
                String value = hashMap.get(key);
                if (value != null) {
                    count++;
                }
            }
        });
        
        measurePerformance("TreeMap - 値の取得", () -> {
            int count = 0;
            for (Integer key : searchKeys) {
                String value = treeMap.get(key);
                if (value != null) {
                    count++;
                }
            }
        });
        
        // 反復処理
        measurePerformance("HashMap - エントリの反復処理", () -> {
            int count = 0;
            for (Map.Entry<Integer, String> entry : hashMap.entrySet()) {
                count += entry.getKey();
            }
        });
        
        measurePerformance("TreeMap - エントリの反復処理", () -> {
            int count = 0;
            for (Map.Entry<Integer, String> entry : treeMap.entrySet()) {
                count += entry.getKey();
            }
        });
    }

    public static void main(String[] args) {
        System.out.println("コレクション実装のパフォーマンス比較");
        System.out.println("データサイズ: " + DATA_SIZE);
        System.out.println("検索サイズ: " + SEARCH_SIZE);
        
        compareListPerformance();
        compareSetPerformance();
        compareMapPerformance();
        
        System.out.println("\n===== 結論 =====");
        System.out.println("1. ArrayList: ランダムアクセスが速い、末尾への追加も効率的");
        System.out.println("2. LinkedList: 先頭への追加/削除が速い、ランダムアクセスは遅い");
        System.out.println("3. HashSet: 追加/検索が非常に速い（O(1)）、順序なし");
        System.out.println("4. TreeSet: 順序付けされた要素が必要な場合に使用、操作はO(log n)");
        System.out.println("5. HashMap: キー/値のペアの追加/検索が非常に速い（O(1)）");
        System.out.println("6. TreeMap: キーでソートされたマップが必要な場合に使用、操作はO(log n)");
    }
}
```

## 発展演習1: hashCode/equalsの実装

### 解答例
```java
package collection.hashcode;

import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Objects;
import java.util.Set;

public class HashCodeEqualsDemo {

    /**
     * 顧客クラス（hashCode/equalsを適切に実装）
     */
    public static class Customer {
        private String id;
        private String name;
        private String email;
        
        public Customer(String id, String name, String email) {
            this.id = id;
            this.name = name;
            this.email = email;
        }
        
        public String getId() { return id; }
        public String getName() { return name; }
        public String getEmail() { return email; }
        
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            
            Customer customer = (Customer) o;
            
            // IDが同じであれば同一の顧客とみなす
            return Objects.equals(id, customer.id);
        }
        
        @Override
        public int hashCode() {
            // IDに基づいてハッシュコードを生成
            return Objects.hash(id);
        }
        
        @Override
        public String toString() {
            return "Customer{id='" + id + "', name='" + name + "', email='" + email + "'}";
        }
    }
    
    /**
     * 注文クラス（hashCode/equalsを適切に実装）
     */
    public static class Order {
        private String orderId;
        private String customerId;
        private double amount;
        private String status;
        
        public Order(String orderId, String customerId, double amount, String status) {
            this.orderId = orderId;
            this.customerId = customerId;
            this.amount = amount;
            this.status = status;
        }
        
        public String getOrderId() { return orderId; }
        public String getCustomerId() { return customerId; }
        public double getAmount() { return amount; }
        public String getStatus() { return status; }
        
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            
            Order order = (Order) o;
            
            // 注文IDが同じであれば同一の注文とみなす
            return Objects.equals(orderId, order.orderId);
        }
        
        @Override
        public int hashCode() {
            // 注文IDに基づいてハッシュコードを生成
            return Objects.hash(orderId);
        }
        
        @Override
        public String toString() {
            return "Order{orderId='" + orderId + "', customerId='" + customerId + 
                   "', amount=" + amount + ", status='" + status + "'}";
        }
    }
    
    /**
     * 製品クラス（hashCode/equalsを不適切に実装）
     */
    public static class ProductBad {
        private String productId;
        private String name;
        private double price;
        
        public ProductBad(String productId, String name, double price) {
            this.productId = productId;
            this.name = name;
            this.price = price;
        }
        
        public String getProductId() { return productId; }
        public String getName() { return name; }
        public double getPrice() { return price; }
        
        // equalsをオーバーライドしているが、hashCodeをオーバーライドしていない
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            
            ProductBad product = (ProductBad) o;
            
            return Objects.equals(productId, product.productId);
        }
        
        // hashCodeをオーバーライドしていないため、Object.hashCode()が使用される
        
        @Override
        public String toString() {
            return "ProductBad{productId='" + productId + "', name='" + name + "', price=" + price + "}";
        }
    }
    
    /**
     * 製品クラス（hashCode/equalsを適切に実装）
     */
    public static class ProductGood {
        private String productId;
        private String name;
        private double price;
        
        public ProductGood(String productId, String name, double price) {
            this.productId = productId;
            this.name = name;
            this.price = price;
        }
        
        public String getProductId() { return productId; }
        public String getName() { return name; }
        public double getPrice() { return price; }
        
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            
            ProductGood product = (ProductGood) o;
            
            return Objects.equals(productId, product.productId);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(productId);
        }
        
        @Override
        public String toString() {
            return "ProductGood{productId='" + productId + "', name='" + name + "', price=" + price + "}";
        }
    }

    public static void main(String[] args) {
        // 顧客の例
        System.out.println("===== 顧客の例（hashCode/equalsを適切に実装） =====");
        
        Customer customer1 = new Customer("C001", "山田太郎", "taro@example.com");
        Customer customer2 = new Customer("C001", "山田太郎", "taro@example.com");
        Customer customer3 = new Customer("C001", "山田次郎", "jiro@example.com"); // 名前とメールが異なるが、IDは同じ
        Customer customer4 = new Customer("C002", "山田太郎", "taro@example.com"); // 名前とメールは同じだが、IDが異なる
        
        System.out.println("customer1.equals(customer2): " + customer1.equals(customer2)); // true
        System.out.println("customer1.equals(customer3): " + customer1.equals(customer3)); // true（IDが同じため）
        System.out.println("customer1.equals(customer4): " + customer1.equals(customer4)); // false
        
        System.out.println("customer1.hashCode(): " + customer1.hashCode());
        System.out.println("customer2.hashCode(): " + customer2.hashCode());
        System.out.println("customer3.hashCode(): " + customer3.hashCode());
        System.out.println("customer4.hashCode(): " + customer4.hashCode());
        
        Set<Customer> customerSet = new HashSet<>();
        customerSet.add(customer1);
        customerSet.add(customer2);
        customerSet.add(customer3);
        customerSet.add(customer4);
        
        System.out.println("HashSetのサイズ: " + customerSet.size()); // 2（customer1, customer2, customer3は同一とみなされる）
        System.out.println("HashSetの内容: " + customerSet);
        
        // 注文の例
        System.out.println("\n===== 注文の例（hashCode/equalsを適切に実装） =====");
        
        Order order1 = new Order("O001", "C001", 5000, "配送中");
        Order order2 = new Order("O001", "C001", 5000, "配送中");
        Order order3 = new Order("O001", "C002", 6000, "準備中"); // 顧客IDと金額と状態が異なるが、注文IDは同じ
        Order order4 = new Order("O002", "C001", 5000, "配送中"); // 顧客IDと金額と状態は同じだが、注文IDが異なる
        
        System.out.println("order1.equals(order2): " + order1.equals(order2)); // true
        System.out.println("order1.equals(order3): " + order1.equals(order3)); // true（注文IDが同じため）
        System.out.println("order1.equals(order4): " + order1.equals(order4)); // false
        
        Map<Order, String> orderMap = new HashMap<>();
        orderMap.put(order1, "注文1の詳細情報");
        orderMap.put(order2, "注文2の詳細情報");
        orderMap.put(order3, "注文3の詳細情報");
        orderMap.put(order4, "注文4の詳細情報");
        
        System.out.println("HashMapのサイズ: " + orderMap.size()); // 2（order1, order2, order3は同一とみなされる）
        System.out.println("order1のマップ値: " + orderMap.get(order1));
        System.out.println("order3のマップ値: " + orderMap.get(order3)); // order1と同じキーとみなされるため、最後に追加された値が取得される
        
        // 製品の例（hashCode/equalsを不適切に実装）
        System.out.println("\n===== 製品の例（hashCode/equalsを不適切に実装） =====");
        
        ProductBad product1 = new ProductBad("P001", "ノートPC", 80000);
        ProductBad product2 = new ProductBad("P001", "ノートPC", 80000);
        
        System.out.println("product1.equals(product2): " + product1.equals(product2)); // true
        System.out.println("product1.hashCode(): " + product1.hashCode());
        System.out.println("product2.hashCode(): " + product2.hashCode());
        
        Set<ProductBad> productBadSet = new HashSet<>();
        productBadSet.add(product1);
        productBadSet.add(product2);
        
        System.out.println("HashSetのサイズ: " + productBadSet.size()); // 2（hashCodeが異なるため、同一とみなされない）
        System.out.println("HashSetの内容: " + productBadSet);
        
        // 製品の例（hashCode/equalsを適切に実装）
        System.out.println("\n===== 製品の例（hashCode/equalsを適切に実装） =====");
        
        ProductGood productGood1 = new ProductGood("P001", "ノートPC", 80000);
        ProductGood productGood2 = new ProductGood("P001", "ノートPC", 80000);
        
        System.out.println("productGood1.equals(productGood2): " + productGood1.equals(productGood2)); // true
        System.out.println("productGood1.hashCode(): " + productGood1.hashCode());
        System.out.println("productGood2.hashCode(): " + productGood2.hashCode());
        
        Set<ProductGood> productGoodSet = new HashSet<>();
        productGoodSet.add(productGood1);
        productGoodSet.add(productGood2);
        
        System.out.println("HashSetのサイズ: " + productGoodSet.size()); // 1（hashCodeが同じため、同一とみなされる）
        System.out.println("HashSetの内容: " + productGoodSet);
        
        // 結論
        System.out.println("\n===== 結論 =====");
        System.out.println("1. equals()とhashCode()は常にペアでオーバーライドする必要がある");
        System.out.println("2. equals()がtrueを返す2つのオブジェクトは、同じhashCode()値を返さなければならない");
        System.out.println("3. hashCode()が同じでも、equals()がfalseを返す場合はある（ハッシュ衝突）");
        System.out.println("4. HashSet, HashMapなどのハッシュベースのコレクションでは、これらのメソッドが正しく実装されていないと予期しない動作をする");
    }
}
```

## 発展演習2: LRUキャッシュの実装

### 解答例
```java
package collection.lru;

import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

public class LRUCacheDemo {

    /**
     * 自前実装のLRUキャッシュ
     * 
     * @param <K> キーの型
     * @param <V> 値の型
     */
    public static class LRUCache<K, V> {
        private final int capacity;
        private final Map<K, Node<K, V>> cache;
        private Node<K, V> head;
        private Node<K, V> tail;
        
        // 双方向リンクリストのノード
        private static class Node<K, V> {
            K key;
            V value;
            Node<K, V> prev;
            Node<K, V> next;
            
            Node(K key, V value) {
                this.key = key;
                this.value = value;
            }
        }
        
        public LRUCache(int capacity) {
            this.capacity = capacity;
            this.cache = new HashMap<>(capacity);
            this.head = null;
            this.tail = null;
        }
        
        /**
         * キャッシュから値を取得します。
         * 
         * @param key キー
         * @return 値（キーが存在しない場合はnull）
         */
        public V get(K key) {
            Node<K, V> node = cache.get(key);
            if (node == null) {
                return null; // キーが存在しない
            }
            
            // ノードを最近使用したものとしてマーク（リストの先頭に移動）
            moveToHead(node);
            
            return node.value;
        }
        
        /**
         * キャッシュに値を設定します。
         * 
         * @param key キー
         * @param value 値
         */
        public void put(K key, V value) {
            Node<K, V> node = cache.get(key);
            
            if (node != null) {
                // 既存のキーの場合は値を更新し、最近使用したものとしてマーク
                node.value = value;
                moveToHead(node);
                return;
            }
            
            // キャパシティを超える場合は、最も長く使用されていないアイテム（末尾）を削除
            if (cache.size() >= capacity) {
                cache.remove(tail.key);
                removeTail();
            }
            
            // 新しいノードを作成し、先頭に追加
            Node<K, V> newNode = new Node<>(key, value);
            cache.put(key, newNode);
            addToHead(newNode);
        }
        
        /**
         * キャッシュの現在のサイズを返します。
         * 
         * @return キャッシュのサイズ
         */
        public int size() {
            return cache.size();
        }
        
        /**
         * キャッシュの内容を文字列形式で返します。
         * 
         * @return キャッシュの内容（最近使用した順）
         */
        @Override
        public String toString() {
            StringBuilder sb = new StringBuilder();
            sb.append("LRUCache{capacity=").append(capacity).append(", size=").append(cache.size()).append(", items=[");
            
            Node<K, V> current = head;
            while (current != null) {
                sb.append(current.key).append("=").append(current.value);
                current = current.next;
                if (current != null) {
                    sb.append(", ");
                }
            }
            
            sb.append("]}");
            return sb.toString();
        }
        
        // ノードをリストの先頭に移動
        private void moveToHead(Node<K, V> node) {
            if (node == head) {
                return; // 既に先頭にある場合は何もしない
            }
            
            // リストから削除
            if (node.prev != null) {
                node.prev.next = node.next;
            }
            if (node.next != null) {
                node.next.prev = node.prev;
            }
            if (node == tail) {
                tail = node.prev;
            }
            
            // 先頭に追加
            node.prev = null;
            node.next = head;
            if (head != null) {
                head.prev = node;
            }
            head = node;
            
            // リストが空だった場合は、末尾も設定
            if (tail == null) {
                tail = node;
            }
        }
        
        // 新しいノードをリストの先頭に追加
        private void addToHead(Node<K, V> node) {
            node.prev = null;
            node.next = head;
            
            if (head != null) {
                head.prev = node;
            }
            
            head = node;
            
            // リストが空だった場合は、末尾も設定
            if (tail == null) {
                tail = node;
            }
        }
        
        // リストの末尾を削除
        private void removeTail() {
            if (tail == null) {
                return;
            }
            
            if (tail.prev != null) {
                tail.prev.next = null;
                tail = tail.prev;
            } else {
                // リストに要素が1つしかない場合
                head = null;
                tail = null;
            }
        }
    }
    
    /**
     * LinkedHashMapを使用したLRUキャッシュ
     * 
     * @param <K> キーの型
     * @param <V> 値の型
     */
    public static class LinkedHashMapLRUCache<K, V> extends LinkedHashMap<K, V> {
        private final int capacity;
        
        public LinkedHashMapLRUCache(int capacity) {
            // 初期容量、ロードファクタ、アクセス順序（true: アクセス順、false: 挿入順）
            super(capacity, 0.75f, true);
            this.capacity = capacity;
        }
        
        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            // サイズがキャパシティを超えたら、最も古いエントリを削除
            return size() > capacity;
        }
        
        @Override
        public String toString() {
            StringBuilder sb = new StringBuilder();
            sb.append("LinkedHashMapLRUCache{capacity=").append(capacity).append(", size=").append(size()).append(", items=[");
            
            boolean first = true;
            for (Map.Entry<K, V> entry : entrySet()) {
                if (!first) {
                    sb.append(", ");
                }
                sb.append(entry.getKey()).append("=").append(entry.getValue());
                first = false;
            }
            
            sb.append("]}");
            return sb.toString();
        }
    }

    public static void main(String[] args) {
        // 自前実装のLRUキャッシュのテスト
        System.out.println("===== 自前実装のLRUキャッシュ =====");
        LRUCache<String, String> cache1 = new LRUCache<>(3);
        
        cache1.put("key1", "value1");
        System.out.println("key1を追加: " + cache1);
        
        cache1.put("key2", "value2");
        System.out.println("key2を追加: " + cache1);
        
        cache1.put("key3", "value3");
        System.out.println("key3を追加: " + cache1);
        
        cache1.get("key1"); // key1を最近使用したものとしてマーク
        System.out.println("key1にアクセス: " + cache1);
        
        cache1.put("key4", "value4"); // キャパシティを超えるため、最も長く使用されていないkey2が削除される
        System.out.println("key4を追加: " + cache1);
        
        System.out.println("key2の値: " + cache1.get("key2")); // null（削除済み）
        System.out.println("key3の値: " + cache1.get("key3")); // value3
        
        cache1.put("key3", "updated-value3"); // 既存のキーの値を更新
        System.out.println("key3を更新: " + cache1);
        
        // LinkedHashMapを使用したLRUキャッシュのテスト
        System.out.println("\n===== LinkedHashMapを使用したLRUキャッシュ =====");
        LinkedHashMapLRUCache<String, String> cache2 = new LinkedHashMapLRUCache<>(3);
        
        cache2.put("key1", "value1");
        System.out.println("key1を追加: " + cache2);
        
        cache2.put("key2", "value2");
        System.out.println("key2を追加: " + cache2);
        
        cache2.put("key3", "value3");
        System.out.println("key3を追加: " + cache2);
        
        cache2.get("key1"); // key1を最近使用したものとしてマーク
        System.out.println("key1にアクセス: " + cache2);
        
        cache2.put("key4", "value4"); // キャパシティを超えるため、最も長く使用されていないkey2が削除される
        System.out.println("key4を追加: " + cache2);
        
        System.out.println("key2の値: " + cache2.get("key2")); // null（削除済み）
        System.out.println("key3の値: " + cache2.get("key3")); // value3
        
        cache2.put("key3", "updated-value3"); // 既存のキーの値を更新
        System.out.println("key3を更新: " + cache2);
        
        // パフォーマンス比較
        System.out.println("\n===== パフォーマンス比較 =====");
        final int CAPACITY = 1000;
        final int OPERATIONS = 100000;
        
        // 自前実装のLRUキャッシュ
        LRUCache<Integer, String> customCache = new LRUCache<>(CAPACITY);
        long startTime1 = System.nanoTime();
        
        for (int i = 0; i < OPERATIONS; i++) {
            int key = i % (CAPACITY * 2); // キャパシティの2倍の範囲でキーを生成
            if (i % 3 == 0) {
                // 33%の確率でget操作
                customCache.get(key);
            } else {
                // 67%の確率でput操作
                customCache.put(key, "value-" + key);
            }
        }
        
        long endTime1 = System.nanoTime();
        long duration1 = (endTime1 - startTime1) / 1_000_000; // ミリ秒に変換
        
        // LinkedHashMapを使用したLRUキャッシュ
        LinkedHashMapLRUCache<Integer, String> linkedHashMapCache = new LinkedHashMapLRUCache<>(CAPACITY);
        long startTime2 = System.nanoTime();
        
        for (int i = 0; i < OPERATIONS; i++) {
            int key = i % (CAPACITY * 2); // キャパシティの2倍の範囲でキーを生成
            if (i % 3 == 0) {
                // 33%の確率でget操作
                linkedHashMapCache.get(key);
            } else {
                // 67%の確率でput操作
                linkedHashMapCache.put(key, "value-" + key);
            }
        }
        
        long endTime2 = System.nanoTime();
        long duration2 = (endTime2 - startTime2) / 1_000_000; // ミリ秒に変換
        
        System.out.println("自前実装のLRUキャッシュ: " + duration1 + " ms");
        System.out.println("LinkedHashMapを使用したLRUキャッシュ: " + duration2 + " ms");
        System.out.println("速度比: " + (double) duration1 / duration2);
        
        // 結論
        System.out.println("\n===== 結論 =====");
        System.out.println("1. LRUキャッシュは、最も長く使用されていないアイテムを削除するキャッシュ戦略");
        System.out.println("2. 自前実装では、HashMapと双方向リンクリストを組み合わせて実装可能");
        System.out.println("3. LinkedHashMapを使用すると、より簡潔かつ効率的に実装可能");
        System.out.println("4. キャッシュは、データベースアクセスや計算コストの高い処理の結果を保存するのに有用");
    }
}
```
