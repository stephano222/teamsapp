# ER 図

README の「7. このアプリで実現すること」と「10. 技術スタック」、画面遷移図（Figma）をもとにした、MVP のデータ設計です。

```mermaid
erDiagram
  users ||--o{ recruitments : "募集する"
  users ||--o{ applications : "応募する"
  users ||--o{ comments : "コメントする"
  recruitments ||--o{ roles : "役割を持つ"
  recruitments ||--o{ applications : "応募を受ける"
  recruitments ||--o{ comments : "コメントを受ける"
  recruitments ||--o{ recruitment_tags : ""
  tags ||--o{ recruitment_tags : ""
  roles ||--o{ applications : "希望される"

  users {
    bigint id PK
    string email "devise・一意"
    string encrypted_password "devise"
    string name "ユーザー名"
    boolean admin "管理者か（既定 false）"
    datetime created_at
    datetime updated_at
  }

  recruitments {
    bigint id PK
    bigint user_id FK "募集者"
    string title "タイトル"
    text purpose "目的"
    boolean beginner_welcome "初心者歓迎マーク"
    boolean new_role_welcome "未経験の役割歓迎マーク"
    integer min_members "最低稼働人数"
    integer capacity "募集人数"
    integer total_hours "総時間数"
    integer owner_weekly_hours "募集者が開発に使える時間"
    string technologies "使う技術"
    text owner_experience "経験したこと"
    text owner_goal "この開発で経験したいこと"
    integer status "enum: 募集中 承認待ち チーム結成 終了"
    integer contact_tool "enum: 連絡ツール（MVP は1つ）"
    string contact_url "招待URL（承認後にメンバーだけに表示）"
    datetime submitted_at "管理者へ提出した日時"
    datetime reviewed_at "管理者が承認・却下した日時"
    datetime team_formed_at "チーム結成の日時（2日以内の連絡の起点）"
    datetime created_at
    datetime updated_at
  }

  roles {
    bigint id PK
    bigint recruitment_id FK
    string name "役割名（フロント・バックエンドなど）"
    integer required_hours "各役割の必要時間"
    boolean new_role_ok "未経験OK"
    datetime created_at
    datetime updated_at
  }

  applications {
    bigint id PK
    bigint user_id FK "応募者"
    bigint recruitment_id FK
    bigint role_id FK "やってみたい役割"
    boolean beginner "初心者として参加か"
    boolean new_role "未経験の役割を希望か"
    integer weekly_hours "開発に使える時間"
    string technologies "使える技術"
    text experience "経験したこと"
    text can_do "今できそうなこと"
    text want_to_try "やってみたいこと"
    text goal "この開発で経験したいこと"
    boolean can_use_contact_tool "指定の連絡ツールを使えるか"
    integer status "enum: 応募中 選定 不選定"
    datetime created_at
    datetime updated_at
  }

  comments {
    bigint id PK
    bigint user_id FK
    bigint recruitment_id FK
    text body
    datetime created_at
    datetime updated_at
  }

  tags {
    bigint id PK
    string name "一意"
    datetime created_at
    datetime updated_at
  }

  recruitment_tags {
    bigint id PK
    bigint recruitment_id FK
    bigint tag_id FK
  }
```

## 設計のポイント

- **チームメンバー**は専用のテーブルを作らず、`applications.status` が「選定」の応募者で表します。連絡先（`contact_url`）は、募集の状態が「チーム結成」のときに、この応募者と募集者だけに表示します（pundit で判定）。
- **募集の状態**は `recruitments.status` の enum で持ちます。「却下」は状態ではなく「募集中に戻す」操作なので、`reviewed_at` に日時だけを残します。
- **管理者**は `users.admin` の真偽値で表します。管理者の承認・却下ページは、この値で表示を切り替えます。
- **連絡ツール**は MVP では募集テーブルの `contact_tool`（enum）と `contact_url` で持ちます。本リリースで用途ごとに複数選べるようにするときに、ツールのマスタテーブルと中間テーブルへ分けます（README の比較どおり）。
- **画像**（募集の画像投稿）は Active Storage を使うため、ER 図には出していません。`active_storage_attachments` が `recruitments` を指します。
- **タグ**は、画面遷移図の「タグ管理ページ」に合わせて `tags` と中間テーブル `recruitment_tags` で持ちます。README が本リリース用に挙げている acts-as-taggable-on（使用技術のタグ付け）とは別物です。
- 応募は、同じ人が同じ募集に二重に応募できないよう、`applications` に `user_id` と `recruitment_id` の組の一意制約をつけます。
- `comments` は、画面遷移図のとおり募集詳細ページの中で扱います。投稿・編集・削除は、コメントを書いた本人だけができます。
