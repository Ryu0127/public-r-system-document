---
featureName: User
featureNameJp: ユーザー
date: 2026-05-09
subdirectory:
---

## 概要
<!-- この機能で何を実現するかを記述 -->

## テーブル定義

### テーブル1
- tableName: tbl_user
- tableNameJp: ユーザー
- className: TblUser

| カラム名 | 型 | Javaの型 | NULL | 備考 |
| --- | --- | --- | --- | --- |
| id | int(11) | Integer | NO | PK, AI |
| user_name | varchar(255) | String | YES | ユーザー名 |
| email | varchar(255) | String | YES | メールアドレス |
| created_datetime | datetime | LocalDateTime | YES | 作成日時 |
| updated_datetime | datetime | LocalDateTime | YES | 更新日時 |

<!-- テーブルが複数ある場合は ### テーブル2 として追加 -->

## API定義

<!-- 同一URLのAPIは apiNameJp を統一してまとめる -->
<!-- HTTPメソッドが異なる場合は ### API2 として追加する -->

### API1
- apiNameJp: ユーザー情報API
- apiName: User
- url: /user
- HTTP: GET

**RequestBody**
```json
```

**Response**
```json
{
  "statusCode": 0,
  "body": {
    "id": 0,
    "userName": "string",
    "email": "string"
  },
  "message": "string"
}
```

**使用テーブル**
- INSERT: なし
- SELECT: tbl_user

### API2
- apiNameJp: ユーザー情報API
- apiName: User
- url: /user
- HTTP: POST

**RequestBody**
```json
{
  "userName": "string",
  "email": "string"
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
- INSERT: tbl_user
- SELECT: なし

<!-- URLが異なる場合は apiNameJp も変えて ### API3 として追加する -->

## 処理概要
<!-- 処理の流れを箇条書きで記述 -->
