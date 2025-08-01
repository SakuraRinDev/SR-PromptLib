## 📋 インタラクティブ・インフォグラフィック作成システム - 完全技術仕様書 v3.0

### 🎯 システム概要指示
単一HTMLファイルで動作する、完全自己完結型のWYSIWYGインフォグラフィック作成・編集システムを作成して下さい。外部ライブラリに依存せず、モダンブラウザで動作し、編集内容を含めた完全なHTMLファイルとして保存・再編集が可能です。このシステムは、技術文書、製品紹介、教育コンテンツ、ポートフォリオなど、様々な用途のインタラクティブなインフォグラフィック作成に対応できます。イメージとして、WixやStudioのようなWeb開発ツールの操作感を目指しています。
### 🏗️ システムアーキテクチャ

#### 必須HTML構造
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>インタラクティブ・インフォグラフィック・エディター</title>
</head>
<body>
    <!-- 背景パターン -->
    <div id="bg-pattern"></div>
    
    <!-- 必須：コントロールボタン -->
    <div class="controls">
        <button class="btn" id="editBtn">編集モード</button>
        <button class="btn" id="saveBtn">HTML保存</button>
    </div>

    <!-- 必須：ファイル入力（非表示） -->
    <input type="file" id="mediaInput" accept="image/*,video/*" multiple>
    
    <!-- 必須：テンプレートモーダル -->
    <div class="template-modal" id="templateModal">
        <div class="template-content">
            <button class="close-modal" id="closeModalBtn">×</button>
            <h2 class="template-header">追加する要素を選択</h2>
            <div class="template-grid">
                <div class="template-item" data-template="text">
                    <div class="template-icon">📝</div>
                    <div class="template-name">テキスト</div>
                </div>
                <div class="template-item" data-template="heading">
                    <div class="template-icon">📰</div>
                    <div class="template-name">見出し</div>
                </div>
                <div class="template-item" data-template="media">
                    <div class="template-icon">🖼️</div>
                    <div class="template-name">画像・動画</div>
                </div>
            </div>
        </div>
    </div>

    <!-- コンテンツエリア -->
    <header class="header"><!-- ヘッダーコンテンツ --></header>
    <!-- セクションはここに動的に追加される -->
    <footer class="footer"><!-- フッターコンテンツ --></footer>

    <script>
        // JavaScriptコードをここに配置
    </script>
</body>
</html>
```

#### グローバル状態管理（必須）
```javascript
// アプリケーションの状態管理
const app = {
    isEditMode: false,          // 編集モードのON/OFF
    draggedElement: null,       // ドラッグ中の要素
    insertPosition: null,       // セクション挿入位置
    currentGrid: null,          // 現在のグリッド
    currentUploadGrid: null     // アップロード対象のグリッド
};

// DOM要素の参照（必須）
const elements = {
    editBtn: document.getElementById('editBtn'),
    saveBtn: document.getElementById('saveBtn'),
    templateModal: document.getElementById('templateModal'),
    closeModalBtn: document.getElementById('closeModalBtn'),
    mediaInput: document.getElementById('mediaInput')
};
```

### 🔧 コア機能の実装詳細

#### 1. **初期化処理（必須）**

```javascript
// DOMContentLoaded時に実行
document.addEventListener('DOMContentLoaded', () => {
    initializeApp();
});

function initializeApp() {
    setupEventListeners();    // イベントリスナー設定
    initializeSections();     // 既存セクションの初期化
    updateSectionNumbers();   // セクション番号更新
    initializeObserver();     // スクロールアニメーション
}
```

**⚠️ 実装時の注意点:**
- 必ず`DOMContentLoaded`後に初期化を実行
- 保存されたHTMLを開いた際も同じ初期化処理が走る

#### 2. **イベントリスナー設定（重要）**

```javascript
function setupEventListeners() {
    // 編集モードボタン
    elements.editBtn.addEventListener('click', toggleEditMode);
    
    // ファイル選択（必須：初期化時に設定）
    if (elements.mediaInput) {
        elements.mediaInput.addEventListener('change', handleMediaFileSelect);
    }
    
    // 保存ボタン
    elements.saveBtn.addEventListener('click', saveHTML);
    
    // モーダルを閉じる
    elements.closeModalBtn.addEventListener('click', function() {
        closeTemplateModal();
    });
    
    // テンプレート選択（必須：既存要素への設定）
    document.querySelectorAll('.template-item').forEach(item => {
        item.addEventListener('click', function(e) {
            handleTemplateSelect(e);
        });
    });
    
    // サイズセレクターを閉じる
    document.addEventListener('click', (e) => {
        if (!e.target.closest('.media-btn')) {
            document.querySelectorAll('.size-selector').forEach(selector => {
                selector.classList.remove('show');
            });
        }
    });
}
```

#### 3. **セクション初期化（最重要）**

```javascript
function initializeSections() {
    const sections = document.querySelectorAll('.section, .full-width-section');
    
    sections.forEach(section => {
        // 重複ラップ防止チェック
        if (!section.parentElement || !section.parentElement.classList.contains('section-wrapper')) {
            wrapSection(section);
        }
        
        // グリッド追加
        if (!section.querySelector('.media-grid')) {
            addMediaGrid(section);
        }
    });

    // 既存の追加ボタンをクリーンアップ（重要）
    document.querySelectorAll('.add-section-container').forEach(el => el.remove());

    // セクション間に追加ボタンを配置
    const wrappers = document.querySelectorAll('.section-wrapper');
    
    // 最初のセクションの前
    if (wrappers.length > 0) {
        const firstWrapper = wrappers[0];
        const addContainer = createAddSectionButton();
        firstWrapper.parentElement.insertBefore(addContainer, firstWrapper);
    }
    
    // 各セクションの後
    wrappers.forEach((wrapper) => {
        const addContainer = createAddSectionButton();
        wrapper.parentElement.insertBefore(addContainer, wrapper.nextSibling);
    });
}
```

**⚠️ 実装時の注意点:**
- 既存の追加ボタンを必ず削除してから再作成
- セクションの重複ラップを防ぐチェックが必須

#### 4. **メディア処理システム（Base64エンコーディング）**

```javascript
// ファイル選択処理
function handleMediaFileSelect(e) {
    const files = Array.from(e.target.files);
    if (files.length === 0) return;
    
    let targetGrid = app.currentUploadGrid || findNearestGrid();
    
    files.forEach(file => {
        // ファイルサイズチェック（推奨）
        const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
        if (file.size > MAX_FILE_SIZE) {
            alert(`${file.name}は10MBを超えています。小さいファイルを選択してください。`);
            return;
        }
        
        const reader = new FileReader();
        reader.onload = (event) => {
            // Base64データURL形式で読み込まれる
            // 形式: data:[<mediatype>][;base64],<data>
            if (targetGrid) {
                createMediaElement(event.target.result, file.type, targetGrid);
            }
        };
        
        // ファイルをData URLとして読み込む（Base64エンコード）
        reader.readAsDataURL(file);
    });
    
    // 入力をリセット
    e.target.value = '';
    app.currentUploadGrid = null;
}

