webサーバ関連の設定

```
http {
	# nginxに常時接続しているクライアントに対するタイムアウト時間
	# 0にすると常時接続を無効化できる
	keepalive_timeout: 60s;
	# 一度オープンしたファイル情報を一時保存
	# キャッシュは最大 1000 登録でき、キャッシュに最後にアクセスがあってから 60 秒間有効
	open_file_cache max=1000 inactive=60s;
	# ファイル読み込みとレスポンス送信にsendfile()システムコールを使うかどうか
	sendfile on;
	# sendfileディレクティブが有効な場合にLinux系ではTCP_CORKオプションが利用される
	tcp_nopush on;

	# 仮想サーバ
	server {
		# IPとポート
		# デフォルト: *:80 (使用できない場合は*:8000)
		listen 80;
		# サーバ名
		server_name alpha.example.com;
		# ドキュメントルート
		root /var/www/html;
		# レスポンスヘッダのContent-type
    charset UTF-8;
    # ログフォーマット
    log_format json escape=json '{ "time":"$time_iso8601",'
                                    '"host":"$remote_addr",'
                                    '"port":"$remote_port",'
                                    '"method":"$request_method",'
                                    '"uri":"$request_uri",'
                                    '"status":"$status",'
                                    '"body_bytes":"$body_bytes_sent",'
                                    '"referer":"$http_referer",'
                                    '"ua":"$http_user_agent",'
                                    '"request_time":"$request_time",'
                                    '"response_time":"$upstream_response_time"}';
	  # アクセスログ出力先
	  access_log /var/log/nginx/access.log json;
	  # エラーログ出力先
    error_log /var/log/nginx/error.log;
	}
	
	# 仮想サーバ
	server {
		listen 127.0.0.1:80;  # アドレスも指定可能
		server_name bravo.example.com;
	}
}
```

## 仮想サーバ

実際は入り口が1つだけど、別のHTTPサーバであるかのように動作して、それぞれ独立した設定ができる。

リクエストに応じて振り分けてんだと思う。

## ドキュメントルート

nginxではrootディレクティブで指定したディレクトリのパスがそのURIのルートにマッピングされる。

例えば、 `https://example.com/images/example.png` にリクエストされた場合、見にいくファイルのパスは `/var/www/html/images/example.png` になる。

## ログフォーマット

escapeパラメータを使用すると、変数内でJSONまたはデフォルト文字のエスケープを設定できる

デフォルトはdefault

noneはエスケープを無効にする

## sendfile

sendfile()を使うとカーネル空間内でファイルの読み込みと送信が完了するため、効率良くファイルの内容をクライアントに送信できる

基本ONにし得

### sendfile()

あるファイル・ディスクリプタから別の ファイル・ディスクリプタへのデータのコピーを行う

read, writeはユーザ空間との間でデータの転送が必要となるから、read, writeを組み合わせるより効率がいい

### tcp_nopush

最も大きなパケットサイズでレスポンスヘッダとファイルの内容を送信できるので、送信パケット数が最小にできる

基本的に有効化したい