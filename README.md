# TOY BOX RANKING

GitHub Pages + Supabase で動かす、イベント用ソロ・ペアランキングのたたき台です。HTML/CSS/JavaScriptのみで、ビルド作業は不要です。

## 機能

- ソロ部門・ペア部門のランキング（各上位10組）
- 6ステージ分の点数入力と合計点の自動計算
- Supabaseへ点数を保存し、別端末と共有
- 管理者ログイン、登録データの編集・削除
- 削除前の確認ダイアログ
- 20秒ごとのランキング再読み込み
- 公開ランキング専用URL（`?view=ranking`）とQRコード作成
- スマホ画面対応

## 1. index.htmlに接続情報を入れる

テキストエディタ（Windowsのメモ帳など）で `index.html` を開き、ファイル後半の次の3項目を置き換えて保存します。

```js
const SUPABASE_URL = "PASTE_PROJECT_URL_HERE";
const SUPABASE_PUBLISHABLE_KEY = "PASTE_PUBLISHABLE_KEY_HERE";
const ADMIN_USER_ID = "PASTE_ADMIN_USER_ID_HERE";
```

- `SUPABASE_URL`: SupabaseのProject URL。例 `https://abcdefgh.supabase.co`（`/rest/v1/` は付けません）
- `SUPABASE_PUBLISHABLE_KEY`: Supabaseの `sb_publishable_...` から始まるキー
- `ADMIN_USER_ID`: Supabaseの Authentication → Users にある管理者の User ID（UUID）

**`sb_secret_...` や `service_role` の秘密鍵は絶対に使わないでください。** 公開用のPublishable keyはブラウザ用ですが、データ保護は必ずSupabaseのRLSポリシーで行います。

## 2. Supabase側の確認

このアプリは、すでに作成した `public.score_entries` テーブルと以下の列を使います。

- `id`, `play_mode`, `player1_name`, `player2_name`, `created_at`
- `p1_stage1` ～ `p1_stage6`
- `p2_stage1` ～ `p2_stage6`

前に設定したRLSが残っていることを確認してください。

- `anon` は `SELECT`（ランキングの閲覧）のみ可能
- `authenticated` はログインしている管理者のみ `INSERT` / `UPDATE` / `DELETE` 可能
- 既存の汎用的な「authenticatedなら誰でも編集可能」ポリシーが残っていないこと
- 管理者のUser IDはSQLポリシーのUUIDとHTML内の `ADMIN_USER_ID` で一致していること

公開ランキングは登録データを閲覧できる仕組みです。参加者名や点数は公開されるので、住所・電話番号・学校の個人情報などを入力しないでください。

## 3. GitHubにアップロード

1. GitHubで新しいリポジトリを作成します（例 `toybox-ranking`）。
2. このフォルダーの `index.html` と `README.md` をリポジトリの一番上（root）にアップロードします。
3. GitHubリポジトリの Settings → Pages を開きます。
4. Build and deployment の Source で `Deploy from a branch` を選び、Branch は `main`、フォルダーは `/(root)` を選んで保存します。
5. 公開されるURLを開き、ランキングが読み込めることを確認します。

GitHub Pagesは公開サイトです。リポジトリを公開すると、HTML内の設定値も閲覧できます。Publishable keyとProject URLはブラウザ向けの値ですが、秘密鍵やパスワードは絶対にHTMLに書かないでください。

## 4. 管理者ログイン

サイトの「管理者メニュー」からSupabase Authで作成した管理者アカウントのメールアドレスとパスワードでログインします。ログインできても、`ADMIN_USER_ID` とRLSの許可ユーザーIDが一致しなければ登録・編集・削除できません。

## 5. 公開ランキングとQRコード

管理者メニューの「公開用QRコード」からQRコードを作成できます。QRコードのリンク先は `https://.../リポジトリ名/?view=ranking` の形式で、閲覧用のランキングだけを表示します。

## 計算ルール

- ソロ：1人目のステージ1～6の合計
- ペア：1人目と2人目のステージ1～6をすべて合計
- 合計点の高い順。合計点が同じ場合は、先に登録されたデータを先に表示します。

## トラブルシューティング

- 「permission denied」「row-level security policy」：RLSとSQLポリシーを確認します。
- ログインできない：Supabase Authentication → Users のアカウント、メール、パスワードを確認します。
- 管理者メニューが使えない：HTMLの `ADMIN_USER_ID` とAuthのUser ID、SQLで許可したUUIDが一致しているか確認します。
- ランキングが読み込めない：Project URL、Publishable key、`score_entries` テーブルと列名を確認します。
