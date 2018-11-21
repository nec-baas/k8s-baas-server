# NEC モバイルバックエンド基盤 Kubernetes テンプレート

NEC モバイルバックエンド基盤: API サーバ / デベロッパコンソールサーバ （以下は BaaS サーバ）のKubernetes テンプレート。

# 注意事項

各ソフトの Docker イメージは、DockerHub より取得します。

- BaaS サーバ： necbaas/api-console-server
- MongodDB： mongo:4
- Fluntd サーバ： necbaas/baas-fluentd

本テンプレートでデプロイした MongoDB の Pod について、

- 格納されたデータは Pod を削除した時点でデータは廃棄されます。
- データの永続化が必要な場合は、Kubernetes の永続ボリューム (PV) を利用してください。

# 設定ファイルのデプロイ

以下のコマンドで設定ファイルをデプロイしてください。

    $ kubectl create -f mongo-secrets.yml -f fluentd-config.yml -f baas-server-config.yml

## 設定ファイルの設定項目

- mongo-secrets.yml

> - MongoDBのユーザ名とパスワードなどの秘密情報( base64 でエンコードした文字列を格納する必要 )
> - Secret name: necbaas-mongo-secrets
> - <span style="color:red"> Cloud Functions サーバと共用するため、投入済の場合は再投入不要</span>

- fluentd-config.yml

> - Fluentd サーバの環境変数設定
> - ConfigMap name: necbaas-fluentd-config
> - <span style="color:red"> Cloud Functions サーバと共用するため、投入済の場合は再投入不要</span>

- baas-server-config.yml

> - BaaS サーバの環境変数設定
> - ConfigMap name: necbaas-baas-server-config

各設定項目は、前述の[モバイルバックエンド基盤： BaaS サーバ Dockerfile ](https://hub.docker.com/r/necbaas/api-server/) と
[モバイルバックエンド基盤： Fluentd Dockerfile ](https://hub.docker.com/r/necbaas/baas-fluentd/)の 設定項目と同じです。

設定方法は「設定ファイル」-> 「Docker イメージ実行時の環境変数」を参照してください。

# BaaS サーバのデプロイ

下記のコマンドで BaaS サーバをデプロイしてください。

    $ kubectl create -f mongo-server.yml -f baas-server.yml

- mongo-server.yml

> - MongoDB デプロイメントとサービス定義

- baas-server.yml

> - BaaS サーバのデプロイメントとサービス定義

# エントリポイントの確認

`node_ip` の確認

    $ kubectl get node -o jsonpath='{..status.addresses[?(@.type=="InternalIP")].address}{"\n"}'

`node_port` の確認

    $ kubectl get svc necbaas-api-console-server -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'

# 動作確認方法

エンドポイントの確認で取得した `node_ip` と `node_port`を利用し、サーバの動作を確認してください。

## デベロッパーコンソール

ブラウザからデベロッパーコンソール(`http://[node_ip]:[node_port]/console`) にアクセスし、ログインしてください。

詳細は [デベロッパーコンソール利用手順](https://nec-baas.github.io/baas-manual/latest/server/ja/server/usage/devconsole.html)を参照してください。

## APIサーバ

ブラウザから(`http://[node_ip]:[node_port]/api`)にアクセスし、以下の JSON 応答が 表示されれば 
API サーバは正常稼働しています。

    {"error":"Not Found"}

# テンプレートの更新

デプロイ後、下記のコマンドでテンプレートの変更を反映してください。

     $ kubectl apply -f << YAML ファイル >>

# テンプレートの削除

下記のコマンドでテンプレートを削除してください。

     $ kubectl delete -f << YAML ファイル >>
