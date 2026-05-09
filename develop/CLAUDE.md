# CLAUDE.md
# Claude Code 自動定義書生成ルール

## 役割
このリポジトリでは要件ファイルから定義書MDファイルを自動生成する。
要件ファイルが `develop/read-doc/` に配置されたら、以下のルールに従って定義書を生成すること。

---

## ディレクトリ構成

```
document/public-r-system-document/
└── develop/
    ├── CLAUDE.md               # このファイル
    ├── api/
    │   ├── template_api.md     # API定義書テンプレート
    │   └── doc/
    │       └── {urlPath}/                   # URLディレクトリ
    │           └── {apiNameJp}/             # API概要ディレクトリ
    │               ├── GET-{urlPath}-{apiNameJp}.md
    │               ├── POST-{urlPath}-{apiNameJp}.md
    │               └── ...（HTTPメソッドごとにファイルを分ける）
    ├── db-table/
    │   ├── template_table.md   # テーブル定義書テンプレート
    │   └── doc/
    │       └── {featureName}/  # 機能名のサブフォルダ
    │           └── {tableName}（{tableNameJp}）.md
    ├── read-doc/               # 要件ファイルの投入先（監視対象）
    ├── done/                   # 処理済み要件ファイルの移動先
    └── script/
        └── _RunScript.md       # Templaterトリガーファイル（自動上書きされる）
```

---

## 実行順序

```
① テーブル定義書.md を生成
    ↓
② _RunScript.md のPLACEHOLDER_PATHを生成したテーブル定義書のパスに書き換える
    ↓
③ Advanced URI 経由で _RunScript.md を開き Templater を実行
    ↓  （テーブルが複数ある場合は②③をテーブル数分繰り返す）
④ API定義書.md を生成
```

---

## 生成ルール

### 実行タイミング
`develop/read-doc/` 配下に `.md` ファイルが存在する場合に実行する。

### urlPath の決定ルール
要件ファイルの url から先頭の `/` を除去し、残りの `/` を `-` に変換する。
```
/user            → user
/switch-bot/lock → switch-bot-lock
```

### featureName の決定ルール
要件ファイルの frontmatter の `featureName` を小文字・ハイフン区切りに変換する。
```
UserManagement → user-management
User           → user
```

### 出力先
- テーブル定義書: `develop/db-table/doc/{featureName}/{tableName}（{tableNameJp}）.md`
- API定義書: `develop/api/doc/{urlPath}/{apiNameJp}/{HTTP}-{urlPath}-{apiNameJp}.md`

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

## テーブル定義書生成後のTemplater自動実行ルール

テーブル定義書を生成したら以下を実行すること。
テーブルが複数ある場合は1ファイルごとに②③を繰り返すこと。

### ② _RunScript.md を書き換える
`develop/script/_RunScript.md` の `PLACEHOLDER_PATH` を
生成したテーブル定義書のパスに置き換えて上書き保存する。

書き換え例：
```
# 変更前
const targetPath = "PLACEHOLDER_PATH";

# 変更後（his_lock_detection の場合）
const targetPath = "develop/db-table/doc/switch-bot/his_lock_detection（施錠検出履歴）.md";
```

### ③ Advanced URI で _RunScript.md を実行する
以下のPowerShellコマンドを実行すること。

```powershell
$vaultName = "public-r-system-document"
$runScript  = "develop/script/_RunScript.md"
$encoded    = [System.Uri]::EscapeDataString($runScript)
$commandId  = [System.Uri]::EscapeDataString("templater-obsidian:replace-in-file-templater")

# ファイルを開く
Start-Process "obsidian://advanced-uri?vault=$vaultName&filepath=$encoded"
Start-Sleep -Seconds 2

# Templater を実行
Start-Process "obsidian://advanced-uri?vault=$vaultName&filepath=$encoded&commandid=$commandId"
Start-Sleep -Seconds 3
```

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

### 同一URLに複数HTTPメソッドがある場合
同じ `urlPath` かつ同じ `apiNameJp` のAPIは同じディレクトリにまとめる。
HTTPメソッド（GET / POST / PUT / DELETE）ごとに別ファイルとして生成する。

### 各セクションの埋め方
| セクション | 埋める内容 |
| --- | --- |
| API-Schema | 空欄のまま |
| API-RequestBody | 要件の RequestBody をコードブロックに記述 |
| API-Response | 要件の Response をコードブロックに記述 |
| DB-SELECT | 要件の使用テーブル SELECT を Obsidian リンク形式で記述 |
| DB-INSERT/UPDATE/DELETE | 要件の使用テーブル INSERT/UPDATE/DELETE を Obsidian リンク形式で記述 |
| 外部API-GET | 空欄のまま |
| シーケンス図 | 空欄のまま |

### Obsidianリンクの形式
```
[[{tableName}（{tableNameJp}）]]
```

---

## 注意事項
- 生成するファイルは必ずテンプレートのフォーマットに従うこと
- 要件ファイルに記載のない項目は空欄にする（削除しない）
- テーブルが複数ある場合はテーブル定義書を複数ファイル生成しTemplaterも複数回実行する
- APIが複数ある場合はAPI定義書を複数ファイル生成する
- サブフォルダが存在しない場合は作成する
- 生成完了後に生成したファイルの一覧を出力すること
