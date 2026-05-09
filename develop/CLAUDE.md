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
                ├── Contexts/       # Aggregate / AggregateList
                └── Application/    # Controller / Request / Response
```

---

## 実行順序

```
① テーブル定義書.md を生成
    ↓
② _RunScript.md 経由で Templater を実行（テーブル数分繰り返す）
   → Infra系コード（Entity / Repository / Mapper / Aggregate / AggregateList）を生成
    ↓
③ API定義書.md を生成（全メソッド分）
    ↓
④ _RunScript-Api.md 経由で Templater を実行（apiNameJpディレクトリ数分繰り返す）
   → Controller / Request / Response を生成 → TODO実装まで続けて行う
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

## 実装統合ルール（TODO補完）

Controller / Request / Response の生成完了後、
`// TODO: implement` を以下のルールに従って実装すること。

---

### 参照するファイル

```
① 生成された Controller ファイル
   generate-code/{timestamp}/src/.../controller/{ApiName}Controller.java

② API定義書（apiNameJpディレクトリ内の全ファイル）
   develop/api/doc/{urlPath}/{apiNameJp}/*.md
   → RequestBody / Response / DB操作の内容を確認する

③ テーブル定義書（API定義書内のリンク先）
   develop/db-table/doc/{featureName}/{tableName}（{tableNameJp}）.md
   → カラム名・Javaの型・フィールド名を確認する
```

---

### Repositoryのメソッド仕様

Repositoryは必ずAggregateを経由すること。Entityを直接渡さない。

| 操作 | メソッド | 引数 | 戻り値 |
| --- | --- | --- | --- |
| 全件取得 | `all()` | なし | `{ClassName}AggregateList` |
| 1件取得 | `find(Integer id)` | id | `{ClassName}Aggregate` |
| 複数取得 | `finds(List<Integer> ids)` | ids | `{ClassName}AggregateList` |
| 登録 | `insertGeneratedKeys({ClassName}Aggregate)` | Aggregate | void |
| 更新 | `update({ClassName}Aggregate)` | Aggregate | void |
| 削除 | `delete({ClassName}Aggregate)` | Aggregate | void |

---

### INSERT の実装パターン

```java
// 1. Entityにリクエストの値をセット
HisLockDetection entity = new HisLockDetection();
entity.deviceId        = request.deviceId;
entity.deviceType      = request.deviceType;
entity.createdDatetime = LocalDateTime.now();

// 2. EntityをAggregateでラップしてRepositoryに渡す
HisLockDetectionAggregate aggregate = HisLockDetectionAggregate.factory(entity);
hisLockDetectionRepository.insertGeneratedKeys(aggregate);

// 3. Responseを組み立てて返す
SwitchBotLockPostResponse response = new SwitchBotLockPostResponse();
response.statusCode = 0;
response.message    = "success";
return response;
```

### SELECT（1件）の実装パターン

```java
// 1. Repositoryから取得（AggregateListまたはAggregateで返る）
HisLockDetectionAggregate aggregate = hisLockDetectionRepository.find(id);

// 2. getEntity()でEntityを取り出してResponseに詰める
HisLockDetection entity = aggregate.getEntity();
SwitchBotLockGetResponse response = new SwitchBotLockGetResponse();
response.statusCode  = 0;
response.message     = "success";
SwitchBotLockGetResponse.Body body = new SwitchBotLockGetResponse.Body();
body.deviceId        = entity.deviceId;
body.lockStatus      = entity.lockStatus;
response.body        = body;
return response;
```

### SELECT（全件）の実装パターン

```java
// AggregateListで返る → getAggregates()でリストを取得
HisLockDetectionAggregateList aggregateList = hisLockDetectionRepository.all();
// 以降はAggregateListからAggregateを取り出してResponseに詰める
```

---

### import追加ルール

実装に応じて以下を必要に応じてimportすること。

```java
import com.example.api.contexts.domain.aggregate.{ClassName}Aggregate;
import com.example.api.contexts.domain.collection.aggregate.{ClassName}AggregateList;
import com.example.api.infrastructure.entity.{ClassName};
import java.time.LocalDateTime;
import java.util.List;
```

---

### 実装できない箇所のルール
- 外部API呼び出しが必要な場合: `// TODO: 外部API実装` を残す
- 判断できないビジネスロジックは `// TODO: ビジネスロジック実装` を残す
- 実装完了後にimport文を整理すること
