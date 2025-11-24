# Image to PPTX Converter

画像を編集可能なPowerPointファイルに変換するWebアプリケーション。

## 🚀 機能

- 画像アップロード（ドラッグ&ドロップ対応）
- AI自動ベクトル化（画像 → SVG → PPTX）
  - **スタックモード対応**: PowerPointで図形を移動しても背景に穴が開かない✨
  - **セマンティックグループ化（実験的）**: Gemini AIで意味的に関連する要素を自動グループ化🤖
- リアルタイム進捗表示
- 編集可能なPowerPointファイルとしてダウンロード
- Clerk認証統合

## 📋 現在の実装状況

### ✅ 完了
- [x] LP デザイン（ヒーローセクション、機能説明）
- [x] Convex バックエンドAPI
  - [x] ファイルアップロード
  - [x] ジョブ管理
  - [x] 進捗追跡
  - [x] ダウンロード機能
- [x] フロントエンド統合
  - [x] 画像アップロード UI
  - [x] 進捗表示
  - [x] ダウンロード機能
  - [x] 認証フロー

### ✅ 完了
- [x] **ベクトル化エンジン**
  - [x] Vectorizer.AI API統合
  - [x] スタックモード（穴あき問題解決）
  - [x] Gap Filler 最適化
  - [x] ファイル名サニタイズ
- [x] **セマンティックグループ化（実験的）**
  - [x] Gemini Vision API統合
  - [x] SVG要素の自動グループ化
  - [x] PowerPoint GroupShape対応
- [x] **PPTX生成**
  - [x] Aspose.Slides統合
  - [x] SVG埋め込み
  - [x] レイアウト自動調整
- [ ] **次のステップ**
  - [ ] SVG→Image変換（セマンティックグループ化の精度向上）
  - [ ] デプロイ設定

## 🔧 セットアップ

### 1. 依存関係のインストール

```bash
npm install
```

### 2. 環境変数の設定

`.env.local` ファイルを作成：

```bash
# Convex
NEXT_PUBLIC_CONVEX_URL=your_convex_deployment_url

# Clerk認証
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_SECRET_KEY=your_clerk_secret_key

# Vectorizer.AI API（必須）
VECTORIZER_API_ID=your_vectorizer_api_id
VECTORIZER_API_SECRET=your_vectorizer_api_secret

# Gemini API（セマンティックグループ化用、オプション）
GEMINI_API_KEY=your_gemini_api_key
ENABLE_SEMANTIC_GROUPING=true  # セマンティックグループ化を有効化
GEMINI_MODEL=gemini-3-pro-preview  # Gemini 3 Pro（最新・推奨）

# デバッグファイル保存（開発用）
SAVE_DEBUG_FILES=true  # 開発時のみtrue推奨（本番環境ではfalse）

# Python ワーカー（未実装のため現時点では不要）
# PYTHON_WORKER_ENDPOINT=http://localhost:8000
```

### 3. Convex のセットアップ

```bash
# Convex開発サーバー起動
npx convex dev
```

別ターミナルで：

```bash
# Next.js開発サーバー起動
npm run dev
```

ブラウザで `http://localhost:3000` を開く。

## 📖 詳細ドキュメント

- [バックエンドAPI仕様](./docs/backend-usage.md)
- [Pythonワーカー仕様](./docs/python-worker-spec.md)
- [環境変数設定](./docs/environment-setup.md)
- [プロジェクトプラン](./docs/plan.md)

## 🤖 セマンティックグループ化（実験的機能）

### 概要

Gemini Vision APIを使用して、ベクトル化されたSVGの要素を**意味的にグループ化**します。

### 問題

通常のベクトル化では、1つの矢印が複数のパス（輪郭、影、ハイライト）に分解され、PowerPointで個別に移動する必要があります。

### 解決策

1. **Vectorizer.AI**: 画像をSVGにベクトル化（各要素にIDを付与）
2. **Gemini Vision API**: **元画像とSVGの両方**を分析し、視覚的特徴（色・質感・形状）から「この座標のこれらのIDは1つの矢印」と判定 🆕
3. **SVG再構築**: 意味的に関連する要素を`<g>`タグでグループ化
4. **Aspose.Slides**: グループ化されたSVGをPowerPointの GroupShape として埋め込み

**🆕 精度向上ポイント:**
- 元画像をbase64化してGeminiに渡すことで、SVGの輪郭だけでなく**色・質感・全体形状**まで理解
- レイアウト単位（ヘッダー／フッター）ではなく、**オブジェクト単位（矢印／アイコン／テキスト）** でグループ化
- 「青い矢印」「薄い矢印」といった視覚的な違いを認識可能に

### 効果

**Before (元画像なし):**
```xml
<!-- レイアウト単位での大雑把なグループ化 -->
<g data-label="Header Section">
  <path id="gen-el-1" .../> <!-- タイトル文字 -->
  <path id="gen-el-2" .../> <!-- ロゴアイコン -->
  <path id="gen-el-3" .../> <!-- 日付ラベル -->
</g>
```
❌ ヘッダー全体が1つのグループ → 個別のオブジェクトを移動できない

