---
layout: default
title: Day 9: SQLパフォーマンスチューニングの実践
---
# Day 9: SQLパフォーマンスチューニングの実践

## 学習の目的と背景

データベースの規模が大きくなるにつれて、SQLクエリのパフォーマンスは重要な課題となります。効率的なクエリは応答時間を短縮し、システム全体のパフォーマンスを向上させます。本日の学習では、PostgreSQLにおけるパフォーマンスチューニングの手法を理解し、実践的なチューニングスキルを習得することを目指します。

## 学習内容の解説

### 1. パフォーマンスチューニングの基本

#### クエリの実行計画を理解する

PostgreSQLでは、EXPLAINコマンドを使用してクエリの実行計画を確認できます。

```sql
-- 基本的な実行計画の確認
EXPLAIN
SELECT * FROM books WHERE price > 50;

-- 実際の実行時間も含めた詳細な実行計画
EXPLAIN ANALYZE
SELECT * FROM books WHERE price > 50;
```

実行計画には以下の情報が含まれます：

- **スキャン方法**: シーケンシャルスキャン、インデックススキャン、ビットマップスキャンなど
- **結合方法**: ネステッドループ結合、ハッシュ結合、マージ結合など
- **コスト推定**: 起動コストと総コスト
- **行数推定**: 各ステップで処理される行数の推定値
- **実際の実行時間**: EXPLAIN ANALYZEを使用した場合のみ

#### コストモデルの理解

PostgreSQLのクエリプランナーは、各操作のコストを推定して最適な実行計画を選択します。コストは以下の要素に基づいて計算されます：

- **ディスクI/O**: ページの読み取りと書き込み
- **CPU処理**: 行の処理、比較、計算など
- **メモリ使用**: ソート、ハッシュテーブルなどの一時データ構造

コストの単位は任意ですが、一般的にはシーケンシャルページ読み取りのコストを基準としています。

### 2. インデックスの活用

#### インデックスの種類

PostgreSQLでは以下の種類のインデックスがサポートされています：

- **B-tree**: 最も一般的なインデックス。等価比較、範囲検索、ソートに適しています。
- **Hash**: 等価比較のみに適しています。
- **GiST**: 地理データや全文検索など、複雑なデータ型に適しています。
- **SP-GiST**: 非バランス型のディスク上のデータ構造を実装しています。
- **GIN**: 複合値（配列など）に適しています。
- **BRIN**: 大きなテーブルの物理的に近接した値に適しています。

#### インデックスの作成と管理

```sql
-- 基本的なインデックスの作成
CREATE INDEX idx_books_price ON books(price);

-- 複合インデックスの作成
CREATE INDEX idx_books_publisher_year ON books(publisher_id, publication_year);

-- ユニークインデックスの作成
CREATE UNIQUE INDEX idx_books_isbn ON books(isbn);

-- インデックスの削除
DROP INDEX idx_books_price;

-- インデックスの再構築
REINDEX INDEX idx_books_price;
```

#### インデックスの選択基準

- **カーディナリティ**: 列の異なる値の数。カーディナリティが高い列（例：主キー）はインデックスに適しています。
- **選択性**: クエリで選択される行の割合。選択性が低い（少数の行が選択される）場合はインデックスが効果的です。
- **更新頻度**: 頻繁に更新される列はインデックスのオーバーヘッドが大きくなります。
- **クエリパターン**: WHERE句、JOIN条件、ORDER BY句で使用される列はインデックスの候補です。

### 3. テーブル設計とデータ型の最適化

#### 正規化と非正規化

- **正規化**: データの冗長性を減らし、整合性を向上させます。複雑なJOINが必要になる場合があります。
- **非正規化**: パフォーマンスのためにデータを複製します。データの整合性を維持するための追加の労力が必要です。

#### 適切なデータ型の選択

- **整数型**: 可能な限り小さい整数型（SMALLINT, INTEGER, BIGINT）を選択します。
- **文字列型**: 固定長の場合はCHAR、可変長の場合はVARCHAR、大きなテキストの場合はTEXTを使用します。
- **日付と時刻**: DATE, TIME, TIMESTAMP, INTERVALなど、適切な型を選択します。
- **特殊データ型**: JSON, JSONB, XML, 配列型など、データの性質に合わせて選択します。

