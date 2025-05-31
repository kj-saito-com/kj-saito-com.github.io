---
layout: default
title: "Day 8: 集計関数と分析関数の実践的活用"
---
# Day 8: 集計関数と分析関数の実践的活用

## 学習の目的と背景

データベースの実務では、単純なデータ取得だけでなく、データの集計や分析が頻繁に必要になります。PostgreSQLは豊富な集計関数と分析関数を提供しており、これらを活用することで複雑なビジネス要件に対応したデータ分析が可能になります。

本日の学習では、PostgreSQLにおける集計関数と分析関数（ウィンドウ関数）の使い方を理解し、実践的なデータ分析スキルを習得することを目指します。

## 学習内容の解説

### 1. 集計関数の基本

集計関数は、複数の行からデータを集約して単一の結果を返す関数です。PostgreSQLでは以下の基本的な集計関数が提供されています：

#### 基本的な集計関数

- **COUNT**: 行数をカウントします
- **SUM**: 数値の合計を計算します
- **AVG**: 数値の平均を計算します
- **MIN**: 最小値を返します
- **MAX**: 最大値を返します

```sql
-- 基本的な集計関数の例
SELECT 
    COUNT(*) AS total_books,
    SUM(price) AS total_price,
    AVG(price) AS average_price,
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM books;
```

#### GROUP BY句との組み合わせ

GROUP BY句を使用すると、指定した列の値ごとに集計を行うことができます。

```sql
-- カテゴリごとの書籍数と平均価格
SELECT 
    c.name AS category,
    COUNT(b.id) AS book_count,
    AVG(b.price) AS average_price
FROM 
    categories c
JOIN 
    book_categories bc ON c.id = bc.category_id
JOIN 
    books b ON bc.book_id = b.id
GROUP BY 
    c.id, c.name
ORDER BY 
    book_count DESC;
```

#### HAVING句による集計結果のフィルタリング

HAVING句を使用すると、集計結果に対して条件を適用できます。

```sql
-- 平均価格が40ドルを超えるカテゴリ
SELECT 
    c.name AS category,
    COUNT(b.id) AS book_count,
    AVG(b.price) AS average_price
FROM 
    categories c
JOIN 
    book_categories bc ON c.id = bc.category_id
JOIN 
    books b ON bc.book_id = b.id
GROUP BY 
    c.id, c.name
HAVING 
    AVG(b.price) > 40
ORDER BY 
    average_price DESC;
```

### 2. 高度な集計関数

PostgreSQLでは、基本的な集計関数に加えて、より高度な集計機能も提供されています。

#### 統計関数

- **STDDEV**: 標準偏差を計算します
- **VARIANCE**: 分散を計算します
- **CORR**: 相関係数を計算します
- **PERCENTILE_CONT**: 連続パーセンタイルを計算します
- **PERCENTILE_DISC**: 離散パーセンタイルを計算します

```sql
-- 価格の統計情報
SELECT 
    AVG(price) AS average_price,
    STDDEV(price) AS price_stddev,
    VARIANCE(price) AS price_variance,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median_price
FROM books;
```

#### 条件付き集計

- **COUNT(DISTINCT ...)**: 重複を除いた行数をカウントします
- **SUM(CASE WHEN ... THEN ... ELSE ... END)**: 条件に基づいて合計を計算します

```sql
-- 価格帯ごとの書籍数
SELECT 
    COUNT(*) AS total_books,
    COUNT(DISTINCT publisher_id) AS publisher_count,
    SUM(CASE WHEN price < 30 THEN 1 ELSE 0 END) AS low_price_books,
    SUM(CASE WHEN price >= 30 AND price < 50 THEN 1 ELSE 0 END) AS mid_price_books,
    SUM(CASE WHEN price >= 50 THEN 1 ELSE 0 END) AS high_price_books
FROM books;
```

#### 文字列集約

- **STRING_AGG**: 文字列を連結します

```sql
-- 出版社ごとの書籍タイトル一覧
SELECT 
    p.name AS publisher,
    STRING_AGG(b.title, ', ' ORDER BY b.publication_year) AS book_titles
FROM 
    publishers p
JOIN 
    books b ON p.id = b.publisher_id
GROUP BY 
    p.id, p.name;
```

### 3. 分析関数（ウィンドウ関数）の基本

