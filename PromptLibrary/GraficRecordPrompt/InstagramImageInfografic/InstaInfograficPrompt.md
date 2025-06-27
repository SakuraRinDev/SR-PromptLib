# Instagram用画像スライドインフォグラフィック作成プロンプト

## 概要
このプロンプトは、Instagram投稿用の正方形（600×600px）画像スライドを**任意の枚数**作成し、個別または一括でダウンロードできるインフォグラフィックを生成するためのテンプレートです。

## 基本要件

### 技術仕様
- **フォーマット**: SVGベースのHTML
- **サイズ**: 600×600px（正方形）
- **枚数**: 可変（2枚以上、上限なし）
- **出力形式**: PNG画像（ダウンロード時）
- **レスポンシブ**: モバイル対応

### デザイン要件
1. **カラースキーム**
   - 各スライドに異なるグラデーション背景
   - 統一感のある配色展開
   - スライド枚数に応じた色彩計画

2. **タイポグラフィ**
   - 日本語フォント対応
   - 階層的なテキストサイズ
   - 適切な余白とline-height

3. **レイアウト**
   - 中央揃えを基本
   - スライド間の一貫性
   - 絵文字の効果的な活用

## 実装テンプレート

### 基本構造
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>[タイトル] - Instagram用スライド</title>
    <style>
        /* 基本スタイル */
        body {
            font-family: 'Noto Sans JP', sans-serif;
            background: #f0f0f0;
            padding: 20px;
        }
        
        .slide-wrapper {
            width: 600px;
            height: 600px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border-radius: 20px;
            overflow: hidden;
            margin: 0 auto 20px;
        }
    </style>
</head>
<body>
    <h1 style="text-align: center;">Instagram用画像スライドインフォグラフィック</h1>
    
    <!-- ダウンロードボタン -->
    <div class="controls" style="text-align: center; margin-bottom: 10px;">
        <button onclick="downloadAllSlides()">📥 全スライドを保存</button>
        <button onclick="selectSlideToDownload()">📥 番号を指定して保存</button>
    </div>
    <p style="text-align: center; color: #636e72; font-size: 14px;">※ スライドは画像で保存されます</p>
    
    <div class="slide-container">
        <!-- スライドを動的に配置 -->
    </div>
</body>
</html>
```

### SVGスライドテンプレート集

#### テンプレート1: タイトルスライド
```svg
<svg width="600" height="600" viewBox="0 0 600 600" xmlns="http://www.w3.org/2000/svg" id="slide[番号]-svg">
    <defs>
        <linearGradient id="bg-gradient[番号]" x1="0%" y1="0%" x2="100%" y2="100%">
            <stop offset="0%" style="stop-color:[色1]"/>
            <stop offset="100%" style="stop-color:[色2]"/>
        </linearGradient>
    </defs>
    
    <rect width="600" height="600" fill="url(#bg-gradient[番号])"/>
    
    <!-- バッジ（オプション） -->
    <rect x="225" y="80" width="150" height="35" rx="17" fill="white" opacity="0.9"/>
    <text x="300" y="102" text-anchor="middle" fill="[色]" font-size="14">[バッジテキスト]</text>
    
    <!-- メインタイトル -->
    <text x="300" y="[Y座標]" text-anchor="middle" fill="white" font-size="36" font-weight="bold">[タイトル]</text>
    
    <!-- サブタイトル -->
    <text x="300" y="[Y座標]" text-anchor="middle" fill="white" font-size="20">[サブタイトル]</text>
</svg>
```

#### テンプレート2: 情報リストスライド
```svg
<svg width="600" height="600" viewBox="0 0 600 600" xmlns="http://www.w3.org/2000/svg" id="slide[番号]-svg">
    <!-- 背景 -->
    <rect width="600" height="600" fill="[背景色]"/>
    
    <!-- タイトル -->
    <text x="300" y="80" text-anchor="middle" fill="[色]" font-size="28" font-weight="bold">[セクションタイトル]</text>
    
    <!-- 情報カード -->
    <rect x="50" y="120" width="500" height="[高さ]" rx="15" fill="white"/>
    
    <!-- リスト項目（繰り返し） -->
    <g transform="translate(80, [Y開始位置])">
        <!-- 項目を動的に追加 -->
    </g>
</svg>
```

#### テンプレート3: ビジュアル中心スライド
```svg
<svg width="600" height="600" viewBox="0 0 600 600" xmlns="http://www.w3.org/2000/svg" id="slide[番号]-svg">
    <!-- 背景 -->
    <rect width="600" height="600" fill="[背景色]"/>
    
    <!-- 中央の大きなアイコン/数値 -->
    <text x="300" y="300" text-anchor="middle" font-size="120" fill="[色]">[メインビジュアル]</text>
    
    <!-- 説明テキスト -->
    <text x="300" y="400" text-anchor="middle" font-size="24" fill="[色]">[説明]</text>
