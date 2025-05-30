# Day 6: PostgreSQL環境構築と基本操作 - 解答例

## 課題1: 書籍管理データベースの設計と実装

```sql
-- データベースの作成
CREATE DATABASE book_management;

-- データベースへの接続
-- A5M2では、接続リストから新しく作成したデータベースを選択します

-- 著者テーブルの作成
CREATE TABLE authors (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    birth_date DATE,
    biography TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 出版社テーブルの作成
CREATE TABLE publishers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    address TEXT,
    phone VARCHAR(20),
    email VARCHAR(100),
    website VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- カテゴリテーブルの作成
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 書籍テーブルの作成
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    isbn VARCHAR(20) UNIQUE NOT NULL,
    publication_year INTEGER,
    price NUMERIC(10, 2) CHECK (price >= 0),
    description TEXT,
    publisher_id INTEGER REFERENCES publishers(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 書籍と著者の中間テーブル（多対多）
CREATE TABLE book_authors (
    book_id INTEGER REFERENCES books(id) ON DELETE CASCADE,
    author_id INTEGER REFERENCES authors(id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, author_id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 書籍とカテゴリの中間テーブル（多対多）
CREATE TABLE book_categories (
    book_id INTEGER REFERENCES books(id) ON DELETE CASCADE,
    category_id INTEGER REFERENCES categories(id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, category_id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- インデックスの作成
CREATE INDEX idx_books_title ON books(title);
CREATE INDEX idx_books_isbn ON books(isbn);
CREATE INDEX idx_books_publisher_id ON books(publisher_id);
CREATE INDEX idx_authors_last_name ON authors(last_name);
CREATE INDEX idx_publishers_name ON publishers(name);
CREATE INDEX idx_categories_name ON categories(name);

-- サンプルデータの挿入
-- 著者データ
INSERT INTO authors (first_name, last_name, birth_date, biography)
VALUES 
('Robert', 'Martin', '1952-12-05', 'Robert Cecil Martin is an American software engineer, instructor, and author.'),
('Martin', 'Fowler', '1963-12-18', 'Martin Fowler is a British software engineer, author and international speaker.'),
('Eric', 'Evans', '1960-01-01', 'Eric Evans is a specialist in domain modeling and design.'),
('Joshua', 'Bloch', '1961-08-28', 'Joshua J. Bloch is an American software engineer and author.'),
('Kathy', 'Sierra', '1957-01-01', 'Kathy Sierra is an American programming instructor and game developer.');

-- 出版社データ
INSERT INTO publishers (name, address, phone, email, website)
VALUES 
('Addison-Wesley', '1 Jacob Way, Reading, MA 01867, USA', '+1-617-944-3700', 'info@aw.com', 'www.aw.com'),
('O''Reilly Media', '1005 Gravenstein Highway North, Sebastopol, CA 95472, USA', '+1-707-827-7000', 'info@oreilly.com', 'www.oreilly.com'),
('Prentice Hall', 'One Lake Street, Upper Saddle River, NJ 07458, USA', '+1-201-236-7000', 'info@prenticehall.com', 'www.prenticehall.com'),
('Manning Publications', '20 Baldwin Road, Shelter Island, NY 11964, USA', '+1-631-823-9484', 'info@manning.com', 'www.manning.com');

-- カテゴリデータ
INSERT INTO categories (name, description)
VALUES 
('Programming', 'Books about programming languages and techniques'),
('Software Engineering', 'Books about software design, architecture, and best practices'),
('Database', 'Books about database design and management'),
('Web Development', 'Books about web technologies and development'),
('DevOps', 'Books about DevOps practices and tools');

-- 書籍データ
INSERT INTO books (title, isbn, publication_year, price, description, publisher_id)
VALUES 
('Clean Code', '9780132350884', 2008, 44.99, 'A handbook of agile software craftsmanship', 1),
('Refactoring', '9780201485677', 1999, 49.99, 'Improving the design of existing code', 1),
('Domain-Driven Design', '9780321125217', 2003, 54.99, 'Tackling complexity in the heart of software', 1),
('Effective Java', '9780134685991', 2017, 49.99, 'Best practices for the Java platform', 1),
('Head First Java', '9780596009205', 2005, 39.99, 'A brain-friendly guide to Java', 2);

-- 書籍と著者の関連付け
INSERT INTO book_authors (book_id, author_id)
VALUES 
(1, 1), -- Clean Code by Robert Martin
(2, 2), -- Refactoring by Martin Fowler
(3, 3), -- Domain-Driven Design by Eric Evans
(4, 4), -- Effective Java by Joshua Bloch
(5, 5); -- Head First Java by Kathy Sierra

-- 書籍とカテゴリの関連付け
INSERT INTO book_categories (book_id, category_id)
VALUES 
(1, 1), (1, 2), -- Clean Code: Programming, Software Engineering
(2, 1), (2, 2), -- Refactoring: Programming, Software Engineering
(3, 2),         -- Domain-Driven Design: Software Engineering
(4, 1),         -- Effective Java: Programming
(5, 1);         -- Head First Java: Programming

-- 複数の著者を持つ書籍の例
INSERT INTO books (title, isbn, publication_year, price, description, publisher_id)
VALUES ('Design Patterns', '9780201633610', 1994, 59.99, 'Elements of Reusable Object-Oriented Software', 1);

-- 通称「Gang of Four」の著者たち
INSERT INTO authors (first_name, last_name, birth_date, biography)
VALUES 
('Erich', 'Gamma', '1961-01-01', 'Erich Gamma is a Swiss computer scientist.'),
('Richard', 'Helm', '1960-01-01', 'Richard Helm is a computer scientist.'),
('Ralph', 'Johnson', '1955-01-01', 'Ralph Johnson is a computer scientist.'),
('John', 'Vlissides', '1961-01-01', 'John Vlissides was a computer scientist.');

-- 書籍と著者の関連付け（複数著者）
INSERT INTO book_authors (book_id, author_id)
VALUES 
(6, 6), -- Design Patterns by Erich Gamma
(6, 7), -- Design Patterns by Richard Helm
(6, 8), -- Design Patterns by Ralph Johnson
(6, 9); -- Design Patterns by John Vlissides

-- 書籍とカテゴリの関連付け
INSERT INTO book_categories (book_id, category_id)
VALUES 
(6, 1), (6, 2); -- Design Patterns: Programming, Software Engineering
```