分析関数（ウィンドウ関数）は、結果セットの「ウィンドウ」（行の集合）に対して計算を行い、各行に対して値を返します。GROUP BY句とは異なり、行の集約は行わず、各行は結果セットに残ります。

#### 基本的な構文

```sql
function_name() OVER ([PARTITION BY column1, column2, ...] [ORDER BY column3, column4, ...] [frame_clause])
```

- **PARTITION BY**: 計算を行うグループを指定します
- **ORDER BY**: 計算を行う順序を指定します
- **frame_clause**: 現在の行に対するウィンドウフレーム（範囲）を指定します

#### ランキング関数

- **ROW_NUMBER**: 各行に一意の連番を割り当てます
- **RANK**: 同じ値の行には同じランクを割り当て、次のランクはスキップします
- **DENSE_RANK**: 同じ値の行には同じランクを割り当て、次のランクはスキップしません
- **NTILE**: 行を指定した数のグループに分割します

```sql
-- 価格に基づく書籍のランキング
SELECT 
    title,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS row_num,
    RANK() OVER (ORDER BY price DESC) AS price_rank,
    DENSE_RANK() OVER (ORDER BY price DESC) AS dense_rank,
    NTILE(4) OVER (ORDER BY price DESC) AS quartile
FROM books;
```

#### パーティション分割

PARTITION BY句を使用すると、指定した列の値ごとに独立した計算を行うことができます。

```sql
-- カテゴリごとの価格ランキング
SELECT 
    c.name AS category,
    b.title,
    b.price,
    RANK() OVER (PARTITION BY c.id ORDER BY b.price DESC) AS category_price_rank
FROM 
    books b
JOIN 
    book_categories bc ON b.id = bc.book_id
JOIN 
    categories c ON bc.category_id = c.id
ORDER BY 
    category, category_price_rank;
```

### 4. 高度な分析関数

PostgreSQLでは、基本的なランキング関数に加えて、より高度な分析機能も提供されています。

#### 集計ウィンドウ関数

集計関数をウィンドウ関数として使用することができます。

```sql
-- 累積合計と移動平均
SELECT 
    b.title,
    b.publication_year,
    b.price,
    SUM(b.price) OVER (ORDER BY b.publication_year) AS cumulative_sum,
    AVG(b.price) OVER (ORDER BY b.publication_year ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS moving_avg
FROM 
    books b
ORDER BY 
    b.publication_year;
```

#### オフセット関数

- **LAG**: 現在の行より前の行の値を取得します
- **LEAD**: 現在の行より後の行の値を取得します
- **FIRST_VALUE**: ウィンドウの最初の行の値を取得します
- **LAST_VALUE**: ウィンドウの最後の行の値を取得します

```sql
-- 前年と翌年の書籍価格との比較
SELECT 
    b.title,
    b.publication_year,
    b.price,
    LAG(b.price) OVER (ORDER BY b.publication_year) AS prev_year_price,
    LEAD(b.price) OVER (ORDER BY b.publication_year) AS next_year_price,
    b.price - LAG(b.price) OVER (ORDER BY b.publication_year) AS price_diff_from_prev
FROM 
    books b
ORDER BY 
    b.publication_year;
```

#### ウィンドウフレーム

ウィンドウフレームを使用すると、現在の行に対する計算の範囲を細かく制御できます。

```sql
-- 様々なウィンドウフレームの例
SELECT 
    b.title,
    b.publication_year,
    b.price,
    AVG(b.price) OVER () AS overall_avg,
    AVG(b.price) OVER (ORDER BY b.publication_year) AS cumulative_avg,
    AVG(b.price) OVER (ORDER BY b.publication_year ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS moving_avg_3rows,
    AVG(b.price) OVER (ORDER BY b.publication_year RANGE BETWEEN INTERVAL '1 YEAR' PRECEDING AND INTERVAL '1 YEAR' FOLLOWING) AS moving_avg_3years
FROM 
    books b
ORDER BY 
    b.publication_year;
```

### 5. 集計関数と分析関数の組み合わせ

集計関数と分析関数を組み合わせることで、より複雑なデータ分析が可能になります。

#### 集計結果と元データの比較

```sql
-- 各書籍の価格と、カテゴリ平均との差
SELECT 
    c.name AS category,
    b.title,
    b.price,
    AVG(b.price) OVER (PARTITION BY c.id) AS category_avg_price,
    b.price - AVG(b.price) OVER (PARTITION BY c.id) AS diff_from_category_avg,
    (b.price / AVG(b.price) OVER (PARTITION BY c.id) - 1) * 100 AS percent_diff
FROM 
    books b
JOIN 
    book_categories bc ON b.id = bc.book_id
JOIN 
    categories c ON bc.category_id = c.id
ORDER BY 
    category, b.price DESC;
```

