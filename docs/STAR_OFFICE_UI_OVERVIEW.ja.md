# Star Office UI — 機能概要

Star Office UI は、AI エージェントの作業状態をリアルタイムで可視化するピクセルアートオフィスダッシュボードです。エージェントは状態に応じてオフィスのゾーン（休憩室、デスク、バグコーナー）間を移動します。マルチエージェントレンダリング、3 言語 UI（中/英/日）、カスタマイズ可能なアート資産、AI 生成の部屋背景（Gemini）、モバイル対応レイアウトをサポートしています。

## 表示内容

- ピクセルアートオフィス背景（トップダウン RPG スタイル、1280×720 キャンバス）
- キャラクターが状態に応じたエリアに移動 — アイドル状態のエージェントはソファに、作業中のエージェントはデスクに、エラー状態のエージェントはバグコーナーに
- 名前と吹き出しで現在のステータスとアクティビティを表示
- インタラクティブな装飾：植物、ポスター、猫、花をクリックしてアニメーションを切り替え
- モバイル対応 — スマートフォンやタブレットで素早くステータス確認が可能

## コア機能

### 1) エージェントステータスレンダリング

バックエンドがエージェントの状態を読み取り、`GET /status`（メインエージェント）と `GET /agents`（全エージェント）で提供します。フロントエンドはこれらのエンドポイントを 2〜2.5 秒ごとにポーリングし、エージェントを正しいオフィスゾーンにレンダリングします。

6 つの状態が 3 つのオフィスエリアにマッピング：

| 状態 | オフィスエリア | 説明 |
|------|---------------|------|
| `idle` | 休憩室（ソファ） | 待機中、アクティブタスクなし |
| `writing` | デスク（ワークスペース） | 文書の作成・整理 |
| `researching` | デスク（ワークスペース） | 情報検索中 |
| `executing` | デスク（ワークスペース） | タスク実行中 |
| `syncing` | デスク（ワークスペース） | 同期・デプロイ中 |
| `error` | バグコーナー | エラーが発生 |

状態正規化エイリアス：`working`/`busy`/`write` → `writing`、`run`/`running`/`execute`/`exec` → `executing`、`sync` → `syncing`、`research`/`search` → `researching`。不明な状態はデフォルトで `idle`。

作業状態は設定可能な TTL（デフォルト 300 秒）後に自動的に `idle` にリセットされます。

### 2) マルチエージェントオフィス

複数のエージェントが同じオフィスに同時に表示可能。各エージェントには以下があります：

- 独立したアバター（メインのスタースプライトまたは 6 種類のゲストアバタースタイル）
- 名前タグ
- 認証状態に応じた色のステータスドット：緑（承認済み）、黄（保留中）、赤（拒否）、灰（オフライン）
- エリア + スロットインデックスによる位置割り当てで重なりを防止

リモートエージェントは join key を使って `POST /join-agent` で参加し、15 秒ごとに `POST /agent-push` でステータス更新をプッシュします。フロントエンドは `GET /agents` エンドポイントから全エージェントをレンダリングします。

### 3) Join Key システム

オフィスに入れる人を制御します。キーは `join-keys.json`（初回起動時に `join-keys.sample.json` から自動生成）に保存されます。

| プロパティ | 型 | 説明 |
|-----------|------|------|
| `key` | string | Join key 文字列（例：`ocj_starteam01`） |
| `reusable` | boolean | エージェント退出後に再利用可能か |
| `maxConcurrent` | number | キーごとの最大同時接続数（デフォルト 3） |
| `expiresAt` | ISO 8601 / null | キーの有効期限 |

デフォルトキー：`ocj_starteam01` 〜 `ocj_starteam08`、各最大 3 同時接続。「アクティブ」の定義：`authStatus == "approved"` かつ最終プッシュ/更新が 300 秒以内。

### 4) アセット管理

右側のアセットドロワー（320px 幅、モバイルでは 92vw）でオフィスアートを完全制御：

- **アセットリスト** — 全フロントエンドアセットの表示/非表示トグル
- **位置編集** — 任意のアセットの X/Y 座標とスケールを調整
- **アップロード** — カスタムアートワークでアセットを置き換え（PNG, WEBP, GIF, SVG, AVIF, JPG）
- **スプライトシート変換** — アップロード時にアニメーション GIF/WEBP を自動変換
- **復元** — `.default` スナップショットまたは `.bak` バックアップから復元
- **パスワード保護** — ドロワーは認証が必要（デフォルトパスワード：`1234`）

### 5) AI 背景生成（Gemini）

Gemini API を使用して RPG スタイルのピクセルアート背景を生成：

- **2 つの速度モード：** Fast（1152×648、高速処理）と Quality（1280×720、フル解像度）
- **ランダムテーマ：** 8bit ダンジョンギルドルーム、Stardew Valley 風の酒場、北欧ファンタジー酒場、魔法技術ワークショップ、エルフの森の宿屋、ピクセルサイバー酒場、砂漠キャラバンの宿、雪山ロッジ
- **カスタムプロンプト：** 自由な説明文で完全カスタム背景を生成
- **非同期生成：** ワーカースレッドで背景を生成しタイムアウトを回避；フロントエンドが進捗をポーリング
- **モデルオプション：** `nanobanana-pro`（nano-banana-pro-preview）または `nanobanana-2`（gemini-2.5-flash-image）