</svg>
```

#### テンプレート4: CTAスライド
```svg
<svg width="600" height="600" viewBox="0 0 600 600" xmlns="http://www.w3.org/2000/svg" id="slide[番号]-svg">
    <!-- アクションエリア -->
    <rect x="60" y="[Y位置]" width="480" height="[高さ]" rx="20" fill="white"/>
    
    <!-- CTA見出し -->
    <text x="300" y="[Y位置]" text-anchor="middle" font-size="28" font-weight="bold">[アクション文言]</text>
    
    <!-- 詳細情報 -->
    <text x="300" y="[Y位置]" text-anchor="middle" font-size="18">[詳細]</text>
</svg>
```

### 動的スライド生成とダウンロード機能
```javascript
// スライド総数を動的に取得
function getSlideCount() {
    return document.querySelectorAll('[id^="slide"][id$="-svg"]').length;
}

// 全スライドダウンロード（枚数に依存しない）
function downloadAllSlides() {
    const slideCount = getSlideCount();
    for (let i = 1; i <= slideCount; i++) {
        setTimeout(() => downloadSlide(i), (i - 1) * 500);
    }
    
    setTimeout(() => {
        alert(`${slideCount}枚のスライドのダウンロードを開始しました。`);
    }, slideCount * 500);
}

// 番号指定ダウンロード
function selectSlideToDownload() {
    const slideCount = getSlideCount();
    const slideNum = prompt(`保存したいスライドの番号を入力してください（1〜${slideCount}）：`);
    
    if (slideNum === null) return;
    
    const num = parseInt(slideNum);
    if (isNaN(num) || num < 1 || num > slideCount) {
        alert(`1から${slideCount}の番号を入力してください。`);
        return;
    }
    
    downloadSlide(num);
}
```

## プロンプト使用例（枚数自由）

```
以下の内容でInstagram用の画像スライドインフォグラフィックを作成してください：

【全体設定】
- スライド枚数: [任意の数]枚
- テーマ: [テーマ名]
- 基本カラー: [メインカラー]

【スライド構成】
スライド1（タイトル）:
- タイプ: タイトルスライド
- メインタイトル: [タイトル]
- サブタイトル: [サブタイトル]
- 背景: グラデーション（[色1]→[色2]）

スライド2（概要）:
- タイプ: 情報リスト
- セクションタイトル: [タイトル]
- 項目:
  - [項目1]
  - [項目2]
  - [項目3]

スライド3（データ）:
- タイプ: ビジュアル中心
- メイン要素: [数値/アイコン]
- 説明: [説明文]

[必要に応じてスライドを追加...]

最終スライド（CTA）:
- タイプ: CTAスライド
- アクション: [行動喚起文]
- 詳細: [連絡先/URL等]

技術要件：
- 600×600pxの正方形
- SVGベース
- ダウンロード機能付き（全スライド/個別選択）
- レスポンシブ対応
```

## スライド枚数別の推奨構成

### 2-3枚構成（シンプル）
1. タイトル + キービジュアル
2. メイン情報
3. CTA（任意）

### 4-6枚構成（標準）
1. タイトルスライド
2. 概要/導入
3-4. 詳細情報（2-3枚）
5. まとめ
6. CTA

### 7-10枚構成（詳細）
1. タイトルスライド
2. 目次/アジェンダ
3-7. コンテンツ（5枚程度）
8. 重要ポイントまとめ
9. 次のステップ
10. CTA/連絡先

## カラーパレット生成ガイド

### 枚数に応じた色彩計画
```javascript
// スライド枚数に応じてグラデーションを生成
function generateColorScheme(slideCount) {
    const baseHue = 250; // 基準色相
    const colors = [];
    
    for (let i = 0; i < slideCount; i++) {
        const hue = (baseHue + (i * 360 / slideCount)) % 360;
        const saturation = 70 - (i * 5); // 徐々に彩度を下げる
        const lightness = 50 + (i * 3); // 徐々に明度を上げる
        
        colors.push({
            primary: `hsl(${hue}, ${saturation}%, ${lightness}%)`,
            secondary: `hsl(${hue}, ${saturation - 10}%, ${lightness + 10}%)`
        });
    }
    
    return colors;
}
```

## ベストプラクティス

### 1. スライド枚数の決め方
- **情報量**: 1スライド1メッセージが基本
- **閲覧時間**: Instagram での平均閲覧時間を考慮
- **ストーリー性**: 起承転結を意識した構成

### 2. 統一感の保ち方
- **デザインシステム**: 共通のマージン、フォントサイズ
- **カラー展開**: 基調色からの派生色使用
- **レイアウトパターン**: 3-4種類のテンプレートを使い回す

### 3. エンゲージメント向上のコツ
- **1枚目**: 興味を引く質問やデータ
- **中間**: 価値ある情報を段階的に展開
- **最終**: 明確なCTAと次のアクション

## 注意事項
1. **スライド番号**: 必ず連番で管理（id="slide1-svg", "slide2-svg"...）
2. **メモリ考慮**: 10枚を超える場合はパフォーマンスに注意
3. **アクセシビリティ**: 各スライドに適切なaria-labelを設定
4. **プレビュー**: 実際のInstagram表示サイズでの確認推奨

このプロンプトテンプレートにより、2枚から任意の枚数まで、柔軟にInstagram用スライドを作成できます。コンテンツの量や目的に応じて、最適な枚数とレイアウトを選択してください。