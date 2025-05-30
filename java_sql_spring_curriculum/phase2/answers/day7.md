# Day 7: 複雑なJOINとサブクエリの活用 - 解答例

## 課題1: 複雑なJOIN操作の実践

### 1. 各出版社ごとに、出版した書籍の数、著者の数（重複なし）、カテゴリの数（重複なし）を取得するクエリ

```sql
SELECT 
    p.name AS publisher_name,
    COUNT(DISTINCT b.id) AS book_count,
    COUNT(DISTINCT a.id) AS author_count,
    COUNT(DISTINCT c.id) AS category_count
FROM 
    publishers p
LEFT JOIN 
    books b ON p.id = b.publisher_id
LEFT JOIN 
    book_authors ba ON b.id = ba.book_id
LEFT JOIN 
    authors a ON ba.author_id = a.id
LEFT JOIN 
    book_categories bc ON b.id = bc.book_id
LEFT JOIN 
    categories c ON bc.category_id = c.id
GROUP BY 
    p.id, p.name
ORDER BY 
    book_count DESC, publisher_name;
```

### 2. 各著者ごとに、執筆した書籍のタイトル一覧と、共著者の名前一覧を取得するクエリ

```sql
SELECT 
    a.first_name || ' ' || a.last_name AS author_name,
    STRING_AGG(DISTINCT b.title, ', ') AS book_titles,
    STRING_AGG(DISTINCT 
        CASE 
            WHEN co.id != a.id THEN co.first_name || ' ' || co.last_name 
            ELSE NULL 
        END, 
        ', ' ORDER BY co.first_name, co.last_name) AS co_authors
FROM 
    authors a
LEFT JOIN 
    book_authors ba ON a.id = ba.author_id
LEFT JOIN 
    books b ON ba.book_id = b.id
LEFT JOIN 
    book_authors ba2 ON b.id = ba2.book_id
LEFT JOIN 
    authors co ON ba2.author_id = co.id
GROUP BY 
    a.id, a.first_name, a.last_name
ORDER BY 
    author_name;
```

### 3. 書籍と著者と出版社とカテゴリを全て結合し、書籍が存在しない著者や、書籍が存在しないカテゴリも含めて取得するクエリ

```sql
SELECT 
    b.title AS book_title,
    a.first_name || ' ' || a.last_name AS author_name,
    p.name AS publisher_name,
    c.name AS category_name
FROM 
    books b
FULL JOIN 
    book_authors ba ON b.id = ba.book_id
FULL JOIN 
    authors a ON ba.author_id = a.id
FULL JOIN 
    publishers p ON b.publisher_id = p.id
FULL JOIN 
    book_categories bc ON b.id = bc.book_id
FULL JOIN 
    categories c ON bc.category_id = c.id
ORDER BY 
    book_title NULLS LAST, 
    author_name NULLS LAST, 
    publisher_name NULLS LAST, 
    category_name NULLS LAST;
```

### 4. 自己結合を使用して、同じ出版社から出版された他の書籍のリストを各書籍ごとに取得するクエリ

```sql
SELECT 
    b1.title AS book_title,
    p.name AS publisher_name,
    STRING_AGG(b2.title, ', ' ORDER BY b2.publication_year, b2.title) AS other_books_by_same_publisher
FROM 
    books b1
JOIN 
    publishers p ON b1.publisher_id = p.id
JOIN 
    books b2 ON b1.publisher_id = b2.publisher_id AND b1.id != b2.id
GROUP BY 
    b1.id, b1.title, p.name
ORDER BY 
    p.name, b1.title;
```

## 課題2: サブクエリの活用

### 1. 平均価格より高い書籍を、カテゴリごとに取得するクエリ（サブクエリを使用）

```sql
SELECT 
    c.name AS category_name,
    b.title,
    b.price
FROM 
    books b
JOIN 
    book_categories bc ON b.id = bc.book_id
JOIN 
    categories c ON bc.category_id = c.id
WHERE 
    b.price > (SELECT AVG(price) FROM books)
ORDER BY 
    c.name, b.price DESC;
```

### 2. 3冊以上の書籍を執筆している著者の一覧と、その著者が執筆した最新の書籍を取得するクエリ