// メディア要素作成
function createMediaElement(src, type, targetGrid) {
    const mediaContainer = document.createElement('div');
    mediaContainer.className = 'media-container fade-in';
    mediaContainer.draggable = true;
    
    let mediaEl;
    if (type.startsWith('image/')) {
        mediaEl = document.createElement('img');
        mediaEl.onload = () => {
            mediaContainer.classList.add('visible');
        };
    } else if (type.startsWith('video/')) {
        mediaEl = document.createElement('video');
        mediaEl.controls = true;
        mediaEl.onloadedmetadata = () => {
            mediaContainer.classList.add('visible');
        };
    }
    
    // Base64エンコードされたデータをsrcに設定
    mediaEl.src = src;
    
    // コントロールUI追加
    const controls = createMediaControls(mediaContainer, targetGrid);
    const sizeSelector = createSizeSelector(mediaContainer);
    
    mediaContainer.appendChild(mediaEl);
    mediaContainer.appendChild(controls);
    mediaContainer.appendChild(sizeSelector);
    
    // ドラッグイベント設定
    mediaContainer.addEventListener('dragstart', handleMediaDragStart);
    mediaContainer.addEventListener('dragend', handleMediaDragEnd);
    
    removePlaceholder(targetGrid);
    targetGrid.appendChild(mediaContainer);
    
    // アニメーション開始
    setTimeout(() => {
        mediaContainer.classList.add('visible');
    }, 10);
}
```

**⚠️ Base64エンコーディングの注意点:**
- ファイルサイズが約133%に増加
- 大きなファイルはページ読み込みを遅くする
- 10MB以上のファイルは非推奨

#### 5. **グリッドシステムと既存要素の再初期化**

```javascript
function addMediaGrid(section) {
    const container = section.querySelector('.container');
    if (!container) return null;
    
    let grid = container.querySelector('.media-grid');
    if (grid) {
        // 既存グリッドのイベント再設定（重要）
        grid.addEventListener('dragover', handleGridDragOver);
        grid.addEventListener('drop', handleGridDrop);
        
        // 既存プレースホルダーのイベント再設定（重要）
        const uploadBtn = grid.querySelector('.upload-btn');
        if (uploadBtn) {
            uploadBtn.onclick = () => {
                app.currentUploadGrid = grid;
                elements.mediaInput.click();
            };
        }
        return grid;
    }
    
    // 新規グリッド作成
    grid = document.createElement('div');
    grid.className = 'media-grid';
    grid.addEventListener('dragover', handleGridDragOver);
    grid.addEventListener('drop', handleGridDrop);
    container.appendChild(grid);
    return grid;
}
```

#### 6. **HTML保存システム（クリーンアップ処理付き）**

```javascript
function saveHTML() {
    // 現在の編集状態を記憶
    const wasEditMode = app.isEditMode;
    if (app.isEditMode) {
        toggleEditMode();
    }
    
    // 編集用UI要素を一時的に削除（重要）
    document.querySelectorAll('.add-section-container').forEach(el => {
        el.remove();
    });
    
    // 編集モード専用要素を非表示に
    document.querySelectorAll('.section-number').forEach(el => {
        el.style.display = 'none';
    });
    
    document.querySelectorAll('.section-controls').forEach(el => {
        el.style.display = 'none';
    });
    
    // HTML全体を保存（Base64メディアデータを含む）
    const htmlContent = document.documentElement.outerHTML;
    const blob = new Blob([htmlContent], { type: 'text/html;charset=utf-8' });
    const url = URL.createObjectURL(blob);
    
    // ダウンロード実行
    const a = document.createElement('a');
    a.href = url;
    a.download = `infographic-${new Date().toISOString().slice(0, 10)}.html`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    
    // 元の状態に復元
    if (wasEditMode) {
        toggleEditMode();
    }
}
```

#### 7. **セクション複製時のイベント再設定（重要）**

```javascript
function duplicateSection(wrapper) {
    const clone = wrapper.cloneNode(true);
    wrapper.parentElement.insertBefore(clone, wrapper.nextElementSibling);
    
    // コントロールボタンのイベント再設定（必須）
    const controls = clone.querySelector('.section-controls');
    if (controls) {
        controls.innerHTML = '';
        const upBtn = createControlButton('⬆️', '上へ移動', () => moveSection(clone, -1));
        const downBtn = createControlButton('⬇️', '下へ移動', () => moveSection(clone, 1));
        const duplicateBtn = createControlButton('📋', '複製', () => duplicateSection(clone));
        const deleteBtn = createControlButton('🗑️', '削除', () => deleteSection(clone));
        
        controls.appendChild(upBtn);
        controls.appendChild(downBtn);
        controls.appendChild(duplicateBtn);
        controls.appendChild(deleteBtn);
    }
    
    // グリッドイベントの再設定（必須）
    const grid = clone.querySelector('.media-grid');
    if (grid) {
        grid.addEventListener('dragover', handleGridDragOver);
        grid.addEventListener('drop', handleGridDrop);
        
        // プレースホルダーのイベント再設定（必須）
        const uploadBtn = grid.querySelector('.upload-btn');
        if (uploadBtn) {
            uploadBtn.onclick = () => {
                app.currentUploadGrid = grid;
                elements.mediaInput.click();
            };
        }
    }
    
    // 編集可能要素の設定
    clone.querySelectorAll('.editable').forEach(el => {
        el.contentEditable = app.isEditMode;
    });
    
    // メディア要素のイベント再設定（必須）
    clone.querySelectorAll('.media-container').forEach(media => {
        media.addEventListener('dragstart', handleMediaDragStart);
        media.addEventListener('dragend', handleMediaDragEnd);
        
        // メディアコントロールの再設定
        const mediaControls = media.querySelector('.media-controls');
        if (mediaControls && grid) {
            const newControls = createMediaControls(media, grid);
            mediaControls.parentNode.replaceChild(newControls, mediaControls);
        }
    });
    
    updateSectionNumbers();
    clone.scrollIntoView({ behavior: 'smooth', block: 'center' });
}
```

### ⚠️ 実装時の必須チェックリスト

#### 1. **HTML構造**
- [ ] ファイル入力要素（`#mediaInput`）が存在する
- [ ] モーダル要素（`#templateModal`）が存在する
- [ ] コントロールボタン（`#editBtn`, `#saveBtn`）が存在する

#### 2. **JavaScript初期化**
- [ ] `DOMContentLoaded`イベントで初期化を実行
- [ ] `elements`オブジェクトですべてのDOM要素を取得
- [ ] 既存要素のイベントリスナーを設定

#### 3. **セクション管理**
- [ ] 重複ラップを防ぐチェックを実装
- [ ] 追加ボタンの重複を防ぐため、既存を削除してから作成
- [ ] セクション番号の更新を忘れない

