---
layout: default
title: Day 10: SQLトラブルシューティングとトランザクション管理 - 解答例
---
# Day 10: SQLトラブルシューティングとトランザクション管理 - 解答例

## 課題1: トラブルシューティング

### クエリ1の解析と修正

**元のクエリ:**
```sql
SELECT employee_id, first_name last_name, salary
FROM employees
WHERE department_id = 10
ORDER salary DESC;
```

**エラーの特定:**
1. `first_name last_name` の間にカンマがない（構文エラー）
2. `ORDER` の後に `BY` がない（構文エラー）

**修正したクエリ:**
```sql
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE department_id = 10
ORDER BY salary DESC;
```

**ベストプラクティス:**
- SQL文を書く前に構文を確認する
- 複数の列を選択する場合は、各列名の間にカンマを入れる
- ORDER BY句を使用する場合は、必ず「ORDER BY」と記述する
- SQLクライアントツールの構文ハイライト機能を活用する

### クエリ2の解析と修正

**元のクエリ:**
```sql
INSERT INTO departments (department_id, department_name, location)
VALUES (10, 'Finance', 'New York');
INSERT INTO employees (employee_id, first_name, last_name, salary, department_id)
VALUES (101, 'John', 'Doe', 5000, 20);
UPDATE employees
SET department_id = 10
WHERE employee_id = 101;
DELETE FROM departments
WHERE department_id = 20;
```

**エラーの特定:**
このクエリセットには、実行順序に関する問題があります。最後の文で department_id = 20 の部門を削除していますが、その部門を参照している従業員（employee_id = 101）が存在します。これは外部キー制約違反になる可能性があります。

**修正したクエリ:**
```sql
INSERT INTO departments (department_id, department_name, location)
VALUES (10, 'Finance', 'New York');
INSERT INTO departments (department_id, department_name, location)
VALUES (20, 'HR', 'Chicago');
INSERT INTO employees (employee_id, first_name, last_name, salary, department_id)
VALUES (101, 'John', 'Doe', 5000, 20);
UPDATE employees
SET department_id = 10
WHERE employee_id = 101;
DELETE FROM departments
WHERE department_id = 20;
```

または、外部キー制約を考慮した順序に変更：

```sql
INSERT INTO departments (department_id, department_name, location)
VALUES (10, 'Finance', 'New York');
INSERT INTO departments (department_id, department_name, location)
VALUES (20, 'HR', 'Chicago');
INSERT INTO employees (employee_id, first_name, last_name, salary, department_id)
VALUES (101, 'John', 'Doe', 5000, 20);
UPDATE employees
SET department_id = 10
WHERE employee_id = 101;
-- 従業員を部門10に移動した後で部門20を削除
DELETE FROM departments
WHERE department_id = 20;
```

**ベストプラクティス:**
- 外部キー制約を持つテーブル間の操作順序に注意する
- トランザクションを使用して、一連の操作を原子的に実行する
- 削除操作の前に、参照整合性を確認する
- ON DELETE CASCADE などの参照アクションを適切に設定する

### クエリ3の解析と修正

**元のクエリ:**
```sql
SELECT e.employee_id, e.first_name, e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE salary > 5000
GROUP BY department_name;
```

**エラーの特定:**
GROUP BY句に含まれていない列（employee_id, first_name, last_name）がSELECT句に含まれています。PostgreSQLでは、GROUP BY句に含まれていない列は、集約関数（SUM, COUNT, AVG など）で囲む必要があります。

**修正したクエリ:**
選択する列をすべてGROUP BY句に含める：
```sql
SELECT e.employee_id, e.first_name, e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE salary > 5000
GROUP BY d.department_name, e.employee_id, e.first_name, e.last_name;
```

または、集約を目的としない場合はGROUP BY句を削除：
```sql
SELECT e.employee_id, e.first_name, e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE salary > 5000
ORDER BY d.department_name;
```

**ベストプラクティス:**
- GROUP BY句を使用する場合は、SELECT句の非集約列をすべてGROUP BY句に含める
- 集約が必要ない場合は、GROUP BY句を使用しない
- クエリの目的（集約か単純な選択か）を明確にする
- 複雑なクエリは、段階的に構築してテストする

## 課題2: トランザクション管理

### テーブル作成

