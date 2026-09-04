# 1m26_02-deploy

1Monthon26_02のtraQインスタンスをデプロイするための設定リポジトリ。
公開ホストは `i-my-traq.trap.show` に統一する。

## NeoShowcase

NeoShowcaseではDocker Composeではなく、同じリポジトリから次の4アプリを作成する。server/clientの`master`へのpushでGHCRの`:master`イメージが更新されるため、GitHub Releaseは不要。

| アプリ | Deploy Type | Build Type | Context | Dockerfile | Database |
| --- | --- | --- | --- | --- | --- |
| Backend | Runtime | Dockerfile | `.` | `Dockerfile.backend` | MariaDB |
| Frontend | Runtime | Dockerfile | `.` | `Dockerfile.frontend` | なし |
| BOT_MAI | Runtime | Dockerfile | `.` | `Dockerfile.bot-mai` | なし |
| BOT_AI | Runtime | Dockerfile | `.` | `Dockerfile.bot-ai` | なし |

EntrypointとCommandは空欄、Auto ShutdownはOFFにする。

URL設定:

| アプリ | Path | HTTP Port | Strip Prefix | h2c |
| --- | --- | --- | --- | --- |
| Backend | `/api` | `3000` | OFF | OFF |
| Backend | `/.well-known` | `3000` | OFF | OFF |
| Frontend | `/` | `80` | OFF | OFF |
| BOT_MAI | `/bot-mai` | `8080` | ON | OFF |
| BOT_AI | `/bot-ai` | `8080` | ON | OFF |

BackendでMariaDBを有効にするとNeoShowcaseが設定する `NS_MARIADB_*` 環境変数を、`Dockerfile.backend`がtraQ用の `TRAQ_MARIADB_*` に変換する。
Backendは非永続ローカルストレージを使う間、起動時に `--repair-images` でUnicode絵文字と全ユーザーのアイコンを再生成する。

BOT_MAIとBOT_AIにはNeoShowcaseで同じ値の環境変数を設定する。

| 環境変数 | 値 |
| --- | --- |
| `TRAQ_DEV_USER_NAME` | Bot管理専用の任意のユーザー名（例: `qbot_deploy`） |
| `TRAQ_DEV_USER_PASSWORD` | 10〜32文字の十分にランダムな共通パスワード |

API URL、公開URL、Bot endpoint、Auto Register、Portは各Dockerfileに設定済み。上記2値は秘密情報を含むためリポジトリへ書かず、NeoShowcaseの環境変数として設定する。

## デプロイ順序

1. serverとclientの変更をそれぞれ`master`へpushする。
2. GitHub Actionsで次のイメージが作成されたことを確認する。
   - `ghcr.io/trap-jp/1m26_02-server:master`
   - `ghcr.io/trap-jp/1m26_02-client:master`
   - `ghcr.io/trap-jp/1m26_02-bot-mai:master`
   - `ghcr.io/trap-jp/1m26_02-bot-ai:master`
3. 初回のみ、各GHCR Packageをpublicにするか、NeoShowcaseからpullできる認証を設定する。
4. NeoShowcaseでBackend、Frontend、BOT_MAI、BOT_AIを作成し、上記URLと環境変数を設定する。
5. Backendが起動してから両Botを起動する。Botは接続できるまで再試行するため、同時起動でも問題ない。

server/clientのRelease Workflowはバージョン配布用として残しているが、このNeoShowcase構成では使用しない。

## ファイルストレージ

環境変数を設定しない場合、アップロードファイルはコンテナ内のローカルストレージに入り、再デプロイ時に失われる可能性がある。本番運用ではBackendに次のS3設定を追加する。

| 環境変数 | 内容 |
| --- | --- |
| `TRAQ_STORAGE_TYPE` | `s3` |
| `TRAQ_STORAGE_S3_BUCKET` | Bucket名 |
| `TRAQ_STORAGE_S3_REGION` | Region |
| `TRAQ_STORAGE_S3_ENDPOINT` | S3互換サービスの場合のendpoint |
| `TRAQ_STORAGE_S3_ACCESSKEY` | Access Key |
| `TRAQ_STORAGE_S3_SECRETKEY` | Secret Key |

Bucketは事前に作成する。添付PNG/PDF、ユーザーアイコン、スタンプ画像を保持するため、公開前に永続ストレージを設定すること。