#### 4. **メディア処理**
- [ ] ファイルサイズチェックを実装（推奨10MB以下）
- [ ] Base64エンコーディングの処理を理解
- [ ] プレースホルダーのイベントを正しく設定

#### 5. **保存処理**
- [ ] 編集用UI要素を一時的に削除
- [ ] 保存後に編集状態を復元
- [ ] ファイル名に日付を含める

#### 6. **イベント再設定**
- [ ] 複製時のすべてのイベントを再設定
- [ ] グリッドのドラッグ&ドロップイベント
- [ ] プレースホルダーのアップロードボタンイベント
- [ ] メディアコントロールのイベント

### 🚨 よくある実装ミスと対策

1. **問題：保存したHTMLを開くとボタンが動作しない**
   - 原因：イベントリスナーが設定されていない
   - 対策：`initializeSections()`で既存要素のイベントを再設定

   よくあるエラーの例　（ポップアップを表示後、ボタンを押してもインサートできない。）
sample_research3.html:446 Uncaught TypeError: Cannot read properties of null (reading 'classList')
    at wrapSection (sample_research3.html:446:35)
    at insertNewSection (sample_research3.html:565:25)
    at handleTemplateSelect (sample_research3.html:558:25)
    at HTMLDivElement.<anonymous> (sample_research3.html:405:51)


2. **問題：セクション追加ボタンが重複する**
   - 原因：既存ボタンを削除せずに追加
   - 対策：`document.querySelectorAll('.add-section-container').forEach(el => el.remove());`

3. **問題：メディアアップロードが動作しない**
   - 原因：プレースホルダーのイベントが未設定
   - 対策：`addMediaGrid()`で既存プレースホルダーのイベントを再設定

4. **問題：複製したセクションが動作しない**
   - 原因：クローンしたDOMのイベントが失われる
   - 対策：`duplicateSection()`ですべてのイベントを再設定

5. **問題：大きなファイルでページが重い**
   - 原因：Base64エンコードによるサイズ増大
   - 対策：ファイルサイズ制限とリサイズ処理を実装

この仕様書に従って実装することで、完全に機能するインタラクティブ・インフォグラフィック作成システムを構築できます。