```sql
WITH prolific_authors AS (
    SELECT 
        a.id,
        a.first_name || ' ' || a.last_name AS author_name,
        COUNT(DISTINCT ba.book_id) AS book_count
    FROM 
        authors a
    JOIN 
        book_authors ba ON a.id = ba.author_id
    GROUP BY 
        a.id, a.first_name, a.last_name
    HAVING 
        COUNT(DISTINCT ba.book_id) >= 3
),
latest_books AS (
    SELECT 
        a.id AS author_id,
        b.title,
        b.publication_year,
        ROW_NUMBER() OVER (PARTITION BY a.id ORDER BY b.publication_year DESC, b.title) AS rn
    FROM 
        authors a
    JOIN 
        book_authors ba ON a.id = ba.author_id
    JOIN 
        books b ON ba.book_id = b.id
    WHERE 
        a.id IN (SELECT id FROM prolific_authors)
)
SELECT 
    pa.author_name,
    pa.book_count,
    lb.title AS latest_book,
    lb.publication_year
FROM 
    prolific_authors pa
JOIN 
    latest_books lb ON pa.id = lb.author_id AND lb.rn = 1
ORDER BY 
    pa.book_count DESC, pa.author_name;
```

### 3. 各カテゴリにおいて、最も高価な書籍と最も安価な書籍を取得するクエリ

```sql
WITH category_price_stats AS (
    SELECT 
        c.id AS category_id,
        c.name AS category_name,
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
)
SELECT 
    cps.category_name,
    min_book.title AS cheapest_book,
    min_book.price AS min_price,
    max_book.title AS most_expensive_book,
    max_book.price AS max_price
FROM 
    category_price_stats cps
JOIN (
    SELECT 
        c.id AS category_id,
        b.title,
        b.price
    FROM 
        books b
    JOIN 
        book_categories bc ON b.id = bc.book_id
    JOIN 
        categories c ON bc.category_id = c.id
    WHERE 
        (c.id, b.price) IN (
            SELECT 
                c2.id, 
                MIN(b2.price)
            FROM 
                categories c2
            JOIN 
                book_categories bc2 ON c2.id = bc2.category_id
            JOIN 
                books b2 ON bc2.book_id = b2.id
            GROUP BY 
                c2.id
        )
) min_book ON cps.category_id = min_book.category_id
JOIN (
    SELECT 
        c.id AS category_id,
        b.title,
        b.price
    FROM 
        books b
    JOIN 
        book_categories bc ON b.id = bc.book_id
    JOIN 
        categories c ON bc.category_id = c.id
    WHERE 
        (c.id, b.price) IN (
            SELECT 
                c2.id, 
                MAX(b2.price)
            FROM 
                categories c2
            JOIN 
                book_categories bc2 ON c2.id = bc2.category_id
            JOIN 
                books b2 ON bc2.book_id = b2.id
            GROUP BY 
                c2.id
        )
) max_book ON cps.category_id = max_book.category_id
ORDER BY 
    cps.category_name;
```

### 4. 書籍を1冊も出版していない出版社を取得するクエリ（EXISTS演算子を使用）

```sql
SELECT 
    p.id,
    p.name,
    p.address,
    p.phone,
    p.email,
    p.website
FROM 
    publishers p
WHERE 
    NOT EXISTS (
        SELECT 1
        FROM books b
        WHERE b.publisher_id = p.id
    )
ORDER BY 
    p.name;
```

### 5. 各出版社の平均書籍価格と、全体の平均価格との差を計算するクエリ

```sql
SELECT 
    p.name AS publisher_name,
    COUNT(b.id) AS book_count,
    ROUND(AVG(b.price)::numeric, 2) AS avg_price,
    ROUND((AVG(b.price) - (SELECT AVG(price) FROM books))::numeric, 2) AS diff_from_overall_avg,
    CASE 
        WHEN AVG(b.price) > (SELECT AVG(price) FROM books) THEN 'Above Average'
        WHEN AVG(b.price) < (SELECT AVG(price) FROM books) THEN 'Below Average'
        ELSE 'Average'
    END AS price_category
FROM 
    publishers p
JOIN 
    books b ON p.id = b.publisher_id
GROUP BY 
    p.id, p.name
ORDER BY 
    diff_from_overall_avg DESC;
```