## 課題2: データベース操作の実践

```sql
-- 1. 特定の著者が書いた全ての書籍を取得
SELECT b.title, b.isbn, b.publication_year, b.price
FROM books b
JOIN book_authors ba ON b.id = ba.book_id
JOIN authors a ON ba.author_id = a.id
WHERE a.last_name = 'Martin' AND a.first_name = 'Robert'
ORDER BY b.publication_year DESC;

-- 2. 特定のカテゴリに属する書籍を価格順に取得
SELECT b.title, b.isbn, b.price, p.name AS publisher
FROM books b
JOIN book_categories bc ON b.id = bc.book_id
JOIN categories c ON bc.category_id = c.id
JOIN publishers p ON b.publisher_id = p.id
WHERE c.name = 'Software Engineering'
ORDER BY b.price DESC;

-- 3. 特定の出版社から出版された書籍の数と平均価格を計算
SELECT p.name AS publisher, 
       COUNT(b.id) AS book_count, 
       AVG(b.price) AS average_price,
       MIN(b.price) AS min_price,
       MAX(b.price) AS max_price
FROM publishers p
JOIN books b ON p.id = b.publisher_id
GROUP BY p.name
ORDER BY book_count DESC;

-- 4. 複数の著者がいる書籍のリストを取得
SELECT b.title, b.isbn, COUNT(ba.author_id) AS author_count,
       STRING_AGG(a.first_name || ' ' || a.last_name, ', ') AS authors
FROM books b
JOIN book_authors ba ON b.id = ba.book_id
JOIN authors a ON ba.author_id = a.id
GROUP BY b.id, b.title, b.isbn
HAVING COUNT(ba.author_id) > 1
ORDER BY author_count DESC, b.title;

-- 5. 特定の書籍の価格を更新（トランザクション使用）
BEGIN;
UPDATE books
SET price = 47.99, updated_at = CURRENT_TIMESTAMP
WHERE isbn = '9780132350884'; -- Clean Code
COMMIT;

-- 6. 新しい著者と書籍を追加し、関連付ける（トランザクション使用）
BEGIN;

-- 新しい著者の追加
INSERT INTO authors (first_name, last_name, birth_date, biography)
VALUES ('Kent', 'Beck', '1961-03-31', 'Kent Beck is an American software engineer and the creator of extreme programming.');

-- 新しい書籍の追加
INSERT INTO books (title, isbn, publication_year, price, description, publisher_id)
VALUES ('Test-Driven Development', '9780321146533', 2002, 39.99, 'By Example', 
        (SELECT id FROM publishers WHERE name = 'Addison-Wesley'));

-- 著者と書籍の関連付け
INSERT INTO book_authors (book_id, author_id)
VALUES (
    (SELECT id FROM books WHERE isbn = '9780321146533'),
    (SELECT id FROM authors WHERE first_name = 'Kent' AND last_name = 'Beck')
);

-- 書籍とカテゴリの関連付け
INSERT INTO book_categories (book_id, category_id)
VALUES (
    (SELECT id FROM books WHERE isbn = '9780321146533'),
    (SELECT id FROM categories WHERE name = 'Programming')
), (
    (SELECT id FROM books WHERE isbn = '9780321146533'),
    (SELECT id FROM categories WHERE name = 'Software Engineering')
);

COMMIT;

-- 7. 特定のカテゴリ名を変更（トランザクション使用）
BEGIN;
UPDATE categories
SET name = 'Database Systems', 
    description = 'Books about database design, management, and optimization',
    updated_at = CURRENT_TIMESTAMP
WHERE name = 'Database';
COMMIT;

-- 8. インデックスの作成によるパフォーマンス向上
-- 書籍タイトルの部分一致検索のためのインデックス
CREATE INDEX idx_books_title_gin ON books USING gin (title gin_trgm_ops);

-- 事前にtrgmモジュールのインストールが必要
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- 出版年と価格の複合インデックス
CREATE INDEX idx_books_year_price ON books(publication_year, price);

-- 著者のフルネーム検索のためのインデックス
CREATE INDEX idx_authors_fullname ON authors((first_name || ' ' || last_name));

-- インデックスの効果を確認するためのEXPLAIN ANALYZE
EXPLAIN ANALYZE
SELECT b.title, b.isbn, b.price
FROM books b
JOIN book_categories bc ON b.id = bc.book_id
JOIN categories c ON bc.category_id = c.id
WHERE c.name = 'Software Engineering' AND b.publication_year > 2000
ORDER BY b.price DESC;
```

