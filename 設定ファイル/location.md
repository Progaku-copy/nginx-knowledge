`location URI　{...}` の、URIに基づいたリクエストに対する設定を行う

下記の例なら、 `/images` へのアクセスは `/var/www/html` ではなく、 `/var/www/img` を見にいく。

```
server {
	server_name www.example.com;
	
	root /var/www/html;
	
	location /images/ {
		root /var/www/img;
	}
}
```

プレフィックスはこんな感じで指定

```
location = /images/ {}
```

| プレフィックス | 説明 |
| --- | --- |
| なし | 前方一致 |
| ^~ | 前方一致
一致したら、正規表現の条件を評価しない |
| = | 完全一致
パスが等しい場合 |
| ~ | 正規表現（大文字・小文字を区別する） |
| ~* | 正規表現（大文字・小文字を区別しない） |