#### パーセンタイルと四分位数

```sql
-- 価格の四分位数と、各書籍の位置付け
WITH price_stats AS (
    SELECT 
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY price) AS q1,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY price) AS q3,
        AVG(price) AS avg_price,
        STDDEV(price) AS stddev_price
    FROM books
)
SELECT 
    b.title,
    b.price,
    ps.q1,
    ps.median,
    ps.q3,
    CASE 
        WHEN b.price < ps.q1 THEN 'Low'
        WHEN b.price < ps.median THEN 'Below Median'
        WHEN b.price < ps.q3 THEN 'Above Median'
        ELSE 'High'
    END AS price_category,
    (b.price - ps.avg_price) / ps.stddev_price AS z_score
FROM 
    books b
CROSS JOIN 
    price_stats ps
ORDER BY 
    b.price;
```

#### 時系列分析

```sql
-- 年ごとの書籍数と平均価格の推移
SELECT 
    b.publication_year,
    COUNT(*) AS book_count,
    AVG(b.price) AS avg_price,
    SUM(COUNT(*)) OVER (ORDER BY b.publication_year) AS cumulative_count,
    AVG(AVG(b.price)) OVER (ORDER BY b.publication_year ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3years
FROM 
    books b
GROUP BY 
    b.publication_year
ORDER BY 
    b.publication_year;
```

### 6. A5M2での実行と分析

A5M2では、集計関数と分析関数を使用したクエリの結果を視覚的に確認できます。

#### 結果のエクスポートと可視化

1. クエリを実行し、結果タブで出力を確認します
2. 結果セットをCSVやExcelにエクスポートします
   - 右クリックメニューから「CSVエクスポート」または「Excelエクスポート」を選択
3. エクスポートしたデータをExcelやその他のツールで可視化します

#### クエリの保存と再利用

1. よく使用するクエリはSQLファイルとして保存します
   - 「ファイル」→「名前を付けて保存」を選択
2. クエリライブラリ機能を使用して、クエリを整理・管理します
   - 「ツール」→「クエリライブラリ」を選択

## 実践課題（演習）

### 課題1: 基本的な集計と分析

前回作成した書籍管理データベースを使用して、以下のクエリを作成してください。

**要件:**
1. 出版社ごとの書籍数、平均価格、最高価格、最低価格を計算するクエリ
2. 出版年ごとの書籍数と平均価格を計算し、書籍数の多い年順に並べるクエリ
3. カテゴリごとの書籍数と平均価格を計算し、書籍が3冊以上あるカテゴリのみを表示するクエリ
4. 著者ごとの書籍数と平均価格を計算し、共著書籍の場合は各著者にカウントするクエリ

**制約条件:**
- 適切な集計関数を使用すること
- 結果が見やすくなるように、適切な列名と並び順を設定すること
- 必要に応じてHAVING句を使用すること

### 課題2: 高度な集計と分析

前回作成した書籍管理データベースを使用して、以下のクエリを作成してください。

**要件:**
1. 書籍の価格を4分位（四分位）に分け、各分位ごとの書籍数と平均価格を計算するクエリ
2. 各書籍の価格と、同じカテゴリ内での価格ランキング（1位、2位、...）を表示するクエリ
3. 各出版社について、出版年ごとの書籍数の累積合計を計算するクエリ
4. 各著者について、執筆した書籍の価格の移動平均（3冊単位）を計算するクエリ
5. 各書籍について、同じ出版社の前の書籍と次の書籍（出版年順）のタイトルと価格を表示するクエリ

**制約条件:**
- 適切な分析関数（ウィンドウ関数）を使用すること
- PARTITION BY句とORDER BY句を適切に組み合わせること
- 必要に応じてウィンドウフレーム句を使用すること
- 共通テーブル式（WITH句）を活用して、複雑なクエリを構造化すること

## 解説と補足

### 集計関数の選択に関するポイント

1. **基本的な集計と高度な集計の使い分け**
   - 単純な合計や平均を求める場合は基本的な集計関数（SUM, AVG, COUNT）を使用
   - データの分布や統計的特性を分析する場合は高度な集計関数（STDDEV, PERCENTILE_CONT）を使用

