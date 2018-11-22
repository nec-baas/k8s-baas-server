# NEC モバイルバックエンド基盤 Kubernetes テンプレート

NEC モバイルバックエンド基盤 サーバ一式を Kubernetes 上で起動するための定義ファイルです。

# 概要

以下のサーバが起動されます。 各サーバの Docker イメージは、 DockerHub より取得します。

* [MongoDB サーバ](https://hub.docker.com/_/mongo/)
* [RabbitMQ サーバ](https://hub.docker.com/_/rabbitmq/)
* [fluentd サーバ](https://hub.docker.com/r/necbaas/baas-fluentd/)
* [BaaS API サーバ](https://hub.docker.com/r/necbaas/api-console-server/)
* [BaaS Console サーバ](https://hub.docker.com/r/necbaas/api-console-server/)
* [BaaS SSE Push サーバ](https://hub.docker.com/r/necbaas/ssepush-server/)
* [BaaS Cloud Functions サーバ](https://hub.docker.com/r/necbaas/cloudfn-server/)

# 注意事項

本定義でデプロイした MongoDB サーバの Pod について、

- Pod が削除されると、 DB データが廃棄されます。
- DB データの永続化が必要な場合は、Kubernetes の永続ボリューム (PV) を利用してください。

# 使い方

## 設定ファイルのデプロイ

以下のコマンドで設定ファイルをデプロイします。

    $ kubectl create -f mongo-secrets.yml -f fluentd-config.yml -f baas-server-config.yml -f ssepush-server-config.yml

### 設定ファイルの設定項目

詳細の設定項目について、 各サーバの Docker イメージ実行時の環境変数を参照してください。

- mongo-secrets.yml

> - MongoDBのユーザ名とパスワードなどの秘密情報( base64 でエンコードした文字列を格納する必要 )
> - Secret name: necbaas-mongo-secrets

- fluentd-config.yml

> - Fluentd サーバの環境変数設定
> - ConfigMap name: necbaas-fluentd-config

- baas-server-config.yml

> - BaaS サーバの環境変数設定
> - ConfigMap name: necbaas-baas-server-config

- ssepush-server-config.yml

> - SSEPush サーバの環境変数設定
> - ConfigMap name: necbaas-ssepush-server-config


## サーバのデプロイ

下記のコマンドで BaaS サーバをデプロイしてください。

    $ kubectl create -f mongo-server.yml -f rabbitmq-server.yml -f baas-server.yml -f ssepush-server.yml

- mongo-server.yml

> - MongoDB デプロイメントとサービス定義

- rabbitmq-server.yml

> - RabbitMQ サーバの設定、デプロイメントとサービス定義
> - Secret name: necbaas-rabbitmq-secrets
> - Deployment name: necbaas-rabbitmq-deploy
> - Service name: necbaas-rabbitmq-server

- baas-server.yml

> - BaaS サーバのデプロイメントとサービス定義

- ssepush-server.yml

> - ssepush-server サーバのデプロイメントとサービス定義

## エントリポイントの確認

### `node_ip` の確認

    $ kubectl get node -o jsonpath='{..status.addresses[?(@.type=="InternalIP")].address}{"\n"}'

### BaaS サーバ `baas_node_port` の確認

    $ kubectl get svc necbaas-api-console-server -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'

### SSEPush サーバ `ssepush_node_port` の確認

    $ kubectl get svc necbaas-ssepush-server -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'
    
## 動作確認方法

### APIサーバ

ブラウザから(`http://[node_ip]:[baas_node_port]/api`)にアクセスし、以下の JSON 応答が 表示されれば 
API サーバは正常稼働しています。

    {"error":"Not Found"}

### デベロッパーコンソール

ブラウザからデベロッパーコンソール(`http://[node_ip]:[baas_node_port]/console`) にアクセスし、ログインしてください。

詳細は [デベロッパーコンソール利用手順](https://nec-baas.github.io/baas-manual/latest/server/ja/server/usage/devconsole.html)を参照してください。

### SEEPush サーバ

下記のコマンドでデプロイ状況を確認します。 `AVAILABLE` -> 1 なら、問題ありません。

    $ kubectl get deployment necbaas-ssepush-deploy

BaaS サーバのデベロッパーコンソールより、SSE Push サーバ 公開 URI を設定してください。
     
     http://[node_ip]:[ssepush_node_port]/ssepush/events

その後、[Node.js 用 SSE Push クライアントライブラリ](https://github.com/nec-baas/baas-ssepush-nodejs)を利用し、動作確認を実施してください。


## テンプレートの更新

デプロイ後、下記のコマンドでテンプレートの変更を反映してください。

     $ kubectl apply -f << YAML ファイル >>

## テンプレートの削除

下記のコマンドでテンプレートを削除してください。

     $ kubectl delete -f << YAML ファイル >>