## 解答の解説

### 課題1の解説

#### 1. 各出版社ごとの書籍数、著者数、カテゴリ数

このクエリでは、LEFT JOINを使用して、書籍が存在しない出版社も結果に含めています。COUNT(DISTINCT ...)を使用することで、重複を排除しています。例えば、同じ著者が複数の書籍を執筆している場合でも、その著者は1人としてカウントされます。

#### 2. 各著者ごとの書籍タイトル一覧と共著者一覧

このクエリでは、自己結合（著者テーブルを2回参照）を使用して、各著者とその共著者を関連付けています。STRING_AGG関数を使用して、書籍タイトルと共著者名を連結しています。CASE式を使用して、著者自身が共著者リストに含まれないようにしています。

#### 3. 書籍、著者、出版社、カテゴリの完全外部結合

このクエリでは、FULL JOINを使用して、関連するレコードが存在しない場合でも全てのエンティティを結果に含めています。ORDER BY句のNULLS LAST指定により、NULL値を持つレコードを最後に表示しています。

#### 4. 同じ出版社の他の書籍リスト

このクエリでは、自己結合（書籍テーブルを2回参照）を使用して、同じ出版社から出版された他の書籍を関連付けています。b1.id != b2.idの条件により、書籍自身が結果に含まれないようにしています。STRING_AGG関数を使用して、他の書籍タイトルを連結しています。

### 課題2の解説

#### 1. 平均価格より高い書籍

このクエリでは、WHERE句でスカラーサブクエリを使用して、全体の平均価格を計算し、その値より高い価格の書籍のみを抽出しています。

#### 2. 3冊以上執筆している著者と最新書籍

このクエリでは、共通テーブル式（CTE）を2つ使用しています。1つ目のCTEでは、3冊以上の書籍を執筆している著者を特定し、2つ目のCTEでは、ウィンドウ関数（ROW_NUMBER）を使用して各著者の書籍を出版年の降順でランク付けしています。最後に、ランクが1の書籍（最新の書籍）のみを選択しています。

#### 3. 各カテゴリの最高価格と最低価格の書籍

このクエリでは、共通テーブル式（CTE）と相関サブクエリを組み合わせて使用しています。CTEでは各カテゴリの最高価格と最低価格を計算し、その後2つのサブクエリを使用して、それぞれの価格に一致する書籍を特定しています。

#### 4. 書籍を出版していない出版社

このクエリでは、NOT EXISTS演算子と相関サブクエリを使用して、関連する書籍が存在しない出版社を特定しています。EXISTS演算子は、サブクエリが少なくとも1行を返す場合にtrueとなります。

#### 5. 出版社ごとの平均価格と全体平均との差

このクエリでは、SELECT句でスカラーサブクエリを使用して、全体の平均価格を計算し、各出版社の平均価格との差を計算しています。CASE式を使用して、平均価格が全体平均より高いか低いかを示す分類も追加しています。

### A5M2での実行のポイント

1. **複雑なクエリの段階的な構築**
   - 複雑なクエリは、まず基本的な部分から始めて、徐々に機能を追加していくと理解しやすくなります
   - A5M2では、クエリの一部を選択して実行することができるため、部分的なテストが容易です

2. **共通テーブル式（CTE）の活用**
   - WITH句を使用した共通テーブル式は、複雑なクエリを読みやすく、管理しやすくします
   - A5M2では、CTEの結果を個別に確認することはできませんが、クエリの構造化に役立ちます

3. **結果セットの検証**
   - 複雑なJOINやサブクエリを使用する場合は、予期しない結果（重複レコードや欠落レコードなど）がないか注意深く確認することが重要です
   - A5M2の結果タブでは、列ヘッダーをクリックしてソートしたり、フィルタを適用したりして、結果を検証できます

4. **パフォーマンスの最適化**
   - EXPLAIN ANALYZEを使用して、クエリの実行計画とパフォーマンスを確認します
   - 特に大きなテーブルに対するクエリでは、インデックスの有無や結合順序が重要になります
