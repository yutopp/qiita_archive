Windows7(x64) SP1

## 1
[https://chocolatey.org/](https://chocolatey.org/)に従って`chocolatey`をインストール．

## 2
PowerShellを管理者権限で開く．以下のコマンドを実行
```
choco install vagrant
choco install virtualbox
choco install git
```

## 3
`コントロール パネル\システムとセキュリティ\システム\システムの詳細設定`から`環境変数` を開いて，
`PATH`に`C:\Program Files (x86)\Git\bin;`を追加

## 完了
`chocolatey`は最高．