Gemini API キーとモデルはドロワーの Gemini 設定セクションまたは `POST /config/gemini` エンドポイントで設定。

### 6) ホームお気に入り

生成またはカスタム背景を最大 30 個お気に入りとして保存：

- **現在を保存** — アクティブな背景をスナップショット
- **適用** — 保存済みのお気に入りに切り替え
- **削除** — 不要なお気に入りを削除

### 7) 昨日のメモ

`../memory/YYYY-MM-DD.md` から最新の日次メモを自動的に読み取り、機密コンテンツ（OpenID トークン、ファイルパス、IP アドレス、メールアドレス、電話番号）をサニタイズして「昨日のメモ」カードとして表示。長いコンテンツは切り詰められ、ランダムな名言が追加されます。

### 8) 3 言語 UI（i18n）

3 言語対応：中国語（`zh`）、英語（`en`）、日本語（`ja`）。

- 言語設定は `localStorage` に `uiLang` として保存
- 右下の言語ボタンで切り替え
- 全 UI 文字列、吹き出し、読み込みメッセージ、ステータステキストが即座に更新
- `I18N` オブジェクトと `t(key)` 関数で翻訳を処理

### 9) モバイルレイアウト

`max-width: 900px` または `pointer: coarse` でレスポンシブデザインが有効：

- ゲームキャンバス：100vw × 66.666vh（上部）
- コントロールパネル：100vw × 33.334vh（下部にスタック）
- タッチフレンドリー：最小ボタン高さ 44px
- アセットドロワー：92vw、バックドロップオーバーレイとボディスクロールロック
- 2 カラムゲストエージェントグリッド

### 10) アニメーション

| キー | ソース | フレームサイズ |
|------|--------|-------------|
| `star_idle` | star-idle-spritesheet | 128×128 |
| `star_working` | star-working-spritesheet-grid | 230×144 |
| `star_researching` | star-researching-spritesheet | 128×105 |
| `sofa_busy` | sofa-busy-spritesheet | 256×256 |
| `error_bug` | error-bug-spritesheet-grid | 180×180 |
| `sync_anim` | sync-animation-spritesheet-grid | 256×256 |
| `cats` | cats-spritesheet | 160×160 |
| `plants` | plants-spritesheet | 160×160 |
| `posters` | posters-spritesheet | 160×160 |
| `coffee_machine` | coffee-machine-spritesheet | 230×230 |
| `serverroom` | serverroom-spritesheet | 180×251 |
| `flowers` | flowers-spritesheet | 65×65 |

ゲストアバター：`guest_anim_1.webp` 〜 `guest_anim_6.webp` および `guest_role_1.png` 〜 `guest_role_6.png`。

### 11) デスクトップペットモード（オプション）

Electron ベースのデスクトップラッパー（`desktop-pet/`）でオフィスを透明なデスクトップペットウィンドウに変換。Python バックエンドを自動起動し、フレームレスウィンドウに UI を読み込みます。

## API エンドポイント

### コアステート

| メソッド | パス | 説明 |
|---------|------|------|
| GET | `/status` | メインエージェントの状態 |
| POST | `/set_state` | メインエージェントの状態を設定 |
| GET | `/agents` | 全エージェント（自動クリーンアップ付き） |
| GET | `/health` | ヘルスチェック |

### マルチエージェント

| メソッド | パス | 説明 |
|---------|------|------|
| POST | `/join-agent` | ゲストエージェントがオフィスに参加 |
| POST | `/agent-push` | ゲストエージェントがステータス更新をプッシュ |
| POST | `/leave-agent` | ゲストエージェントが退出 |
| POST | `/agent-approve` | 保留中のゲストを承認 |
| POST | `/agent-reject` | 保留中のゲストを拒否 |

### コンテンツ

| メソッド | パス | 説明 |
|---------|------|------|
| GET | `/yesterday-memo` | サニタイズ済み昨日のメモ |

### アセット管理

| メソッド | パス | 説明 |
|---------|------|------|
| POST | `/assets/auth` | アセットドロワーの認証 |
| GET | `/assets/auth/status` | ドロワー認証状態の確認 |
| GET | `/assets/list` | 全フロントエンドアセットの一覧 |
| GET | `/assets/positions` | アセット位置の取得 |
| POST | `/assets/positions` | アセット位置の設定 |
| GET | `/assets/defaults` | デフォルト位置の取得 |
| POST | `/assets/defaults` | デフォルト位置の設定 |
| POST | `/assets/upload` | 新しいアセットのアップロード |
| POST | `/assets/restore-default` | `.default` からアセットを復元 |
| POST | `/assets/restore-prev` | `.bak` からアセットを復元 |

### AI 背景生成