```sql
-- accountsテーブルの作成
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    account_number VARCHAR(20) UNIQUE NOT NULL,
    owner_name VARCHAR(100) NOT NULL,
    balance NUMERIC(15, 2) NOT NULL CHECK (balance >= 0)
);

-- transactionsテーブルの作成
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    from_account INTEGER REFERENCES accounts(id),
    to_account INTEGER REFERENCES accounts(id),
    amount NUMERIC(15, 2) NOT NULL CHECK (amount > 0),
    transaction_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL DEFAULT 'completed',
    description TEXT
);

-- サンプルデータの挿入
INSERT INTO accounts (account_number, owner_name, balance)
VALUES 
    ('1001-2345-6789', 'Alice Smith', 10000.00),
    ('2002-3456-7890', 'Bob Johnson', 5000.00),
    ('3003-4567-8901', 'Charlie Brown', 2000.00);
```

### 送金処理のストアドプロシージャ

```sql
-- 送金処理を行うストアドプロシージャ
CREATE OR REPLACE PROCEDURE transfer_funds(
    p_from_account VARCHAR(20),
    p_to_account VARCHAR(20),
    p_amount NUMERIC(15, 2),
    p_description TEXT DEFAULT NULL
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_from_id INTEGER;
    v_to_id INTEGER;
    v_from_balance NUMERIC(15, 2);
    v_transaction_id INTEGER;
BEGIN
    -- トランザクションの開始
    BEGIN
        -- 送金元と送金先のアカウントIDを取得
        SELECT id, balance INTO v_from_id, v_from_balance
        FROM accounts
        WHERE account_number = p_from_account
        FOR UPDATE; -- 行ロックを取得
        
        IF NOT FOUND THEN
            RAISE EXCEPTION '送金元アカウント % が見つかりません', p_from_account;
        END IF;
        
        SELECT id INTO v_to_id
        FROM accounts
        WHERE account_number = p_to_account
        FOR UPDATE; -- 行ロックを取得
        
        IF NOT FOUND THEN
            RAISE EXCEPTION '送金先アカウント % が見つかりません', p_to_account;
        END IF;
        
        -- 同一アカウントへの送金をチェック
        IF v_from_id = v_to_id THEN
            RAISE EXCEPTION '同一アカウントへの送金はできません';
        END IF;
        
        -- 残高不足をチェック
        IF v_from_balance < p_amount THEN
            RAISE EXCEPTION '残高不足: 現在の残高は % です', v_from_balance;
        END IF;
        
        -- 送金元の残高を減少
        UPDATE accounts
        SET balance = balance - p_amount
        WHERE id = v_from_id;
        
        -- 送金先の残高を増加
        UPDATE accounts
        SET balance = balance + p_amount
        WHERE id = v_to_id;
        
        -- 送金履歴の記録
        INSERT INTO transactions (from_account, to_account, amount, description)
        VALUES (v_from_id, v_to_id, p_amount, p_description)
        RETURNING id INTO v_transaction_id;
        
        -- トランザクションのコミット
        COMMIT;
        
        RAISE NOTICE '送金が完了しました。取引ID: %', v_transaction_id;
    EXCEPTION
        WHEN OTHERS THEN
            -- エラーが発生した場合はロールバック
            ROLLBACK;
            RAISE;
    END;
END;
$$;
```

### テストケース

```sql
-- テストケース1: 正常な送金処理
CALL transfer_funds('1001-2345-6789', '2002-3456-7890', 1000.00, '正常な送金テスト');

-- 結果の確認
SELECT * FROM accounts;
SELECT * FROM transactions;

-- テストケース2: 残高不足による送金失敗
CALL transfer_funds('3003-4567-8901', '1001-2345-6789', 3000.00, '残高不足テスト');

-- テストケース3: 同時実行による競合
-- 別々のセッションで以下を実行
-- セッション1
BEGIN;
CALL transfer_funds('1001-2345-6789', '3003-4567-8901', 500.00, '競合テスト1');
COMMIT;

-- セッション2
BEGIN;
CALL transfer_funds('2002-3456-7890', '1001-2345-6789', 300.00, '競合テスト2');
COMMIT;
```

### 同時実行テストの結果分析

上記の同時実行テストでは、FOR UPDATE句によって行レベルのロックが取得されるため、一方のトランザクションが完了するまで、もう一方のトランザクションは待機状態になります。これにより、データの整合性が保たれます。