2. **NULL値の扱い**
   - COUNT(*)はNULL値を含む全ての行をカウントしますが、COUNT(column)はNULL値を除外します
   - 他の集計関数（SUM, AVG, MIN, MAX）もNULL値を無視します
   - NULL値を特別に扱いたい場合は、COALESCE関数を使用して代替値を設定できます

3. **GROUP BY句の列選択**
   - GROUP BY句には、集計しない全ての列を含める必要があります
   - 集計関数を適用した列はGROUP BY句に含めません
   - PostgreSQLでは、GROUP BY句に含まれていない列をSELECT句に含めるとエラーになります

4. **HAVING句とWHERE句の使い分け**
   - WHERE句は集計前の行に対するフィルタリングに使用します
   - HAVING句は集計後の結果に対するフィルタリングに使用します
   - パフォーマンスを考慮すると、可能な限りWHERE句でフィルタリングを行うべきです

### 分析関数の活用ポイント

1. **パーティション分割の考慮**
   - データをどのようにグループ化するかを考慮し、適切なPARTITION BY句を設定します
   - パーティションが大きすぎると計算が遅くなり、小さすぎると意味のある結果が得られない可能性があります

2. **順序付けの重要性**
   - 累積計算や移動平均などの場合、ORDER BY句の順序が結果に大きく影響します
   - 日付や連番などの明確な順序を持つ列を使用することが望ましいです

3. **ウィンドウフレームの選択**
   - デフォルトのフレームは、ORDER BY句がない場合は全パーティション、ある場合は「現在の行までの全ての行」です
   - 移動平均などの計算には、ROWS BETWEEN n PRECEDING AND n FOLLOWINGのようなフレームが適しています
   - 時間ベースの分析には、RANGE BETWEEN INTERVAL ... PRECEDING AND INTERVAL ... FOLLOWINGが便利です

4. **複数の分析関数の組み合わせ**
   - 同じクエリ内で複数の分析関数を使用できます
   - 各関数に異なるパーティションや順序を指定できます
   - 複雑な分析では、共通テーブル式（WITH句）を使用して段階的に計算を行うと理解しやすくなります

### A5M2での実行と分析のポイント

1. **大量データの扱い**
   - 大量のデータに対して集計や分析を行う場合、クエリの実行に時間がかかる可能性があります
   - LIMIT句を使用して、開発中は少量のデータでテストすることが効率的です
   - 本番実行前にEXPLAIN ANALYZEでクエリプランを確認することをお勧めします

2. **結果の検証**
   - 集計結果や分析結果が期待通りかを確認するために、簡単なケースで手計算と比較することが有効です
   - 異常値や外れ値がある場合、集計結果に大きな影響を与える可能性があるため、データの前処理を検討します

3. **パフォーマンスの最適化**
   - 集計や分析に使用する列にインデックスを作成することで、パフォーマンスが向上する場合があります
   - 特に、GROUP BY句やPARTITION BY句、ORDER BY句で使用される列にインデックスを作成すると効果的です
   - 複雑な計算を行う場合は、マテリアライズドビューの使用を検討します

## 実務での活用シーン

### 集計関数の活用シーン

1. **ビジネスレポートの作成**
   - 売上集計（日次、月次、年次）
   - 顧客セグメント別の購入分析
   - 製品カテゴリ別の在庫状況

2. **KPI（重要業績評価指標）の監視**
   - 売上目標の達成率
   - 顧客獲得コスト
   - 顧客生涯価値（LTV）の計算

3. **異常検知と品質管理**
   - 平均からの逸脱の検出
   - 統計的プロセス管理
   - 外れ値の特定

4. **予算計画と予測**
   - 過去のデータに基づく予算配分
   - トレンド分析と将来予測
   - シナリオ分析

### 分析関数の活用シーン

1. **ランキングと順位付け**
   - 売上トップ製品の特定
   - 顧客ロイヤルティランキング
   - 従業員パフォーマンス評価

2. **時系列分析**
   - 売上の成長率計算
   - 季節変動の特定
   - 移動平均によるトレンド分析

3. **比較分析**
   - 前年同期比の計算
   - 計画と実績の差異分析
   - 市場シェアの変動分析

4. **セグメント分析**
   - 顧客セグメント内でのパフォーマンス比較
   - 地域別の売上貢献度分析
   - 製品カテゴリ内での価格ポジショニング