**After (元画像あり + 強化プロンプト):**
```xml
<!-- オブジェクト単位での細かいグループ化 -->
<g data-label="Text: Dashboard Title">
  <path id="gen-el-1" .../>  <!-- 'D' -->
  <path id="gen-el-2" .../>  <!-- 'a' -->
  <!-- ... -->
</g>
<g data-label="Blue Arrow Pointing Right">
  <path id="gen-el-10" .../> <!-- 矢印の輪郭 -->
  <path id="gen-el-11" .../> <!-- 矢印の影 -->
  <path id="gen-el-12" .../> <!-- 矢印のハイライト -->
</g>
<g data-label="Lock Icon">
  <path id="gen-el-20" .../>
  <path id="gen-el-21" .../>
</g>
```
✅ PowerPointで矢印／アイコン／テキストを個別に移動可能！

### 有効化方法

`.env.local` に追加：

```bash
GEMINI_API_KEY=your_gemini_api_key
ENABLE_SEMANTIC_GROUPING=true
GEMINI_MODEL=gemini-3-pro-preview  # Gemini 3 Pro（推奨）
```

### モデル選択

| モデル名 | 品質 | 推論力 | 用途 |
|---------|------|--------|------|
| **`gemini-3-pro-preview`** | ⭐⭐⭐⭐⭐ | 🧠🧠🧠🧠🧠 | **推奨**: Gemini 3、高度な推論とセマンティック分析 |
| `gemini-1.5-pro` | ⭐⭐⭐⭐ | 🧠🧠🧠🧠 | 安定版、本番環境向け |
| `gemini-2.0-flash-exp` | ⭐⭐⭐ | 🧠🧠🧠 | 高速処理優先 |
| `gemini-1.5-flash` | ⭐⭐⭐ | 🧠🧠 | コスト削減優先 |

