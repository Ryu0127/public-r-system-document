---
featureName: FeatureName
featureNameJp: 機能名
date: 2026-05-09
---

## 概要
<!-- この機能で何を実現するかを記述 -->

## テーブル定義

### テーブル1
- tableName: tbl_example
- tableNameJp: サンプル
- className: TblExample

| カラム名 | 型 | Javaの型 | NULL | 備考 |
| --- | --- | --- | --- | --- |
| id | int(11) | Integer | NO | PK, AI |
| name | varchar(255) | String | YES | 名前 |
| created_datetime | datetime | LocalDateTime | YES | 作成日時 |
| updated_datetime | datetime | LocalDateTime | YES | 更新日時 |

<!-- テーブルが複数ある場合は ### テーブル2 として追加 -->

## API定義

### API1
- apiNameJp: サンプルAPI
- apiName: SampleApi
- url: /sample
- HTTP: POST

**RequestBody**
```json
{
  "name": "string"
}
```

**Response**
```json
{
  "statusCode": 0,
  "message": "string"
}
```

**使用テーブル**
- INSERT: tbl_example
- SELECT: （なし）

<!-- APIが複数ある場合は ### API2 として追加 -->

## 処理概要
<!-- 処理の流れを箇条書きで記述 -->
