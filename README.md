# NEC モバイルバックエンド基盤 Kubernetes テンプレート

NEC モバイルバックエンド基盤 サーバ一式を Kubernetes 上で起動するための定義ファイルです。

# 概要

以下のサーバが起動されます。 各サーバの Docker イメージは、 DockerHub より取得します。

* [MongoDB サーバ](https://hub.docker.com/_/mongo/)
* [RabbitMQ サーバ](https://hub.docker.com/_/rabbitmq/)
* [Fluentd サーバ](https://hub.docker.com/r/necbaas/baas-fluentd/)
* [BaaS API サーバ](https://hub.docker.com/r/necbaas/api-console-server/)
* [BaaS Console サーバ](https://hub.docker.com/r/necbaas/api-console-server/)
* [BaaS SSE Push サーバ](https://hub.docker.com/r/necbaas/ssepush-server/)
* [BaaS Cloud Functions サーバ](https://hub.docker.com/r/necbaas/cloudfn-server/)

# 注意事項

本定義でデプロイした MongoDB サーバの Pod について

- Pod が削除されると、 DB データが廃棄されます。
- DB データの永続化が必要な場合は、Kubernetes の永続ボリューム (PV) を利用してください。

# 使い方

## 設定ファイルのデプロイ

以下のコマンドで設定ファイルをデプロイします。

    $ kubectl create -f mongo-secrets.yml \
                     -f fluentd-config.yml \
                     -f baas-server-config.yml \
                     -f ssepush-server-config.yml \
                     -f cloudfn-server-config.yml

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

- cloudfn-server-config.yml

> - Cloudfn サーバの環境変数設定
> - ConfigMap name: cloudfn-server-config

## サーバのデプロイ

利用サーバをデプロイします。

    $ kubectl create -f mongo-server.yml \
                     -f rabbitmq-server.yml \
                     -f fluentd-server.yml

    # POD 状態の確認
    $ kubectl get pods
    
上記サーバの POD が　"running" 状態になった後、 necbaas 各サーバをデプロイします。                     

    $ kubectl create -f api-server.yml \
                     -f console-server.yml \
                     -f ssepush-server.yml \
                     -f cloudfn-server.yml

- mongo-server.yml

> - MongoDB デプロイメントとサービス定義

- rabbitmq-server.yml

> - RabbitMQ サーバの設定、デプロイメントとサービス定義
> - Secret name: necbaas-rabbitmq-secrets
> - Deployment name: necbaas-rabbitmq-deploy
> - Service name: necbaas-rabbitmq-server

- fluentd-server.yml

> - API/Console サーバが利用する Fluentd サーバのデプロイメントとサービス定義

- api-server.yml

> - API サーバのデプロイメントとサービス定義

- console-server.yml

> - Console サーバのデプロイメントとサービス定義

- ssepush-server.yml

> - SSEPush サーバのデプロイメントとサービス定義

- cloudfn-server.yml

> - Cloud Functions サーバのデプロイメント

## エントリポイントの確認

### `node_ip` の確認

    $ kubectl get node -o jsonpath='{..status.addresses[?(@.type=="InternalIP")].address}{"\n"}'

### API サーバ `api_node_port` の確認

    $ kubectl get svc necbaas-api-server -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'

### Console サーバ `console_node_port` の確認

    $ kubectl get svc necbaas-console-server -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'

### SSEPush サーバ `ssepush_node_port` の確認

    $ kubectl get svc necbaas-ssepush-server -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'
    
## 動作確認方法

### APIサーバ

ブラウザから(`http://[node_ip]:[api_node_port]/api`)にアクセスし、以下の JSON 応答が 表示されれば 
API サーバは正常稼働しています。

    {"error":"Not Found"}

### デベロッパーコンソール

ブラウザからデベロッパーコンソール(`http://[node_ip]:[console_node_port]/console`) にアクセスし、ログインしてください。

詳細は [デベロッパーコンソール利用手順](https://nec-baas.github.io/baas-manual/latest/server/ja/server/usage/devconsole.html)を参照してください。

### SEEPush サーバ

下記のコマンドでデプロイ状況を確認します。 `AVAILABLE` -> 1 なら、問題ありません。

    $ kubectl get deployment necbaas-ssepush-deploy

BaaS サーバのデベロッパーコンソールより、SSE Push サーバ 公開 URI と AMQP 設定を実施してください。
     
- SSE Push サーバ 公開 URI
     
> - http://[node_ip]:[ssepush_node_port]/ssepush/events

- AMQP 設定

> - AMQPサーバアドレス: `necbaas-rabbitmq-server:5672`
> - 認証ユーザ名: `rabbitmq`
> - 認証パスワード: `rabbitmq`

その後、[Node.js 用 SSE Push クライアントライブラリ](https://github.com/nec-baas/baas-ssepush-nodejs)を利用し、動作確認を実施してください。

### Cloud Functions サーバ

下記のコマンドでデプロイ状況を確認します。 `AVAILABLE` -> 1 なら、問題ありません。

    $ kubectl get deployment necbaas-cloudfn-deploy

BaaS サーバのデベロッパーコンソール → 「システム設定」より、 API サーバ 内部向けベース URI と AMQP 設定を実施してください。 

- API サーバ 内部向けベース URI

> - http://necbaas-api-server:8080/api

- AMQP 設定
 
> - SSEPush サーバの動作確認で設定済の場合は不要になります。

ユーザコードの動作確認について、 [Cloud Functions サーバ 利用手順書](https://nec-baas.github.io/baas-manual/latest/server/ja/cloudfunctions-server/index.html) → 動作確認を参照してください。

## テンプレートの更新

デプロイ後、下記のコマンドでテンプレートの変更を反映してください。

     $ kubectl apply -f << YAML ファイル >>

## テンプレートの削除

下記のコマンドでテンプレートを削除してください。

     $ kubectl delete -f << YAML ファイル >>
