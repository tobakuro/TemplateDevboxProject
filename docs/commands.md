<!-- 開発でよく使うコマンドをまとめて記載 -->

# よく使うコマンド集

> このドキュメントには、開発中に **よく使うコマンド** (ビルド・テスト・デプロイ・DB操作など) をプロジェクトに合わせて記載してください。

## Zone.Identifier ファイルの削除

WindowsでダウンロードしたZIPをExplorer経由で展開・移動すると `*Zone.Identifier` という残骸ファイルが生成されます。
`devbox shell` に入るたびに自動削除されますが（`devbox.json` の `init_hook` 参照）、手動で削除したい場合は以下を実行してください。

```bash
find . -name '*Zone.Identifier' -type f -print -delete
```
