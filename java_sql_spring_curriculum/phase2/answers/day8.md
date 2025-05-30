# Day 8: 集計関数と分析関数の実践的活用 - 解答例

## 課題1: 基本的な集計と分析

### 1. 出版社ごとの書籍数、平均価格、最高価格、最低価格を計算するクエリ

```sql
SELECT 
    p.name AS publisher_name,
    COUNT(b.id) AS book_count,
    ROUND(AVG(b.price)::numeric, 2) AS average_price,
    MAX(b.price) AS max_price,
    MIN(b.price) AS min_price,
    SUM(b.price) AS total_price
FROM 
    publishers p
LEFT JOIN 
    books b ON p.id = b.publisher_id
GROUP BY 
    p.id, p.name
ORDER BY 
    book_count DESC, publisher_name;
```

### 2. 出版年ごとの書籍数と平均価格を計算し、書籍数の多い年順に並べるクエリ

```sql
SELECT 
    b.publication_year,
    COUNT(b.id) AS book_count,
    ROUND(AVG(b.price)::numeric, 2) AS average_price,
    MIN(b.price) AS min_price,
    MAX(b.price) AS max_price
FROM 
    books b
WHERE 
    b.publication_year IS NOT NULL
GROUP BY 
    b.publication_year
ORDER BY 
    book_count DESC, b.publication_year;
```

### 3. カテゴリごとの書籍数と平均価格を計算し、書籍が3冊以上あるカテゴリのみを表示するクエリ

```sql
SELECT 
    c.name AS category_name,
    COUNT(b.id) AS book_count,
    ROUND(AVG(b.price)::numeric, 2) AS average_price,
    MIN(b.price) AS min_price,
    MAX(b.price) AS max_price
FROM 
    categories c
JOIN 
    book_categories bc ON c.id = bc.category_id
JOIN 
    books b ON bc.book_id = b.id
GROUP BY 
    c.id, c.name
HAVING 
    COUNT(b.id) >= 3
ORDER BY 
    book_count DESC, category_name;
```

### 4. 著者ごとの書籍数と平均価格を計算し、共著書籍の場合は各著者にカウントするクエリ

```sql
SELECT 
    a.first_name || ' ' || a.last_name AS author_name,
    COUNT(b.id) AS book_count,
    ROUND(AVG(b.price)::numeric, 2) AS average_price,
    STRING_AGG(b.title, ', ' ORDER BY b.publication_year, b.title) AS book_titles
FROM 
    authors a
JOIN 
    book_authors ba ON a.id = ba.author_id
JOIN 
    books b ON ba.book_id = b.id
GROUP BY 
    a.id, a.first_name, a.last_name
ORDER BY 
    book_count DESC, author_name;
```

## 課題2: 高度な集計と分析

### 1. 書籍の価格を4分位（四分位）に分け、各分位ごとの書籍数と平均価格を計算するクエリ

```sql
WITH price_quartiles AS (
    SELECT 
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY price) AS q1,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS q2,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY price) AS q3
    FROM books
),
book_with_quartile AS (
    SELECT 
        b.id,
        b.title,
        b.price,
        CASE 
            WHEN b.price <= pq.q1 THEN 1
            WHEN b.price <= pq.q2 THEN 2
            WHEN b.price <= pq.q3 THEN 3
            ELSE 4
        END AS quartile
    FROM 
        books b
    CROSS JOIN 
        price_quartiles pq
)
SELECT 
    quartile,
    CASE 
        WHEN quartile = 1 THEN 'First Quartile (0-25%)'
        WHEN quartile = 2 THEN 'Second Quartile (25-50%)'
        WHEN quartile = 3 THEN 'Third Quartile (50-75%)'
        WHEN quartile = 4 THEN 'Fourth Quartile (75-100%)'
    END AS quartile_name,
    COUNT(*) AS book_count,
    ROUND(AVG(price)::numeric, 2) AS average_price,
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM 
    book_with_quartile
GROUP BY 
    quartile
ORDER BY 
    quartile;
```

### 2. 各書籍の価格と、同じカテゴリ内での価格ランキング（1位、2位、...）を表示するクエリ

```sql
SELECT 
    c.name AS category_name,
    b.title,
    b.price,
    RANK() OVER (PARTITION BY c.id ORDER BY b.price DESC) AS price_rank,
    COUNT(*) OVER (PARTITION BY c.id) AS total_books_in_category,
    ROUND((b.price / AVG(b.price) OVER (PARTITION BY c.id) * 100)::numeric - 100, 2) AS percent_above_avg
FROM 
    books b
JOIN 
    book_categories bc ON b.id = bc.book_id
JOIN 
    categories c ON bc.category_id = c.id
ORDER BY 
    category_name, price_rank;
```

### 3. 各出版社について、出版年ごとの書籍数の累積合計を計算するクエリ

```sql
WITH publisher_yearly_books AS (
    SELECT 
        p.id AS publisher_id,
        p.name AS publisher_name,
        b.publication_year,
        COUNT(*) AS yearly_book_count
    FROM 
        publishers p
    JOIN 
        books b ON p.id = b.publisher_id
    WHERE 
        b.publication_year IS NOT NULL
    GROUP BY 
        p.id, p.name, b.publication_year
)
SELECT 
    publisher_name,
    publication_year,
    yearly_book_count,
    SUM(yearly_book_count) OVER (PARTITION BY publisher_id ORDER BY publication_year) AS cumulative_book_count
FROM 
    publisher_yearly_books
ORDER BY 
    publisher_name, publication_year;
```

### 4. 各著者について、執筆した書籍の価格の移動平均（3冊単位）を計算するクエリ

