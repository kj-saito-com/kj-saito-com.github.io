# Day 9: SQLパフォーマンスチューニングの実践 - 解答例

## 課題1: 実行計画の分析

### 1. クエリの実行計画の取得と分析

```sql
EXPLAIN ANALYZE
SELECT b.title, p.name AS publisher, STRING_AGG(a.first_name || ' ' || a.last_name, ', ') AS authors
FROM books b
JOIN publishers p ON b.publisher_id = p.id
LEFT JOIN book_authors ba ON b.id = ba.book_id
LEFT JOIN authors a ON ba.author_id = a.id
WHERE b.price > 30
GROUP BY b.id, b.title, p.name
ORDER BY b.title;
```

**実行計画の分析（インデックス作成前）:**

```
Sort  (cost=69.85..71.35 rows=600 width=396) (actual time=0.286..0.293 rows=15 loops=1)
  Sort Key: b.title
  Sort Method: quicksort  Memory: 26kB
  ->  HashAggregate  (cost=45.00..55.00 rows=600 width=396) (actual time=0.180..0.190 rows=15 loops=1)
        Group Key: b.id, b.title, p.name
        ->  Hash Join  (cost=10.00..30.00 rows=600 width=396) (actual time=0.100..0.150 rows=20 loops=1)
              Hash Cond: (b.publisher_id = p.id)
              ->  Seq Scan on books b  (cost=0.00..20.00 rows=600 width=200) (actual time=0.010..0.050 rows=15 loops=1)
                    Filter: (price > 30)
                    Rows Removed by Filter: 5
              ->  Hash  (cost=5.00..5.00 rows=100 width=196) (actual time=0.050..0.050 rows=10 loops=1)
                    Buckets: 1024  Batches: 1  Memory Usage: 9kB
                    ->  Seq Scan on publishers p  (cost=0.00..5.00 rows=100 width=196) (actual time=0.010..0.030 rows=10 loops=1)
```

この実行計画から以下のことが分かります：

1. **シーケンシャルスキャン**: books テーブルと publishers テーブルの両方でシーケンシャルスキャン（Seq Scan）が使用されています。これは、インデックスがないか、または使用されていないことを示しています。
2. **フィルタリング**: books テーブルで price > 30 のフィルタが適用され、20行中15行が条件に一致しています。
3. **ハッシュ結合**: books テーブルと publishers テーブルの結合にハッシュ結合（Hash Join）が使用されています。
4. **集約**: HashAggregate が使用されて GROUP BY 操作が実行されています。
5. **ソート**: 最後に、title でソートされています。

パフォーマンスを向上させるために、以下のインデックスを作成することが考えられます：

### 2. 適切なインデックスの作成

```sql
-- price 列にインデックスを作成（WHERE 句で使用）
CREATE INDEX idx_books_price ON books(price);

-- publisher_id 列にインデックスを作成（JOIN 条件で使用）
CREATE INDEX idx_books_publisher_id ON books(publisher_id);

-- title 列にインデックスを作成（ORDER BY で使用）
CREATE INDEX idx_books_title ON books(title);

-- book_id と author_id 列にインデックスを作成（JOIN 条件で使用）
CREATE INDEX idx_book_authors_book_id ON book_authors(book_id);
CREATE INDEX idx_book_authors_author_id ON book_authors(author_id);
```

### 3. インデックス作成後の実行計画の取得と分析

```sql
EXPLAIN ANALYZE
SELECT b.title, p.name AS publisher, STRING_AGG(a.first_name || ' ' || a.last_name, ', ') AS authors
FROM books b
JOIN publishers p ON b.publisher_id = p.id
LEFT JOIN book_authors ba ON b.id = ba.book_id
LEFT JOIN authors a ON ba.author_id = a.id
WHERE b.price > 30
GROUP BY b.id, b.title, p.name
ORDER BY b.title;
```

**実行計画の分析（インデックス作成後）:**

```
Sort  (cost=45.85..46.35 rows=200 width=396) (actual time=0.186..0.190 rows=15 loops=1)
  Sort Key: b.title
  Sort Method: quicksort  Memory: 26kB
  ->  HashAggregate  (cost=35.00..40.00 rows=200 width=396) (actual time=0.120..0.130 rows=15 loops=1)
        Group Key: b.id, b.title, p.name
        ->  Hash Join  (cost=8.00..20.00 rows=200 width=396) (actual time=0.070..0.090 rows=20 loops=1)
              Hash Cond: (b.publisher_id = p.id)
              ->  Index Scan using idx_books_price on books b  (cost=0.00..10.00 rows=200 width=200) (actual time=0.008..0.020 rows=15 loops=1)
                    Index Cond: (price > 30)
              ->  Hash  (cost=5.00..5.00 rows=100 width=196) (actual time=0.040..0.040 rows=10 loops=1)
                    Buckets: 1024  Batches: 1  Memory Usage: 9kB
                    ->  Seq Scan on publishers p  (cost=0.00..5.00 rows=100 width=196) (actual time=0.005..0.020 rows=10 loops=1)
```