**Gemini 3 Proの特徴:**
- ✨ **高度な推論**: Dynamic Thinkingで複雑なセマンティック分析に最適
- 📅 **最新知識**: 2025年1月までの知識カットオフ
- 🎯 **1M トークンコンテキスト**: 大規模なSVG分析に対応
- 🔗 **詳細**: [Gemini 3 Developer Guide](https://ai.google.dev/gemini-api/docs/gemini-3)

### 注意事項

- 🧪 **実験的機能**: 現在はテキストベースの分析のみ（画像入力は今後実装予定）
- ⚡ **追加処理時間**: Gemini API呼び出しにより数秒の遅延が発生
- 💰 **API使用料**: Gemini APIの使用料が発生します
- 🔄 **モデル更新**: Experimentalモデル（exp-xxxx）は予告なく変更される可能性があります

## ⚠️ 重要な注意事項

### 🎉 完全動作可能！

すべての主要機能が実装されています：

1. ✅ 画像のアップロード → **正常動作**
2. ✅ ベクトル化（Vectorizer.AI） → **正常動作**
3. ✅ スタックモード対応 → **穴あき問題解決**
4. ✅ セマンティックグループ化（オプション） → **実験的機能**
5. ✅ PPTX生成とダウンロード → **正常動作**

### 起動方法

**最速で試すには：**

```bash
# Python ワーカー起動
cd python-worker
docker-compose up --build
```

詳細は `python-worker/QUICKSTART.md` を参照してください。

## 🛠️ 技術スタック

- **フロントエンド**: Next.js 16, React 19, TailwindCSS 4, Framer Motion
- **バックエンド**: Convex (サーバーレス)
- **認証**: Clerk
- **AI処理**:
  - 画像ベクトル化: Vectorizer.AI API
  - セマンティック分析: Google Gemini Vision API
  - SVG操作: @xmldom/xmldom
- **PPTX生成**:
  - Aspose.Slides for Node.js via Java
  - SVG → PowerPoint GroupShape埋め込み

## 📝 使い方

1. サインイン
2. 画像をドラッグ&ドロップまたはファイル選択
3. 自動変換開始
   - ベクトル化（Vectorizer.AI）
   - セマンティックグループ化（有効化している場合）
   - PPTX生成
4. 完成したPPTXをダウンロード
5. PowerPointで開く
   - ✅ **スタックモード**: 図形を移動しても背景に穴が開かない
   - ✅ **セマンティックグループ化**: 意味的にまとまった図形が自動グループ化
   - 必要に応じてグループ化解除して、個別の図形として編集可能

## 🛠️ 開発者向け：デバッグファイル保存機能

開発時に詳細なログとファイルを保存する機能を用意しています。

### 有効化方法

`.env.local` に以下を追加：

```bash
SAVE_DEBUG_FILES=true
```

### 保存される内容

毎回の変換実行で `tmp/aspose-vectorize/YYYYMMDD_HHMMSS/` フォルダが作成され、以下が保存されます：

```
tmp/aspose-vectorize/20251124_063000/
├── vectorized.svg           # ベクトル化されたSVG（セマンティックグループ化適用後）
├── vectorized-output.pptx   # 生成されたPowerPointファイル
├── process.log              # 詳細な処理ログ（各ステップのタイムスタンプ付き）
├── metadata.json            # 実行環境情報（ファイルサイズ、設定、Node.jsバージョン等）
└── xml/                     # PPTXから抽出した全XMLファイル
    ├── [Content_Types].xml
    ├── _rels_.rels
    ├── ppt_presentation.xml
    ├── ppt_slides_slide1.xml
    └── ...（その他のXMLファイル）
```

### 主な用途

- **SVG構造の確認**: セマンティックグループ化が正しく適用されているか
- **PowerPoint互換性の検証**: 生成されたPPTXをPowerPointで開いて確認
- **XMLデバッグ**: Aspose.Slidesが生成するXML構造の詳細確認
- **パフォーマンス分析**: 各ステップの処理時間をログから確認
- **問題のトラブルシュート**: エラー発生時の状態を完全に保存

### ログの例

```log
[2024-11-24T06:30:15.123Z] [START] Converting example.jpg (1048576 bytes)
[2024-11-24T06:30:15.234Z] [STEP 1] Vectorizing with Vectorizer.AI...
[2024-11-24T06:30:18.456Z] [STEP 1] SVG generated (524288 characters)
[2024-11-24T06:30:18.567Z] [STEP 2] Applying semantic grouping with Gemini AI...
[2024-11-24T06:30:45.789Z] [STEP 2] Semantic grouping applied (524288 → 589824 chars)
[2024-11-24T06:30:45.890Z] [STEP 3] Embedding SVG into PPTX with Aspose.Slides...
[2024-11-24T06:30:46.123Z] [STEP 3] Slide dimensions: 9144000x6858000
[2024-11-24T06:30:46.234Z] [STEP 3] SVG placement: x=0, y=0, w=9144000, h=6858000
[2024-11-24T06:30:48.456Z] [STEP 3] Added 3 shapes to slide
[2024-11-24T06:30:50.789Z] [STEP 3] PPTX generated (2097152 bytes)
[2024-11-24T06:30:51.012Z] [saveDebugFiles] Debug files saved to: tmp/aspose-vectorize/20241124_063015
[2024-11-24T06:30:51.123Z] [END] Conversion completed successfully
```

### 注意事項

- **本番環境では必ず無効化してください**（ストレージ容量とパフォーマンスへの影響）
- ファイルは自動削除されないため、定期的に `tmp/aspose-vectorize/` を手動でクリーンアップしてください

---

## 🤝 コントリビューション

このプロジェクトは開発中です。特に以下の貢献を歓迎します：

- Python ワーカーの実装
- ベクトル化アルゴリズムの最適化
- UI/UX改善
- バグ報告

### コントリビューション手順

1. **Issue の作成**
   - バグ報告、機能提案、タスク記録用のテンプレートを用意しています
   - [Issue一覧](../../issues)から適切なテンプレートを選択してください

2. **コミットメッセージ規約**
   - このプロジェクトでは [Conventional Commits](https://www.conventionalcommits.org/ja/v1.0.0/) を採用しています
   - コミットメッセージは以下の形式で記述してください：

   ```
   <type>: <subject>
   
   [optional body]
   ```

   **主要な type:**
   - `feat:` 新機能追加
   - `fix:` バグ修正
   - `docs:` ドキュメントのみの変更
   - `style:` コードの意味に影響しない変更（フォーマット、セミコロン等）
   - `refactor:` バグ修正や機能追加を伴わないコードの変更
   - `perf:` パフォーマンス改善
   - `test:` テストの追加・修正
   - `chore:` ビルドプロセスやツールの変更

   **例（日本語必須）:**
   ```bash
   # 正しい例（日本語で記述）
   git commit -m "feat: PDF→PPTX変換機能を追加"
   git commit -m "fix: 画像アップロードエラーを修正"
   git commit -m "docs: セットアップ手順を更新"
   
   # ❌ 間違い（英語は拒否されます）
   git commit -m "feat: add PDF to PPTX conversion"  # エラー
   git commit -m "fix: resolve image upload error"   # エラー
   ```
   
   **注意：** このプロジェクトでは日本語でのコミットメッセージが必須です。
   英語のみのメッセージは自動的に拒否されます。

3. **Pull Request の作成**
   - PRテンプレートに従って変更内容を記載してください
   - レビュー前に以下を確認：
     - [ ] Lintエラーがない (`npm run lint`)
     - [ ] 既存機能に影響がない
     - [ ] 関連Issueをリンクしている

詳細は [CONTRIBUTING.md](./CONTRIBUTING.md) を参照してください。

## 📄 ライセンス

MIT License

## 🔗 関連リンク

- [Convex](https://www.convex.dev/)
- [Clerk](https://clerk.com/)
- [python-pptx](https://python-pptx.readthedocs.io/)
- [vtracer](https://github.com/visioncortex/vtracer)

---

**現在のバージョン**: 0.1.0-alpha  
**最終更新**: 2024年11月21日