#### 成功した実装サンプル
```javascript
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>インタラクティブ・インフォグラフィック・エディター</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700;900&family=Roboto+Mono:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary-color: #0db7ed;
            --secondary-color: #384d54;
            --accent-color: #00d4ff;
            --bg-dark: #1a1a1a;
            --bg-light: #f8f9fa;
            --text-primary: #333333;
            --text-secondary: #666666;
            --card-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Noto Sans JP', sans-serif;
            background: var(--bg-light);
            color: var(--text-primary);
            line-height: 1.8;
            overflow-x: hidden;
            position: relative;
        }

        /* 背景パターン */
        #bg-pattern {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            opacity: 0.05;
            pointer-events: none;
            background-image: 
                repeating-linear-gradient(45deg, transparent, transparent 35px, rgba(13, 183, 237, 0.1) 35px, rgba(13, 183, 237, 0.1) 70px);
            z-index: 0;
        }

        /* コントロールボタン */
        .controls {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 1000;
            display: flex;
            gap: 10px;
        }

        .btn {
            background: var(--primary-color);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 30px;
            font-weight: 700;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(13, 183, 237, 0.3);
        }

        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(13, 183, 237, 0.4);
        }

        .btn.editing {
            background: #28a745;
        }

        #mediaInput {
            display: none;
        }

        /* メインコンテナ */
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 20px;
            position: relative;
            z-index: 1;
        }

        /* ヘッダー */
        .header {
            text-align: center;
            padding: 100px 0 80px;
            background: linear-gradient(135deg, var(--bg-dark) 0%, var(--secondary-color) 100%);
            color: white;
            position: relative;
            overflow: hidden;
        }

        .header::before {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: radial-gradient(circle, rgba(13, 183, 237, 0.1) 0%, transparent 70%);
            animation: rotate 30s linear infinite;
        }

        @keyframes rotate {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }

        .main-title {
            font-size: clamp(3rem, 8vw, 6rem);
            font-weight: 900;
            margin-bottom: 20px;
            position: relative;
            z-index: 1;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        .subtitle {
            font-size: clamp(1.2rem, 3vw, 1.8rem);
            font-weight: 400;
            opacity: 0.9;
            position: relative;
            z-index: 1;
        }

        .docker-icon {
            font-size: 120px;
            margin: 40px 0;
            filter: drop-shadow(0 10px 20px rgba(13, 183, 237, 0.5));
        }

        /* セクション */
        .section {
            padding: 80px 0;
            position: relative;
        }

        .section-title {
            font-size: clamp(2rem, 5vw, 3rem);
            font-weight: 900;
            color: var(--secondary-color);
            text-align: center;
            margin-bottom: 60px;
            position: relative;
        }

        .section-title::after {
            content: '';
            position: absolute;
            bottom: -15px;
            left: 50%;
            transform: translateX(-50%);
            width: 80px;
            height: 4px;
            background: var(--primary-color);
            border-radius: 2px;
        }

        /* カードグリッド */
        .card-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 30px;
            margin-bottom: 60px;
        }

        .card {
            background: white;
            border-radius: 16px;
            padding: 40px;
            box-shadow: var(--card-shadow);
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }

        .card::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 4px;
            background: linear-gradient(90deg, var(--primary-color), var(--accent-color));
        }

        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
        }

        .card-number {
            width: 50px;
            height: 50px;
            background: var(--primary-color);
            color: white;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 900;
            font-size: 24px;
            margin-bottom: 20px;
        }

        .card-title {
            font-size: 1.5rem;
            font-weight: 700;
            color: var(--secondary-color);
            margin-bottom: 15px;
        }

        .card-content {
            color: var(--text-secondary);
            line-height: 1.8;
        }

        /* コマンドブロック */
        .command-block {
            background: var(--bg-dark);
            color: #00ff00;
            padding: 30px;
            border-radius: 12px;
            font-family: 'Roboto Mono', monospace;
            overflow-x: auto;
            margin: 30px 0;
            box-shadow: inset 0 2px 10px rgba(0, 0, 0, 0.3);
            position: relative;
        }

        .command-block::before {
            content: 'Terminal';
            position: absolute;
            top: 10px;
            left: 15px;
            font-size: 12px;
            color: #666;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .command-line {
            margin: 10px 0;
            display: block;
        }

        .command-comment {
            color: #666;
        }

        /* フルワイドセクション */
        .full-width-section {
            background: linear-gradient(135deg, rgba(13, 183, 237, 0.05) 0%, rgba(0, 212, 255, 0.05) 100%);
            padding: 80px 0;
            margin: 60px 0;
        }

        /* ステップタイムライン */
        .timeline {
            position: relative;
            padding: 40px 0;
        }

        .timeline::before {
            content: '';
            position: absolute;
            left: 50%;
            top: 0;
            bottom: 0;
            width: 2px;
            background: var(--primary-color);
            transform: translateX(-50%);
        }

        .timeline-item {
            display: flex;
            align-items: center;
            margin-bottom: 60px;
            position: relative;
        }

        .timeline-item:nth-child(even) {
            flex-direction: row-reverse;
        }

        .timeline-content {
            flex: 1;
            padding: 30px;
            background: white;
            border-radius: 12px;
            box-shadow: var(--card-shadow);
            margin: 0 40px;
        }

        .timeline-marker {
            width: 30px;
            height: 30px;
            background: var(--primary-color);
            border-radius: 50%;
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            box-shadow: 0 0 0 6px rgba(13, 183, 237, 0.2);
        }

        /* 特徴リスト */
        .feature-list {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 30px;
            margin: 40px 0;
        }

        .feature-item {
            display: flex;
            align-items: flex-start;
            gap: 20px;
        }

        .feature-icon {
            font-size: 30px;
            color: var(--primary-color);
            flex-shrink: 0;
        }

        /* グリッドシステム */
        .media-grid {
            display: grid;
            grid-template-columns: repeat(12, 1fr);
            gap: 20px;
            margin: 30px 0;
            min-height: 100px;
            position: relative;
        }

        .media-grid.drop-zone {
            border: 2px dashed var(--primary-color);
            border-radius: 12px;
            padding: 20px;
            background: rgba(13, 183, 237, 0.05);
        }

        .grid-placeholder {
            grid-column: span 12;
            display: flex;
            align-items: center;
            justify-content: center;
            color: var(--text-secondary);
            font-size: 14px;
            padding: 40px;
            text-align: center;
            background: rgba(13, 183, 237, 0.02);
            border-radius: 8px;
            border: 1px dashed rgba(13, 183, 237, 0.2);
        }

        /* メディア要素 */
        .media-container {
            position: relative;
            border-radius: 12px;
            overflow: hidden;
            box-shadow: var(--card-shadow);
            transition: all 0.3s ease;
            cursor: move;
            grid-column: span 6;
        }

        .media-container.full-width {
            grid-column: span 12;
        }

        .media-container.third-width {
            grid-column: span 4;
        }

        .media-container.two-thirds-width {
            grid-column: span 8;
        }

        .media-container.dragging {
            opacity: 0.5;
            cursor: grabbing;
        }

        .media-container.drag-over {
            transform: scale(1.02);
            box-shadow: 0 8px 30px rgba(13, 183, 237, 0.3);
        }

        .media-container img,
        .media-container video {
            width: 100%;
            height: auto;
            display: block;
        }

        /* メディアコントロール */
        .media-controls {
            position: absolute;
            top: 10px;
            right: 10px;
            display: flex;
            gap: 8px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .media-container:hover .media-controls {
            opacity: 1;
        }

        .media-btn {
            background: rgba(0, 0, 0, 0.7);
            color: white;
            border: none;
            width: 32px;
            height: 32px;
            border-radius: 4px;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 16px;
            transition: all 0.2s;
        }

        .media-btn:hover {
            background: rgba(0, 0, 0, 0.9);
            transform: scale(1.1);
        }

        .media-btn.delete {
            background: rgba(231, 76, 60, 0.9);
        }

        .media-btn.delete:hover {
            background: rgba(231, 76, 60, 1);
        }

        /* サイズ選択ドロップダウン */
        .size-selector {
            position: absolute;
            top: 50px;
            right: 10px;
            background: white;
            border-radius: 8px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
            padding: 8px;
            display: none;
            z-index: 100;
        }

        .size-selector.show {
            display: block;
        }

        .size-option {
            padding: 8px 16px;
            cursor: pointer;
            border-radius: 4px;
            transition: background 0.2s;
        }

        .size-option:hover {
            background: var(--bg-light);
        }

        .size-option.active {
            background: var(--primary-color);
            color: white;
        }

        /* セクション編集UI */
        .section-wrapper {
            position: relative;
        }

        .section-controls {
            position: absolute;
            top: 20px;
            right: 20px;
            display: none;
            gap: 8px;
            z-index: 100;
        }

        .editing-mode .section-wrapper:hover .section-controls {
            display: flex;
        }

        /* セクション追加ボタン */
        .add-section-container {
            display: none;
            padding: 40px 0;
            text-align: center;
        }

        .editing-mode .add-section-container {
            display: block;
        }

        .add-section-btn {
            background: white;
            border: 2px dashed var(--primary-color);
            color: var(--primary-color);
            padding: 20px 40px;
            border-radius: 30px;
            font-weight: 700;
            cursor: pointer;
            transition: all 0.3s ease;
            font-size: 16px;
        }

        .add-section-btn:hover {
            background: var(--primary-color);
            color: white;
            border-style: solid;
        }

        /* セクションテンプレートモーダル */
        .template-modal {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.8);
            display: none;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }

        .template-modal.show {
            display: flex;
        }

        .template-content {
            background: white;
            border-radius: 20px;
            padding: 40px;
            max-width: 600px;
            width: 90%;
            max-height: 80vh;
            overflow-y: auto;
            position: relative;
        }

        .template-header {
            font-size: 24px;
            font-weight: 700;
            margin-bottom: 30px;
            text-align: center;
        }

        .template-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 20px;
        }

        .template-item {
            border: 2px solid #e0e0e0;
            border-radius: 12px;
            padding: 20px;
            cursor: pointer;
            transition: all 0.3s ease;
            text-align: center;
        }

        .template-item:hover {
            border-color: var(--primary-color);
            transform: translateY(-2px);
            box-shadow: 0 4px 20px rgba(13, 183, 237, 0.2);
        }

        .template-icon {
            font-size: 48px;
            margin-bottom: 10px;
        }

        .template-name {
            font-weight: 700;
            color: var(--text-primary);
        }

        .close-modal {
            position: absolute;
            top: 20px;
            right: 20px;
            background: none;
            border: none;
            font-size: 30px;
            cursor: pointer;
            color: #666;
        }

        /* セクション番号 */
        .section-number {
            position: absolute;
            top: 10px;
            left: 10px;
            background: var(--primary-color);
            color: white;
            width: 30px;
            height: 30px;
            border-radius: 50%;
            display: none;
            align-items: center;
            justify-content: center;
            font-weight: 700;
            font-size: 14px;
        }

        .editing-mode .section-number {
            display: flex;
        }

        /* フッター */
        .footer {
            background: var(--bg-dark);
            color: white;
            text-align: center;
            padding: 60px 0;
            margin-top: 100px;
        }

        .footer-logo {
            font-size: 48px;
            margin-bottom: 20px;
        }

        /* 編集可能な要素のスタイル */
        [contenteditable="true"] {
            outline: 2px dashed var(--primary-color);
            outline-offset: 4px;
            transition: outline-color 0.3s;
        }

        [contenteditable="true"]:focus {
            outline-color: var(--accent-color);
            background: rgba(13, 183, 237, 0.05);
        }

        /* レスポンシブ */
        @media (max-width: 768px) {
            .timeline::before {
                left: 30px;
            }
            
            .timeline-item {
                flex-direction: column !important;
                padding-left: 60px;
            }
            
            .timeline-marker {
                left: 30px;
            }
            
            .timeline-content {
                margin: 0;
            }
            
            .section-controls {
                top: 50px;
                right: 10px;
            }
            
            .media-grid {
                grid-template-columns: repeat(6, 1fr);
            }
            
            .media-container {
                grid-column: span 6 !important;
            }
        }

        /* アニメーション */
        .fade-in {
            opacity: 0;
            transform: translateY(30px);
            transition: all 0.8s ease;
        }

        .fade-in.visible {
            opacity: 1;
            transform: translateY(0);
        }

        /* アップロードボタンのスタイル */
        .upload-btn {
            background: var(--secondary-color);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 700;
            transition: all 0.3s;
        }

        .upload-btn:hover {
            background: var(--primary-color);
        }
    </style>
</head>
<body>
    <div id="bg-pattern"></div>
    
    <!-- コントロール -->
    <div class="controls">
        <button class="btn" id="editBtn">編集モード</button>
        <button class="btn" id="saveBtn">HTML保存</button>
    </div>

    <!-- ファイル入力（非表示） -->
    <input type="file" id="mediaInput" accept="image/*,video/*" multiple>
    
    <!-- セクションテンプレートモーダル -->
    <div class="template-modal" id="templateModal">
        <div class="template-content">
            <button class="close-modal" id="closeModalBtn">×</button>
            <h2 class="template-header">追加する要素を選択</h2>
            <div class="template-grid">
                <div class="template-item" data-template="text">
                    <div class="template-icon">📝</div>
                    <div class="template-name">テキスト</div>
                </div>
                <div class="template-item" data-template="heading">
                    <div class="template-icon">📰</div>
                    <div class="template-name">見出し</div>
                </div>
                <div class="template-item" data-template="media">
                    <div class="template-icon">🖼️</div>
                    <div class="template-name">画像・動画</div>
                </div>
            </div>
        </div>
    </div>

    <!-- ヘッダー -->
    <header class="header">
        <div class="container">
            <div class="docker-icon">✨</div>
            <h1 class="main-title editable">タイトルを入力</h1>
            <p class="subtitle editable">サブタイトルを入力</p>
        </div>
    </header>

    <!-- サンプルセクション1 -->
    <section class="section">
        <div class="container">
            <h2 class="section-title editable">見出しを入力</h2>
            <div class="card">
                <p class="card-content editable">
                    ここにテキストを入力してください。編集モードをオンにすると、このテキストを直接編集できます。
                </p>
            </div>
        </div>
    </section>

    <!-- サンプルセクション2 -->
    <section class="full-width-section">
        <div class="container">
            <h2 class="section-title editable">セクションタイトル</h2>
            <div class="card-grid">
                <div class="card fade-in">
                    <div class="card-number">1</div>
                    <h3 class="card-title editable">項目1</h3>
                    <p class="card-content editable">
                        説明文をここに入力します。
                    </p>
                </div>
                <div class="card fade-in">
                    <div class="card-number">2</div>
                    <h3 class="card-title editable">項目2</h3>
                    <p class="card-content editable">
                        説明文をここに入力します。
                    </p>
                </div>
                <div class="card fade-in">
                    <div class="card-number">3</div>
                    <h3 class="card-title editable">項目3</h3>
                    <p class="card-content editable">
                        説明文をここに入力します。
                    </p>
                </div>
            </div>
        </div>
    </section>

    <!-- フッター -->
    <footer class="footer">
        <div class="container">
            <div class="footer-logo">🌟</div>
            <h2 class="editable">フッタータイトル</h2>
            <p class="editable">フッターの説明文</p>
            <p style="margin-top: 20px; opacity: 0.7;">
                <span class="editable">© 2025 Your Project</span>
            </p>
        </div>
    </footer>

    <script>
        // アプリケーションの状態管理
        const app = {
            isEditMode: false,
            draggedElement: null,
            insertPosition: null,
            currentGrid: null,
            currentUploadGrid: null
        };

        // DOM要素の参照
        const elements = {
            editBtn: document.getElementById('editBtn'),
            saveBtn: document.getElementById('saveBtn'),
            templateModal: document.getElementById('templateModal'),
            closeModalBtn: document.getElementById('closeModalBtn'),
            mediaInput: document.getElementById('mediaInput')
        };

        // 初期化
        document.addEventListener('DOMContentLoaded', () => {
            initializeApp();
        });

        function initializeApp() {
            setupEventListeners();
            initializeSections();
            updateSectionNumbers();
            initializeObserver();
        }

        // イベントリスナーの設定
        function setupEventListeners() {
            // 編集モードボタン
            elements.editBtn.addEventListener('click', toggleEditMode);
            
            // ファイル選択
            if (elements.mediaInput) {
                elements.mediaInput.addEventListener('change', handleMediaFileSelect);
            }
            
            // 保存ボタン
            elements.saveBtn.addEventListener('click', saveHTML);
            
            // モーダルを閉じる
            elements.closeModalBtn.addEventListener('click', function() {
                closeTemplateModal();
            });
            
            // テンプレート選択
            document.querySelectorAll('.template-item').forEach(item => {
                item.addEventListener('click', function(e) {
                    handleTemplateSelect(e);
                });
            });
            
            // グローバルクリックイベント（サイズセレクターを閉じる）
            document.addEventListener('click', (e) => {
                if (!e.target.closest('.media-btn')) {
                    document.querySelectorAll('.size-selector').forEach(selector => {
                        selector.classList.remove('show');
                    });
                }
            });
        }

        // 編集モードの切り替え
        function toggleEditMode() {
            app.isEditMode = !app.isEditMode;
            document.body.classList.toggle('editing-mode', app.isEditMode);
            
            // 編集可能要素の設定
            const editables = document.querySelectorAll('.editable');
            editables.forEach(el => {
                el.contentEditable = app.isEditMode;
            });
            
            // グリッドの表示設定
            const grids = document.querySelectorAll('.media-grid');
            grids.forEach(grid => {
                if (app.isEditMode) {
                    grid.classList.add('drop-zone');
                    if (grid.children.length === 0) {
                        addPlaceholder(grid);
                    }
                } else {
                    grid.classList.remove('drop-zone');
                    removePlaceholder(grid);
                }
            });
            
            elements.editBtn.textContent = app.isEditMode ? '編集終了' : '編集モード';
            elements.editBtn.classList.toggle('editing', app.isEditMode);
        }

        // メディアファイル選択処理
        function handleMediaFileSelect(e) {
            const files = Array.from(e.target.files);
            if (files.length === 0) return;
            
            let targetGrid = app.currentUploadGrid || findNearestGrid();
            
            files.forEach(file => {
                const reader = new FileReader();
                reader.onload = (event) => {
                    if (targetGrid) {
                        createMediaElement(event.target.result, file.type, targetGrid);
                    }
                };
                reader.readAsDataURL(file);
            });
            
            e.target.value = '';
            app.currentUploadGrid = null;
        }

        // 最も近いグリッドを探す
        function findNearestGrid() {
            const grids = document.querySelectorAll('.media-grid');
            let targetGrid = null;
            let minDistance = Infinity;
            
            const viewportCenter = window.innerHeight / 2;
            
            grids.forEach(grid => {
                const rect = grid.getBoundingClientRect();
                const gridCenter = rect.top + rect.height / 2;
                const distance = Math.abs(gridCenter - viewportCenter);
                
                if (distance < minDistance) {
                    minDistance = distance;
                    targetGrid = grid;
                }
            });
            
            return targetGrid || grids[0];
        }

        // テンプレート選択処理
        function handleTemplateSelect(e) {
            const template = e.currentTarget.dataset.template;
            let newSection;
            
            switch(template) {
                case 'text':
                    newSection = createTextSection();
                    break;
                case 'heading':
                    newSection = createHeadingSection();
                    break;
                case 'media':
                    newSection = createMediaSection();
                    break;
            }
            
            if (newSection) {
                insertNewSection(newSection);
                closeTemplateModal();
            }
        }

        // モーダル表示
        function showTemplateModal() {
            elements.templateModal.classList.add('show');
        }

        // モーダルを閉じる
        function closeTemplateModal() {
            elements.templateModal.classList.remove('show');
            app.insertPosition = null;
            app.currentGrid = null;
        }

        // セクション初期化
        function initializeSections() {
            const sections = document.querySelectorAll('.section, .full-width-section');
            
            sections.forEach(section => {
                // すでにラップされているかチェック
                if (!section.parentElement || !section.parentElement.classList.contains('section-wrapper')) {
                    wrapSection(section);
                }
                
                // グリッドがない場合は追加
                if (!section.querySelector('.media-grid')) {
                    addMediaGrid(section);
                }
            });

            // 既存の追加ボタンをすべて削除
            document.querySelectorAll('.add-section-container').forEach(el => el.remove());

            // セクション間に追加ボタンを配置
            const wrappers = document.querySelectorAll('.section-wrapper');
            
            // 最初のセクションの前に追加ボタンを配置
            if (wrappers.length > 0) {
                const firstWrapper = wrappers[0];
                const addContainer = createAddSectionButton();
                firstWrapper.parentElement.insertBefore(addContainer, firstWrapper);
            }
            
            // 各セクションの後に追加ボタンを配置
            wrappers.forEach((wrapper) => {
                const addContainer = createAddSectionButton();
                wrapper.parentElement.insertBefore(addContainer, wrapper.nextSibling);
            });
        }

        // セクションをラップ
        function wrapSection(section) {
            // すでにラップされている場合はスキップ
            if (section.parentElement && section.parentElement.classList.contains('section-wrapper')) {
                return;
            }
            
            const wrapper = document.createElement('div');
            wrapper.className = 'section-wrapper';
            section.parentElement.insertBefore(wrapper, section);
            wrapper.appendChild(section);
            
            // セクション番号
            const number = document.createElement('div');
            number.className = 'section-number';
            wrapper.appendChild(number);
            
            // コントロールボタン
            const controls = document.createElement('div');
            controls.className = 'section-controls';
            
            const upBtn = createControlButton('⬆️', '上へ移動', () => moveSection(wrapper, -1));
            const downBtn = createControlButton('⬇️', '下へ移動', () => moveSection(wrapper, 1));
            const duplicateBtn = createControlButton('📋', '複製', () => duplicateSection(wrapper));
            const deleteBtn = createControlButton('🗑️', '削除', () => deleteSection(wrapper));
            
            controls.appendChild(upBtn);
            controls.appendChild(downBtn);
            controls.appendChild(duplicateBtn);
            controls.appendChild(deleteBtn);
            wrapper.appendChild(controls);
        }

        // グリッド追加
        function addMediaGrid(section) {
            const container = section.querySelector('.container');
            if (!container) return null;
            
            let grid = container.querySelector('.media-grid');
            if (grid) {
                // 既存のグリッドにイベントリスナーを追加
                grid.addEventListener('dragover', handleGridDragOver);
                grid.addEventListener('drop', handleGridDrop);
                
                // 既存のプレースホルダーのイベントも再設定
                const uploadBtn = grid.querySelector('.upload-btn');
                if (uploadBtn) {
                    uploadBtn.onclick = () => {
                        app.currentUploadGrid = grid;
                        elements.mediaInput.click();
                    };
                }
                return grid;
            }
            
            grid = document.createElement('div');
            grid.className = 'media-grid';
            
            grid.addEventListener('dragover', handleGridDragOver);
            grid.addEventListener('drop', handleGridDrop);
            
            container.appendChild(grid);
            return grid;
        }

        // セクション追加ボタン作成
        function createAddSectionButton() {
            const container = document.createElement('div');
            container.className = 'add-section-container';
            
            const btn = document.createElement('button');
            btn.className = 'add-section-btn';
            btn.innerHTML = '+ 新しい要素を追加';
            
            btn.addEventListener('click', function() {
                app.insertPosition = container;
                const prevSection = container.previousElementSibling;
                if (prevSection && prevSection.classList.contains('section-wrapper')) {
                    const section = prevSection.querySelector('.section, .full-width-section');
                    app.currentGrid = section ? section.querySelector('.media-grid') : null;
                } else {
                    app.currentGrid = null;
                }
                showTemplateModal();
            });
            
            container.appendChild(btn);
            return container;
        }

        // セクションテンプレート作成
        function createTextSection() {
            const section = document.createElement('section');
            section.className = 'section fade-in visible';
            section.innerHTML = `
                <div class="container">
                    <p class="editable" style="font-size: 1.1rem; line-height: 1.8;">
                        新しいテキストをここに入力してください。編集モードでクリックすると編集できます。
                    </p>
                </div>
            `;
            return section;
        }

        function createHeadingSection() {
            const section = document.createElement('section');
            section.className = 'section fade-in visible';
            section.innerHTML = `
                <div class="container">
                    <h2 class="section-title editable">新しい見出し</h2>
                </div>
            `;
            return section;
        }

        function createMediaSection() {
            const section = document.createElement('section');
            section.className = 'section fade-in visible';
            section.innerHTML = `
                <div class="container">
                    <div class="media-grid"></div>
                </div>
            `;
            return section;
        }

        // 新しいセクションを挿入
        function insertNewSection(newSection) {
            if (!newSection || !app.insertPosition) return;
            
            const wrapper = document.createElement('div');
            wrapper.className = 'section-wrapper';
            wrapper.appendChild(newSection);
            
            const number = document.createElement('div');
            number.className = 'section-number';
            wrapper.appendChild(number);
            
            const controls = document.createElement('div');
            controls.className = 'section-controls';
            
            const upBtn = createControlButton('⬆️', '上へ移動', () => moveSection(wrapper, -1));
            const downBtn = createControlButton('⬇️', '下へ移動', () => moveSection(wrapper, 1));
            const duplicateBtn = createControlButton('📋', '複製', () => duplicateSection(wrapper));
            const deleteBtn = createControlButton('🗑️', '削除', () => deleteSection(wrapper));
            
            controls.appendChild(upBtn);
            controls.appendChild(downBtn);
            controls.appendChild(duplicateBtn);
            controls.appendChild(deleteBtn);
            wrapper.appendChild(controls);
            
            // 挿入位置の次に新しいセクションを挿入
            app.insertPosition.parentElement.insertBefore(wrapper, app.insertPosition.nextSibling);
            
            // 新しい追加ボタンを配置
            const addContainer = createAddSectionButton();
            wrapper.parentElement.insertBefore(addContainer, wrapper.nextSibling);
            
            const grid = addMediaGrid(newSection);
            if (grid && app.isEditMode) {
                grid.classList.add('drop-zone');
                if (grid.children.length === 0) {
                    addPlaceholder(grid);
                }
            }
            
            newSection.querySelectorAll('.editable').forEach(el => {
                el.contentEditable = app.isEditMode;
            });
            
            updateSectionNumbers();
            newSection.scrollIntoView({ behavior: 'smooth', block: 'center' });
        }

        // メディア要素作成
        function createMediaElement(src, type, targetGrid) {
            const mediaContainer = document.createElement('div');
            mediaContainer.className = 'media-container fade-in';
            mediaContainer.draggable = true;
            
            let mediaEl;
            if (type.startsWith('image/')) {
                mediaEl = document.createElement('img');
                mediaEl.onload = () => {
                    mediaContainer.classList.add('visible');
                };
            } else if (type.startsWith('video/')) {
                mediaEl = document.createElement('video');
                mediaEl.controls = true;
                mediaEl.onloadedmetadata = () => {
                    mediaContainer.classList.add('visible');
                };
            }
            
            mediaEl.src = src;
            
            const controls = createMediaControls(mediaContainer, targetGrid);
            const sizeSelector = createSizeSelector(mediaContainer);
            
            mediaContainer.appendChild(mediaEl);
            mediaContainer.appendChild(controls);
            mediaContainer.appendChild(sizeSelector);
            
            mediaContainer.addEventListener('dragstart', handleMediaDragStart);
            mediaContainer.addEventListener('dragend', handleMediaDragEnd);
            
            removePlaceholder(targetGrid);
            targetGrid.appendChild(mediaContainer);
            
            // フェードインアニメーションのトリガー
            setTimeout(() => {
                mediaContainer.classList.add('visible');
            }, 10);
        }

        // メディアコントロール作成
        function createMediaControls(mediaContainer, targetGrid) {
            const controls = document.createElement('div');
            controls.className = 'media-controls';
            
            const sizeBtn = createMediaButton('↔️', 'サイズ変更', (e) => {
                e.stopPropagation();
                const selector = mediaContainer.querySelector('.size-selector');
                selector.classList.toggle('show');
            });
            
            const upBtn = createMediaButton('↑', '上へ移動', () => moveMedia(mediaContainer, -1));
            const downBtn = createMediaButton('↓', '下へ移動', () => moveMedia(mediaContainer, 1));
            const deleteBtn = createMediaButton('×', '削除', () => {
                mediaContainer.remove();
                checkGridPlaceholder(targetGrid);
            });
            deleteBtn.classList.add('delete');
            
            controls.appendChild(sizeBtn);
            controls.appendChild(upBtn);
            controls.appendChild(downBtn);
            controls.appendChild(deleteBtn);
            
            return controls;
        }

        // メディアボタン作成
        function createMediaButton(text, title, onClick) {
            const btn = document.createElement('button');
            btn.className = 'media-btn';
            btn.innerHTML = text;
            btn.title = title;
            btn.onclick = onClick;
            return btn;
        }

        // サイズセレクター作成
        function createSizeSelector(mediaContainer) {
            const sizeSelector = document.createElement('div');
            sizeSelector.className = 'size-selector';
            
            const sizes = [
                { label: '1/3幅', value: '4', class: 'third-width' },
                { label: '1/2幅', value: '6', class: '' },
                { label: '2/3幅', value: '8', class: 'two-thirds-width' },
                { label: '全幅', value: '12', class: 'full-width' }
            ];
            
            sizes.forEach(size => {
                const option = document.createElement('div');
                option.className = 'size-option' + (size.value === '6' ? ' active' : '');
                option.dataset.size = size.value;
                option.textContent = size.label;
                
                option.onclick = (e) => {
                    e.stopPropagation();
                    mediaContainer.className = 'media-container fade-in visible';
                    if (size.class) mediaContainer.classList.add(size.class);
                    
                    sizeSelector.querySelectorAll('.size-option').forEach(opt => opt.classList.remove('active'));
                    option.classList.add('active');
                    sizeSelector.classList.remove('show');
                };
                
                sizeSelector.appendChild(option);
            });
            
            return sizeSelector;
        }

        // コントロールボタン作成
        function createControlButton(icon, title, onClick) {
            const btn = document.createElement('button');
            btn.className = 'media-btn';
            btn.innerHTML = icon;
            btn.title = title;
            btn.onclick = onClick;
            return btn;
        }

        // プレースホルダー管理
        function addPlaceholder(grid) {
            if (!grid.querySelector('.grid-placeholder')) {
                const placeholder = document.createElement('div');
                placeholder.className = 'grid-placeholder';
                placeholder.innerHTML = `
                    <div style="text-align: center;">
                        <p>ここに画像や動画をドラッグ＆ドロップ</p>
                        <p>または</p>
                        <button class="upload-btn" style="margin-top: 10px;">
                            ファイルを選択
                        </button>
                    </div>
                `;
                
                const uploadBtn = placeholder.querySelector('.upload-btn');
                uploadBtn.onclick = () => {
                    app.currentUploadGrid = grid;
                    elements.mediaInput.click();
                };
                
                grid.appendChild(placeholder);
            }
        }

        function removePlaceholder(grid) {
            const placeholder = grid.querySelector('.grid-placeholder');
            if (placeholder) placeholder.remove();
        }

        function checkGridPlaceholder(grid) {
            if (!app.isEditMode || !grid) return;
            
            const hasMedia = grid.querySelector('.media-container');
            if (!hasMedia) {
                addPlaceholder(grid);
            } else {
                removePlaceholder(grid);
            }
        }

        // メディア移動
        function moveMedia(mediaContainer, direction) {
            const grid = mediaContainer.parentElement;
            const items = Array.from(grid.children).filter(child => 
                child.classList.contains('media-container')
            );
            const currentIndex = items.indexOf(mediaContainer);
            const newIndex = currentIndex + direction;
            
            if (newIndex >= 0 && newIndex < items.length) {
                if (direction === -1) {
                    grid.insertBefore(mediaContainer, items[newIndex]);
                } else {
                    if (items[newIndex + 1]) {
                        grid.insertBefore(mediaContainer, items[newIndex + 1]);
                    } else {
                        grid.appendChild(mediaContainer);
                    }
                }
            }
        }

        // セクション操作
        function moveSection(wrapper, direction) {
            const sections = Array.from(document.querySelectorAll('.section-wrapper'));
            const currentIndex = sections.indexOf(wrapper);
            const newIndex = currentIndex + direction;
            
            if (newIndex >= 0 && newIndex < sections.length) {
                const targetSection = sections[newIndex];
                
                if (direction === -1) {
                    targetSection.parentElement.insertBefore(wrapper, targetSection);
                } else {
                    if (targetSection.nextElementSibling) {
                        targetSection.parentElement.insertBefore(wrapper, targetSection.nextElementSibling);
                    } else {
                        targetSection.parentElement.appendChild(wrapper);
                    }
                }
                
                updateSectionNumbers();
                wrapper.scrollIntoView({ behavior: 'smooth', block: 'center' });
            }
        }

        function duplicateSection(wrapper) {
            const clone = wrapper.cloneNode(true);
            
            wrapper.parentElement.insertBefore(clone, wrapper.nextElementSibling);
            
            // イベントリスナーを再設定
            const controls = clone.querySelector('.section-controls');
            if (controls) {
                controls.innerHTML = '';
                const upBtn = createControlButton('⬆️', '上へ移動', () => moveSection(clone, -1));
                const downBtn = createControlButton('⬇️', '下へ移動', () => moveSection(clone, 1));
                const duplicateBtn = createControlButton('📋', '複製', () => duplicateSection(clone));
                const deleteBtn = createControlButton('🗑️', '削除', () => deleteSection(clone));
                
                controls.appendChild(upBtn);
                controls.appendChild(downBtn);
                controls.appendChild(duplicateBtn);
                controls.appendChild(deleteBtn);
            }
            
            // グリッドイベントを再設定
            const grid = clone.querySelector('.media-grid');
            if (grid) {
                grid.addEventListener('dragover', handleGridDragOver);
                grid.addEventListener('drop', handleGridDrop);
                if (app.isEditMode) {
                    grid.classList.add('drop-zone');
                }
                
                // プレースホルダーのイベントも再設定
                const placeholder = grid.querySelector('.grid-placeholder');
                if (placeholder) {
                    const uploadBtn = placeholder.querySelector('.upload-btn');
                    if (uploadBtn) {
                        uploadBtn.onclick = () => {
                            app.currentUploadGrid = grid;
                            elements.mediaInput.click();
                        };
                    }
                }
            }
            
            // 編集可能要素を設定
            clone.querySelectorAll('.editable').forEach(el => {
                el.contentEditable = app.isEditMode;
            });
            
            // メディア要素のイベントを再設定
            clone.querySelectorAll('.media-container').forEach(media => {
                media.addEventListener('dragstart', handleMediaDragStart);
                media.addEventListener('dragend', handleMediaDragEnd);
                
                // メディアコントロールのイベントを再設定
                const controls = media.querySelector('.media-controls');
                if (controls) {
                    controls.innerHTML = '';
                    const newControls = createMediaControls(media, grid);
                    controls.parentNode.replaceChild(newControls, controls);
                }
            });
            
            updateSectionNumbers();
            clone.scrollIntoView({ behavior: 'smooth', block: 'center' });
        }

        function deleteSection(wrapper) {
            if (confirm('このセクションを削除しますか？')) {
                const nextElement = wrapper.nextElementSibling;
                if (nextElement && nextElement.classList.contains('add-section-container')) {
                    nextElement.remove();
                }
                wrapper.remove();
                updateSectionNumbers();
            }
        }

        function updateSectionNumbers() {
            const wrappers = document.querySelectorAll('.section-wrapper');
            wrappers.forEach((wrapper, index) => {
                const number = wrapper.querySelector('.section-number');
                if (number) {
                    number.textContent = index + 1;
                }
            });
        }

        // ドラッグ&ドロップ処理
        function handleMediaDragStart(e) {
            if (!app.isEditMode) return;
            app.draggedElement = e.currentTarget;
            app.draggedElement.classList.add('dragging');
            e.dataTransfer.effectAllowed = 'move';
        }

        function handleMediaDragEnd(e) {
            if (app.draggedElement) {
                app.draggedElement.classList.remove('dragging');
                app.draggedElement = null;
            }
        }

        function handleGridDragOver(e) {
            if (!app.isEditMode || !app.draggedElement) return;
            e.preventDefault();
            
            const afterElement = getDragAfterElement(e.currentTarget, e.clientY);
            if (afterElement == null) {
                e.currentTarget.appendChild(app.draggedElement);
            } else {
                e.currentTarget.insertBefore(app.draggedElement, afterElement);
            }
        }

        function handleGridDrop(e) {
            e.preventDefault();
            if (app.draggedElement) {
                const oldGrid = app.draggedElement.parentElement;
                const newGrid = e.currentTarget;
                
                if (oldGrid !== newGrid) {
                    checkGridPlaceholder(oldGrid);
                    checkGridPlaceholder(newGrid);
                }
            }
        }

        function getDragAfterElement(container, y) {
            const draggableElements = [...container.querySelectorAll('.media-container:not(.dragging)')];
            
            return draggableElements.reduce((closest, child) => {
                const box = child.getBoundingClientRect();
                const offset = y - box.top - box.height / 2;
                
                if (offset < 0 && offset > closest.offset) {
                    return { offset: offset, element: child };
                } else {
                    return closest;
                }
            }, { offset: Number.NEGATIVE_INFINITY }).element;
        }

        // HTML保存
        function saveHTML() {
            // 編集モードを一時的に終了
            const wasEditMode = app.isEditMode;
            if (app.isEditMode) {
                toggleEditMode();
            }
            
            // 一時的な要素を削除
            document.querySelectorAll('.add-section-container').forEach(el => {
                el.remove();
            });
            
            // セクション番号を非表示に
            document.querySelectorAll('.section-number').forEach(el => {
                el.style.display = 'none';
            });
            
            // セクションコントロールを非表示に
            document.querySelectorAll('.section-controls').forEach(el => {
                el.style.display = 'none';
            });
            
            const htmlContent = document.documentElement.outerHTML;
            const blob = new Blob([htmlContent], { type: 'text/html;charset=utf-8' });
            const url = URL.createObjectURL(blob);
            
            const a = document.createElement('a');
            a.href = url;
            a.download = `infographic-${new Date().toISOString().slice(0, 10)}.html`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            
            // 元の状態に戻す
            if (wasEditMode) {
                toggleEditMode();
            }
        }

        // Intersection Observer
        function initializeObserver() {
            const observerOptions = {
                threshold: 0.1,
                rootMargin: '0px 0px -50px 0px'
            };
            
            const observer = new IntersectionObserver((entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        entry.target.classList.add('visible');
                    }
                });
            }, observerOptions);
            
            document.querySelectorAll('.fade-in').forEach(el => {
                observer.observe(el);
            });
        }
    </script>
</body>
</html>
```