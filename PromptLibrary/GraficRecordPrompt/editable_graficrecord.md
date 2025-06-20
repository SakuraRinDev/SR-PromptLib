# 編集可能グラフィックレコーディング風インフォグラフィック作成ガイドライン

次のプロンプトは、 非常にレベルの高いデザインを有した**日本語インフォグラフィック HTML** を自動生成するための要件定義です。
ここでは **デザイン仕様** と **技術仕様** のみを列挙します。

---

## 🖌️ デザイン仕様

### 1. カラースキーム（Sakura-Blue パレット）
| 名称 | HEX | 主要用途 |
|------|-----|---------|
| Primary | #1E3A8A | タイトル・重要背景
| Secondary | #3B82F6 | アクセント・リンク
| Accent | #60A5FA | ハイライト・装飾
| Neon-Blue | #00D4FF | ホバー・グローエフェクト
| BG-Dark | #FFFFFF | 基本背景
| BG-Light | #F5F5F5 | セクション背景

*背景はホワイト( BG-Dark )、本文は #000000、補助テキストは #333333 を推奨。上記 6 色を CSS 変数で定義し、グラデーション ( Primary→Secondary ) を多用してブランド感を統一する。*

### 2. タイポグラフィ
- Web フォント: `Shippori Mincho` (日本語見出し) / `Bebas Neue` (英数字・強調数字) / `A-OTF-UDShinGoPr6N-Bold` (本文)
- タイトル (h1): **clamp(4rem,10vw,8rem) / 700 / グラデーション( #0048A8→#0070FF ) / -webkit-text-fill-color:transparent**
- サブタイトル (h2): clamp(2.5rem,6vw,4rem) / 700 / #002F6F / 余白下 2rem
- 見出し (h3): clamp(1.8rem,4vw,2.5rem) / 700 / #0048A8
- 本文: 16px / 500 / #333333 / 行間 1.8
- 英数字やカウンターには `Bebas Neue` を使用して視覚的リズムを演出

### 3. レイアウト
1. ヘッダー: 左にタイトル、右に日付 & 出典
2. 2 カラム構成: 左 50% / 右 50%
3. カード: 白背景 + 角丸 12px + 微細シャドウ（0 2px 6px rgba(0,0,0,0.08)）
4. セクション間余白: **64px** を基本、要素間 **24px**
5. グラスモーフィズム: 重要データを強調する際に適用 （blur 10px / 背景透過 0.1）
6. 16:9 比率で視線誘導を考慮（Z レイヤー×余白×タイポ階層）
7. ワイヤーフレームアート: 3D シティスケープを `<canvas id="wireframe-bg">` に描画し、
   - ラインカラーを rgba(0,0,0,0.35) / 縁取りグローに Neon-Blue を利用
   - `pointer-events:none;` で UI 操作を阻害しない
   - マウス移動に応じたパララックス回転 (`rotationSpeed:0.0003`) と波動 (`waveAmplitude:80`)
   - ビルディング生成はグリッド間隔 80px、テーパー係数 0.4-1.0、スパイア率 25%

### 4. 最終仕上げ
- プロフェッショナルフッター: ロゴ配置 + 出典
- 微細なアニメーションや絵文字で説得力を向上（過度な動きは避ける）

---

## ⚙️ 技術仕様

1. **HTML/CSS/JavaScript 単一ファイル** で提供（外部依存は Google Fonts のみ）。
2. 画面比率は 16:9。`<meta name="viewport" content="width=device-width, initial-scale=1">` を設定。
3. **編集モードボタン（必須）**
   - `id="toggleEdit"` ボタンを固定表示。
   - クリックで `document.body.contentEditable` & `document.designMode` を ON/OFF。
   - ON 時のボタンテキスト: `終了(保存: ファイル → 名前を付けてページを保存)`。

<!-- ❶ 編集モードボタン（参考コード） -->
<pre><code>&lt;button id="toggleEdit" style="position:fixed;top:15px;right:15px;z-index:9999;"&gt;編集モード&lt;/button&gt;
&lt;script&gt;
  const btn = document.getElementById('toggleEdit');
  let editing = false;
  btn.onclick = () =&gt; {
    editing = !editing;
    document.body.contentEditable = editing;          // ページ全体を editable に
    document.designMode     = editing ? 'on' : 'off'; // 矢印キーでカーソル移動可
    btn.textContent         = editing ? '終了(保存:右クリック-&gt;名前を付けて保存)' : '編集モード';
  };
&lt;/script&gt;
</code></pre>

4. 背景装飾（任意）
   - キャンバスもしくは SVG によるワイヤーフレーム／波形など。
   - `pointer-events:none;` を設定し右クリック時に画像保存メニューが出ないようにする。
   - `id="wireframe-bg"` キャンバスを推奨。3D シティや波形など **ラインアート** を実装し、FPS 60 を維持するよう最適化する。
5. CSS 変数 (`:root {--primary-color: …}`) でテーマカラーを一元管理。
6. 各セクションを `<section>` で区切り、Intersection Observer でフェードイン。

---