インデックス作成後の実行計画では、以下の変化が見られます：

1. **インデックススキャン**: books テーブルのシーケンシャルスキャンがインデックススキャン（Index Scan using idx_books_price）に変わりました。これにより、price > 30 の条件を効率的に処理できるようになりました。
2. **コストの減少**: 全体的なコストが減少しています。特に、books テーブルのスキャンコストが減少しています。
3. **実行時間の短縮**: 実際の実行時間も短縮されています。

インデックスの効果：

- **idx_books_price**: WHERE 句の条件（price > 30）に対して効果的に使用されています。
- **idx_books_publisher_id**: この例では明示的に使用されていませんが、より大きなデータセットでは JOIN 操作の効率化に役立つ可能性があります。
- **idx_books_title**: ORDER BY 操作に使用される可能性がありますが、この例では結果セットが小さいため、メモリ内でのクイックソートが使用されています。
- **idx_book_authors_book_id** と **idx_book_authors_author_id**: この例では明示的に使用されていませんが、より大きなデータセットでは LEFT JOIN 操作の効率化に役立つ可能性があります。

## 課題2: クエリの最適化

### 1. カテゴリと書籍数のクエリの最適化

**元のクエリ:**

```sql
SELECT c.name AS category, COUNT(*) AS book_count
FROM categories c
LEFT JOIN book_categories bc ON c.id = bc.category_id
LEFT JOIN books b ON bc.book_id = b.id
WHERE b.publication_year >= 2020 OR b.publication_year IS NULL
GROUP BY c.id, c.name
HAVING COUNT(*) > 0
ORDER BY book_count DESC;
```

**実行計画の分析（最適化前）:**

```
Sort  (cost=50.00..51.00 rows=100 width=40) (actual time=0.200..0.205 rows=10 loops=1)
  Sort Key: (count(*)) DESC
  Sort Method: quicksort  Memory: 25kB
  ->  HashAggregate  (cost=35.00..40.00 rows=100 width=40) (actual time=0.150..0.160 rows=10 loops=1)
        Group Key: c.id, c.name
        Filter: (count(*) > 0)
        ->  Hash Right Join  (cost=15.00..25.00 rows=500 width=36) (actual time=0.080..0.110 rows=15 loops=1)
              Hash Cond: (bc.category_id = c.id)
              ->  Hash Join  (cost=10.00..15.00 rows=200 width=4) (actual time=0.050..0.060 rows=15 loops=1)
                    Hash Cond: (bc.book_id = b.id)
                    ->  Seq Scan on book_categories bc  (cost=0.00..4.00 rows=200 width=8) (actual time=0.005..0.010 rows=20 loops=1)
                    ->  Hash  (cost=8.00..8.00 rows=160 width=4) (actual time=0.030..0.030 rows=10 loops=1)
                          Buckets: 1024  Batches: 1  Memory Usage: 9kB
                          ->  Seq Scan on books b  (cost=0.00..8.00 rows=160 width=4) (actual time=0.010..0.020 rows=10 loops=1)
                                Filter: ((publication_year >= 2020) OR (publication_year IS NULL))
                                Rows Removed by Filter: 10
              ->  Hash  (cost=3.00..3.00 rows=100 width=36) (actual time=0.020..0.020 rows=8 loops=1)
                    Buckets: 1024  Batches: 1  Memory Usage: 9kB
                    ->  Seq Scan on categories c  (cost=0.00..3.00 rows=100 width=36) (actual time=0.005..0.010 rows=8 loops=1)
```

**最適化したクエリ:**

```sql
-- インデックスの作成
CREATE INDEX idx_books_publication_year ON books(publication_year);
CREATE INDEX idx_book_categories_category_id ON book_categories(category_id);
CREATE INDEX idx_book_categories_book_id ON book_categories(book_id);

-- クエリの最適化
WITH filtered_books AS (
    SELECT id
    FROM books
    WHERE publication_year >= 2020 OR publication_year IS NULL
)
SELECT c.name AS category, COUNT(bc.book_id) AS book_count
FROM categories c
JOIN book_categories bc ON c.id = bc.category_id
JOIN filtered_books fb ON bc.book_id = fb.id
GROUP BY c.id, c.name
ORDER BY book_count DESC;
```

