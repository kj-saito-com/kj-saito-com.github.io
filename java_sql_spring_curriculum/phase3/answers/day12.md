---
layout: default
title: Day 12: TERASOLUNAフレームワークの概要とTODOアプリケーションの設計 - 解答例
---
# Day 12: TERASOLUNAフレームワークの概要とTODOアプリケーションの設計 - 解答例

## コア演習1: TODOアプリケーションの要件定義書作成

```
# TODOアプリケーション要件定義書

## 1. 概要
本アプリケーションは、ユーザーがTODO（タスク）を管理するためのWebアプリケーションである。
ユーザーは自分のTODOを登録、更新、削除、完了/未完了の管理ができる。
TERASOLUNAフレームワークを使用し、保守性と拡張性を重視した設計・実装を行う。

## 2. 機能要件

### 2.1 ユーザー管理機能
#### 2.1.1 ユーザー登録
- ユーザー名、パスワード、確認用パスワードを入力して登録できる
- ユーザー名は一意であること
- パスワードは8文字以上、英数字混在であること
- 確認用パスワードと一致することを確認する

#### 2.1.2 ログイン/ログアウト
- ユーザー名とパスワードによる認証を行う
- 認証失敗時は適切なエラーメッセージを表示する
- ログイン状態はセッションで管理する
- ログアウト機能を提供する

### 2.2 TODO管理機能
#### 2.2.1 TODO一覧表示
- ログインユーザーのTODOを一覧表示する
- 完了/未完了でフィルタリングできる
- 作成日時の降順で表示する

#### 2.2.2 TODO登録
- TODOのタイトルを入力して登録できる
- タイトルは1文字以上、100文字以内であること
- 登録時は未完了状態で作成される
- 作成日時と作成者（ログインユーザー）が記録される

#### 2.2.3 TODO更新
- TODOのタイトルを更新できる
- タイトルは1文字以上、100文字以内であること
- 更新日時が記録される

#### 2.2.4 TODO削除
- TODOを削除できる
- 削除前に確認ダイアログを表示する

#### 2.2.5 TODO完了/未完了の切り替え
- TODOの完了/未完了状態を切り替えられる
- 切り替え時に更新日時が記録される

## 3. 非機能要件

### 3.1 セキュリティ
#### 3.1.1 認証
- Spring Securityを使用した認証機能を実装する
- パスワードはBCryptでハッシュ化して保存する
- セッションタイムアウトは30分とする

#### 3.1.2 認可
- ユーザーは自分のTODOのみ操作可能とする
- 他ユーザーのTODOへのアクセスは拒否する
- 認可エラー時は適切なエラーページを表示する

#### 3.1.3 入力値検証
- すべての入力値に対してサーバーサイドでのバリデーションを実施する
- XSS対策としてHTMLエスケープを行う
- SQLインジェクション対策としてプリペアドステートメントを使用する

### 3.2 保守性
#### 3.2.1 アーキテクチャ
- TERASOLUNAフレームワークの推奨アーキテクチャに準拠する
- アプリケーション層、ドメイン層、インフラストラクチャ層の責務を明確に分離する
- インターフェースと実装を分離し、依存性の注入を活用する

#### 3.2.2 例外処理
- 適切な例外ハンドリングを実装する
- システム例外とビジネス例外を区別する
- ユーザーに分かりやすいエラーメッセージを表示する

#### 3.2.3 ログ出力
- 操作ログ、エラーログを適切に出力する
- ログレベルを適切に設定する
- センシティブな情報（パスワードなど）はログに出力しない

### 3.3 パフォーマンス
#### 3.3.1 応答時間
- 画面表示は3秒以内に完了すること
- データベースアクセスを最適化すること
- 必要に応じてインデックスを設定すること

#### 3.3.2 同時アクセス
- 10ユーザー程度の同時アクセスに対応すること
- デッドロックが発生しないようにトランザクション設計を行うこと

#### 3.3.3 リソース使用
- メモリリークが発生しないようにすること
- コネクションプールを適切に設定すること
```

## コア演習2: TODOアプリケーションの画面設計

```
# TODOアプリケーション画面設計書

## 1. 画面遷移図

```
[ログイン画面] ← → [ユーザー登録画面]
    ↓ (ログイン成功)
