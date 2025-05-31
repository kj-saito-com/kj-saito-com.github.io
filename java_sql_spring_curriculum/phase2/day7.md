---
layout: default
title: "Day 7: 複雑なJOINとサブクエリの活用"
---
# Day 7: 複雑なJOINとサブクエリの活用

## 学習の目的と背景

データベースの実務では、単一テーブルからの単純なデータ取得だけでなく、複数テーブルからの複雑なデータ結合や、条件に基づいた高度なデータ抽出が必要になります。JOINとサブクエリはSQLの中でも特に重要な機能であり、これらを適切に活用することで、複雑なビジネス要件に対応したデータ取得が可能になります。

本日の学習では、PostgreSQLにおけるJOINの種類とその使い分け、サブクエリの活用方法について理解を深め、実践的なクエリ作成スキルを習得することを目指します。

## 学習内容の解説

### 1. JOINの基本と種類

JOINは、複数のテーブルを結合してデータを取得するための操作です。PostgreSQLでは以下の種類のJOINがサポートされています：

#### INNER JOIN（内部結合）

両方のテーブルで一致する行のみを返します。

```sql
SELECT a.column_name, b.column_name
FROM table_a a
INNER JOIN table_b b ON a.common_field = b.common_field;
```

#### LEFT JOIN（左外部結合）

左側のテーブル（FROM句で指定したテーブル）の全ての行と、右側のテーブルの一致する行を返します。右側のテーブルに一致する行がない場合は、NULL値が返されます。

```sql
SELECT a.column_name, b.column_name
FROM table_a a
LEFT JOIN table_b b ON a.common_field = b.common_field;
```

#### RIGHT JOIN（右外部結合）

右側のテーブルの全ての行と、左側のテーブルの一致する行を返します。左側のテーブルに一致する行がない場合は、NULL値が返されます。

```sql
SELECT a.column_name, b.column_name
FROM table_a a
RIGHT JOIN table_b b ON a.common_field = b.common_field;
```

#### FULL JOIN（完全外部結合）

両方のテーブルの全ての行を返します。一致する行がない場合は、対応する側にNULL値が返されます。

```sql
SELECT a.column_name, b.column_name
FROM table_a a
FULL JOIN table_b b ON a.common_field = b.common_field;
```

#### CROSS JOIN（交差結合）

左側のテーブルの各行と右側のテーブルの各行を組み合わせた全ての可能な組み合わせを返します（直積）。

```sql
SELECT a.column_name, b.column_name
FROM table_a a
CROSS JOIN table_b b;
```

#### SELF JOIN（自己結合）

同じテーブルを自分自身と結合します。階層関係や関連レコードを取得する場合に使用します。

```sql
SELECT a.column_name, b.column_name
FROM table_a a
JOIN table_a b ON a.id = b.parent_id;
```

### 2. 複雑なJOIN操作

#### 複数テーブルの結合

3つ以上のテーブルを結合する場合は、複数のJOIN句を連続して使用します。

```sql
SELECT a.column1, b.column2, c.column3
FROM table_a a
JOIN table_b b ON a.id = b.a_id
JOIN table_c c ON b.id = c.b_id;
```

#### 異なる結合条件の組み合わせ

複数の条件を使用して結合することも可能です。

```sql
SELECT a.column1, b.column2
FROM table_a a
JOIN table_b b ON a.id = b.a_id AND a.status = 'active';
```

#### USING句の活用

結合キーの名前が両方のテーブルで同じ場合は、USING句を使用して簡潔に記述できます。

```sql
SELECT a.column1, b.column2
FROM table_a a
JOIN table_b b USING (common_column);
```

### 3. サブクエリの基本と種類

サブクエリは、別のSQLクエリ内に埋め込まれたSELECTステートメントです。以下の種類があります：

#### スカラーサブクエリ

単一の値を返すサブクエリです。

```sql
SELECT column_name, 
       (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;
```

#### 行サブクエリ

単一の行（複数の列）を返すサブクエリです。

```sql
SELECT *
FROM employees
WHERE (department_id, salary) = (SELECT department_id, MAX(salary)
                                FROM employees
                                WHERE department_id = 10
                                GROUP BY department_id);
```

#### テーブルサブクエリ

複数の行と列を返すサブクエリです。FROM句で使用されることが多いです。

```sql
SELECT a.column1, b.column2
FROM table_a a
JOIN (SELECT id, column2 FROM table_b WHERE condition) b ON a.id = b.id;
```

#### 相関サブクエリ

外部クエリの値を参照するサブクエリです。

```sql
SELECT e.employee_id, e.salary
FROM employees e
WHERE e.salary > (SELECT AVG(salary)
                 FROM employees
                 WHERE department_id = e.department_id);
```