| メソッド | パス | 説明 |
|---------|------|------|
| POST | `/assets/generate-rpg-background` | 非同期背景生成の開始 |
| GET | `/assets/generate-rpg-background/poll` | 生成進捗のポーリング |
| POST | `/assets/restore-reference-background` | オリジナルリファレンス背景の復元 |
| POST | `/assets/restore-last-generated-background` | 最後の AI 生成背景の復元 |

### ホームお気に入り

| メソッド | パス | 説明 |
|---------|------|------|
| GET | `/assets/home-favorites/list` | 保存済みお気に入りの一覧（最大 30） |
| POST | `/assets/home-favorites/save-current` | 現在の背景を保存 |
| POST | `/assets/home-favorites/apply` | 保存済みお気に入りを適用 |
| POST | `/assets/home-favorites/delete` | お気に入りを削除 |

### 設定

| メソッド | パス | 説明 |
|---------|------|------|
| GET | `/config/gemini` | Gemini API 設定の取得（マスクされたキー、モデル） |
| POST | `/config/gemini` | Gemini API キーとモデルの設定 |

### ページ

| メソッド | パス | 説明 |
|---------|------|------|
| GET | `/` | メインピクセルオフィス UI |
| GET | `/join` | エージェント参加フォーム |
| GET | `/invite` | 招待手順 |
| GET | `/electron-standalone` | Electron スタンドアロン版 |

## 環境変数

| 変数 | デフォルト | 説明 |
|------|---------|------|
| `STAR_BACKEND_PORT` | `19000` | Flask サーバーポート |
| `FLASK_SECRET_KEY` / `STAR_OFFICE_SECRET` | `star-office-dev-secret` | Flask セッションシークレット（本番：≥ 24 文字） |
| `ASSET_DRAWER_PASS` | `1234` | アセットドロワーパスワード（本番：≥ 8 文字、`1234` 以外） |
| `GEMINI_API_KEY` / `GOOGLE_API_KEY` | — | 画像生成用 Gemini API キー |
| `GEMINI_MODEL` | `nanobanana-pro` | 画像生成用 Gemini モデル |
| `AUTO_ROTATE_HOME_ON_PAGE_OPEN` | `0` | ページ表示時にホーム背景をローテーション |
| `AUTO_ROTATE_MIN_INTERVAL_SECONDS` | `60` | 自動ローテーションの最小間隔 |
| `STAR_OFFICE_STATE_FILE` | `state.json` | メイン状態ファイルのパス |
| `STAR_OFFICE_ENV` / `FLASK_ENV` | — | `production` に設定してセキュリティ強化 |
| `OPENCLAW_WORKSPACE` | 自動検出 | メモ検索用 OpenClaw ワークスペースパス |

## セキュリティ

- **アセットドロワー認証：** パスワードベース、セッション保存（`asset_editor_authed`）、HttpOnly + SameSite=Lax クッキー、12 時間のセッション有効期間
- **本番モード：** `FLASK_SECRET_KEY` は ≥ 24 文字で弱いマーカーなし；`ASSET_DRAWER_PASS` は ≥ 8 文字で `1234` 以外；チェック失敗で起動停止
- **ランタイム設定：** `runtime-config.json` は `chmod 0600`
- **コンテンツサニタイズ：** 昨日のメモから OpenID トークン、ファイルパス、IP アドレス、メールアドレス、電話番号を除去

## PrivateClawd 統合

Star Office が [PrivateClawd](https://privateclawd.com) の一部としてデプロイされると、追加の統合機能が有効になります：

- **マルチテナンシー：** ステートエンドポイント（`/status`、`/agents`）は Next.js プロキシによりインターセプトされ、Flask のファイルベースの状態ではなく PostgreSQL からユーザーごとのデータを返す
- **書き込みエンドポイントのブロック：** `/set_state`、`/agent-push`、`/join-agent`、`/leave-agent` はノーオペレーションの成功レスポンスを返す（エージェント状態はボットライフサイクルで管理）
- **認証済みリバースプロキシ：** 全リクエストは `/api/star-office/proxy/` を経由し、完全なセッション認証を実施
- **URL 書き換え：** HTML 内のルート相対パスがプロキシプレフィックスに書き換えられ、iframe が正しく動作
- **ボットステータス同期：** ボットライフサイクルイベント（デプロイ、開始、停止、エラー）が `syncStarOfficeAgent()` を通じて対応する Star Office エージェント状態を自動更新
- **エージェントごとのオプトイン：** `star-office` スキルが有効なエージェントのみがオフィスに表示
- **ダッシュボード統合：** サイドバーリンク、メインダッシュボードのクイック起動カード、ツールバー付き iframe ビュー（エージェント数、更新、全画面、新しいタブ）

## アート資産ライセンス

- **コード：** MIT ライセンス
- **アート資産：** 商用利用禁止 — 商用デプロイ時はキャラクター/シーンアセットをオリジナルアートワークに置き換える必要あり
- メインキャラクタースプライトは著作権フリーの猫アートワークを使用
- ゲストキャラクターアニメーションは [LimeZu](https://limezu.itch.io/animated-mini-characters-2-platform-free) のフリーアセットを使用 — 再配布時は帰属表示を維持