#### パーティショニング

大きなテーブルをより小さな物理的な「パーティション」に分割することで、パフォーマンスを向上させることができます。

```sql
-- パーティションテーブルの作成
CREATE TABLE sales (
    id SERIAL,
    sale_date DATE,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (sale_date);

-- パーティションの作成
CREATE TABLE sales_2023 PARTITION OF sales
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE sales_2024 PARTITION OF sales
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

### 4. クエリの最適化テクニック

#### WHERE句の最適化

- **インデックス列の使用**: WHERE句でインデックス列を使用します。
- **関数の回避**: インデックス列に関数を適用すると、インデックスが使用されなくなる場合があります。
- **LIKE演算子の注意点**: 前方一致（'abc%'）はインデックスを使用できますが、後方一致（'%abc'）や中間一致（'%abc%'）はインデックスを使用できません。

```sql
-- インデックスを使用する例
SELECT * FROM books WHERE isbn = '9781234567890';

-- インデックスを使用しない例（関数の使用）
SELECT * FROM books WHERE UPPER(title) = 'DATABASE DESIGN';

-- 代替案（関数インデックスの使用）
CREATE INDEX idx_books_upper_title ON books(UPPER(title));
SELECT * FROM books WHERE UPPER(title) = 'DATABASE DESIGN';
```

#### JOIN操作の最適化

- **結合順序**: 小さいテーブルから大きいテーブルへの結合が一般的に効率的です。
- **結合方法**: ネステッドループ結合（小さいテーブル）、ハッシュ結合（大きいテーブル、等価結合）、マージ結合（ソート済みデータ）の特性を理解します。
- **結合条件**: 結合条件にインデックスを作成します。

```sql
-- 結合条件にインデックスを作成
CREATE INDEX idx_books_publisher_id ON books(publisher_id);
CREATE INDEX idx_publishers_id ON publishers(id);

-- 効率的な結合
SELECT b.title, p.name
FROM books b
JOIN publishers p ON b.publisher_id = p.id
WHERE b.price > 50;
```

#### 集約関数と GROUP BY の最適化

- **インデックスの活用**: GROUP BY句で使用される列にインデックスを作成します。
- **HAVING vs WHERE**: 可能な限りWHERE句でフィルタリングを行い、HAVING句は集約後のフィルタリングにのみ使用します。

```sql
-- 効率的な集約
SELECT publisher_id, AVG(price)
FROM books
WHERE publication_year > 2020
GROUP BY publisher_id
HAVING COUNT(*) > 5;
```

#### サブクエリとCTEの最適化

- **相関サブクエリ**: 外部クエリの各行に対して実行されるため、パフォーマンスに影響する可能性があります。
- **非相関サブクエリ**: 一度だけ実行されるため、一般的に効率的です。
- **共通テーブル式（CTE）**: 複雑なクエリを分解し、可読性を向上させます。

```sql
-- 非相関サブクエリ
SELECT *
FROM books
WHERE publisher_id IN (SELECT id FROM publishers WHERE country = 'USA');

