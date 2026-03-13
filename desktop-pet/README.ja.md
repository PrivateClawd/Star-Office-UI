# Star Office Tauri デスクトップシェル

このディレクトリは `Star-Office-UI` をデスクトップアプリケーション（透明ウィンドウ）としてパッケージ化し、起動時にバックエンドプロセスを自動的に起動します。

## 開発

まず、リポジトリルートで Python 環境を準備します：

```bash
cd Star-Office-UI
uv venv .venv
uv pip install -r backend/requirements.txt --python .venv/bin/python
```

次に Tauri を起動します：

```bash
cd desktop-pet
npm install
npm run dev
```

## バックエンド自動起動ロジック

- 優先：`../.venv/bin/python backend/app.py`
- フォールバック：`python3 backend/app.py`
- 最終手段：`python backend/app.py`

ウィンドウのデフォルト URL：

- `http://127.0.0.1:19000/?desktop=1`

## オプション環境変数

- `STAR_PROJECT_ROOT`：プロジェクトルートディレクトリ（デフォルトで自動検出）
- `STAR_BACKEND_PYTHON`：カスタム Python 実行パス
- `STAR_BACKEND_URL`：デスクトップウィンドウ用のカスタム URL