## 解答の解説

### 課題1の解説

この解答例では、書籍管理データベースの設計と実装を行っています。

1. **テーブル設計**
   - 主要エンティティ（書籍、著者、出版社、カテゴリ）ごとに個別のテーブルを作成
   - 多対多の関係を表現するための中間テーブル（book_authors, book_categories）を作成
   - 各テーブルに適切な列と制約を設定

2. **正規化**
   - 第1正規形: 各列が原子的な値を持つように設計
   - 第2正規形: 部分関数従属性を排除（例: 書籍と著者の関係を中間テーブルで表現）
   - 第3正規形: 推移的関数従属性を排除（例: 出版社の情報を別テーブルに分離）

3. **制約の実装**
   - 主キー制約: 各テーブルの`id`列にPRIMARY KEY制約
   - 外部キー制約: 関連テーブルへの参照にREFERENCES制約
   - 一意性制約: ISBNにUNIQUE制約
   - 値の検証: 価格に対するCHECK制約（0以上）

4. **メタデータの追加**
   - 作成日時と更新日時の列を追加し、データの履歴管理をサポート
   - DEFAULT CURRENT_TIMESTAMPを使用して自動的に現在時刻を設定

5. **インデックスの作成**
   - 頻繁に検索される列（タイトル、ISBN、著者名など）にインデックスを作成
   - 外部キー列にもインデックスを作成し、結合操作のパフォーマンスを向上

### 課題2の解説

この解答例では、作成したデータベースに対する様々な操作を実装しています。

1. **検索クエリ**
   - JOIN操作を使用して関連テーブルのデータを結合
   - WHERE句による絞り込み
   - ORDER BY句によるソート
   - GROUP BY句とHAVING句による集計と絞り込み
   - 集計関数（COUNT, AVG, MIN, MAX）の使用
   - STRING_AGG関数を使用した文字列の集約

2. **更新操作**
   - トランザクションを使用した一貫性の確保
   - BEGIN/COMMITによるトランザクションの明示的な制御
   - UPDATE文による既存データの更新
   - INSERT文による新規データの追加
   - サブクエリを使用した関連データの取得と参照

3. **パフォーマンス最適化**
   - 様々な種類のインデックスの作成
   - 全文検索のためのGINインデックスとtrgm拡張機能の使用
   - 複合インデックスによる複数条件での検索の最適化
   - EXPLAIN ANALYZEを使用したクエリプランの分析

4. **高度なSQL機能**
   - サブクエリの使用
   - 集計関数と分析関数の活用
   - 文字列操作と連結
   - 拡張機能の活用

### A5M2での実行のポイント

1. **トランザクション管理**
   - A5M2では、SQLエディタでBEGIN/COMMITステートメントを使用してトランザクションを制御できます
   - 「自動コミット」設定がオンになっている場合は、明示的なトランザクション制御が必要ない場合もあります

2. **結果の確認**
   - クエリ実行後、結果タブで出力を確認できます
   - 結果セットはCSVやExcelにエクスポート可能です

3. **実行計画の分析**
   - EXPLAIN ANALYZEの結果は、テキスト形式で表示されます
   - 実行計画を分析して、インデックスの効果やクエリのボトルネックを特定できます

4. **エラーハンドリング**
   - SQLエラーが発生した場合、エラーメッセージが表示されます
   - トランザクション内でエラーが発生した場合、ROLLBACKを使用して変更を取り消すことができます