[TODO一覧画面] → [ログイン画面] (ログアウト)
```

## 2. 画面レイアウト

### 2.1 ログイン画面

```
+------------------------------------------+
|                                          |
|             TODOアプリケーション          |
|                                          |
+------------------------------------------+
|                                          |
|  ユーザー名: [                      ]    |
|                                          |
|  パスワード: [                      ]    |
|                                          |
|  [ログイン]                              |
|                                          |
|  アカウントをお持ちでない方は[こちら]    |
|                                          |
+------------------------------------------+
```

#### 項目説明
- ユーザー名: テキスト入力欄、必須
- パスワード: パスワード入力欄、必須
- ログイン: ボタン、クリックでログイン処理実行
- こちら: リンク、クリックでユーザー登録画面に遷移

### 2.2 ユーザー登録画面

```
+------------------------------------------+
|                                          |
|             TODOアプリケーション          |
|                                          |
+------------------------------------------+
|                                          |
|  ユーザー名: [                      ]    |
|                                          |
|  パスワード: [                      ]    |
|                                          |
|  パスワード(確認): [                ]    |
|                                          |
|  [登録]                                  |
|                                          |
|  ログイン画面に[戻る]                    |
|                                          |
+------------------------------------------+
```

#### 項目説明
- ユーザー名: テキスト入力欄、必須、一意制約あり
- パスワード: パスワード入力欄、必須、8文字以上、英数字混在
- パスワード(確認): パスワード入力欄、必須、パスワードと一致
- 登録: ボタン、クリックでユーザー登録処理実行
- 戻る: リンク、クリックでログイン画面に遷移

### 2.3 TODO一覧画面

```
+------------------------------------------+
|                                          |
|  TODOアプリケーション    [ログアウト]    |
|                                          |
+------------------------------------------+
|                                          |
|  ようこそ、{ユーザー名}さん              |
|                                          |
|  新規TODO: [                      ] [追加]|
|                                          |
|  表示: [すべて ▼]                        |
|                                          |
|  +------------------------------------+  |
|  | □ 牛乳を買う           [編集][削除]|  |
|  | ■ 報告書を提出する     [編集][削除]|  |
|  | □ 会議の準備をする     [編集][削除]|  |
|  +------------------------------------+  |
|                                          |
+------------------------------------------+
```

#### 項目説明
- ログアウト: リンク、クリックでログアウト処理実行
- 新規TODO: テキスト入力欄、1文字以上、100文字以内
- 追加: ボタン、クリックでTODO登録処理実行
- 表示: ドロップダウンリスト、「すべて」「未完了」「完了」から選択
- TODOリスト:
  - チェックボックス: クリックでTODO完了/未完了切り替え
  - TODO内容: テキスト表示
  - 編集: リンク、クリックでTODO編集モードに切り替え
  - 削除: リンク、クリックでTODO削除処理実行（確認ダイアログあり）
```

## コア演習3: TODOアプリケーションのデータベース設計

```
# TODOアプリケーションデータベース設計書

## 1. テーブル定義

### 1.1 usersテーブル
| カラム名 | データ型 | 制約 | 説明 |
|---------|---------|------|------|
| username | VARCHAR(50) | PRIMARY KEY | ユーザー名 |
| password | VARCHAR(100) | NOT NULL | パスワード（BCryptでハッシュ化） |
| role | VARCHAR(20) | NOT NULL | ロール（ROLE_USER, ROLE_ADMIN） |
| enabled | BOOLEAN | NOT NULL | 有効フラグ |

### 1.2 todoテーブル
| カラム名 | データ型 | 制約 | 説明 |
|---------|---------|------|------|
| todo_id | VARCHAR(36) | PRIMARY KEY | TODOのID（UUID） |
| todo_title | VARCHAR(100) | NOT NULL | TODOのタイトル |
| finished | BOOLEAN | NOT NULL | 完了フラグ（true: 完了, false: 未完了） |
| created_at | TIMESTAMP | NOT NULL | 作成日時 |
| created_by | VARCHAR(50) | NOT NULL, FOREIGN KEY | 作成者（usersテーブルのusernameを参照） |
| updated_at | TIMESTAMP | | 更新日時 |

## 2. ER図

```
+---------------+       +---------------+
|    users      |       |     todo      |
+---------------+       +---------------+
| PK username   |       | PK todo_id    |
|    password   |       |    todo_title |
|    role       |       |    finished   |
|    enabled    |       |    created_at |
+---------------+       |    created_by |
        ^               |    updated_at |
        |               +---------------+
        |                      |
        +----------------------+
             created_by (FK)
