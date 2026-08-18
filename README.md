# 1m26_02-deploy

1Monthon26_02のtraQインスタンスをデプロイするための設定リポジトリ。

## NeoShowcase

NeoShowcaseではDocker Composeではなく、同じリポジトリから次の2アプリを作成する。

| アプリ | Deploy Type | Build Type | Context | Dockerfile | Database |
| --- | --- | --- | --- | --- | --- |
| Backend | Runtime | Dockerfile | `.` | `Dockerfile.backend` | MariaDB |
| Frontend | Runtime | Dockerfile | `.` | `Dockerfile.frontend` | なし |

EntrypointとCommandは空欄、Auto ShutdownはOFFにする。

URL設定:

| アプリ | Path | HTTP Port | Strip Prefix | h2c |
| --- | --- | --- | --- | --- |
| Backend | `/api` | `3000` | OFF | OFF |
| Backend | `/.well-known` | `3000` | OFF | OFF |
| Frontend | `/` | `80` | OFF | OFF |

BackendでMariaDBを有効にするとNeoShowcaseが設定する `NS_MARIADB_*` 環境変数を、`Dockerfile.backend`がtraQ用の `TRAQ_MARIADB_*` に変換する。

`config.neoshowcase.yml` のローカルストレージはコンテナ内に保存される。永続化が必要な運用ではS3等の外部ストレージに変更すること。