```sql
WITH author_books AS (
    SELECT 
        a.id AS author_id,
        a.first_name || ' ' || a.last_name AS author_name,
        b.title,
        b.publication_year,
        b.price,
        ROW_NUMBER() OVER (PARTITION BY a.id ORDER BY b.publication_year, b.id) AS book_seq
    FROM 
        authors a
    JOIN 
        book_authors ba ON a.id = ba.author_id
    JOIN 
        books b ON ba.book_id = b.id
)
SELECT 
    author_name,
    title,
    publication_year,
    price,
    ROUND(AVG(price) OVER (PARTITION BY author_id ORDER BY book_seq ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)::numeric, 2) AS moving_avg_3books,
    COUNT(*) OVER (PARTITION BY author_id) AS total_books
FROM 
    author_books
ORDER BY 
    author_name, book_seq;
```

### 5. 各書籍について、同じ出版社の前の書籍と次の書籍（出版年順）のタイトルと価格を表示するクエリ

```sql
SELECT 
    p.name AS publisher_name,
    b.title,
    b.publication_year,
    b.price,
    LAG(b.title) OVER (PARTITION BY b.publisher_id ORDER BY b.publication_year, b.id) AS prev_book_title,
    LAG(b.price) OVER (PARTITION BY b.publisher_id ORDER BY b.publication_year, b.id) AS prev_book_price,
    LEAD(b.title) OVER (PARTITION BY b.publisher_id ORDER BY b.publication_year, b.id) AS next_book_title,
    LEAD(b.price) OVER (PARTITION BY b.publisher_id ORDER BY b.publication_year, b.id) AS next_book_price
FROM 
    books b
JOIN 
    publishers p ON b.publisher_id = p.id
ORDER BY 
    p.name, b.publication_year, b.id;
```

## 解答の解説

### 課題1の解説

#### 1. 出版社ごとの書籍数と価格統計

このクエリでは、LEFT JOINを使用して、書籍が存在しない出版社も結果に含めています。基本的な集計関数（COUNT, AVG, MAX, MIN, SUM）を使用して、各出版社の書籍に関する統計情報を計算しています。ROUND関数とキャスト（::numeric）を使用して、平均価格を小数点以下2桁に丸めています。

#### 2. 出版年ごとの書籍数と平均価格

このクエリでは、出版年ごとにグループ化し、各年の書籍数と価格統計を計算しています。WHERE句を使用して、出版年がNULLの書籍を除外しています。結果は書籍数の多い順にソートされています。

#### 3. 書籍が3冊以上あるカテゴリ

このクエリでは、カテゴリごとにグループ化し、HAVING句を使用して書籍数が3冊以上のカテゴリのみをフィルタリングしています。JOINを使用して、カテゴリ、書籍カテゴリ中間テーブル、書籍テーブルを結合しています。

#### 4. 著者ごとの書籍数と平均価格

このクエリでは、著者ごとにグループ化し、各著者が関わった書籍の数と平均価格を計算しています。共著書籍の場合、各著者に対して個別にカウントされます。STRING_AGG関数を使用して、各著者が執筆した書籍のタイトルを連結しています。

### 課題2の解説

#### 1. 価格の四分位数分析

このクエリでは、共通テーブル式（CTE）を2つ使用しています。最初のCTE（price_quartiles）では、PERCENTILE_CONT関数を使用して価格の四分位数（Q1, Q2, Q3）を計算しています。2つ目のCTE（book_with_quartile）では、各書籍を適切な四分位に分類しています。最後に、四分位ごとにグループ化して統計情報を計算しています。

#### 2. カテゴリ内での価格ランキング

このクエリでは、RANK関数を使用して、各カテゴリ内で価格の高い順に書籍をランク付けしています。PARTITION BY句を使用して、カテゴリごとに独立したランキングを作成しています。また、各カテゴリの平均価格と比較した割合も計算しています。

#### 3. 出版社ごとの累積書籍数

このクエリでは、共通テーブル式（CTE）を使用して、各出版社の年ごとの書籍数を計算しています。その後、ウィンドウ関数のSUM ... OVER (PARTITION BY ... ORDER BY ...)を使用して、累積合計を計算しています。

#### 4. 著者ごとの書籍価格の移動平均

このクエリでは、共通テーブル式（CTE）を使用して、各著者の書籍を出版年順に並べ、連番（book_seq）を割り当てています。その後、ウィンドウ関数のAVG ... OVER (... ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)を使用して、3冊単位（前の書籍、現在の書籍、次の書籍）の移動平均を計算しています。

#### 5. 前後の書籍情報の表示

このクエリでは、LAG関数とLEAD関数を使用して、同じ出版社の前の書籍と次の書籍（出版年順）の情報を取得しています。PARTITION BY句を使用して、出版社ごとに独立した前後関係を作成しています。

### A5M2での実行のポイント

1. **共通テーブル式（CTE）の活用**
   - WITH句を使用した共通テーブル式は、複雑なクエリを読みやすく、管理しやすくします
   - A5M2では、CTEの結果を個別に確認することはできませんが、クエリの構造化に役立ちます

2. **ウィンドウ関数の使用**
   - PARTITION BY句とORDER BY句を適切に組み合わせることで、意図した結果を得ることができます
   - ウィンドウフレーム（ROWS BETWEEN ... AND ...）を使用して、計算の範囲を制御できます

3. **結果の検証**
   - 複雑な集計や分析を行う場合は、小さなデータセットで結果を手計算と比較することが有効です
   - 特に、累積計算や移動平均などの結果が期待通りかを確認することが重要です

4. **パフォーマンスの考慮**
   - 大量のデータに対して複雑な分析を行う場合、クエリの最適化が重要になります
   - 特に、ウィンドウ関数を多用する場合は、インデックスの活用やクエリの構造化を検討します