-- 共通テーブル式（CTE）
WITH usa_publishers AS (
    SELECT id FROM publishers WHERE country = 'USA'
)
SELECT *
FROM books
WHERE publisher_id IN (SELECT id FROM usa_publishers);
```

### 5. データベース設定の最適化

#### メモリ関連のパラメータ

- **shared_buffers**: PostgreSQLがデータキャッシュに使用するメモリ量。一般的にはシステムメモリの25%程度。
- **work_mem**: ソートやハッシュ操作に使用するメモリ量。複雑なクエリでは増やすことでパフォーマンスが向上する場合があります。
- **maintenance_work_mem**: VACUUM、CREATE INDEX、ALTER TABLEなどのメンテナンス操作に使用するメモリ量。

#### ディスクI/O関連のパラメータ

- **effective_cache_size**: オペレーティングシステムのディスクキャッシュを含む、利用可能なメモリの推定値。
- **random_page_cost**: ランダムディスクページアクセスのコスト。SSDの場合は低い値（1.1〜2.0）が適切です。
- **seq_page_cost**: シーケンシャルディスクページアクセスのコスト。通常は1.0に設定されています。

#### 自動バキュームの設定

- **autovacuum**: 自動的にVACUUMとANALYZEを実行するプロセス。
- **autovacuum_vacuum_threshold**: VACUUMをトリガーする更新または削除された行数のしきい値。
- **autovacuum_analyze_threshold**: ANALYZEをトリガーする挿入、更新、または削除された行数のしきい値。

### 6. モニタリングとトラブルシューティング

#### システムカタログとビュー

PostgreSQLは、データベースの状態を監視するための多くのシステムカタログとビューを提供しています。

- **pg_stat_activity**: 現在実行中のプロセスとクエリに関する情報。
- **pg_stat_user_tables**: テーブルのアクセス統計。
- **pg_stat_user_indexes**: インデックスの使用統計。
- **pg_locks**: 現在のロック情報。

```sql
-- アクティブなクエリの確認
SELECT pid, query, query_start, state
FROM pg_stat_activity
WHERE state = 'active';

-- テーブル統計の確認
SELECT schemaname, relname, seq_scan, seq_tup_read, idx_scan, idx_tup_fetch
FROM pg_stat_user_tables;

-- インデックス統計の確認
SELECT schemaname, relname, indexrelname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes;
```

#### スロークエリログ

スロークエリログを有効にすると、実行時間が長いクエリを記録できます。

```sql
-- スロークエリログの設定
ALTER SYSTEM SET log_min_duration_statement = 1000; -- 1000ミリ秒以上のクエリをログに記録
ALTER SYSTEM SET log_statement = 'none'; -- 'none', 'ddl', 'mod', 'all'のいずれか
```

#### クエリのキャンセルと終了

長時間実行されているクエリをキャンセルまたは終了させることができます。

```sql
-- クエリのキャンセル（通常の方法）
SELECT pg_cancel_backend(pid);

