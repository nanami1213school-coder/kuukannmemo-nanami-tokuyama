# API設計

## 1. 概要

本アプリでは、MVPの実装にFirebase Authentication、Cloud Firestore、Cloudinaryを使用する。

本設計書では、将来アプリのデータ処理をサーバー化した場合を想定し、REST APIを設計する。

認証処理はFirebase Authenticationを利用し、アプリ独自のログインAPIおよびアカウント登録APIは作成しない。

## 2. 基本仕様

| 項目 | 内容 |
|---|---|
| 通信方式 | HTTPS |
| データ形式 | JSON |
| 文字コード | UTF-8 |
| APIバージョン | v1 |
| ベースURL | `https://api.example.com/v1` |
| 認証方式 | Firebase IDトークン |
| 日時形式 | ISO 8601 |
| タイムゾーン | UTC |

`api.example.com`は設計上の仮URLであり、MVPでは実際のサーバーを構築しない。

## 3. 認証方法

認証が必要なAPIでは、HTTPヘッダーにFirebase Authenticationから取得したIDトークンを設定する。

```http
Authorization: Bearer {Firebase ID Token}
Content-Type: application/json
```

サーバーはIDトークンを検証し、ログイン中のユーザーを特定する。

## 4. API一覧

### ユーザーAPI

| API ID | メソッド | エンドポイント | 概要 | MVP |
|---|---|---|---|---:|
| API-001 | GET | `/users/me` | ログイン中のユーザー情報を取得する | ○ |
| API-002 | PATCH | `/users/me` | ユーザー情報を変更する | 将来 |

### 部屋API

| API ID | メソッド | エンドポイント | 概要 | MVP |
|---|---|---|---|---:|
| API-101 | GET | `/rooms` | 所有または共有されている部屋の一覧を取得する | ○ |
| API-102 | POST | `/rooms` | 新しい部屋を登録する | ○ |
| API-103 | GET | `/rooms/{roomId}` | 指定した部屋の情報を取得する | ○ |
| API-104 | PATCH | `/rooms/{roomId}` | 部屋名または部屋画像を変更する | ○ |
| API-105 | DELETE | `/rooms/{roomId}` | 部屋を削除する | 将来 |

### メモAPI

| API ID | メソッド | エンドポイント | 概要 | MVP |
|---|---|---|---|---:|
| API-201 | GET | `/rooms/{roomId}/memos` | 部屋に登録されたメモ一覧を取得する | ○ |
| API-202 | POST | `/rooms/{roomId}/memos` | 部屋画像上にメモを登録する | ○ |
| API-203 | GET | `/rooms/{roomId}/memos/{memoId}` | 指定したメモを取得する | ○ |
| API-204 | PATCH | `/rooms/{roomId}/memos/{memoId}` | 指定したメモを編集する | ○ |
| API-205 | DELETE | `/rooms/{roomId}/memos/{memoId}` | 指定したメモを削除する | 将来 |

### 共有管理API

| API ID | メソッド | エンドポイント | 概要 | MVP |
|---|---|---|---|---:|
| API-301 | GET | `/rooms/{roomId}/members` | 部屋を共有しているユーザーを取得する | ○ |
| API-302 | POST | `/rooms/{roomId}/members` | メールアドレスを指定してユーザーを追加する | ○ |
| API-303 | DELETE | `/rooms/{roomId}/members/{userId}` | 共有ユーザーを部屋から削除する | 将来 |

### 画像アップロードAPI

| API ID | メソッド | エンドポイント | 概要 | MVP |
|---|---|---|---|---:|
| API-401 | POST | `/uploads/signature` | Cloudinaryへの署名付きアップロード情報を取得する | 将来 |

MVPではCloudinaryのUnsigned Upload Presetを利用する。

将来サーバー化する場合は、API-401で署名を発行し、CloudinaryのAPI SecretをiOSアプリ内に保存しない構成へ変更する。

## 5. HTTPステータスコード