ただし、デッドロックのリスクもあります。例えば、セッション1がアカウント1をロックし、セッション2がアカウント2をロックした後、セッション1がアカウント2をロックしようとし、セッション2がアカウント1をロックしようとすると、デッドロックが発生します。

このような場合、PostgreSQLはデッドロックを検出し、一方のトランザクションをロールバックします。ストアドプロシージャ内のエラーハンドリングにより、ロールバックされたトランザクションは適切に処理されます。

## 課題3: データ整合性の確保

### テーブル作成と制約の実装

```sql
-- authorsテーブルの作成
CREATE TABLE authors (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    birth_year INTEGER CHECK (birth_year > 0),
    nationality VARCHAR(50)
);

-- publishersテーブルの作成
CREATE TABLE publishers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    country VARCHAR(50)
);

-- booksテーブルの作成
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL, -- タイトルは必須
    author_id INTEGER REFERENCES authors(id),
    publisher_id INTEGER REFERENCES publishers(id),
    publication_year INTEGER CHECK (publication_year > 0),
    price NUMERIC(10, 2) CHECK (price > 0), -- 価格は0より大きい
    stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0), -- 在庫数は0以上
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- book_price_historyテーブルの作成（価格変更履歴用）
CREATE TABLE book_price_history (
    id SERIAL PRIMARY KEY,
    book_id INTEGER NOT NULL REFERENCES books(id),
    old_price NUMERIC(10, 2),
    new_price NUMERIC(10, 2),
    change_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    change_user VARCHAR(100) DEFAULT CURRENT_USER
);

-- ordersテーブルの作成
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled')), -- ステータスの制約
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- order_itemsテーブルの作成
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES orders(id),
    book_id INTEGER NOT NULL REFERENCES books(id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### トリガーの実装

```sql
-- 注文作成時に在庫を減少させるトリガー関数
CREATE OR REPLACE FUNCTION decrease_book_stock()
RETURNS TRIGGER AS $$
BEGIN
    -- 在庫の確認と減少
    UPDATE books
    SET stock = stock - NEW.quantity
    WHERE id = NEW.book_id AND stock >= NEW.quantity;
    
    -- 在庫が足りない場合はエラーを発生
    IF NOT FOUND THEN
        RAISE EXCEPTION '書籍ID % の在庫が不足しています', NEW.book_id;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_decrease_book_stock
AFTER INSERT ON order_items
FOR EACH ROW
EXECUTE FUNCTION decrease_book_stock();

-- 注文キャンセル時に在庫を元に戻すトリガー関数
CREATE OR REPLACE FUNCTION restore_book_stock()
RETURNS TRIGGER AS $$
BEGIN
    -- 注文がキャンセルされた場合のみ処理
    IF NEW.status = 'cancelled' AND OLD.status != 'cancelled' THEN
        -- 注文に含まれる全ての商品の在庫を元に戻す
        UPDATE books b
        SET stock = b.stock + oi.quantity
        FROM order_items oi
        WHERE oi.order_id = NEW.id AND oi.book_id = b.id;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_restore_book_stock
AFTER UPDATE ON orders
FOR EACH ROW
EXECUTE FUNCTION restore_book_stock();

-- 書籍の価格変更を記録するトリガー関数
CREATE OR REPLACE FUNCTION log_book_price_change()
RETURNS TRIGGER AS $$
BEGIN
    -- 価格が変更された場合のみ記録
    IF NEW.price != OLD.price THEN
        INSERT INTO book_price_history (book_id, old_price, new_price)
        VALUES (OLD.id, OLD.price, NEW.price);
    END IF;
    
    -- updated_atを更新
    NEW.updated_at = CURRENT_TIMESTAMP;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_log_book_price_change
BEFORE UPDATE ON books
FOR EACH ROW
EXECUTE FUNCTION log_book_price_change();

-- 注文ステータス更新時にupdated_atを更新するトリガー
CREATE OR REPLACE FUNCTION update_order_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_update_order_timestamp
BEFORE UPDATE ON orders
FOR EACH ROW
EXECUTE FUNCTION update_order_timestamp();
```

### データ整合性検証のためのテストケース

```sql
-- サンプルデータの挿入
INSERT INTO authors (name, birth_year, nationality)
VALUES 
    ('J.K. Rowling', 1965, 'British'),
    ('George Orwell', 1903, 'British'),
    ('Haruki Murakami', 1949, 'Japanese');