### 4. サブクエリの活用パターン

#### WHERE句でのサブクエリ

条件式の一部としてサブクエリを使用します。

```sql
-- IN演算子との組み合わせ
SELECT *
FROM employees
WHERE department_id IN (SELECT department_id
                       FROM departments
                       WHERE location_id = 1700);

-- EXISTS演算子との組み合わせ
SELECT *
FROM departments d
WHERE EXISTS (SELECT 1
             FROM employees e
             WHERE e.department_id = d.department_id
             AND e.salary > 10000);
```

#### FROM句でのサブクエリ（派生テーブル）

サブクエリの結果をテーブルとして扱います。

```sql
SELECT d.department_name, e.avg_salary
FROM departments d
JOIN (SELECT department_id, AVG(salary) AS avg_salary
     FROM employees
     GROUP BY department_id) e ON d.department_id = e.department_id;
```

#### SELECT句でのサブクエリ

各行に対して計算された値を取得します。

```sql
SELECT e.employee_id, e.first_name,
       (SELECT COUNT(*)
        FROM employees
        WHERE manager_id = e.employee_id) AS direct_reports
FROM employees e;
```

#### HAVING句でのサブクエリ

グループ化された結果に対する条件としてサブクエリを使用します。

```sql
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);
```

### 5. JOINとサブクエリの使い分け

同じ結果を得るためにJOINとサブクエリの両方を使用できる場合がありますが、以下のような使い分けの指針があります：

#### JOINを使用する場合

- 複数のテーブルから列を取得する必要がある場合
- 結合条件が明確で、結果セットが比較的小さい場合
- 外部結合（LEFT JOIN、RIGHT JOIN、FULL JOIN）が必要な場合

#### サブクエリを使用する場合

- 集計結果と元のデータを比較する場合
- 条件に基づいて行をフィルタリングする場合
- 複雑な条件や階層的なデータ構造を扱う場合
- 一時的な結果セットを作成する場合

### 6. パフォーマンスの考慮事項

#### 実行計画の確認

EXPLAIN ANALYZEを使用して、クエリの実行計画とパフォーマンスを確認できます。

```sql
EXPLAIN ANALYZE
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 10000;
```

#### インデックスの活用

結合キーや頻繁に検索される列にインデックスを作成することで、パフォーマンスを向上させることができます。

```sql
CREATE INDEX idx_employees_department_id ON employees(department_id);
CREATE INDEX idx_departments_department_id ON departments(department_id);
```

#### クエリの最適化

- 不要な列や行を取得しないようにする
- 適切な結合順序を考慮する
- 可能な限り条件を早く適用する
- 大きなテーブルに対するCROSS JOINを避ける

## 実践課題（演習）

### 課題1: 複雑なJOIN操作の実践

前回作成した書籍管理データベースを使用して、以下のクエリを作成してください。

**要件:**
1. 各出版社ごとに、出版した書籍の数、著者の数（重複なし）、カテゴリの数（重複なし）を取得するクエリ
2. 各著者ごとに、執筆した書籍のタイトル一覧と、共著者の名前一覧を取得するクエリ
3. 書籍と著者と出版社とカテゴリを全て結合し、書籍が存在しない著者や、書籍が存在しないカテゴリも含めて取得するクエリ
4. 自己結合を使用して、同じ出版社から出版された他の書籍のリストを各書籍ごとに取得するクエリ

**制約条件:**
- 適切なJOINの種類（INNER JOIN、LEFT JOIN、RIGHT JOIN、FULL JOIN）を選択すること
- 結果が見やすくなるように、適切な列名と並び順を設定すること

### 課題2: サブクエリの活用

前回作成した書籍管理データベースを使用して、以下のクエリを作成してください。

**要件:**
1. 平均価格より高い書籍を、カテゴリごとに取得するクエリ（サブクエリを使用）
2. 3冊以上の書籍を執筆している著者の一覧と、その著者が執筆した最新の書籍を取得するクエリ
3. 各カテゴリにおいて、最も高価な書籍と最も安価な書籍を取得するクエリ
4. 書籍を1冊も出版していない出版社を取得するクエリ（EXISTS演算子を使用）
5. 各出版社の平均書籍価格と、全体の平均価格との差を計算するクエリ

**制約条件:**
- WHERE句、FROM句、SELECT句、HAVING句でそれぞれサブクエリを使用すること
- 相関サブクエリと非相関サブクエリの両方を使用すること
- 可能な限り効率的なクエリを作成すること

## 解説と補足

### JOINの選択に関するポイント

