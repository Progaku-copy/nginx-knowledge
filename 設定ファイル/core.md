```
# nginxのworkerプロセス数を設定
# 2プロセスで動かす
worker_processes 2;
# エラーログ
error_log logs/error.log warn;
```

## worker_process

基本的にはCPUのコア数と同じ数にするといい

autoを指定することでCUPのコア数と同じ数のワーカープロセスが起動できる

```bash
# CPUのコア数を確認
grep processor /proc/cpuinfo | wc -l
```

## エラーログ

| パラメータ | 説明 |
| --- | --- |
| emerg | サーバが実行できないエラー |
| alert | 即時対応しなければいけないエラー |
| crit | 致命的エラー |
| error | 一般的エラー |
| warn | ワーニング |
| notice | 注意が必要な情報 |
| info | 一般情報 |
| debug | デバッグログ |

## if

`if (条件式) {…}`

否定は`!`

`!=` , `!-f`

| 演算子 | 説明 |
| --- | --- |
| = | 文字列一致 |
| - | 正規表現マッチ |
| -* | 大文字小文字を区別しない正規表現マッチ |
| -f パス | 指定パスにファイルが存在するかどうか |
| -d パス | 指定パスがディレクトリかどうか |

## set

変数定義

and, or演算子がないので、変数にマジックナンバーを代入して実現

```
# 端末がiPhoneまたはiPadであるかどうかを判定したい
location {
  set $is_apple_mobile 0;
  if ($ http_user_agent ~ "iPhone") {
    set $is_apple_mobile 1;
  }
  if ($ http_user_agent ~ "iPad") {
    set $is_apple_mobile 1;
  }
  
  if ($is_apple_mobile) {...}
}
```