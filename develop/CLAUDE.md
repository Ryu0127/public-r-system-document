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
        ├── _RunScript.md           # テーブル定義書用トリガー
        ├── _RunScript-Api.md       # API用トリガー
        └── code-templates/
            └── _Java-Template/
                ├── Infrastructure/ # Entity / Repository / Mapper 等
                ├── Contexts/       # Aggregate 等
                └── Application/    # Controller / Request / Response
```

---

## 実行順序

```
① テーブル定義書.md を生成
    ↓
② _RunScript.md 経由で Templater を実行（テーブル数分繰り返す）
   → Infra系コード（Entity / Repository / Mapper / Aggregate）を生成
    ↓
③ API定義書.md を生成（全メソッド分）
    ↓
④ _RunScript-Api.md 経由で Templater を実行（apiNameJpディレクトリ数分繰り返す）
   → Controller / Request / Response を生成
```

---

## 生成ルール

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

テーブルが複数ある場合は1ファイルごとに②を繰り返すこと。

### ② _RunScript.md を書き換えて実行する

`develop/script/_RunScript.md` の `PLACEHOLDER_PATH` を
生成したテーブル定義書のパスに書き換えて以下を実行する。

```powershell
$vaultName = "public-r-system-document"
$runScript  = "develop/script/_RunScript.md"
$encoded    = [System.Uri]::EscapeDataString($runScript)
$commandId  = [System.Uri]::EscapeDataString("templater-obsidian:replace-in-file-templater")

Start-Process "obsidian://advanced-uri?vault=$vaultName&filepath=$encoded"
Start-Sleep -Seconds 2
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

## API定義書生成後のTemplater自動実行ルール

apiNameJpディレクトリ単位（= Controllerの単位）で④を繰り返すこと。
1回の実行で Controller + Request + Response が全て生成される。

### ④ _RunScript-Api.md を書き換えて実行する

`develop/script/_RunScript-Api.md` の `PLACEHOLDER_PATH` を
対象の apiNameJp ディレクトリ内のいずれか1つのAPIファイルのパスに書き換えて以下を実行する。
（スクリプトがディレクトリ内の全ファイルを自動で読み込むため、どのファイルでも良い）

```powershell
$vaultName = "public-r-system-document"
$runScript  = "develop/script/_RunScript-Api.md"
$encoded    = [System.Uri]::EscapeDataString($runScript)
$commandId  = [System.Uri]::EscapeDataString("templater-obsidian:replace-in-file-templater")

Start-Process "obsidian://advanced-uri?vault=$vaultName&filepath=$encoded"
Start-Sleep -Seconds 2
Start-Process "obsidian://advanced-uri?vault=$vaultName&filepath=$encoded&commandid=$commandId"
Start-Sleep -Seconds 3
```

### 生成されるファイル
```
develop/generate-code/{timestamp}/
└── src/main/java/com/example/api/
    └── application/
        ├── controller/
        │   └── {ApiName}Controller.java
        └── resource/
            ├── request/
            │   └── {ApiName}{Http}Request.java  （メソッド数分）
            └── response/
                └── {ApiName}{Http}Response.java （メソッド数分）
```

---

## 注意事項
- 生成するファイルは必ずテンプレートのフォーマットに従うこと
- 要件ファイルに記載のない項目は空欄にする（削除しない）
- テーブルが複数ある場合はテーブル定義書を複数ファイル生成しTemplaterも複数回実行する
- APIが複数ある場合はAPI定義書を複数ファイル生成する
- apiNameJpディレクトリが複数ある場合はTemplaterをディレクトリ数分実行する
- サブフォルダが存在しない場合は作成する
- 生成完了後に生成したファイルの一覧を出力すること
