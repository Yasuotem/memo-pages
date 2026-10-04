# Quartoの導入
Quarto と Python を組み合わせることで、実行可能な Python コードブロックを含む美しいレポートやドキュメント、スライドを作成できます

## 基本的な環境構築
### Quarto CLIのインストールを以下のどちらかの手順で実行する
- [Quarto公式ダウンロードページ](https://quarto.org/docs/get-started/)から
インストーラをダウンロードして実行する

- Scoopを使いインストールする
    1. extras バケットを追加する(まだ追加していない場合)<br>
    `scoop bucket add extras`
    1. Quarto をインストールする<br>
    `scoop install qurto`

　インストール出来ているか確認<br>
　`quarto --version`

### Pythonの準備
1. 仮想環境を使用しているならactivateする
1. Quarto が Python コードを実行するために必要なパッケージをインストールする<br>
`pip install jupyter pandas matplotlib`
1. インストールが終わったら Quarto が正しく Python 環境を認識できているか確認する<br>
`quarto check jupyter`<br>
画面に`Jupyter kernel Python 3 ... found`のように表示されれば準備完了

### VS Codeの準備
1. VS Code を開き、拡張マーケットプレイスを開く
1. 「Quarto」と検索し、公式開発の拡張機能をインストールする
1. 同時に、公式の「Python」拡張機能もインストールされていることを確認する

### クイックスタート
1. VS Codeで新しいファイルを作成して、名前を`test.qmd`にして保存
1. ファイルに以下のテキストを張り付けます
```qmd
---
title: "Quarto Python テスト"
format: html
---

## はじめての Quarto

これは Quarto と Python のテストドキュメントです。

```{python}
import matplotlib.pyplot as plt

# データの作成とプロット
x = [1, 2, 3, 4]
y = [10, 20, 25, 30]

plt.plot(x, y, marker='o')
plt.title("Sample Plot")
plt.show()
```
1. ターミナルで以下を実行する<br>
`quarto preview test.qmd`
1. ブラウザが自動で立ち上がり、コードの実行結果が含まれたHTMLページが表示される