# 暗号化についてのメモ
## 古い業務システムでパスワードがハードコードされているような状況の代替手段について
### 問題点
- ソースコードを見れば誰でも見れる
- ログに出ていれば誰にでも見れる
閉鎖網でも内部の人間が悪意を持てばDBにアクセスできる
### 解決策
- DB情報は「設定レイヤー」に置く
    コード(ロジック)と設定(接続情報)は分離する
- 暗号化された設定ファイルにする
    - DPAPI
    - Windows Credential Manager
    - Oracle Wallet
    - 環境変数(閉鎖網ならこれでも可)
- パスワード変更が「設定変更」で済む構造にする
### 手順
# 1. 接続文字列をコードから分離する
### VB/.NET の場合（2012 年文化に合わせた最小変更）
#### ① App.config / Web.config に接続文字列を移す
```xml
<connectionStrings>
    <add name="OracleDb"
         connectionString="User Id=USER;Password=PASS;Data Source=DB;" />
</connectionStrings>
```

#### ② コード側は「設定レイヤー」を参照するだけ
```vb
Dim conStr As String = ConfigurationManager.ConnectionStrings("OracleDb").ConnectionString
```

これで **コードと設定が分離**される。

## ✔ バッチ / VBS の場合
### ① 接続情報を外部ファイルに出す（config.ini）
```
USER=USER1
PASS=PASS1
DSN=ORCL
```

### ② バッチ側で読み込む
```bat
for /f "tokens=1,2 delims==" %%a in (config.ini) do set %%a=%%b
```

これで **ハードコードが消える**。



---



---

# 🧩 2. 設定レイヤーを暗号化する（DPAPI / Credential Manager / Oracle Wallet）

ここが本丸。  
閉域網でも **内部不正対策＋保守性向上**のために必須。

---

## 🧩 A. DPAPI（Windows の暗号化 API）  
**最も簡単で、2012 年システムでも確実に使える。**

### ✔ 暗号化（PowerShell）
```powershell
$Encrypted = ConvertTo-SecureString "PASS1" -AsPlainText -Force
$Encrypted | ConvertFrom-SecureString | Out-File "pass.enc"
```

### ✔ 復号（VB/.NET）
```vb
Dim enc As String = File.ReadAllText("pass.enc")
Dim secure As SecureString = ConvertTo-SecureString(enc)
Dim pass As String = Marshal.PtrToStringUni(Marshal.SecureStringToBSTR(secure))
```

### ✔ メリット
- Windows が暗号鍵を管理する  
- ファイルを盗まれても復号できない  
- 閉域網でも確実に動く  
- 2012 年の VB/.NET でも使える

---

## 🧩 B. Windows Credential Manager  
**パスワードを OS に預ける方式。**

### ✔ 登録（PowerShell）
```powershell
cmdkey /add:OracleDb /user:USER1 /pass:PASS1
```

### ✔ 取得（VB/.NET）
```vb
Dim cred As New Credential("OracleDb")
Dim user = cred.Username
Dim pass = cred.Password
```

### ✔ メリット
- パスワード変更が GUI でできる  
- コード側は「名前」を参照するだけ  
- 最も保守性が高い

---

## 🧩 C. Oracle Wallet  
**Oracle 公式の「接続情報の暗号化ストア」。**

### ✔ wallet 作成
```bash
mkstore -wrl ./wallet -create
mkstore -wrl ./wallet -createCredential DB USER1 PASS1
```

### ✔ sqlnet.ora
```
WALLET_LOCATION = (SOURCE = (METHOD = FILE) (METHOD_DATA = (DIRECTORY = ./wallet)))
SQLNET.WALLET_OVERRIDE = TRUE
```

### ✔ 接続文字列
```
User Id=/;Data Source=DB;
```

### ✔ メリット
- Oracle が公式にサポート  
- パスワード変更が wallet の更新だけで済む  
- DB 接続文字列からパスワードが消える

---

## 🧩 D. 環境変数（閉域網なら最小構成として可）
```bat
set ORA_USER=USER1
set ORA_PASS=PASS1
```

コード側：
```vb
Dim user = Environment.GetEnvironmentVariable("ORA_USER")
Dim pass = Environment.GetEnvironmentVariable("ORA_PASS")
```

### ✔ メリット
- 最も簡単  
- ハードコードが消える  
- 閉域網なら現実的  
- ただし暗号化は弱い

---

# 🧩 3. パスワード変更を「設定変更」で済む構造にする

ここが最重要。

### ✔ 変更手順（DPAPI の場合）
1. PowerShell で新しいパスワードを暗号化  
2. `pass.enc` を差し替える  
3. アプリ再起動

→ コード改修なし  
→ デプロイなし  
→ ログイン情報が安全に更新される

---

### ✔ 変更手順（Credential Manager の場合）
1. Windows の「資格情報マネージャー」でパスワードを変更  
2. アプリは自動で新しいパスワードを使う

→ 最も美しい  
→ 完全に設定レイヤー化

---

### ✔ 変更手順（Oracle Wallet の場合）
1. `mkstore -modifyCredential` で更新  
2. wallet を差し替える

→ Oracle 公式方式  
→ DB 接続文字列は一切変更不要

---

# 🎯 最終まとめ（あなたの美学に合わせた最適解）

| 方法 | 安全性 | 保守性 | 2012 システムとの相性 | コメント |
|------|--------|--------|------------------------|----------|
| DPAPI | ◎ | ○ | ◎ | 最も現実的で導入しやすい |
| Credential Manager | ◎ | ◎ | ○ | 保守性最強 |
| Oracle Wallet | ◎ | ◎ | △ | Oracle 文化に合う |
| 環境変数 | △ | ○ | ◎ | 閉域網なら最小構成 |