| ステータスコード | 意味 |
|---:|---|
| 200 | 取得または更新成功 |
| 201 | 新規登録成功 |
| 204 | 削除成功 |
| 400 | リクエスト内容が不正 |
| 401 | ログインしていない、またはトークンが無効 |
| 403 | 操作権限がない |
| 404 | 対象データが存在しない |
| 409 | 同じユーザーがすでに共有されている |
| 422 | 入力内容に問題がある |
| 500 | サーバー内部エラー |

## 6. 共通エラーレスポンス

APIでエラーが発生した場合は、以下の形式で返却する。

```json
{
  "error": {
    "code": "ROOM_NOT_FOUND",
    "message": "指定された部屋が見つかりません。"
  }
}
```

## 7. 権限

- ログイン済みユーザーのみAPIを利用できる
- 部屋の所有者と共有ユーザーは、部屋およびメモを閲覧できる
- 部屋の所有者と共有ユーザーは、メモの登録と編集ができる
- 部屋画像の変更と共有ユーザーの管理は、部屋の所有者のみ実行できる

## 8. API詳細仕様

### API-102 部屋登録

新しい部屋を登録する。

#### リクエスト

```http
POST /v1/rooms
Authorization: Bearer {Firebase ID Token}
Content-Type: application/json
```

#### リクエストボディ

```json
{
  "roomName": "リビング",
  "imageUrl": "https://res.cloudinary.com/example/image/upload/room.jpg",
  "imagePublicId": "rooms/room-image-001"
}
```

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| roomName | String | ○ | 登録する部屋名 |
| imageUrl | String | ○ | Cloudinaryに保存した画像のURL |
| imagePublicId | String | ○ | Cloudinary上の画像ID |

#### 入力条件

| 項目名 | 条件 |
|---|---|
| roomName | 1文字以上50文字以内 |
| imageUrl | URL形式であること |
| imagePublicId | 空文字ではないこと |

`roomId`、`ownerId`、`createdAt`、`updatedAt`はサーバー側で設定するため、リクエストには含めない。

#### 成功レスポンス

```http
HTTP/1.1 201 Created
Content-Type: application/json
```

```json
{
  "room": {
    "roomId": "room-001",
    "ownerId": "firebase-user-001",
    "roomName": "リビング",
    "imageUrl": "https://res.cloudinary.com/example/image/upload/room.jpg",
    "imagePublicId": "rooms/room-image-001",
    "createdAt": "2026-09-10T04:00:00Z",
    "updatedAt": "2026-09-10T04:00:00Z"
  }
}
```

#### エラーレスポンス例

```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json
```

```json
{
  "error": {
    "code": "INVALID_ROOM_NAME",
    "message": "部屋名は1文字以上50文字以内で入力してください。"
  }
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 400 | INVALID_REQUEST | JSONの形式が正しくない |
| 401 | UNAUTHORIZED | Firebase IDトークンがない、または無効 |
| 422 | INVALID_ROOM_NAME | 部屋名が入力条件を満たしていない |
| 422 | INVALID_IMAGE | 画像URLまたはPublic IDが正しくない |
| 500 | INTERNAL_SERVER_ERROR | サーバー内部で処理に失敗した |

#### 処理内容

1. Firebase IDトークンを検証する
2. ログインユーザーの`userId`を取得する
3. リクエスト内容を検証する
4. 新しい`roomId`を発行する
5. ログインユーザーのIDを`ownerId`に設定する
6. 部屋情報をデータベースへ保存する
7. 登録した部屋情報を返却する



### API-101 部屋一覧取得

ログイン中のユーザーが所有している部屋と、共有されている部屋の一覧を取得する。

#### リクエスト

```http
GET /v1/rooms
Authorization: Bearer {Firebase ID Token}
```

リクエストボディは使用しない。

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "rooms": [
    {
      "roomId": "room-001",
      "ownerId": "firebase-user-001",
      "roomName": "リビング",
      "imageUrl": "https://res.cloudinary.com/example/image/upload/living-room.jpg",
      "accessRole": "owner",
      "createdAt": "2026-09-10T04:00:00Z",
      "updatedAt": "2026-09-10T04:00:00Z"
    },
    {
      "roomId": "room-002",
      "ownerId": "firebase-user-002",
      "roomName": "会議室A",
      "imageUrl": "https://res.cloudinary.com/example/image/upload/meeting-room.jpg",
      "accessRole": "editor",
      "createdAt": "2026-09-11T06:00:00Z",
      "updatedAt": "2026-09-12T07:30:00Z"
    }
  ]
}
```