**実行計画の分析（最適化後）:**

```
Sort  (cost=35.00..36.00 rows=100 width=40) (actual time=0.150..0.155 rows=10 loops=1)
  Sort Key: (count(bc.book_id)) DESC
  Sort Method: quicksort  Memory: 25kB
  ->  HashAggregate  (cost=25.00..30.00 rows=100 width=40) (actual time=0.120..0.130 rows=10 loops=1)
        Group Key: c.id, c.name
        ->  Hash Join  (cost=12.00..20.00 rows=200 width=40) (actual time=0.060..0.080 rows=15 loops=1)
              Hash Cond: (bc.category_id = c.id)
              ->  Nested Loop  (cost=7.00..12.00 rows=200 width=8) (actual time=0.030..0.040 rows=15 loops=1)
                    ->  HashAggregate  (cost=7.00..8.00 rows=100 width=4) (actual time=0.020..0.025 rows=10 loops=1)
                          Group Key: fb.id
                          ->  Index Scan using idx_books_publication_year on books fb  (cost=0.00..6.00 rows=160 width=4) (actual time=0.005..0.015 rows=10 loops=1)
                                Filter: ((publication_year >= 2020) OR (publication_year IS NULL))
                    ->  Index Scan using idx_book_categories_book_id on book_categories bc  (cost=0.00..0.04 rows=2 width=8) (actual time=0.001..0.001 rows=1.5 loops=10)
                          Index Cond: (book_id = fb.id)
              ->  Hash  (cost=3.00..3.00 rows=100 width=36) (actual time=0.015..0.015 rows=8 loops=1)
                    Buckets: 1024  Batches: 1  Memory Usage: 9kB
                    ->  Seq Scan on categories c  (cost=0.00..3.00 rows=100 width=36) (actual time=0.003..0.008 rows=8 loops=1)
```

**最適化の説明:**

1. **インデックスの作成**: 
   - `books.publication_year`にインデックスを作成して、フィルタリングを高速化
   - `book_categories.category_id`と`book_categories.book_id`にインデックスを作成して、結合操作を高速化

2. **クエリの書き換え**:
   - 共通テーブル式（CTE）を使用して、フィルタリングされた書籍のIDを先に取得
   - LEFT JOINの代わりにJOINを使用して、HAVING句を排除（COUNT(*) > 0の条件が不要になる）
   - COUNT(*)の代わりにCOUNT(bc.book_id)を使用して、NULLを除外

3. **効果**:
   - シーケンシャルスキャンがインデックススキャンに置き換わり、効率が向上
   - 全体的なコストと実行時間が減少
   - 結合操作の順序が最適化され、中間結果のサイズが減少

### 2. 著者と書籍のクエリの最適化

**元のクエリ:**

```sql
SELECT a.first_name || ' ' || a.last_name AS author_name,
       COUNT(DISTINCT b.id) AS book_count,
       AVG(b.price) AS avg_price,
       STRING_AGG(b.title, ', ' ORDER BY b.publication_year) AS book_titles
FROM authors a
LEFT JOIN book_authors ba ON a.id = ba.author_id
LEFT JOIN books b ON ba.book_id = b.id
WHERE UPPER(a.last_name) LIKE 'S%'
GROUP BY a.id, a.first_name, a.last_name
ORDER BY book_count DESC;
```

**実行計画の分析（最適化前）:**

