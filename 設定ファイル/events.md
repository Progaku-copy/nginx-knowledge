イベント駆動方式に関する設定

```
events {
	# ワーカーが処理するコネクション数(デフォルト512)
	worker_connections: 1024;
}
```

## worker_connection

足りなくなると `worker_connections are not enough` みたいなエラーが出る

クライアント以外のコネクション数(アップストリームサーバとのコネクションなど)も含む

```bash
# 同時にオープンできるファイル数を確認
# これを設定の参考にする
ulimit -n
```