#### レスポンス項目

| 項目名 | 型 | 説明 |
|---|---|---|
| rooms | Array | 取得した部屋の配列 |
| roomId | String | 部屋を識別するID |
| ownerId | String | 部屋を所有するユーザーのID |
| roomName | String | 部屋名 |
| imageUrl | String | 部屋画像のURL |
| accessRole | String | ログインユーザーの権限 |
| createdAt | String | 部屋の作成日時 |
| updatedAt | String | 部屋の更新日時 |

`accessRole`には、所有者の場合は`owner`、共有ユーザーの場合は`editor`が設定される。

対象の部屋がない場合は、エラーではなく空の配列を返す。

```json
{
  "rooms": []
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンがない、または無効 |
| 500 | INTERNAL_SERVER_ERROR | 部屋一覧の取得に失敗した |

#### 処理内容

1. Firebase IDトークンを検証する
2. ログインユーザーの`userId`を取得する
3. ユーザーが所有または共有されている部屋を検索する
4. 各部屋の`accessRole`を判定する
5. 部屋一覧を返却する

---

### API-103 部屋詳細取得

指定された部屋の詳細情報を取得する。

#### パスパラメータ

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| roomId | String | ○ | 取得する部屋のID |

#### リクエスト

```http
GET /v1/rooms/room-001
Authorization: Bearer {Firebase ID Token}
```

リクエストボディは使用しない。

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "room": {
    "roomId": "room-001",
    "ownerId": "firebase-user-001",
    "roomName": "リビング",
    "imageUrl": "https://res.cloudinary.com/example/image/upload/living-room.jpg",
    "imagePublicId": "rooms/living-room-001",
    "accessRole": "owner",
    "createdAt": "2026-09-10T04:00:00Z",
    "updatedAt": "2026-09-10T04:00:00Z"
  }
}
```

メモ情報はこのAPIには含めず、`API-201 メモ一覧取得`で別に取得する。

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンがない、または無効 |
| 403 | FORBIDDEN | 部屋を閲覧する権限がない |
| 404 | ROOM_NOT_FOUND | 指定された部屋が存在しない |
| 500 | INTERNAL_SERVER_ERROR | 部屋情報の取得に失敗した |

#### エラーレスポンス例

```http
HTTP/1.1 404 Not Found
Content-Type: application/json
```

```json
{
  "error": {
    "code": "ROOM_NOT_FOUND",
    "message": "指定された部屋が見つかりません。"
  }
}
```

#### 処理内容

1. Firebase IDトークンを検証する
2. 指定された`roomId`の部屋を検索する
3. ログインユーザーが所有者または共有ユーザーか確認する
4. ログインユーザーの`accessRole`を判定する
5. 部屋の詳細情報を返却する


### API-104 部屋更新

指定した部屋の名前または部屋画像を変更する。

部屋を更新できるのは、部屋の所有者だけとする。

#### パスパラメータ

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| roomId | String | ○ | 更新する部屋のID |

#### リクエスト

```http
PATCH /v1/rooms/room-001
Authorization: Bearer {Firebase ID Token}
Content-Type: application/json
```

```json
{
  "roomName": "新しいリビング",
  "imageUrl": "https://res.cloudinary.com/example/image/upload/new-room.jpg",
  "imagePublicId": "rooms/new-room-001"
}
```

#### リクエスト項目

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| roomName | String |  | 変更後の部屋名 |
| imageUrl | String |  | 変更後の部屋画像URL |
| imagePublicId | String |  | 変更後のCloudinary Public ID |