-- クエリの強制終了（最終手段）
SELECT pg_terminate_backend(pid);
```

## 実践課題（演習）

### 課題1: 実行計画の分析

前回作成した書籍管理データベースを使用して、以下のクエリの実行計画を分析してください。

**要件:**
1. 以下のクエリの実行計画を取得し、分析してください。
   ```sql
   SELECT b.title, p.name AS publisher, STRING_AGG(a.first_name || ' ' || a.last_name, ', ') AS authors
   FROM books b
   JOIN publishers p ON b.publisher_id = p.id
   LEFT JOIN book_authors ba ON b.id = ba.book_id
   LEFT JOIN authors a ON ba.author_id = a.id
   WHERE b.price > 30
   GROUP BY b.id, b.title, p.name
   ORDER BY b.title;
   ```

2. 上記クエリのパフォーマンスを向上させるために、適切なインデックスを作成してください。

3. インデックス作成後に再度実行計画を取得し、変化を分析してください。

**制約条件:**
- EXPLAIN ANALYZEを使用して、実際の実行時間も含めた詳細な実行計画を取得すること
- インデックスの作成理由を説明すること
- 実行計画の変化を詳細に分析すること

### 課題2: クエリの最適化

前回作成した書籍管理データベースを使用して、以下のクエリを最適化してください。

**要件:**
1. 以下のクエリのパフォーマンスを分析し、最適化してください。
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

2. 以下のクエリのパフォーマンスを分析し、最適化してください。
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

3. 以下のクエリのパフォーマンスを分析し、最適化してください。
   ```sql
   SELECT b1.title, b1.price,
          (SELECT AVG(b2.price) FROM books b2 WHERE b2.publisher_id = b1.publisher_id) AS publisher_avg_price,
          (SELECT MAX(b3.price) FROM books b3 WHERE b3.publisher_id = b1.publisher_id) AS publisher_max_price
   FROM books b1
   WHERE b1.price > (SELECT AVG(price) * 1.5 FROM books)
   ORDER BY b1.price DESC;
   ```

**制約条件:**
- 各クエリの最適化前後の実行計画を比較すること
- インデックスの作成、クエリの書き換え、またはその両方を検討すること
- 最適化の理由と効果を説明すること

### 課題3: データベース設計の最適化

前回作成した書籍管理データベースの設計を見直し、パフォーマンスの観点から最適化を提案してください。

**要件:**
1. テーブル設計（正規化レベル、データ型など）の観点から最適化を提案してください。
2. インデックス戦略（どの列にインデックスを作成すべきか）を提案してください。
3. パーティショニングが有効な場合は、パーティショニング戦略を提案してください。
4. その他のパフォーマンス最適化（制約、外部キーなど）を提案してください。

**制約条件:**
- 提案の理由と期待される効果を説明すること
- トレードオフ（パフォーマンスvs保守性など）を考慮すること
- 実際のビジネス要件（検索パターン、更新頻度など）を考慮すること

## 解説と補足

### 実行計画の読み方

実行計画は下から上に読みます。各ノードは以下の情報を含みます：

1. **操作タイプ**: Seq Scan（シーケンシャルスキャン）、Index Scan（インデックススキャン）、Hash Join（ハッシュ結合）など
2. **コスト**: `cost=0.00..10.00`の形式で表示され、最初の数字は起動コスト、2番目の数字は総コスト
3. **行数**: `rows=1000`の形式で表示され、返される行数の推定値
4. **幅**: `width=32`の形式で表示され、各行の平均バイト数
5. **実際の時間**: EXPLAIN ANALYZEを使用した場合、`actual time=0.016..0.020`の形式で表示され、最初の数字は最初の行を返すまでの時間、2番目の数字は全ての行を返すまでの時間（ミリ秒単位）
6. **実際の行数**: EXPLAIN ANALYZEを使用した場合、`rows=10`の形式で表示され、実際に返された行数
7. **ループ数**: EXPLAIN ANALYZEを使用した場合、`loops=1`の形式で表示され、操作が実行された回数

```
Sort  (cost=69.85..71.35 rows=600 width=396) (actual time=0.286..0.293 rows=15 loops=1)
  Sort Key: b.title
  Sort Method: quicksort  Memory: 26kB
  ->  Hash Join  (cost=10.00..40.00 rows=600 width=396) (actual time=0.100..0.200 rows=15 loops=1)
        Hash Cond: (b.publisher_id = p.id)
        ->  Seq Scan on books b  (cost=0.00..20.00 rows=600 width=200) (actual time=0.010..0.050 rows=15 loops=1)
              Filter: (price > 30)
        ->  Hash  (cost=5.00..5.00 rows=100 width=196) (actual time=0.050..0.050 rows=10 loops=1)
              Buckets: 1024  Batches: 1  Memory Usage: 9kB
              ->  Seq Scan on publishers p  (cost=0.00..5.00 rows=100 width=196) (actual time=0.010..0.030 rows=10 loops=1)