1. **INNER JOINとOUTER JOINの使い分け**
   - データの完全性が重要な場合はINNER JOINを使用
   - 欠損データも含めて全体像を把握したい場合はOUTER JOIN（LEFT/RIGHT/FULL）を使用

2. **複数テーブルの結合順序**
   - 最も制限の厳しいテーブル（行数が少ないテーブル）から結合を始めると効率的
   - 結合条件で使用される列にインデックスがあるかを確認

3. **USING句とON句の使い分け**
   - 結合キーの名前が同じ場合はUSING句を使用すると簡潔
   - 複雑な結合条件や異なる列名での結合にはON句を使用

4. **NATURAL JOINの注意点**
   - NATURAL JOINは同名の列を自動的に結合するため、予期しない結果になる可能性がある
   - 明示的なON句やUSING句を使用する方が安全

### サブクエリの最適化ポイント

1. **非相関サブクエリと相関サブクエリの選択**
   - 非相関サブクエリ（外部クエリに依存しないサブクエリ）は一度だけ実行されるため、一般的に効率的
   - 相関サブクエリ（外部クエリの各行に対して実行されるサブクエリ）は、行数が少ない場合や、EXISTS演算子と組み合わせる場合に有効

2. **INとEXISTSの使い分け**
   - サブクエリの結果が少ない場合はIN演算子が効率的
   - サブクエリの結果が多い場合や、存在確認のみが必要な場合はEXISTS演算子が効率的

3. **サブクエリとJOINの変換**
   - 多くの場合、サブクエリはJOINに変換できる
   - どちらが効率的かはデータ量やインデックスの有無によって異なるため、EXPLAIN ANALYZEで検証することが重要

4. **共通テーブル式（CTE）の活用**
   - 複雑なサブクエリは、WITH句（CTE）を使用して可読性を向上させることができる
   - 再帰的CTEを使用して階層データを効率的に処理できる

### A5M2でのクエリ実行と分析

1. **複数のクエリの実行**
   - A5M2では、セミコロンで区切られた複数のSQLステートメントを一度に実行できます
   - 各ステートメントの結果は別々のタブに表示されます

2. **実行計画の分析**
   - 「SQL実行」タブでEXPLAIN ANALYZEを実行すると、クエリの実行計画が表示されます
   - 実行計画では、スキャン方法（シーケンシャルスキャン、インデックススキャン）、結合方法（ネステッドループ、ハッシュ結合、マージ結合）、コストなどを確認できます

3. **クエリ結果の操作**
   - 結果セットはCSVやExcelにエクスポートできます
   - 結果セットの列をクリックすると、その列でソートできます
   - フィルタ機能を使用して、結果セットを絞り込むことができます

4. **クエリの保存と再利用**
   - よく使用するクエリはSQLファイルとして保存できます
   - クエリライブラリ機能を使用して、クエリを整理・管理できます

## 実務での活用シーン

### JOINの活用シーン

1. **データ分析と報告**
   - 売上データと顧客データを結合して顧客セグメント別の売上分析
   - 製品データと在庫データを結合して在庫状況レポートの作成
   - 従業員データと部門データを結合して組織構造の分析

2. **マスタデータ管理**
   - 複数のマスタテーブル間の関連性の確認と整合性チェック
   - マスタデータの統合と重複排除
   - リレーショナルデータの完全性検証

3. **トランザクションデータの処理**
   - 注文データと注文明細データを結合して注文情報の完全な取得
   - 取引データと顧客データを結合して取引履歴の分析
   - 支払いデータと請求データを結合して未払い項目の特定

4. **データ移行と統合**
   - 複数のソースシステムからのデータ統合
   - レガシーシステムと新システム間のデータマッピング
   - データウェアハウス構築のためのETL（抽出・変換・ロード）処理

### サブクエリの活用シーン

1. **高度なデータフィルタリング**
   - 特定の条件を満たすレコードのみを抽出（例：平均以上の売上を持つ製品）
   - 複数の条件に基づく段階的なフィルタリング
   - 動的な条件に基づくデータ選択

2. **集計と分析**
   - グループごとの集計値と全体の集計値の比較
   - ランキングや順位付け（例：カテゴリ別の売上上位製品）
   - 時系列データの期間比較（例：前年同期比の計算）

3. **データ検証と監査**
   - 不整合データの検出（例：関連レコードが存在しないレコードの特定）
   - ビジネスルールの検証（例：許容範囲外の値を持つレコードの特定）
   - 重複データの検出と解決

4. **複雑なビジネスロジックの実装**
   - 階層データの処理（例：組織構造、製品カテゴリ階層）
   - 条件付き集計（例：特定の条件を満たす項目のみの合計）
   - 複数の関連エンティティにまたがるビジネスルールの適用
