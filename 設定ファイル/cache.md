```
http {
	proxy_temp_path /var/lib/nginx/cache/nginx_temp;
	proxy_cache_path /var/lib/nginx/cache/nginx levels=1 keys_zone=cache:4M inactive=1d max_size=100M;
	proxy_cache cache;
	proxy_cahce_key "$scheme://$host$request_url$is_args$args";
	expires 8h;
}
```

## proxy_cache_path

`proxy_cache_path 保存先パス levels=ディレクトリの階層レベル keys_zone=キーゾーン名:サイズ inactive=有効期限;` 

proxy_cache_pathとproxy_temp_pathは同じファイルシステム上に配置するのがパフォーマンス上よい

### keys_zone

nginxでは、プロセス間でキャッシュを共有するためにキャッシュのメタデータを共有メモリに保存する。

そのためのキーゾーン名と共有メモリの容量を指定している。

必要なキーゾーン容量の見積もり

```
必要容量 = キャッシュの総容量 ÷ 1ファイルの平均サイズ × 128B
```

### levels

このハッシュ値の何文字目でディレクトリを分割するか指定。

キャッシュファイルのディレクトリを分割しておきことで、古いファイルシステムに存在するディレクトリ内のファイル数制限を回避できる。キャッシュ容量が大きく、ファイル数が増える場合はディレクトリを分割するように指定した方がいい。

例: `levels=1:2`

最後の1文字が第一階層、最後から2, 3文字目が第二階層のディレクティブ名に使用される。

- ハッシュ値: 0c41da539d4e4470469ead97c6e024c4b
- 配置されるファイルパス: var/cache/nginx/b/c4/0c41da539d4e4470469ead97c6e024c4b

## proxy_cache

`proxy_cache キーゾーン名`

## proxy_cahce_key

`proxy_cahce_key キャッシュキー文字列` 

この値をキーとしてキャッシュの同一性を確認する

## 有効期限の指定

### キーゾーン後ごとのキャッシュ有効期限を設定

`proxy_cache_path` の`inactive` 

指定した有効期限より長くリクエストされないとキャッシュファイルが削除される

### オリジンサーバのレスポンスヘッダに有効期限を指定

上の方が優先順位が高い

ブラウザのローカルキャッシュは`Chahe-Contros`と`Expirers`の値に左右される

例えば`Chace-Control`にno-cacheを指定するとブラウザは毎回サーバーに問い合わせ、更新がない場合のみキャッシュを利用するようになる。

対して、 `X-Accel-Expires` はnginxのキャッシュのみを制御。ブラウザのキャッシュ有効期限よりも長くnginxにキャッシュさせたい場合に使う。

`add_header` でレスポンスヘッダを追加できる

- X-Accel-Expires
    
    `add_header X-Accel-Expires 86400;`
    
    コンテンツキャッシュの有効期限が86400s(1d)
    
- Cache-Control
- Expirers
    
    `expired 10d;` ⇒ `Cache-Control: max-age=60`
    
    **private**
    
    個人情報とかをプロキシサーバーにキャッシュするのは良くないので、privateでキャッシュの共有を禁止する。これをつけるよキャッシュプロキシにはキャッシュせず、ユーザ個別のブラウザのみキャッシュさせることができる。
    
    ただし、expiredディレクティブでのパラメータ付与はできないので、add_headerディレクティブを使う必要がある。
    
    `Cache-Control: private, max-age=60`
    

### レスポンスヘッダで指定されなかった場合のHTTPステータスコードやHTTPメソッド有効期限を指定

**ステータスコードで指定**

`proxy_cache_valid 200 1h` 

anyにすると明示的に指定していないすべてのステータスコードが対象になる

`proxy_cache_valid any 10s` 

**HTTPメソッドで指定**

デフォルトはGET, HEAD

`proxy_cache_methods GET HEAD POST;`