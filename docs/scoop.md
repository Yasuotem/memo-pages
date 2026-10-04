# Scoop導入メモ
Scoopはパッケージマネージャーで
win環境でのコマンドラインツールを一括で管理することができる

## Scoopのインストール手順
1. PowerShellを開く
1. 実行ポリシー変更（必要なら）
    スクリプトの実行を許可するために、以下のコマンドを入力
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
1. インストールコマンドの実行
```powershell
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```
これで自動的に[Scoop公式サイト](https://scoop.sh/)から最新版がダウンロードされ、インストールが完了する

## 基本的な使い方
- アプリを検索する
```powershell
scoop search <アプリ名>
```
- アプリをインストールする(例:git)
```powershell
scoop install git
```
- インストール済みのアプリ一覧を見る
```powershell
scoop list
```
- Scoop自身やアプリを更新する
```powershell
scoop update
```

###　注意点:インストール直後の「コマンドが見つからない」
アプリをインストールした直後、同じPowerShellウインドウのままだと
パスが反映されていない状態になることがあるため、以下の方法で解決する
1. PowerShell(またはコマンドプロンプト)を一度閉じて開きなおす
1. 以下のコマンドを実行して、環境変数を現在のウインドウに即時反映させる
```powershell
refreshenv
```

### gitbashとScoopで同じアプリがある
1. どっちが動いてるか確認する
```bash
which openssl
```
- 表示が /c/Users/ユーザー名/scoop/shims/openssl なら Scoop版が優先 されています。
- 表示が /usr/bin/openssl なら Git Bash内蔵版が優先 されています。
1. GitBash内でもScoop版を使いたい場合
GitBashの設定ファイル(~/.bashrcまたは~/.bash_profile)を開き、一番下の行に
以下の記述を追加して、Scoopのパスを最優先(先頭)にするよう強制する
```bash
export PATH="$HOME/scoop/shims:$PATH"
```