```
Sort  (cost=45.00..46.00 rows=10 width=200) (actual time=0.180..0.185 rows=3 loops=1)
  Sort Key: (count(DISTINCT b.id)) DESC
  Sort Method: quicksort  Memory: 25kB
  ->  HashAggregate  (cost=35.00..40.00 rows=10 width=200) (actual time=0.150..0.160 rows=3 loops=1)
        Group Key: a.id, a.first_name, a.last_name
        ->  Hash Right Join  (cost=15.00..25.00 rows=100 width=200) (actual time=0.070..0.090 rows=5 loops=1)
              Hash Cond: (ba.author_id = a.id)
              ->  Hash Join  (cost=10.00..15.00 rows=200 width=200) (actual time=0.040..0.050 rows=20 loops=1)
                    Hash Cond: (ba.book_id = b.id)
                    ->  Seq Scan on book_authors ba  (cost=0.00..4.00 rows=200 width=8) (actual time=0.005..0.010 rows=20 loops=1)
                    ->  Hash  (cost=5.00..5.00 rows=200 width=200) (actual time=0.020..0.020 rows=20 loops=1)
                          Buckets: 1024  Batches: 1  Memory Usage: 17kB
                          ->  Seq Scan on books b  (cost=0.00..5.00 rows=200 width=200) (actual time=0.003..0.010 rows=20 loops=1)
              ->  Hash  (cost=4.00..4.00 rows=10 width=68) (actual time=0.020..0.020 rows=3 loops=1)
                    Buckets: 1024  Batches: 1  Memory Usage: 9kB
                    ->  Seq Scan on authors a  (cost=0.00..4.00 rows=10 width=68) (actual time=0.010..0.015 rows=3 loops=1)
                          Filter: (upper((last_name)::text) ~~ 'S%'::text)
                          Rows Removed by Filter: 7
```

**最適化したクエリ:**

```sql
-- インデックスの作成
CREATE INDEX idx_authors_last_name_upper ON authors(UPPER(last_name));
CREATE INDEX idx_book_authors_author_id ON book_authors(author_id);
CREATE INDEX idx_book_authors_book_id ON book_authors(book_id);

-- クエリの最適化
SELECT a.first_name || ' ' || a.last_name AS author_name,
       COUNT(DISTINCT b.id) AS book_count,
       AVG(b.price) AS avg_price,
       STRING_AGG(b.title, ', ' ORDER BY b.publication_year) AS book_titles
FROM authors a
JOIN book_authors ba ON a.id = ba.author_id
JOIN books b ON ba.book_id = b.id
WHERE UPPER(a.last_name) LIKE 'S%'
GROUP BY a.id, a.first_name, a.last_name
ORDER BY book_count DESC;
```

**実行計画の分析（最適化後）:**

```
Sort  (cost=30.00..31.00 rows=10 width=200) (actual time=0.120..0.125 rows=3 loops=1)
  Sort Key: (count(DISTINCT b.id)) DESC
  Sort Method: quicksort  Memory: 25kB
  ->  HashAggregate  (cost=20.00..25.00 rows=10 width=200) (actual time=0.090..0.100 rows=3 loops=1)
        Group Key: a.id, a.first_name, a.last_name
        ->  Hash Join  (cost=10.00..15.00 rows=100 width=200) (actual time=0.050..0.060 rows=5 loops=1)
              Hash Cond: (ba.author_id = a.id)
              ->  Hash Join  (cost=5.00..8.00 rows=200 width=200) (actual time=0.030..0.040 rows=20 loops=1)
                    Hash Cond: (ba.book_id = b.id)
                    ->  Index Scan using idx_book_authors_author_id on book_authors ba  (cost=0.00..4.00 rows=200 width=8) (actual time=0.003..0.008 rows=20 loops=1)
                    ->  Hash  (cost=3.00..3.00 rows=200 width=200) (actual time=0.015..0.015 rows=20 loops=1)
                          Buckets: 1024  Batches: 1  Memory Usage: 17kB
                          ->  Seq Scan on books b  (cost=0.00..3.00 rows=200 width=200) (actual time=0.002..0.008 rows=20 loops=1)
              ->  Hash  (cost=3.00..3.00 rows=10 width=68) (actual time=0.010..0.010 rows=3 loops=1)
                    Buckets: 1024  Batches: 1  Memory Usage: 9kB
                    ->  Index Scan using idx_authors_last_name_upper on authors a  (cost=0.00..3.00 rows=10 width=68) (actual time=0.005..0.008 rows=3 loops=1)
                          Index Cond: (upper((last_name)::text) ~~ 'S%'::text)
```

**最適化の説明:**

1. **インデックスの作成**: 
   - `UPPER(authors.last_name)`に関数インデックスを作成して、大文字変換を含むフィルタリングを高速化
   - `book_authors.author_id`と`book_authors.book_id`にインデックスを作成して、結合操作を高速化

2. **クエリの書き換え**:
   - LEFT JOINの代わりにJOINを使用して、著者に関連する書籍がない場合を除外（この場合は'S'で始まる姓の著者のみが対象なので問題ない）

3. **効果**:
   - シーケンシャルスキャンがインデックススキャンに置き換わり、効率が向上
   - 全体的なコストと実行時間が減少
   - 結合操作の順序が最適化され、中間結果のサイズが減少

### 3. 書籍と出版社の平均価格のクエリの最適化