```

## 3. インデックス

### 3.1 usersテーブル
- PRIMARY KEY (username)

### 3.2 todoテーブル
- PRIMARY KEY (todo_id)
- INDEX idx_todo_created_by (created_by)
- INDEX idx_todo_finished (finished)

## 4. 制約

### 4.1 usersテーブル
- PRIMARY KEY (username)

### 4.2 todoテーブル
- PRIMARY KEY (todo_id)
- FOREIGN KEY (created_by) REFERENCES users(username)
```

## 発展演習: TODOアプリケーションのクラス設計

```
# TODOアプリケーションクラス設計書

## 1. アプリケーション層

### 1.1 Controller

#### 1.1.1 TodoController
- パッケージ: com.example.todo.app.todo
- 責務: TODO管理に関するリクエストを処理する
- 主なメソッド:
  - list(): TODO一覧表示
  - create(TodoForm): TODO登録
  - finish(String): TODO完了
  - reopen(String): TODO未完了に戻す
  - delete(String): TODO削除

#### 1.1.2 AuthController
- パッケージ: com.example.todo.app.auth
- 責務: 認証・認可に関するリクエストを処理する
- 主なメソッド:
  - showLoginForm(): ログイン画面表示
  - showRegistrationForm(): ユーザー登録画面表示
  - register(UserForm): ユーザー登録

### 1.2 Form

#### 1.2.1 TodoForm
- パッケージ: com.example.todo.app.todo
- 責務: TODOの入力データを保持する
- 主なフィールド:
  - todoTitle: String (バリデーション: NotEmpty, Size(max=100))

#### 1.2.2 UserForm
- パッケージ: com.example.todo.app.auth
- 責務: ユーザー登録の入力データを保持する
- 主なフィールド:
  - username: String (バリデーション: NotEmpty, Size(max=50))
  - password: String (バリデーション: NotEmpty, Size(min=8))
  - confirmPassword: String (バリデーション: NotEmpty, Size(min=8))

## 2. ドメイン層

### 2.1 Model

#### 2.1.1 Todo
- パッケージ: com.example.todo.domain.model
- 責務: TODOのドメインモデル
- 主なフィールド:
  - todoId: String
  - todoTitle: String
  - finished: boolean
  - createdAt: Date
  - createdBy: String
  - updatedAt: Date

#### 2.1.2 User
- パッケージ: com.example.todo.domain.model
- 責務: ユーザーのドメインモデル
- 主なフィールド:
  - username: String
  - password: String
  - role: String
  - enabled: boolean

### 2.2 Service

#### 2.2.1 TodoService
- パッケージ: com.example.todo.domain.service
- 責務: TODOに関するビジネスロジックを提供する
- 主なメソッド:
  - findAll(String): List<Todo> - ユーザーのTODO一覧を取得
  - create(Todo): Todo - TODOを登録
  - finish(String, String): Todo - TODOを完了状態にする
  - reopen(String, String): Todo - TODOを未完了状態に戻す
  - delete(String, String): void - TODOを削除

#### 2.2.2 UserService
- パッケージ: com.example.todo.domain.service
- 責務: ユーザーに関するビジネスロジックを提供する
- 主なメソッド:
  - register(User): User - ユーザーを登録
  - loadUserByUsername(String): UserDetails - Spring Securityの認証に使用

### 2.3 Repository

#### 2.3.1 TodoRepository
- パッケージ: com.example.todo.domain.repository
- 責務: TODOのデータアクセスを抽象化する
- 主なメソッド:
  - findAll(String): List<Todo> - ユーザーのTODO一覧を取得
  - findOne(String): Optional<Todo> - 指定されたIDのTODOを取得
  - create(Todo): Todo - TODOを登録
  - update(Todo): Todo - TODOを更新
  - delete(String): void - TODOを削除
  - countByCreatedBy(String): long - ユーザーのTODO数を取得

#### 2.3.2 UserRepository
- パッケージ: com.example.todo.domain.repository
- 責務: ユーザーのデータアクセスを抽象化する
- 主なメソッド:
  - findOne(String): Optional<User> - 指定されたユーザー名のユーザーを取得
  - create(User): User - ユーザーを登録
  - exists(String): boolean - 指定されたユーザー名のユーザーが存在するか確認

## 3. インフラストラクチャ層

### 3.1 RepositoryImpl

#### 3.1.1 TodoRepositoryImpl
- パッケージ: com.example.todo.domain.repository.mybatis
- 責務: TodoRepositoryの実装
- 依存: TodoRepositoryMapper (MyBatisのマッパーインターフェース)

#### 3.1.2 UserRepositoryImpl
- パッケージ: com.example.todo.domain.repository.mybatis
- 責務: UserRepositoryの実装
- 依存: UserRepositoryMapper (MyBatisのマッパーインターフェース)

### 3.2 Mapper

#### 3.2.1 TodoRepositoryMapper
- パッケージ: com.example.todo.domain.repository.mybatis
- 責務: MyBatisのマッパーインターフェース
- 主なメソッド:
  - findAll(String): List<Todo>
  - findOne(String): Todo
  - create(Todo): void
  - update(Todo): void
  - delete(String): void
  - countByCreatedBy(String): long

#### 3.2.2 UserRepositoryMapper
- パッケージ: com.example.todo.domain.repository.mybatis
- 責務: MyBatisのマッパーインターフェース
- 主なメソッド:
  - findOne(String): User
  - create(User): void
  - exists(String): int

## 4. クラス図

```
+------------------+       +------------------+       +------------------+
|  TodoController  |------>|   TodoService    |------>| TodoRepository   |
+------------------+       +------------------+       +------------------+
        |                          |                          |
        |                          |                          |
        v                          v                          v
