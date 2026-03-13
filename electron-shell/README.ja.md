# Star Desktop Pet（Electron シェル）

このディレクトリは Electron 版のデスクトップシェルです。既存の Tauri 版と並行して動作し、段階的な移行をサポートします。

## 機能

- 既存のフロントエンドを再利用：`http://127.0.0.1:19000/?desktop=1`
- ミニページを再利用：`desktop-pet/src/minimized.html`
- 起動時に Python バックエンドを自動起動（未実行の場合）
- メインウィンドウ / ミニウィンドウの切り替え
- システムトレイ（メニューバー）常駐メニュー
- preload で `window.__TAURI__` 互換レイヤーを注入し、既存フロントエンドロジックの変更を最小化

## 起動方法

```bash
cd electron-shell
npm install
npm run dev
```

## オプション環境変数

- `STAR_PROJECT_ROOT`：プロジェクトルートディレクトリ（デフォルトで自動検出）
- `STAR_BACKEND_PYTHON`：バックエンド Python 実行パス
- `STAR_BACKEND_HOST`：バックエンドホスト（デフォルト `127.0.0.1`）
- `STAR_BACKEND_PORT`：バックエンドポート（デフォルト `19000`）

## 備考

- 現時点では「実行可能な移行スケルトン」です。まずデスクトップコンテナ層の置き換えが目標です。
- 既存の Tauri ディレクトリは影響を受けません。いつでもロールバックや並行比較が可能です。
