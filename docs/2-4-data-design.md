# データ設計

## 1. User（ユーザー）

アプリを利用するユーザーの情報を管理する。

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| userId | String | ○ | Firebase AuthenticationのUID |
| userName | String | ○ | アプリ上に表示するユーザー名 |
| email | String | ○ | ログインおよび共有先の指定に使用する |
| createdAt | Timestamp | ○ | アカウント作成日時 |
| updatedAt | Timestamp | ○ | 更新日時 |

パスワードはFirestoreには保存せず、Firebase Authenticationで管理する。

## 2. Room（部屋）

登録した部屋と部屋写真の情報を管理する。

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| roomId | String | ○ | 部屋を識別するID |
| ownerId | String | ○ | 部屋を作成したユーザーのID |
| roomName | String | ○ | 部屋名 |
| imageUrl | String | ○ | Cloudinaryに保存した部屋画像のURL |
| imagePublicId | String | ○ | Cloudinary上の画像を識別するID |
| createdAt | Timestamp | ○ | 部屋の作成日時 |
| updatedAt | Timestamp | ○ | 部屋の更新日時 |

画像本体はCloudinaryに保存し、Firestoreには画像URLとPublic IDを保存する。

## 3. Memo（メモ）

部屋画像上に配置するメモを管理する。

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| memoId | String | ○ | メモを識別するID |
| roomId | String | ○ | メモを配置した部屋のID |
| createdBy | String | ○ | メモを作成したユーザーのID |
| title | String | ○ | メモのタイトル |
| body | String |  | 自由記載のメモ本文 |
| pinX | Double | ○ | 部屋画像上の横方向の位置 |
| pinY | Double | ○ | 部屋画像上の縦方向の位置 |
| pinColor | String | ○ | 選択したピンカラー |
| imageUrl | String |  | 添付画像のURL |
| imagePublicId | String |  | Cloudinary上の添付画像ID |
| createdAt | Timestamp | ○ | メモの作成日時 |
| updatedAt | Timestamp | ○ | メモの更新日時 |

`pinX`と`pinY`は、端末の画面サイズが変わっても同じ位置にピンを表示できるように、画像に対する割合として保存する。

例：

```text
画像の中央：pinX = 0.5、pinY = 0.5
画像の左上：pinX = 0.0、pinY = 0.0
画像の右下：pinX = 1.0、pinY = 1.0
```

## 4. RoomMember（共有メンバー）

部屋を共有しているユーザーを管理する。

| 項目名 | 型 | 必須 | 説明 |
|---|---|---:|---|
| roomMemberId | String | ○ | 共有情報を識別するID |
| roomId | String | ○ | 共有する部屋のID |
| userId | String | ○ | 共有相手のユーザーID |
| role | String | ○ | 部屋内での権限 |
| joinedAt | Timestamp | ○ | 部屋が共有された日時 |

### 権限

| role | 権限 |
|---|---|
| owner | 部屋・メモの操作および共有メンバーの管理ができる |
| editor | 部屋を閲覧し、メモの追加・編集ができる |

## データの関係

- 1人のユーザーは複数の部屋を所有できる
- 1つの部屋には複数のメモを登録できる
- 1つの部屋を複数のユーザーと共有できる
- 1人のユーザーは複数の部屋に参加できる

```text
User  1 ─── 0..* Room
Room  1 ─── 0..* Memo
User  1 ─── 0..* RoomMember
Room  1 ─── 0..* RoomMember
```

