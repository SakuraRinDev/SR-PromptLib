# Instagram用画像スライドインフォグラフィック作成プロンプト V2.0

## 概要
Instagram投稿用の正方形（600×600px）画像スライドを**任意の枚数**作成し、個別または一括でダウンロードできるおしゃれなインフォグラフィックを生成するためのテンプレートです。

## 基本仕様

### 技術要件
- **フォーマット**: SVGベースのHTML
- **サイズ**: 600×600px（正方形）
- **フォント**: Kaisei Decal（メイン）
- **アイコン**: Font Awesome 6（多用）
- **出力**: 高品質PNG画像

### Cosmic Eye カラーパレット
```css
--cosmic-night: #0f0f23;        /* メイン背景 */
--midnight-nebula: #16213e;     /* サブ背景 */
--astral-shadow: #1a1a2e;       /* ダーク要素 */
--stellar-aqua: #4ecdc4;        /* アクセント1 */
--celestial-coral: #ff6b6b;     /* アクセント2 */
--lunar-gold: #f9ca24;          /* ハイライト */
--galaxy-beige: #ddd0c0;        /* 中間色 */
--infinity-white: #f5f1e8;      /* テキスト背景 */
```

## 実装テンプレート

### 基本HTML構造
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>[タイトル] - Instagram用スライド</title>
    
    <!-- フォント・アイコン読み込み -->
    <link href="https://fonts.googleapis.com/css2?family=Kaisei+Decol:wght@400;500;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        body {
            font-family: 'Kaisei Decol', serif;
            background: linear-gradient(135deg, #0f0f23 0%, #16213e 50%, #1a1a2e 100%);
            padding: 20px;
        }
        
        .main-header h1 {
            color: #f5f1e8;
            background: linear-gradient(45deg, #4ecdc4, #ff6b6b, #f9ca24);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        
        .slide-wrapper {
            width: 600px;
            height: 600px;
            border-radius: 24px;
            margin: 0 auto 30px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.15);
        }
        
        .btn {
            background: linear-gradient(135deg, #4ecdc4, #ff6b6b);
            color: #f5f1e8;
            border: none;
            padding: 12px 24px;
            border-radius: 25px;
            font-family: 'Kaisei Decol', serif;
            cursor: pointer;
            margin: 0 10px;
        }
    </style>
</head>
<body>
    <div class="main-header">
        <h1><i class="fab fa-instagram"></i> Instagram用スライド</h1>
    </div>
    
    <div class="controls">
        <button class="btn" onclick="downloadAllSlides()">
            <i class="fas fa-download"></i> 全スライドを保存
        </button>
        <button class="btn" onclick="selectSlideToDownload()">
            <i class="fas fa-image"></i> 番号指定保存
        </button>
    </div>
    
    <div class="slide-container">
        <!-- スライドをここに配置 -->
    </div>
    
    <script>
        // ダウンロード機能
        async function downloadSlide(slideNumber) {
            const svgElement = document.getElementById(`slide${slideNumber}-svg`);
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
            
            canvas.width = 1200; // 2倍解像度
            canvas.height = 1200;
            ctx.scale(2, 2);
            
            const svgData = new XMLSerializer().serializeToString(svgElement);
            const svgBlob = new Blob([svgData], { type: 'image/svg+xml;charset=utf-8' });
            const svgUrl = URL.createObjectURL(svgBlob);
            
            const img = new Image();
            img.onload = function() {
                ctx.drawImage(img, 0, 0);
                canvas.toBlob(function(blob) {
                    const url = URL.createObjectURL(blob);
                    const a = document.createElement('a');
                    a.href = url;
                    a.download = `instagram-slide-${slideNumber}.png`;
                    a.click();
                    URL.revokeObjectURL(url);
                    URL.revokeObjectURL(svgUrl);
                }, 'image/png', 1.0);
            };
            img.src = svgUrl;
        }
        
        function downloadAllSlides() {
            const slideCount = document.querySelectorAll('[id^="slide"][id$="-svg"]').length;
            for (let i = 1; i <= slideCount; i++) {
                setTimeout(() => downloadSlide(i), (i - 1) * 800);
            }
        }
        
        function selectSlideToDownload() {
            const slideCount = document.querySelectorAll('[id^="slide"][id$="-svg"]').length;
            const slideNum = prompt(`保存したいスライドの番号を入力（1〜${slideCount}）：`);
            const num = parseInt(slideNum);
            if (!isNaN(num) && num >= 1 && num <= slideCount) {
                downloadSlide(num);
            }
        }
    </script>
</body>
</html>
```

## SVGスライドテンプレート

### タイトルスライド
```svg
<svg width="600" height="600" viewBox="0 0 600 600" xmlns="http://www.w3.org/2000/svg" id="slide1-svg">
    <defs>
        <linearGradient id="bg1" x1="0%" y1="0%" x2="100%" y2="100%">
            <stop offset="0%" style="stop-color:#0f0f23"/>
            <stop offset="100%" style="stop-color:#1a1a2e"/>
        </linearGradient>
    </defs>
    
    <rect width="600" height="600" fill="url(#bg1)"/>
    
    <!-- 装飾 -->
    <circle cx="100" cy="100" r="80" fill="rgba(78,205,196,0.1)"/>
    <circle cx="500" cy="500" r="60" fill="rgba(255,107,107,0.08)"/>
    
    <!-- メインタイトル -->
    <text x="300" y="250" text-anchor="middle" fill="#f5f1e8" font-size="42" 
          font-family="Kaisei Decol" font-weight="700">[タイトル]</text>
    
    <!-- サブタイトル -->
    <text x="300" y="300" text-anchor="middle" fill="#ddd0c0" font-size="18" 
          font-family="Kaisei Decol">[サブタイトル]</text>
</svg>
```

### データスライド
```svg
<svg width="600" height="600" viewBox="0 0 600 600" xmlns="http://www.w3.org/2000/svg" id="slide2-svg">
    <rect width="600" height="600" fill="#16213e"/>
    
    <!-- タイトル -->
    <text x="300" y="80" text-anchor="middle" fill="#f5f1e8" font-size="28" 
          font-family="Kaisei Decol" font-weight="700">📊 データ</text>
    
    <!-- データカード -->
    <rect x="50" y="120" width="240" height="180" rx="20" fill="#f5f1e8"/>
    <text x="170" y="180" text-anchor="middle" fill="#ff6b6b" font-size="48" 
          font-family="Kaisei Decol" font-weight="700">[数値]</text>
    <text x="170" y="210" text-anchor="middle" fill="#1a1a2e" font-size="16" 
          font-family="Kaisei Decol">[説明]</text>
</svg>
```

## プロンプト使用例

```
以下でInstagram用スライドを作成してください：

【設定】
- 枚数: 5枚
- テーマ: [テーマ名]
- カラー: Cosmic Eye パレット
- フォント: Kaisei Decol
- アイコン: Font Awesome 6多用

【構成】
スライド1: タイトル
- 「🌌 [メインタイトル]」
- サブ: [サブタイトル]
- 背景: コスミックグラデーション

スライド2-4: コンテンツ
- 各スライドの内容を簡潔に

スライド5: CTA
- 「[行動喚起文]」
- 連絡先情報

【要望】
- 宇宙的なデザイン
- グロー効果
- 高品質PNG出力
```

## Font Awesome活用例

### よく使うアイコン
```html
<!-- ビジネス -->
<i class="fas fa-rocket"></i>     <!-- 🚀 成長 -->
<i class="fas fa-chart-line"></i> <!-- 📈 分析 -->
<i class="fas fa-bullseye"></i>   <!-- 🎯 ターゲット -->

<!-- SNS -->
<i class="fab fa-instagram"></i>  <!-- Instagram -->
<i class="fab fa-twitter"></i>    <!-- Twitter -->
<i class="fab fa-youtube"></i>    <!-- YouTube -->

<!-- 宇宙テーマ -->
<i class="fas fa-satellite"></i>  <!-- 📡 衛星 -->
<i class="fas fa-globe"></i>      <!-- 🌍 地球 -->
<i class="fas fa-star"></i>       <!-- ⭐ 星 -->
```

### アイコンカラーリング
```css
.icon-primary { color: #4ecdc4; }   /* ステラアクア */
.icon-accent { color: #ff6b6b; }    /* セレスティアルコーラル */
.icon-highlight { color: #f9ca24; } /* ルナーゴールド */
```

このテンプレートで、Cosmic Eyeカラーパレットと Kaisei Decol フォント、Font Awesome アイコンを活用したプロフェッショナルなInstagram用スライドが効率的に作成できます。