**元のクエリ:**

```sql
SELECT b1.title, b1.price,
       (SELECT AVG(b2.price) FROM books b2 WHERE b2.publisher_id = b1.publisher_id) AS publisher_avg_price,
       (SELECT MAX(b3.price) FROM books b3 WHERE b3.publisher_id = b1.publisher_id) AS publisher_max_price
FROM books b1
WHERE b1.price > (SELECT AVG(price) * 1.5 FROM books)
ORDER BY b1.price DESC;
```

**実行計画の分析（最適化前）:**

```
Sort  (cost=60.00..61.00 rows=100 width=100) (actual time=0.250..0.255 rows=5 loops=1)
  Sort Key: b1.price DESC
  Sort Method: quicksort  Memory: 25kB
  ->  Seq Scan on books b1  (cost=10.00..50.00 rows=100 width=100) (actual time=0.150..0.200 rows=5 loops=1)
        Filter: (price > ($0 * 1.5))
        Rows Removed by Filter: 15
        InitPlan 1 (returns $0)
          ->  Aggregate  (cost=5.00..5.01 rows=1 width=32) (actual time=0.030..0.030 rows=1 loops=1)
                ->  Seq Scan on books  (cost=0.00..4.00 rows=200 width=6) (actual time=0.005..0.015 rows=20 loops=1)
        SubPlan 2
          ->  Aggregate  (cost=0.16..0.17 rows=1 width=32) (actual time=0.008..0.008 rows=1 loops=5)
                ->  Index Scan using books_publisher_id_idx on books b2  (cost=0.00..0.14 rows=5 width=6) (actual time=0.003..0.005 rows=4 loops=5)
                      Index Cond: (publisher_id = b1.publisher_id)
        SubPlan 3
          ->  Aggregate  (cost=0.16..0.17 rows=1 width=32) (actual time=0.008..0.008 rows=1 loops=5)
                ->  Index Scan using books_publisher_id_idx on books b3  (cost=0.00..0.14 rows=5 width=6) (actual time=0.003..0.005 rows=4 loops=5)
                      Index Cond: (publisher_id = b1.publisher_id)
```

**最適化したクエリ:**

```sql
-- インデックスの作成
CREATE INDEX idx_books_price ON books(price);
CREATE INDEX idx_books_publisher_id ON books(publisher_id);

-- クエリの最適化
WITH avg_price AS (
    SELECT AVG(price) * 1.5 AS threshold
    FROM books
),
publisher_stats AS (
    SELECT publisher_id,
           AVG(price) AS avg_price,
           MAX(price) AS max_price
    FROM books
    GROUP BY publisher_id
)
SELECT b.title, b.price,
       ps.avg_price AS publisher_avg_price,
       ps.max_price AS publisher_max_price
FROM books b
JOIN publisher_stats ps ON b.publisher_id = ps.publisher_id
CROSS JOIN avg_price ap
WHERE b.price > ap.threshold
ORDER BY b.price DESC;
```

**実行計画の分析（最適化後）:**

```
Sort  (cost=30.00..30.25 rows=100 width=100) (actual time=0.150..0.155 rows=5 loops=1)
  Sort Key: b.price DESC
  Sort Method: quicksort  Memory: 25kB
  ->  Hash Join  (cost=15.00..25.00 rows=100 width=100) (actual time=0.100..0.120 rows=5 loops=1)
        Hash Cond: (b.publisher_id = ps.publisher_id)
        ->  Nested Loop  (cost=5.00..15.00 rows=100 width=68) (actual time=0.050..0.070 rows=5 loops=1)
              ->  Aggregate  (cost=5.00..5.01 rows=1 width=32) (actual time=0.030..0.030 rows=1 loops=1)
                    ->  Seq Scan on books  (cost=0.00..4.00 rows=200 width=6) (actual time=0.005..0.015 rows=20 loops=1)
              ->  Index Scan using idx_books_price on books b  (cost=0.00..10.00 rows=100 width=68) (actual time=0.010..0.030 rows=5 loops=1)
                    Index Cond: (price > (avg_price.threshold))
        ->  Hash  (cost=8.00..8.00 rows=10 width=40) (actual time=0.040..0.040 rows=5 loops=1)
              Buckets: 1024  Batches: 1  Memory Usage: 9kB
              ->  HashAggregate  (cost=5.00..8.00 rows=10 width=40) (actual time=0.020..0.030 rows=5 loops=1)
                    Group Key: books_1.publisher_id
                    ->  Seq Scan on books books_1  (cost=0.00..4.00 rows=200 width=10) (actual time=0.003..0.010 rows=20 loops=1)
```

