# Claude Code 自動定義書生成ルール

## 役割

このリポジトリでは要件ファイルから定義書MDファイルを自動生成する。 要件ファイルが `develop/read-doc/` に配置されたら、以下のルールに従って定義書を生成すること。

---

## ディレクトリ構成

```
document/public-r-system-document/
└── develop/
    ├── CLAUDE.md                       # このファイル
    ├── api/
    │   ├── template_api.md             # API定義書テンプレート
    │   └── doc/                        # API定義書の出力先
    ├── db-table/
    │   ├── template_table.md           # テーブル定義書テンプレート
    │   └── doc/                        # テーブル定義書の出力先
    ├── read-doc/                       # 要件ファイルの投入先（監視対象）
    └── requirements/                   # 処理済み要件ファイルの移動先
        ├── requirements-template.md    # 要件テンプレート
        └── doc/                        # 要件ファイルの管理先
```

---

## 生成ルール

### 実行タイミング

`develop/read-doc/` 配下に `.md` ファイルが存在する場合に実行する。

### 出力先

- テーブル定義書: `develop/db-table/doc/{tableName}（{tableNameJp}）.md`
- API定義書: `develop/api/doc/{HTTP}-{urlをハイフン区切り}-{apiNameJp}.md`

### ファイル名の例

```
テーブル定義書: tbl_event（イベント）.md
API定義書:      POST-switch-bot-施錠検出API.md
```

---

## テーブル定義書の生成ルール

`develop/db-table/template_table.md` をベースに以下を埋める。

### frontmatter

```
tableName   : 要件の tableName をそのまま使用
tableNameJp : 要件の tableNameJp をそのまま使用
className   : 要件の className をそのまま使用
```

### テーブル（カラム定義）

要件ファイルのテーブル定義をそのまま転記する。

### dataviewjsボタン

テンプレートのコードをそのままコピーする。変更不要。

---

## API定義書の生成ルール

`develop/api/template_api.md` をベースに以下を埋める。

### frontmatter

```
apiNameJp : 要件の apiNameJp をそのまま使用
apiName   : 要件の apiName をそのまま使用
url       : 要件の url をそのまま使用
HTTP      : 要件の HTTP をそのまま使用
```

### 各セクションの埋め方

|セクション|埋める内容|
|---|---|
|API-Schema|空欄のまま|
|API-RequestBody|要件の RequestBody をコードブロックに記述|
|API-Response|要件の Response をコードブロックに記述|
|DB-SELECT|要件の使用テーブル SELECT を Obsidian リンク形式で記述|
|DB-INSERT/UPDATE/DELETE|要件の使用テーブル INSERT/UPDATE/DELETE を Obsidian リンク形式で記述|
|外部API-GET|空欄のまま|
|シーケンス図|空欄のまま|

### Obsidianリンクの形式

```
[[{tableName}（{tableNameJp}）]]
```

---

## 注意事項

- 生成するファイルは必ずテンプレートのフォーマットに従うこと
- 要件ファイルに記載のない項目は空欄にする（削除しない）
- テーブルが複数ある場合はテーブル定義書を複数ファイル生成する
- APIが複数ある場合はAPI定義書を複数ファイル生成する
- 生成完了後に生成したファイルの一覧を出力すること