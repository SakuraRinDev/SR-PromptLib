# **Sakura Rin デザインガイドライン 設定プロンプト**

## 基本コンセプト

**Sakura Rin**のインフォグラフィックデザインシステムは、現代的で親しみやすく、かつプロフェッショナルな印象を与えるデザインガイドラインです。日本の美意識と現代的なUIデザインを融合させた独自のアプローチを採用しています。

---

## タイポグラフィシステム

### フォントファミリー
- **メインフォント**: `Kiwi Maru` (Google Fonts)
- **特徴**: 丸みのある手書き風フォント、親しみやすさと可読性を両立

### フォントウェイト
- **500 (Medium)**: タイトル・強調テキスト用
- **400 (Regular)**: 見出し・本文テキスト用  
- **300 (Light)**: 補足・注釈テキスト用

### サイズ階層
- **H1 (メインタイトル)**: 64px (デスクトップ) → 36px (モバイル)
- **H2 (セクションタイトル)**: 36px (デスクトップ) → 24px (モバイル)
- **H3 (サブタイトル)**: 24px
- **本文**: 16px

### 文字装飾
- **太字の文字間隔**: `letter-spacing: 2px`
- **標準の文字間隔**: `letter-spacing: 1px`
- **行間**: `line-height: 1.6`

---

## カラーシステム

### テーマ1: ライトテーマ（ソーダブルー）
- **背景色**: `#E0F7FA` (ライトシアン)
- **メインカラー**: `#00ACC1` (シアン600)
- **サブカラー**: `#00BCD4` (シアン500)
- **補助カラー**: `#80DEEA` (シアン200)
- **テキスト**: `#006064` (シアン900)

### テーマ2: ダークテーマ（コバルトナイト）
- **背景色**: `#0C1929` (ダークネイビー)
- **メインカラー**: `#F0F9FF` (ホワイト)
- **サブカラー**: `#60A5FA` (ブルー400)
- **補助カラー**: `#1E3A5F` (ダークブルー)
- **テキスト**: `#F0F9FF` (ホワイト)

---

## レイアウトシステム

### グリッドシステム
```css
/* 情報グリッド */
grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));

/* サービスグリッド */
grid-template-columns: repeat(auto-fit, minmax(340px, 1fr));

/* コンポーネントグリッド */
grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
```

### スペーシング
- **基本間隔**: 20px, 35px, 60px, 80px, 100px
- **セクション間**: 40-60px
- **要素間**: 20-30px

### 最大幅
- **コンテナ**: `max-width: 1400px`
- **中央配置**: `margin: 0 auto`

---

## コンポーネントスタイル

### カードコンポーネント
- **背景**: テーマ依存の背景色
- **ボーダー**: `border-left: 5px solid [アクセントカラー]`
- **角丸**: `border-radius: 10-20px`
- **パディング**: `padding: 20-40px`

### ボタンスタイル
- **プライマリボタン**: メインカラー背景、白文字
- **セカンダリボタン**: サブカラー背景、白文字
- **アウトラインボタン**: 透明背景、カラーボーダー

### 角丸サイズ
- **小**: `10px`
- **中**: `15px`
- **大**: `20px`
- **特大**: `25px`

---

## アニメーション & インタラクション

### 基本トランジション
```css
transition: all 0.3s ease;
```

### ホバーエフェクト
- **リフト効果**: `transform: translateY(-10px)`
- **スケール効果**: `transform: scale(1.1)`
- **回転効果**: `animation: rotate 25s linear infinite`

### シャドウシステム
- **通常**: `box-shadow: 0 5px 20px rgba(68,85,102,0.05)`
- **ホバー**: `box-shadow: 0 15px 40px rgba(0,188,212,0.15)`
- **強調**: `box-shadow: 0 20px 60px rgba(68,85,102,0.3)`

---

## レスポンシブデザイン

### ブレークポイント
- **モバイル**: `320px - 480px`
- **タブレット**: `481px - 768px`
- **デスクトップ**: `769px以上`

### モバイル調整
- **フォントサイズ**: 約30%縮小
- **パディング**: 50%縮小
- **グリッド**: 1カラム固定
- **ボタン**: フルワイド

### メディアクエリ例
```css
/* ベース（モバイル）スタイル */
.component {
  padding: 15px;
  font-size: 16px;
  grid-template-columns: 1fr;
}

/* タブレット以上 */
@media (min-width: 768px) {
  .component {
    padding: 30px;
    font-size: 18px;
    grid-template-columns: repeat(2, 1fr);
  }
}

/* デスクトップ以上 */
@media (min-width: 1200px) {
  .component {
    padding: 40px;
    font-size: 20px;
    grid-template-columns: repeat(3, 1fr);
  }
}
```

---

## テーマ切り替えシステム

### 実装方法
```javascript
function switchTheme(theme) {
    document.body.className = theme;
    localStorage.setItem('selectedTheme', theme);
}

// 初期化
window.onload = function() {
    const savedTheme = localStorage.getItem('selectedTheme') || 'theme1';
    switchTheme(savedTheme);
}
```

### テーマ管理
- **CSS変数使用**: テーマごとの色分離
- **ローカルストレージ**: ユーザー設定保存
- **スムーズ切り替え**: `transition: all 0.3s ease`

---

## ベストプラクティス

### 設計原則
1. **一貫性の維持**: 全体を通じて統一されたスタイル
2. **アクセシビリティ**: コントラスト比4.5:1以上
3. **パフォーマンス**: CSS最適化、画像最適化
4. **拡張性**: 将来的な機能追加を考慮
5. **命名規則**: BEM命名規則推奨
6. **クロスブラウザ対応**: 主要ブラウザでの動作確認

### パフォーマンス最適化
- **画像**: WebP形式、遅延読み込み
- **CSS**: クリティカルCSS、不要セレクタ削除
- **JavaScript**: 非同期読み込み、バンドルサイズ削減

### アクセシビリティ
- **タッチターゲット**: 最小44px × 44px
- **キーボード操作**: フォーカス状態明確化
- **スクリーンリーダー**: セマンティックHTML使用

---

## 実装時の注意点

#### HTMLファイル1枚にまとめる事

### HTML構造
```html
<div class="container">
  <section class="section">
    <h2 class="section-title">セクションタイトル</h2>
    <div class="component-grid">
      <div class="component-example">
        <!-- コンテンツ -->
      </div>
    </div>
  </section>
</div>
```

### CSS クラス命名
- **コンテナ**: `.container`
- **セクション**: `.section`
- **グリッド**: `.[content]-grid`
- **カード**: `.[content]-card`
- **デモ**: `.[content]-demo`

### JavaScript
- **テーマ切り替え**: グローバル関数として実装
- **イベントリスナー**: 適切なクリーンアップ
- **ローカルストレージ**: エラーハンドリング

---

