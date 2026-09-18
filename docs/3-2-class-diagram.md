
# クラス図

## 1. 概要

空間メモを構成するModel、View、ViewModel、Serviceの関係を示す。

図が複雑になることを防ぐため、Modelクラス図とMVVM構成図を分けて作成する。

## 2. Modelクラス図

```mermaid
classDiagram
    direction LR

    class AppUser {
        +String userId
        +String userName
        +String email
        +Date createdAt
        +Date updatedAt
    }

    class Room {
        +String roomId
        +String ownerId
        +String roomName
        +String imageUrl
        +String imagePublicId
        +Date createdAt
        +Date updatedAt
    }

    class Memo {
        +String memoId
        +String roomId
        +String createdBy
        +String title
        +String body
        +Double pinX
        +Double pinY
        +PinColor pinColor
        +String? imageUrl
        +String? imagePublicId
        +Date createdAt
        +Date updatedAt
    }

    class RoomMember {
        +String roomMemberId
        +String roomId
        +String userId
        +String role
        +Date joinedAt
    }

    class PinColor {
        <<enumeration>>
        dustyPink
        peachBeige
        softYellow
        sage
        mint
        dustyAqua
        smokyBlue
        lavender
        mauve
        greige
    }

    class ImageUploadResult {
        +String imageUrl
        +String imagePublicId
    }

    AppUser "1" --> "0..*" Room : owns
    AppUser "1" --> "0..*" Memo : creates
    AppUser "1" --> "0..*" RoomMember : joins
    Room "1" *-- "0..*" Memo : contains
    Room "1" *-- "0..*" RoomMember : shares
    Memo --> PinColor : uses
```

`ImageUploadResult`はCloudinaryへのアップロード結果を一時的に受け取るためのModelであり、Firestoreにはそのまま保存しない。

## 3. MVVM構成図

```mermaid
classDiagram
    direction LR

    class LoginView {
        <<View>>
    }

    class AccountRegistrationView {
        <<View>>
    }

    class RoomListView {
        <<View>>
    }

    class RoomRegistrationView {
        <<View>>
    }

    class RoomDetailView {
        <<View>>
    }

    class MemoEditorView {
        <<View>>
    }

    class SharingManagementView {
        <<View>>
    }

    class SettingsView {
        <<View>>
    }

    class AuthViewModel {
        <<ViewModel>>
        +Bool isAuthenticated
        +AppUser? currentUser
        +login(email, password)
        +register(userName, email, password)
        +logout()
    }

    class RoomListViewModel {
        <<ViewModel>>
        +Array~Room~ rooms
        +Bool isLoading
        +loadRooms()
    }

    class RoomRegistrationViewModel {
        <<ViewModel>>
        +String roomName
        +Data? selectedImage
        +Bool isSaving
        +saveRoom()
    }

    class RoomDetailViewModel {
        <<ViewModel>>
        +Room? room
        +Array~Memo~ memos
        +loadRoom()
        +loadMemos()
        +updateRoomImage()
    }

    class MemoEditorViewModel {
        <<ViewModel>>
        +String title
        +String body
        +PinColor selectedColor
        +Data? selectedImage
        +saveMemo()
    }

    class SharingViewModel {
        <<ViewModel>>
        +Array~RoomMember~ members
        +String email
        +loadMembers()
        +addMember()
    }

    class SettingsViewModel {
        <<ViewModel>>
        +AppUser? currentUser
        +loadUser()
        +logout()
    }

    class AuthService {
        <<Service>>
        +register(email, password)
        +login(email, password)
        +logout()
        +currentUserId()
    }

    class UserService {
        <<Service>>
        +createUser(user)
        +fetchUser(userId)
    }

    class RoomService {
        <<Service>>
        +fetchRooms(userId)
        +fetchRoom(roomId)
        +createRoom(room)
        +updateRoom(room)
    }

    class MemoService {
        <<Service>>
        +fetchMemos(roomId)
        +fetchMemo(roomId, memoId)
        +createMemo(memo)
        +updateMemo(memo)
    }

    class SharingService {
        <<Service>>
        +fetchMembers(roomId)
        +addMember(roomId, email)
    }

    class CloudinaryService {
        <<Service>>
        +uploadImage(data)
    }

    LoginView --> AuthViewModel
    AccountRegistrationView --> AuthViewModel
    RoomListView --> RoomListViewModel
    RoomRegistrationView --> RoomRegistrationViewModel
    RoomDetailView --> RoomDetailViewModel
    MemoEditorView --> MemoEditorViewModel
    SharingManagementView --> SharingViewModel
    SettingsView --> SettingsViewModel

    AuthViewModel ..> AuthService
    AuthViewModel ..> UserService
    RoomListViewModel ..> RoomService
    RoomRegistrationViewModel ..> RoomService
    RoomRegistrationViewModel ..> CloudinaryService
    RoomDetailViewModel ..> RoomService
    RoomDetailViewModel ..> MemoService
    RoomDetailViewModel ..> CloudinaryService
    MemoEditorViewModel ..> MemoService
    MemoEditorViewModel ..> CloudinaryService
    SharingViewModel ..> SharingService
    SettingsViewModel ..> UserService
    SettingsViewModel ..> AuthService
```

## 4. クラスの種類

| 種類 | Swiftでの形式 | 役割 |
|---|---|---|
| Model | `struct`または`enum` | アプリで扱うデータを表す |
| View | `struct` | SwiftUIの画面を表示する |
| ViewModel | `class` | 画面状態と処理を管理する |
| Service | `class` | FirebaseやCloudinaryとの通信を行う |

## 5. 矢印の意味

| 表記 | 意味 |
|---|---|
| `View → ViewModel` | ViewがViewModelを使用する |
| `ViewModel ⇢ Service` | ViewModelがServiceを呼び出す |
| `1 → 0..*` | 1件のデータが0件以上のデータと関係する |
| `◆` | 親データが子データを所有する関係 |
