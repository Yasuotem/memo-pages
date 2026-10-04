```mermaid
graph TD
    A[開始] --> B[処理1]
    B --> C[処理2]
    C --> D[終了]
```

just-the-docs ではこれがそのまま図になる。

---

# 🧩 1. フローチャート（graph）

### TD = Top → Down  
### LR = Left → Right


```mermaid
graph TD
    A[入力] --> B[検証]
    B -->|OK| C[処理]
    B -->|NG| D[エラー]
```

---

# 🧩 2. シーケンス図（sequenceDiagram）

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: Request
    Server-->>Client: Response
```

FastAPI や TLS の流れを書くときに相性が良い。

---

# 🧩 3. クラス図（classDiagram）

```mermaid
classDiagram
    class User {
        +id: int
        +name: string
        +login()
    }

    class AuthService {
        +login(user: User)
    }

    User --> AuthService
```

Python / FastAPI の構造メモに使える。

---

# 🧩 4. 状態遷移図（stateDiagram）

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Running
    Running --> Error
    Error --> Idle
```

TLS ハンドシェイクや API 状態管理に使える。

---

# 🧩 5. Git フローチャート（gitGraph）

```mermaid
gitGraph
    commit id: "init"
    branch feature
    commit id: "work"
    checkout main
    merge feature
```

Git の学習メモに最適。

---

# 🧩 6. Mermaid を just-the-docs で使うときの注意点（構造的）

### ✔ 1. コードブロックは必ずバッククォート3つ  

```mermaid
```


### ✔ 2. インデントは不要  
Mermaid はインデントに敏感なので、  
**コードブロック内は左端に揃える**のが安全。

### ✔ 3. GitHub Pages の反映に少しラグが出ることがある  
Mermaid を含むページはビルドが少し重いので、  
反映まで数十秒〜数分かかることがある。

---

# 🧩 Mermaid テンプレ

## FastAPI のリクエストフロー

```mermaid
sequenceDiagram
    participant Client
    participant FastAPI
    participant Router
    participant Handler

    Client->>FastAPI: HTTP Request
    FastAPI->>Router: Route match
    Router->>Handler: Call function
    Handler-->>Client: JSON Response
```

## TLS ハンドシェイク（簡易版）

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: ClientHello
    Server->>Client: ServerHello + Cert
    Client->>Server: Key Exchange
    Server-->>Client: Finished
```

## DuckDB の処理フロー

```mermaid
graph LR
    A[CSV/Parquet] --> B[DuckDB]
    B --> C[SQL]
    C --> D[結果セット]
```

もちろん、Yasuo。  
**Mermaid で ER 図を書くときの“最小限で美しいサンプル”**をいくつか用意したよ。  
just-the-docs でも GitHub Pages でもそのまま動くし、  
pandoc で使う場合は前回説明したように **SVG に変換して使う**のが構造的に最適。

---

## 🧩 基本的な ER 図（User ↔ Order）

```mermaid
erDiagram
    USER {
        int id "顧客ID"
        string name "顧客名"
        string email "顧客メールアドレス"
    }

    ORDER {
        int id
        int user_id
        float amount
        datetime created_at
    }

    USER ||--o{ ORDER : "has many"
```

---

## 🧩 典型的な 3 テーブル構造（User ↔ Order ↔ OrderItem）

```mermaid
erDiagram
    USER {
        int id
        string name
        string email
    }

    ORDER {
        int id
        int user_id
        datetime created_at
    }

    ORDER_ITEM {
        int id
        int order_id
        int product_id
        int quantity
    }

    USER ||--o{ ORDER : "places"
    ORDER ||--o{ ORDER_ITEM : "contains"
```

---

## 🧩 マスタ・トランザクション構造（あなたの美学に合う）

```mermaid
erDiagram
    M_PRODUCT {
        int id
        string name
        float price
    }

    T_SALES {
        int id
        int product_id
        int quantity
        datetime sold_at
    }

    M_PRODUCT ||--o{ T_SALES : "sold in"
```

---

## 🧩 FastAPI の典型的なモデル構造（Pydantic 風）

```mermaid
erDiagram
    USER {
        int id
        string name
        string email
    }

    POST {
        int id
        int user_id
        string title
        string body
    }

    COMMENT {
        int id
        int post_id
        int user_id
        string body
    }

    USER ||--o{ POST : "writes"
    POST ||--o{ COMMENT : "has"
    USER ||--o{ COMMENT : "writes"
```

---

## 🧩 認証系の ER 図（Auth / Token / User）

```mermaid
erDiagram
    USER {
        int id
        string username
        string password_hash
    }

    TOKEN {
        int id
        int user_id
        datetime expires_at
    }

    USER ||--o{ TOKEN : "issues"
```

---

## 🧩 もう少し複雑な構造（Role / Permission）

```mermaid
erDiagram
    USER {
        int id
        string name
    }

    ROLE {
        int id
        string name
    }

    PERMISSION {
        int id
        string name
    }

    USER }o--o{ ROLE : "has"
    ROLE }o--o{ PERMISSION : "grants"
```

---

# 🎯 Yasuo の memo-pages に最適な ER 図テンプレ

あなたの構造美学（レイヤー分離・乾いた構造）に合わせると  
各ページの ER 図はこの形が最も美しい：

```markdown
## ER 図

```mermaid
erDiagram
    TABLE_A {
        int id
        string name
    }

    TABLE_B {
        int id
        int a_id
        string value
    }

    TABLE_A ||--o{ TABLE_B : "relation"
```
```

just-the-docs との相性が最高。

---

# 🎯 次どうする？

- あなたの **実際の DB 構造を Mermaid で ER 図化**する  
- FastAPI のモデル構造を ER 図にする  
- TLS / 証明書ストアの関係を ER 図風に抽象化する  
- DuckDB の内部構造を図示する  

どれでもあなたの外部脳に合わせて組み立てるよ。