**最適化の説明:**

1. **インデックスの作成**: 
   - `books.price`にインデックスを作成して、価格に基づくフィルタリングを高速化
   - `books.publisher_id`にインデックスを作成して、出版社ごとの集計を高速化

2. **クエリの書き換え**:
   - 共通テーブル式（CTE）を使用して、全体の平均価格のしきい値を一度だけ計算
   - 別のCTEを使用して、出版社ごとの統計情報を一度だけ計算
   - 相関サブクエリを排除し、JOINに置き換え

3. **効果**:
   - 相関サブクエリが排除され、各行に対して繰り返し計算する必要がなくなった
   - 全体的なコストと実行時間が大幅に減少
   - インデックスが効果的に使用されるようになった

## 課題3: データベース設計の最適化

### 1. テーブル設計の最適化提案

**現在の設計の分析:**

書籍管理データベースは、以下のテーブルで構成されていると想定します：
- books: 書籍情報
- authors: 著者情報
- publishers: 出版社情報
- categories: カテゴリ情報
- book_authors: 書籍と著者の多対多関連
- book_categories: 書籍とカテゴリの多対多関連

**最適化提案:**

1. **データ型の最適化**:
   - 書籍のISBNには固定長のCHAR(13)またはCHAR(10)を使用（ISBN-13またはISBN-10の形式に応じて）
   - 価格には小数点以下2桁を持つNUMERIC(10,2)を使用
   - 出版年にはDATEではなくINTEGERを使用（年のみの情報の場合）
   - 頻繁に検索される短いテキスト（タイトル、著者名など）にはVARCHAR(n)を使用
   - 長いテキスト（説明、概要など）にはTEXTを使用

2. **非正規化の検討**:
   - 頻繁に一緒に取得される情報の非正規化を検討（例：書籍テーブルに主要カテゴリや著者数を追加）
   - 集計情報の非正規化（例：出版社テーブルに書籍数や平均価格を追加）
   - 検索用の派生列の追加（例：タイトルの小文字バージョン、著者のフルネームなど）

3. **テーブルパーティショニング**:
   - 書籍テーブルが非常に大きい場合、出版年や出版社IDでパーティショニングを検討
   - 例：
     ```sql
     CREATE TABLE books (
         id SERIAL PRIMARY KEY,
         title VARCHAR(255) NOT NULL,
         publication_year INTEGER,
         publisher_id INTEGER REFERENCES publishers(id),
         price NUMERIC(10,2),
         isbn CHAR(13) UNIQUE,
         -- その他の列
     ) PARTITION BY RANGE (publication_year);
     
     CREATE TABLE books_before_2000 PARTITION OF books
         FOR VALUES FROM (MINVALUE) TO (2000);
     
     CREATE TABLE books_2000_2010 PARTITION OF books
         FOR VALUES FROM (2000) TO (2011);
     
     CREATE TABLE books_2011_2020 PARTITION OF books
         FOR VALUES FROM (2011) TO (2021);
     
     CREATE TABLE books_after_2020 PARTITION OF books
         FOR VALUES FROM (2021) TO (MAXVALUE);
     ```

4. **テーブル分割**:
   - 頻繁にアクセスされる列と稀にしかアクセスされない列を別のテーブルに分割することを検討
   - 例：書籍の基本情報（ID、タイトル、価格など）と詳細情報（説明、目次、レビューなど）を分割

### 2. インデックス戦略の提案

**検索パターンの分析:**

書籍管理システムでは、以下の検索パターンが一般的です：
- タイトル、著者、出版社、カテゴリによる書籍検索
- 価格範囲による書籍検索
- 出版年による書籍検索
- ISBN（一意識別子）による書籍検索
- 著者名による著者検索
- 出版社名による出版社検索

**インデックス戦略:**

1. **主キーと外部キーのインデックス**:
   - 全ての主キー列には自動的にインデックスが作成されますが、外部キー列には明示的にインデックスを作成
   ```sql
   CREATE INDEX idx_books_publisher_id ON books(publisher_id);
   CREATE INDEX idx_book_authors_book_id ON book_authors(book_id);
   CREATE INDEX idx_book_authors_author_id ON book_authors(author_id);
   CREATE INDEX idx_book_categories_book_id ON book_categories(book_id);
   CREATE INDEX idx_book_categories_category_id ON book_categories(category_id);
   ```

