# Snort

## Snort ルールヘッダ

 Snortルールヘッダとは、`/etc/snort ディレクトリ以下に配置される `*.rules` ファイルに記述する検知対象のルールを指します。

### ルールヘッダの構成

 ```
<アクション> <プロトコル> <ソースIPアドレス> <ディスティネーションIPアドレス> <方向指示子> <サーバIPアドレス>  <ポート番号>
```

#### アクション
  1. alert
    - ルールにマッチしたパケットを記録し、アラートを出力する
  1. log
    - ルールにマッチしたパケットを記録する（アラートは出力しない）
  1. pass
    - ルールにマッチしたパケットを無視する（記録もせず、アラートも出力しない）
  1. activate
    - ルールにマッチしたパケットが存在した場合、アラートを出力し対応するdynamicルールアクションを呼び出す
  1. dynamic
    - 対応するactivateルールアクションから呼び出され、logルールアクションと同様に記録のみを行う

### snort inline

 snort inlineは、iptablesがsnortルールを使用してパケットをフィルタリングするように設定します。

### Snortルールの非アクティブ化
  - local.rule にpassルールを入力し、-o オプションでSnortを再起動する。
  - ルールの前に「#」を置きルールをコメントアウトし、Snortを再起動する。

### SOモジュール
  - Snortのコンテキストで、SOルールとは読み込み可能なSnortモジュールのことです。