INSERT INTO publishers (name, country)
VALUES 
    ('Bloomsbury', 'UK'),
    ('Penguin Books', 'UK'),
    ('Kodansha', 'Japan');

INSERT INTO books (title, author_id, publisher_id, publication_year, price, stock)
VALUES 
    ('Harry Potter and the Philosopher''s Stone', 1, 1, 1997, 15.99, 100),
    ('1984', 2, 2, 1949, 12.50, 50),
    ('Norwegian Wood', 3, 3, 1987, 14.75, 30);

-- テストケース1: 正常な注文処理
INSERT INTO orders (customer_id, status)
VALUES (1, 'pending');

INSERT INTO order_items (order_id, book_id, quantity, price)
VALUES (1, 1, 2, 15.99);

-- 在庫の確認
SELECT id, title, stock FROM books WHERE id = 1;

-- テストケース2: 在庫不足による注文失敗
INSERT INTO orders (customer_id, status)
VALUES (2, 'pending');

-- 以下はエラーになるはず（在庫が足りない）
INSERT INTO order_items (order_id, book_id, quantity, price)
VALUES (2, 2, 100, 12.50);

-- テストケース3: 注文キャンセルと在庫の復元
INSERT INTO orders (customer_id, status)
VALUES (3, 'pending');

INSERT INTO order_items (order_id, book_id, quantity, price)
VALUES (3, 3, 5, 14.75);

-- 在庫の確認
SELECT id, title, stock FROM books WHERE id = 3;

-- 注文をキャンセル
UPDATE orders SET status = 'cancelled' WHERE id = 3;

-- 在庫が元に戻ったことを確認
SELECT id, title, stock FROM books WHERE id = 3;

-- テストケース4: 価格変更と履歴記録
UPDATE books SET price = 16.99 WHERE id = 1;

-- 価格変更履歴の確認
SELECT * FROM book_price_history;

-- テストケース5: 制約違反のテスト
-- タイトルなしの書籍を挿入（エラーになるはず）
INSERT INTO books (author_id, publisher_id, publication_year, price, stock)
VALUES (1, 1, 2000, 19.99, 10);

-- 負の価格の書籍を挿入（エラーになるはず）
INSERT INTO books (title, author_id, publisher_id, publication_year, price, stock)
VALUES ('Invalid Book', 1, 1, 2000, -19.99, 10);

-- 無効なステータスの注文を作成（エラーになるはず）
INSERT INTO orders (customer_id, status)
VALUES (4, 'invalid_status');
```

### テスト結果の分析

上記のテストケースを実行すると、以下のような結果が期待されます：

1. **正常な注文処理**: 注文が作成され、対応する書籍の在庫が減少します。
2. **在庫不足による注文失敗**: 在庫が足りない場合、エラーが発生し、注文項目の挿入が失敗します。
3. **注文キャンセルと在庫の復元**: 注文がキャンセルされると、対応する書籍の在庫が元に戻ります。
4. **価格変更と履歴記録**: 書籍の価格が変更されると、変更履歴がbook_price_historyテーブルに記録されます。
5. **制約違反のテスト**: 制約に違反するデータの挿入や更新は、エラーになります。

これらのテストケースにより、実装した制約とトリガーが正しく機能していることを確認できます。データ整合性が保たれ、ビジネスルールが適切に実装されていることが検証できます。

### 追加の改善点

1. **トランザクション管理の強化**:
   - 複数のテーブルを更新する操作は、トランザクション内で行うべきです。
   - 特に、注文処理と在庫更新は、原子的に行われるべきです。

2. **エラー処理の改善**:
   - より詳細なエラーメッセージを提供することで、問題の診断と解決が容易になります。
   - 特定のエラーに対して、適切な回復処理を実装することも検討すべきです。

3. **監査ログの拡張**:
   - 価格変更だけでなく、在庫変更や注文ステータス変更なども記録することで、より完全な監査証跡を提供できます。

4. **パフォーマンスの最適化**:
   - 大量の注文や在庫更新がある場合、トリガーのパフォーマンスが問題になる可能性があります。
   - バッチ処理や非同期処理の導入を検討すべきです。

5. **ビジネスルールの拡張**:
   - 割引や税金の計算
   - 顧客ごとの購入制限
   - 予約注文と在庫管理の連携
   - 返品処理と在庫の調整

これらの改善点を実装することで、より堅牢で柔軟なシステムを構築できます。