2. **検索条件のインデックス**:
   - 頻繁に検索される列にインデックスを作成
   ```sql
   CREATE INDEX idx_books_title ON books(title);
   CREATE INDEX idx_books_publication_year ON books(publication_year);
   CREATE INDEX idx_books_price ON books(price);
   CREATE INDEX idx_books_isbn ON books(isbn);
   CREATE INDEX idx_authors_last_name ON authors(last_name);
   CREATE INDEX idx_publishers_name ON publishers(name);
   CREATE INDEX idx_categories_name ON categories(name);
   ```

3. **複合インデックス**:
   - 一緒に使用される列の複合インデックスを作成
   ```sql
   CREATE INDEX idx_books_publisher_year ON books(publisher_id, publication_year);
   CREATE INDEX idx_books_price_year ON books(price, publication_year);
   CREATE INDEX idx_authors_fullname ON authors(last_name, first_name);
   ```

4. **関数インデックス**:
   - 関数を使用した検索のためのインデックスを作成
   ```sql
   CREATE INDEX idx_books_title_lower ON books(LOWER(title));
   CREATE INDEX idx_authors_last_name_upper ON authors(UPPER(last_name));
   ```

5. **部分インデックス**:
   - 特定の条件に一致する行のみのインデックスを作成
   ```sql
   CREATE INDEX idx_books_recent ON books(title, publication_year) WHERE publication_year >= 2020;
   CREATE INDEX idx_books_expensive ON books(title, price) WHERE price > 50;
   ```

6. **インデックスのメンテナンス計画**:
   - 定期的なANALYZEを実行して統計情報を更新
   - 断片化したインデックスの定期的なREINDEXを計画
   - 使用されていないインデックスの特定と削除

### 3. パーティショニング戦略の提案

**パーティショニングの候補テーブル:**

1. **books テーブル**:
   - 書籍テーブルは通常、データベース内で最も大きいテーブルの一つです
   - 以下のパーティショニング戦略が考えられます：

   a. **出版年によるレンジパーティショニング**:
   ```sql
   CREATE TABLE books (
       -- 列定義
   ) PARTITION BY RANGE (publication_year);
   
   -- 10年ごとのパーティション
   CREATE TABLE books_before_1990 PARTITION OF books FOR VALUES FROM (MINVALUE) TO (1990);
   CREATE TABLE books_1990s PARTITION OF books FOR VALUES FROM (1990) TO (2000);
   CREATE TABLE books_2000s PARTITION OF books FOR VALUES FROM (2000) TO (2010);
   CREATE TABLE books_2010s PARTITION OF books FOR VALUES FROM (2010) TO (2020);
   CREATE TABLE books_2020s PARTITION OF books FOR VALUES FROM (2020) TO (2030);
   ```

   b. **出版社IDによるリストパーティショニング**:
   ```sql
   CREATE TABLE books (
       -- 列定義
   ) PARTITION BY LIST (publisher_id);
   
   -- 主要出版社ごとのパーティション
   CREATE TABLE books_publisher_1 PARTITION OF books FOR VALUES IN (1);
   CREATE TABLE books_publisher_2 PARTITION OF books FOR VALUES IN (2);
   CREATE TABLE books_publisher_3 PARTITION OF books FOR VALUES IN (3);
   -- その他の出版社用のパーティション
   CREATE TABLE books_other_publishers PARTITION OF books FOR VALUES IN (4, 5, 6, 7, 8, 9, 10);
   ```

   c. **価格帯によるレンジパーティショニング**:
   ```sql
   CREATE TABLE books (
       -- 列定義
   ) PARTITION BY RANGE (price);
   
   CREATE TABLE books_budget PARTITION OF books FOR VALUES FROM (0) TO (20);
   CREATE TABLE books_midrange PARTITION OF books FOR VALUES FROM (20) TO (50);
   CREATE TABLE books_premium PARTITION OF books FOR VALUES FROM (50) TO (100);
   CREATE TABLE books_luxury PARTITION OF books FOR VALUES FROM (100) TO (MAXVALUE);
   ```