変更する項目だけを送信する。ただし、画像を変更する場合は`imageUrl`と`imagePublicId`を両方送信する。

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "room": {
    "roomId": "room-001",
    "ownerId": "firebase-user-001",
    "roomName": "新しいリビング",
    "imageUrl": "https://res.cloudinary.com/example/image/upload/new-room.jpg",
    "imagePublicId": "rooms/new-room-001",
    "accessRole": "owner",
    "createdAt": "2026-09-10T04:00:00Z",
    "updatedAt": "2026-09-17T03:00:00Z"
  }
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 400 | INVALID_REQUEST | 更新項目が指定されていない |
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 403 | FORBIDDEN | ログインユーザーが部屋の所有者ではない |
| 404 | ROOM_NOT_FOUND | 指定された部屋が存在しない |
| 422 | INVALID_ROOM_NAME | 部屋名が入力条件を満たしていない |
| 422 | INVALID_IMAGE | 画像情報が正しくない |
| 500 | INTERNAL_SERVER_ERROR | 部屋情報の更新に失敗した |

---

### API-201 メモ一覧取得

指定した部屋に登録されているメモの一覧を取得する。

#### リクエスト

```http
GET /v1/rooms/room-001/memos
Authorization: Bearer {Firebase ID Token}
```

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "memos": [
    {
      "memoId": "memo-001",
      "title": "エアコンの型番",
      "pinX": 0.72,
      "pinY": 0.31,
      "pinColor": "sage",
      "updatedAt": "2026-09-17T03:30:00Z"
    },
    {
      "memoId": "memo-002",
      "title": "電池交換",
      "pinX": 0.25,
      "pinY": 0.68,
      "pinColor": "dusty-pink",
      "updatedAt": "2026-09-17T04:00:00Z"
    }
  ]
}
```

部屋にメモがない場合は、空の配列を返す。

```json
{
  "memos": []
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 403 | FORBIDDEN | 部屋を閲覧する権限がない |
| 404 | ROOM_NOT_FOUND | 指定された部屋が存在しない |
| 500 | INTERNAL_SERVER_ERROR | メモ一覧の取得に失敗した |

---

### API-202 メモ登録

指定した部屋画像上に新しいメモを登録する。

#### リクエスト

```http
POST /v1/rooms/room-001/memos
Authorization: Bearer {Firebase ID Token}
Content-Type: application/json
```

```json
{
  "title": "エアコンの型番",
  "body": "型番：ABC-1234\n2026年9月にフィルターを交換",
  "pinX": 0.72,
  "pinY": 0.31,
  "pinColor": "sage",
  "imageUrl": "https://res.cloudinary.com/example/image/upload/air-conditioner.jpg",
  "imagePublicId": "memos/air-conditioner-001"
}
```

#### リクエスト項目

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| title | String | ○ | メモのタイトル |
| body | String |  | 自由記載のメモ本文 |
| pinX | Double | ○ | 画像上の横位置 |
| pinY | Double | ○ | 画像上の縦位置 |
| pinColor | String | ○ | 選択したピンカラーID |
| imageUrl | String |  | 添付画像のURL |
| imagePublicId | String |  | 添付画像のCloudinary Public ID |

#### 入力条件

| 項目名 | 条件 |
|---|---|
| title | 1文字以上100文字以内 |
| body | 2,000文字以内 |
| pinX | `0.0`以上`1.0`以下 |
| pinY | `0.0`以上`1.0`以下 |
| pinColor | 定義済みの10色から選択する |
| 添付画像 | URLとPublic IDを両方指定するか、両方省略する |

`memoId`、`roomId`、`createdBy`、`createdAt`、`updatedAt`はサーバー側で設定する。

#### 成功レスポンス

```http
HTTP/1.1 201 Created
Content-Type: application/json
```

```json
{
  "memo": {
    "memoId": "memo-001",
    "roomId": "room-001",
    "createdBy": "firebase-user-001",
    "title": "エアコンの型番",
    "body": "型番：ABC-1234\n2026年9月にフィルターを交換",
    "pinX": 0.72,
    "pinY": 0.31,
    "pinColor": "sage",
    "imageUrl": "https://res.cloudinary.com/example/image/upload/air-conditioner.jpg",
    "imagePublicId": "memos/air-conditioner-001",
    "createdAt": "2026-09-17T04:30:00Z",
    "updatedAt": "2026-09-17T04:30:00Z"
  }
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 400 | INVALID_REQUEST | JSONの形式が正しくない |
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 403 | FORBIDDEN | 部屋へメモを登録する権限がない |
| 404 | ROOM_NOT_FOUND | 指定された部屋が存在しない |
| 422 | INVALID_TITLE | タイトルが入力条件を満たしていない |
| 422 | INVALID_PIN_POSITION | ピン位置が範囲外 |
| 422 | INVALID_PIN_COLOR | 定義されていない色が指定された |
| 422 | INVALID_IMAGE | 添付画像情報が正しくない |
| 500 | INTERNAL_SERVER_ERROR | メモの登録に失敗した |

#### 処理内容

1. Firebase IDトークンを検証する
2. 指定した部屋が存在するか確認する
3. ユーザーが所有者または共有ユーザーか確認する
4. リクエスト内容を検証する
5. 新しい`memoId`を発行する
6. ピン位置とメモ内容を保存する
7. 登録したメモ情報を返却する


### API-203 メモ詳細取得

指定したメモの内容を取得する。

#### リクエスト

```http
GET /v1/rooms/room-001/memos/memo-001
Authorization: Bearer {Firebase ID Token}
```

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "memo": {
    "memoId": "memo-001",
    "roomId": "room-001",
    "createdBy": "firebase-user-001",
    "title": "エアコンの型番",
    "body": "型番：ABC-1234\n2026年9月にフィルターを交換",
    "pinX": 0.72,
    "pinY": 0.31,
    "pinColor": "sage",
    "imageUrl": "https://res.cloudinary.com/example/image/upload/air-conditioner.jpg",
    "imagePublicId": "memos/air-conditioner-001",
    "createdAt": "2026-09-17T04:30:00Z",
    "updatedAt": "2026-09-17T04:30:00Z"
  }
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 403 | FORBIDDEN | 部屋を閲覧する権限がない |
| 404 | ROOM_NOT_FOUND | 部屋が存在しない |
| 404 | MEMO_NOT_FOUND | メモが存在しない |
| 500 | INTERNAL_SERVER_ERROR | メモの取得に失敗した |

---

### API-204 メモ更新

指定したメモの内容を変更する。

部屋の所有者と共有ユーザーのどちらもメモを編集できる。

#### リクエスト

```http
PATCH /v1/rooms/room-001/memos/memo-001
Authorization: Bearer {Firebase ID Token}
Content-Type: application/json
```

```json
{
  "title": "エアコンの型番と点検記録",
  "body": "型番：ABC-1234\n次回点検予定：2026年12月",
  "pinColor": "lavender"
}
```

#### 更新可能な項目

| 項目名 | 型 | 説明 |
|---|---|---|
| title | String | メモのタイトル |
| body | String | メモ本文 |
| pinColor | String | ピンカラーID |
| imageUrl | Stringまたはnull | 添付画像URL |
| imagePublicId | Stringまたはnull | Cloudinary Public ID |

変更する項目だけを送信する。

添付画像を削除する場合は、以下のように両方へ`null`を指定する。

```json
{
  "imageUrl": null,
  "imagePublicId": null
}
```

MVPではピン位置の移動を行わないため、`pinX`と`pinY`は更新対象に含めない。

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "memo": {
    "memoId": "memo-001",
    "roomId": "room-001",
    "createdBy": "firebase-user-001",
    "title": "エアコンの型番と点検記録",
    "body": "型番：ABC-1234\n次回点検予定：2026年12月",
    "pinX": 0.72,
    "pinY": 0.31,
    "pinColor": "lavender",
    "imageUrl": "https://res.cloudinary.com/example/image/upload/air-conditioner.jpg",
    "imagePublicId": "memos/air-conditioner-001",
    "createdAt": "2026-09-17T04:30:00Z",
    "updatedAt": "2026-09-17T05:00:00Z"
  }
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 400 | INVALID_REQUEST | 更新項目が指定されていない |
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 403 | FORBIDDEN | メモを編集する権限がない |
| 404 | MEMO_NOT_FOUND | メモが存在しない |
| 422 | INVALID_TITLE | タイトルが入力条件を満たしていない |
| 422 | INVALID_PIN_COLOR | 定義されていない色が指定された |
| 422 | INVALID_IMAGE | 添付画像情報が正しくない |
| 500 | INTERNAL_SERVER_ERROR | メモの更新に失敗した |

---

### API-301 共有メンバー一覧取得

指定した部屋の所有者と共有ユーザーを取得する。

共有情報を取得できるのは部屋の所有者だけとする。

#### リクエスト

```http
GET /v1/rooms/room-001/members
Authorization: Bearer {Firebase ID Token}
```

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "owner": {
    "userId": "firebase-user-001",
    "userName": "Nanami",
    "email": "nanami@example.com",
    "role": "owner"
  },
  "members": [
    {
      "userId": "firebase-user-002",
      "userName": "Sakura",
      "email": "sakura@example.com",
      "role": "editor",
      "joinedAt": "2026-09-17T05:30:00Z"
    }
  ]
}
```

共有ユーザーがいない場合は、`members`に空の配列を返す。

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 403 | FORBIDDEN | ログインユーザーが部屋の所有者ではない |
| 404 | ROOM_NOT_FOUND | 部屋が存在しない |
| 500 | INTERNAL_SERVER_ERROR | 共有情報の取得に失敗した |

---

### API-302 共有メンバー追加

メールアドレスを指定して、登録済みユーザーを部屋へ追加する。

共有メンバーを追加できるのは部屋の所有者だけとする。

#### リクエスト

```http
POST /v1/rooms/room-001/members
Authorization: Bearer {Firebase ID Token}
Content-Type: application/json
```

```json
{
  "email": "sakura@example.com"
}
```

追加するユーザーの権限は、サーバー側で`editor`に設定する。

#### 成功レスポンス

```http
HTTP/1.1 201 Created
Content-Type: application/json
```

```json
{
  "member": {
    "roomMemberId": "member-001",
    "roomId": "room-001",
    "userId": "firebase-user-002",
    "userName": "Sakura",
    "email": "sakura@example.com",
    "role": "editor",
    "joinedAt": "2026-09-17T05:30:00Z"
  }
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 403 | FORBIDDEN | ログインユーザーが部屋の所有者ではない |
| 404 | ROOM_NOT_FOUND | 部屋が存在しない |
| 404 | USER_NOT_FOUND | メールアドレスに該当するユーザーがいない |
| 409 | MEMBER_ALREADY_EXISTS | ユーザーがすでに共有されている |
| 422 | INVALID_EMAIL | メールアドレスの形式が正しくない |
| 500 | INTERNAL_SERVER_ERROR | 共有メンバーの追加に失敗した |

#### 処理内容

1. Firebase IDトークンを検証する
2. ログインユーザーが部屋の所有者か確認する
3. メールアドレスに該当するユーザーを検索する
4. すでに共有されていないか確認する
5. `editor`権限で共有情報を登録する
6. 登録した共有メンバー情報を返却する

---

### API-303 共有メンバー削除

指定したユーザーを部屋の共有メンバーから削除する。

この機能は将来機能とする。

#### リクエスト

```http
DELETE /v1/rooms/room-001/members/firebase-user-002
Authorization: Bearer {Firebase ID Token}
```

#### 成功レスポンス

```http
HTTP/1.1 204 No Content
```

レスポンスボディは返さない。

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 403 | FORBIDDEN | ログインユーザーが部屋の所有者ではない |
| 404 | ROOM_NOT_FOUND | 部屋が存在しない |
| 404 | MEMBER_NOT_FOUND | 指定した共有メンバーが存在しない |
| 422 | OWNER_CANNOT_BE_REMOVED | 部屋の所有者を削除しようとした |
| 500 | INTERNAL_SERVER_ERROR | 共有メンバーの削除に失敗した |

### API-001 ログインユーザー情報取得

ログイン中のユーザー情報を取得する。

設定画面にユーザー名とメールアドレスを表示するために使用する。

#### リクエスト

```http
GET /v1/users/me
Authorization: Bearer {Firebase ID Token}
```

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "user": {
    "userId": "firebase-user-001",
    "userName": "Nanami",
    "email": "nanami@example.com",
    "createdAt": "2026-09-10T01:00:00Z",
    "updatedAt": "2026-09-10T01:00:00Z"
  }
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンがない、または無効 |
| 404 | USER_NOT_FOUND | ユーザー情報が存在しない |
| 500 | INTERNAL_SERVER_ERROR | ユーザー情報の取得に失敗した |

---

### API-002 ログインユーザー情報更新

ログイン中のユーザー名を変更する。

この機能は将来機能とする。

#### リクエスト

```http
PATCH /v1/users/me
Authorization: Bearer {Firebase ID Token}
Content-Type: application/json
```

```json
{
  "userName": "新しいユーザー名"
}
```

メールアドレスとパスワードの変更は、本人確認や再認証が必要なため、Firebase Authenticationを利用して別途処理する。

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "user": {
    "userId": "firebase-user-001",
    "userName": "新しいユーザー名",
    "email": "nanami@example.com",
    "createdAt": "2026-09-10T01:00:00Z",
    "updatedAt": "2026-09-17T06:00:00Z"
  }
}
```

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 404 | USER_NOT_FOUND | ユーザー情報が存在しない |
| 422 | INVALID_USER_NAME | ユーザー名が入力条件を満たしていない |
| 500 | INTERNAL_SERVER_ERROR | ユーザー情報の更新に失敗した |

---

### API-401 Cloudinaryアップロード署名発行

Cloudinaryへ署名付きで画像をアップロードするための署名情報を取得する。

このAPIは将来サーバー化した場合に使用する。MVPではCloudinaryのUnsigned Upload Presetを利用する。

#### リクエスト

```http
POST /v1/uploads/signature
Authorization: Bearer {Firebase ID Token}
Content-Type: application/json
```

```json
{
  "uploadType": "room"
}
```

#### リクエスト項目

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| uploadType | String | ○ | 画像の用途。`room`または`memo` |

#### 成功レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "upload": {
    "cloudName": "example-cloud",
    "apiKey": "123456789012345",
    "timestamp": 1789610400,
    "signature": "generated-cloudinary-signature",
    "folder": "space-memo/firebase-user-001/rooms",
    "expiresAt": "2026-09-17T06:05:00Z"
  }
}
```

CloudinaryのAPI Secretは、レスポンスに含めずサーバー内だけで管理する。

#### 主なエラー

| ステータスコード | エラーコード | 発生条件 |
|---:|---|---|
| 401 | UNAUTHORIZED | Firebase IDトークンが無効 |
| 422 | INVALID_UPLOAD_TYPE | `room`または`memo`以外が指定された |
| 500 | SIGNATURE_GENERATION_FAILED | 署名の生成に失敗した |

#### 処理内容

1. Firebase IDトークンを検証する
2. ログインユーザーの`userId`を取得する
3. `uploadType`を検証する
4. ユーザーごとの保存先フォルダを決定する
5. サーバー内のCloudinary API Secretで署名を生成する
6. 署名とアップロードに必要な情報を返却する
7. iOSアプリからCloudinaryへ画像を直接アップロードする

## 9. API設計上の注意事項

- すべての通信にHTTPSを使用する
- Firebase IDトークンでログインユーザーを確認する
- `ownerId`や`createdBy`をアプリから自由に指定させない
- 取得・更新前に部屋へのアクセス権限を確認する
- Cloudinary API SecretをiOSアプリやGitHubに保存しない
- 日時はサーバー側で設定する
- エラー形式を全APIで統一する
- MVPではREST APIを実装せず、将来サーバー化した場合の設計として扱う