+------------------+       +------------------+       +------------------+
|    TodoForm      |       |      Todo        |       |TodoRepositoryImpl|
+------------------+       +------------------+       +------------------+
                                                              |
                                                              v
                                                     +------------------+
                                                     |TodoRepositoryMapper|
                                                     +------------------+

+------------------+       +------------------+       +------------------+
|  AuthController  |------>|   UserService    |------>| UserRepository   |
+------------------+       +------------------+       +------------------+
        |                          |                          |
        |                          |                          |
        v                          v                          v
+------------------+       +------------------+       +------------------+
|    UserForm      |       |      User        |       |UserRepositoryImpl|
+------------------+       +------------------+       +------------------+
                                                              |
                                                              v
                                                     +------------------+
                                                     |UserRepositoryMapper|
                                                     +------------------+
```

## 5. 依存関係

- アプリケーション層はドメイン層に依存する
- ドメイン層はインフラストラクチャ層に依存しない（依存性逆転の原則）
- インフラストラクチャ層はドメイン層に依存する

## 6. トランザクション境界

- サービス層のメソッドをトランザクション境界とする
- 読み取り専用のメソッドには@Transactional(readOnly = true)を付与する
- 更新を伴うメソッドには@Transactionalを付与する
```

### 解説

この発展演習では、TERASOLUNAフレームワークの推奨アーキテクチャに基づいて、TODOアプリケーションのクラス設計を行いました。

1. **アプリケーション層**
   - Controller: Webリクエストを受け付け、適切なServiceを呼び出す役割を担います
   - Form: ユーザー入力データを保持し、バリデーションを行います

2. **ドメイン層**
   - Model: ビジネスデータを表現します
   - Service: ビジネスロジックを実装し、トランザクション境界となります
   - Repository: データアクセスを抽象化するインターフェースを提供します

3. **インフラストラクチャ層**
   - RepositoryImpl: Repositoryインターフェースの実装を提供します
   - Mapper: MyBatisのマッパーインターフェースを提供します

この階層化アーキテクチャにより、以下のメリットが得られます：

- 関心事の分離による保守性の向上
- 各層を独立してテストできるテスタビリティの向上
- 技術変更の影響範囲の局所化

特に、ドメイン層がインフラストラクチャ層に依存しない「依存性逆転の原則」を採用することで、データアクセス技術の変更（例えば、MyBatisからJPAへの変更）があっても、ドメイン層のコードを変更する必要がありません。