```

### インデックス選択のガイドライン

1. **主キーと外部キー**: 主キーには自動的にインデックスが作成されますが、外部キーには明示的にインデックスを作成する必要があります。
2. **WHERE句で頻繁に使用される列**: 検索条件で頻繁に使用される列にはインデックスを作成します。
3. **JOIN条件で使用される列**: 結合条件で使用される列にはインデックスを作成します。
4. **ORDER BY句で使用される列**: ソート操作を高速化するために、ORDER BY句で使用される列にはインデックスを作成します。
5. **複合インデックス**: 複数の列が一緒に使用される場合は、複合インデックスを検討します。複合インデックスの列順序は重要です（最も選択的な列を最初に配置することが一般的）。
6. **カバリングインデックス**: クエリで必要な全ての列をインデックスに含めることで、テーブルへのアクセスを回避できます。
7. **インデックスのオーバーヘッド**: インデックスはデータ更新時のオーバーヘッドを増加させるため、更新が頻繁な列には注意が必要です。

### クエリ最適化のチェックリスト

1. **インデックスの確認**: 適切なインデックスが存在するか確認します。
2. **WHERE句の最適化**: インデックスを活用できるようにWHERE句を調整します。
3. **JOIN順序の最適化**: 小さいテーブルから大きいテーブルへの結合を検討します。
4. **サブクエリの最適化**: 相関サブクエリを非相関サブクエリやJOINに変換できないか検討します。
5. **集約関数の最適化**: GROUP BY句の前にWHERE句でフィルタリングを行います。
6. **LIMIT句の活用**: 全ての結果が必要ない場合はLIMIT句を使用します。
7. **不要な列の排除**: SELECT *の代わりに必要な列のみを指定します。
8. **一時テーブルやCTEの活用**: 複雑なクエリを分解して可読性と効率を向上させます。

### A5M2でのパフォーマンス分析

A5M2では、以下の機能を使用してパフォーマンスを分析できます：

1. **実行計画の表示**: クエリを選択し、「実行計画」ボタンをクリックするか、クエリの前に「EXPLAIN」または「EXPLAIN ANALYZE」を追加します。
2. **テーブル統計の表示**: テーブルを右クリックし、「統計情報」を選択します。
3. **インデックスの管理**: テーブルを右クリックし、「インデックス」を選択します。
4. **システムカタログの参照**: 「システムオブジェクト」ツリーからシステムカタログやビューを参照できます。

## 実務での活用シーン

### パフォーマンスチューニングの実務シーン

1. **アプリケーションの応答時間改善**
   - ユーザーからの応答時間が遅いという報告を受けた場合
   - 特定の画面やレポートの読み込みが遅い場合
   - バッチ処理の実行時間が長い場合

2. **システムリソースの最適化**
   - CPU使用率が高い場合
   - メモリ使用量が多い場合
   - ディスクI/Oが多い場合
   - コネクション数が多い場合

3. **スケーラビリティの向上**
   - データ量の増加に伴うパフォーマンス低下
   - ユーザー数の増加に伴うパフォーマンス低下
   - 同時実行数の増加に伴うパフォーマンス低下

4. **特定のクエリの最適化**
   - レポート生成クエリの高速化
   - 検索機能の応答時間改善
   - バッチ処理クエリの最適化
   - リアルタイムダッシュボードの更新速度向上

### インデックス戦略の実務シーン

1. **検索機能の最適化**
   - ユーザー検索画面での検索条件に基づくインデックス作成
   - 全文検索のための特殊インデックス（GIN, GiST）の活用
   - 複合検索条件に対する複合インデックスの設計

2. **レポート生成の高速化**
   - レポートで使用される集計条件に基づくインデックス作成
   - ソート操作を高速化するためのインデックス作成
   - 頻繁に結合されるテーブルの結合キーにインデックス作成

3. **トランザクション処理の最適化**
   - 頻繁に更新される列のインデックス戦略の見直し
   - 一意性制約を持つ列へのユニークインデックスの作成
   - 外部キー参照の高速化のためのインデックス作成

4. **バッチ処理の効率化**
   - 大量データ処理時のインデックス戦略
   - 一時的なインデックスの作成と削除
   - メンテナンス作業（VACUUM, ANALYZE）のスケジューリング

### データベース設計の最適化シーン

1. **新規システム設計時**
   - 業務要件に基づく適切な正規化レベルの決定
   - 将来の拡張性を考慮したテーブル設計
   - 適切なデータ型の選択とストレージ最適化

2. **既存システムのリファクタリング**
   - パフォーマンス問題を解決するためのテーブル再設計
   - 非正規化による読み取りパフォーマンスの向上
   - パーティショニングによる大規模テーブルの管理改善

3. **データウェアハウス設計**
   - 分析クエリに最適化されたスキーマ設計（スタースキーマ、スノーフレークスキーマ）
   - 集計テーブルの作成と管理
   - パーティショニングとインデックス戦略の最適化

4. **マイグレーションプロジェクト**
   - レガシーシステムからの移行時のスキーマ最適化
   - 異なるデータベース製品間の移行時の最適化
   - 複数のシステムの統合時のスキーマ設計
