# 3Dトラック積載シミュレーター (3D Truck Loading Simulator)

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Ready-brightgreen)](https://pages.github.com/)
[![Three.js](https://img.shields.io/badge/Three.js-r128-blue)](https://threejs.org/)
[![TailwindCSS](https://img.shields.io/badge/TailwindCSS-v3-38bdf8)](https://tailwindcss.com/)

3D空間上でトラック（10tウイング車、4t中型、2t小型）への段ボール・貨物混載パッキングをリアルタイムに計算・描画・検証できるWebアプリケーションです。

---

## 🌟 主な改修・改善機能

### 1. 積み残し貨物のトラック外可視化 (Overflow / Staging Area)
- **課題の解決**: 荷台長（$X \le L/2$）を超過した段ボールを描画スキップ（非表示）にするのではなく、トラック後方ゲート外の保管エリア（Staging Area）に赤色ハイライトで整列描画します。
- **視覚的区別**:
  - **荷台内貨物**: 通常カラー（各品目の指定色）＋通常エッジ
  - **トラック外積み残し貨物**: 赤色半透明シェーディング（`#ef4444`）＋赤色ワイヤーフレームライン（`#ff1e1e`）
  - **保管エリア床面**: 赤色境界ライン枠・警告グリッド
- **ツールチップ連動**: 3D貨物をクリックすると「状態: 積み残し（荷台外・要対応）」または「状態: 荷台内積載 (OK)」のステータスバッジを表示。

### 2. 混載高密度パッキングロジック (3D BLF / Extreme Point 法)
- **バラ積み・隙間充填対応**:
  - 従来の品目単位での単純な壁積み（Wall Building）では、天井付近や側面に大きなデッドスペースが発生し、容積率に余裕があっても荷台から溢れていました。
  - 新アルゴリズムでは、大型ケースを配置した後に生じる**上部隙間（天井付近）や側面の空き空間**に、後続の小型・薄型ケース（ウェットティッシュ・詰め替え品等）を自動で滑り込ませて充填します。
- **配送ステージ順序の厳守**:
  - Stage 1（奥側・キャビン寄り） $\rightarrow$ Stage 2（手前側・後方ゲート寄り）の配送順序制約を完全遵守。
- **マルチオリエンテーション（6方向回転試行）**:
  - 各段ボールの $(L, W, H)$ を6通りの姿勢で動的評価し、隙間に最も高密度かつ安定して収まる姿勢を選択。
- **全数格納の達成**:
  - 低床10tウイング車において、35品目・全792ケース（約46.5 m³）のデッドスペースを排除し、荷台内への**全数格納完了（積み残し0ケース）**を実現。

### 3. KPI・アラート・操作UIの完全連動
- **ステータスアラートバー**:
  - 積み残し発生時: `【警告】積み残し発生: XX ケースが溢れています`（赤色警告表示）
  - 全数格納時: `全数格納完了 (積み残し 0ケース) [100% 格納]`（緑色完了表示）
- **パッキングモード切替ボタン**:
  - `3D高密度バラ積み (BLF)` と `従来ウォール積み` をワンクリックで切り替え、アルゴリズムによる積載効率の違いや積み残し発生の様子を比較検証可能。
- **車両プリセット切替**:
  - 低床10tウイング (960×240×260cm)
  - 4t中型ウイング (620×220×230cm)
  - 2t小型ドライバン (310×180×190cm)
  - 車両サイズを変更すると、積載可能容量に応じてリアルタイムに積み残しが再計算され、後方に溢れ分が描画されます。

---

## 🚀 GitHub Pages での公開手順

本リポジトリは静的単一ファイル（SPA）構成のため、GitHub Pages を有効化するだけで即座にWeb上に公開できます。

### Step 1. GitHub 上で新規リポジトリを作成
1. [GitHub](https://github.com/) にログインし、`New repository` をクリックします。
2. リポジトリ名（例: `3d-truck-loading-simulator`）を入力し、`Public` を選択して作成します（READMEや.gitignoreは追加しないで空のリポジトリを作成）。

### Step 2. リモートを追加してプッシュ
ターミナルまたはPowerShellで以下を実行します：

```bash
# リモートURLの登録（USERNAMEとREPO_NAMEをご自身のリポジトリに変更）
git remote add origin https://github.com/USERNAME/3d-truck-loading-simulator.git

# メインブランチをプッシュ
git branch -M main
git push -u origin main
```

### Step 3. GitHub Pages の有効化
1. GitHubリポジトリのページで **Settings** タブをクリックします。
2. 左メニューの **Pages** を選択します。
3. **Build and deployment** の **Source** で `Deploy from a branch` を選択します。
4. **Branch** で `main`（または `master`）、フォルダは `/ (root)` を選択し、**Save** をクリックします。
5. 数分後に発行されるURL（`https://USERNAME.github.io/3d-truck-loading-simulator/`）にアクセスすれば、ブラウザ上で誰でもシミュレーターを利用できます。

---

## 💻 ローカル環境での起動方法

インストールやビルドツールは不要です。

1. 本リポジトリの `index.html` を任意のブラウザ（Google Chrome, Edge, Safari など）で直接ダブルクリックして開くだけで動作します。
2. または、VS Code の拡張機能「Live Server」等で開くことも可能です。

---

## 📁 ファイル構成

```text
├── index.html                 # シミュレーター本体（UI, 3Dレンダラー, パッキングエンジン統合版）
├── truck-data-sample.json     # 実測荷姿データ（35品目・全792ケースのサンプルJSON）
├── README.md                  # プロジェクト説明書
└── .gitignore                 # Git除外設定
```

---

## 🛠 使用技術
- **Three.js (r128)**: トラック車体・荷台・段ボールの3Dグラフィックス描画
- **OrbitControls**: 360度カメラ回転・パン・ズーム操作
- **Tailwind CSS (CDN)**: レスポンシブで洗練されたモダンUI
- **Lucide Icons**: 直感的なSVGアイコン群
- **Pure JavaScript**: 高速な3Dパッキング計算（依存ライブラリなし）