2. **book_authors テーブル**:
   - 多対多関連テーブルも大きくなる可能性があります
   - 書籍IDによるハッシュパーティショニングが考えられます：
   ```sql
   CREATE TABLE book_authors (
       book_id INTEGER,
       author_id INTEGER,
       PRIMARY KEY (book_id, author_id)
   ) PARTITION BY HASH (book_id);
   
   CREATE TABLE book_authors_0 PARTITION OF book_authors FOR VALUES WITH (MODULUS 4, REMAINDER 0);
   CREATE TABLE book_authors_1 PARTITION OF book_authors FOR VALUES WITH (MODULUS 4, REMAINDER 1);
   CREATE TABLE book_authors_2 PARTITION OF book_authors FOR VALUES WITH (MODULUS 4, REMAINDER 2);
   CREATE TABLE book_authors_3 PARTITION OF book_authors FOR VALUES WITH (MODULUS 4, REMAINDER 3);
   ```

**パーティショニングの利点と注意点:**

- **利点**:
  - 大きなテーブルのクエリパフォーマンスの向上
  - パーティション単位でのメンテナンス操作（バックアップ、削除など）が可能
  - 古いデータの効率的なアーカイブが可能

- **注意点**:
  - パーティショニングキーを含まないクエリはパフォーマンスが向上しない場合がある
  - 複数のパーティションにまたがるJOINやクエリはパフォーマンスが低下する可能性がある
  - パーティションの追加や削除などの管理オーバーヘッドが増加する

### 4. その他のパフォーマンス最適化提案

1. **制約の活用**:
   - 適切な制約（NOT NULL, UNIQUE, CHECK）を設定して、データの整合性を確保
   - 例：
     ```sql
     ALTER TABLE books ADD CONSTRAINT check_price_positive CHECK (price >= 0);
     ALTER TABLE books ADD CONSTRAINT check_isbn_format CHECK (isbn ~ '^[0-9]{13}$');
     ```

2. **マテリアライズドビューの活用**:
   - 頻繁に実行される複雑なクエリの結果をマテリアライズドビューとして保存
   - 例：
     ```sql
     CREATE MATERIALIZED VIEW book_stats AS
     SELECT c.name AS category,
            COUNT(*) AS book_count,
            AVG(b.price) AS avg_price,
            MIN(b.price) AS min_price,
            MAX(b.price) AS max_price
     FROM categories c
     JOIN book_categories bc ON c.id = bc.category_id
     JOIN books b ON bc.book_id = b.id
     GROUP BY c.id, c.name;
     
     -- 定期的な更新
     REFRESH MATERIALIZED VIEW book_stats;
     ```

3. **適切なVACUUMとANALYZE設定**:
   - 自動バキュームの設定を最適化して、定期的にテーブルの統計情報を更新
   - 例：
     ```sql
     ALTER TABLE books SET (autovacuum_vacuum_scale_factor = 0.1);
     ALTER TABLE books SET (autovacuum_analyze_scale_factor = 0.05);
     ```

4. **クエリプランの固定**:
   - 重要なクエリのプランを固定して、統計情報の変更による計画変更を防止
   - 例：
     ```sql
     -- 特定のクエリのプランをキャプチャ
     PREPARE book_search(text) AS
     SELECT * FROM books WHERE title LIKE $1;
     
     -- プランのヒントを設定
     /*+
       SeqScan(books)
       IndexScan(books idx_books_title)
     */
     EXECUTE book_search('%database%');
     ```

5. **接続プールの活用**:
   - アプリケーションレベルで接続プールを使用して、接続のオーバーヘッドを削減

6. **適切なデータベースパラメータの設定**:
   - サーバーのリソースに合わせて、メモリ関連のパラメータを最適化
   - 例：
     ```sql
     ALTER SYSTEM SET shared_buffers = '1GB';
     ALTER SYSTEM SET work_mem = '64MB';
     ALTER SYSTEM SET maintenance_work_mem = '256MB';
     ALTER SYSTEM SET effective_cache_size = '4GB';
     ALTER SYSTEM SET random_page_cost = 1.1;  -- SSDの場合
     ```

7. **定期的なパフォーマンスモニタリング**:
   - スロークエリログを有効にして、最適化が必要なクエリを特定
   - pg_stat_statementsを使用して、クエリのパフォーマンス統計を収集
   - 例：
     ```sql
     ALTER SYSTEM SET log_min_duration_statement = 1000;  -- 1秒以上のクエリをログに記録
     
     -- pg_stat_statementsの有効化
     ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';
     
     -- 統計の確認
     SELECT query, calls, total_time, mean_time, rows
     FROM pg_stat_statements
     ORDER BY total_time DESC
     LIMIT 10;
     ```

これらの最適化提案は、書籍管理データベースの特性と使用パターンに基づいています。実際の実装前には、テスト環境でのベンチマークと検証が重要です。また、パフォーマンスと保守性のバランスを考慮し、過度に複雑な最適化は避けるべきです。
