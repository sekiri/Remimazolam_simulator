# Remimazolam PK/PD Simulator

レミマゾラム（remimazolam）の **PK/PD 推定シミュレータ**です。
投与プロトコル（静的投与 bolus の量・時刻、持続投与の開始時刻と速度、その後の速度変更）を入力すると、血中濃度 **Cp**、効果部位濃度 **Ce**、そして鎮静深度 **MOAA/S** の推移をリアルタイムに計算・可視化します。
（※目標濃度 Ce を入力して必要な投与速度を逆算する TCI ではありません）

> **単一 HTML ファイルだけで動作します。外部ライブラリ・サーバー・ネットワーク通信は一切不要です。**
> `index.html` をダブルクリックしてブラウザで開くだけで使えます。

---

## ⚠️ 免責事項 / Disclaimer

- 本ツールは **研究・教育目的** のためだけに作られました。
- **実際の臨床判断・麻酔管理・投薬の決定には使用しないでください。**
- 表示される Cp / Ce / MOAA/S はすべて **予測値** です。実際の患者の反応は年齢・体質・併用薬・基礎疾患等で大きく異なります。
- 使用に起因するいかなる損害にも責任を負いかねます。
- 本ツールは医療機器ではありません。

> This tool is for **research and educational purposes only**. It is **not** a medical device and must **not** be used to guide clinical decisions or patient care. All values are model-based predictions, not measurements.

---

## 機能一覧 / Features

- **3コンパートメント PK モデル**（Masui 2022）で Cp（血中）と Ce（効果部位/脳相当）を算出
- **静的投与（bolus）**：任意の時刻・量を複数回追加可能
- **持続投与（infusion）**：1 ポンプとして扱い、**ある時点での速度変更を何回でも追加**
  - 各変更 = その時刻から新速度で続行
  - **速度 0 = その時刻で投与終了**
- **効果部位平衡定数 ke0**：Masui 2022 重回帰式（年齢/体重/身長/性別/ASAで個別化）または Chon 2024 固定値 0.27 min⁻¹ から選択
- **MOAA/S（鎮静深度）の予測**：Ce と Ce50（MOAA/S ≤4 / ≤3 / ≤2 / ≤1 = 0.302 / 0.397 / 0.483 / 0.654 µg/mL, Chon 2024）から各時刻の予測スコア
- **インタラクティブなチャート**（自前 canvas 描画、外部ライブラリなし）
  - ホイール：濃度軸（Y）ズーム — **下端は 0 固定、上端のみ可変**
  - **Shift + ホイール**：時間軸（X）ズーム（カーソル中心）
  - ドラッグ：時間軸パン / **ピンチ**：時間・濃度同時ズーム（タッチ対応）
  - **Ce 中心縮尺**（既定）/ **Cp+Ce 全体縮尺** にトグル切替
  - Cp が表示上端を超過した場合は ▲ マーカーで示す
- **CSV 出力**：時間・Cp・Ce・予測 MOAA/S をダウンロード
- **チャートのドラッグ / ズーム**：
  - ドラッグ（マウス・タッチ）＝ 時間軸の左右移動（パン）。区間幅は不変で、境界（0分・終了時間）で停止
  - `＋X` / `−X` ボタン ＝ 表示中心を基準にした時間軸の拡大 / 縮小（全表示に戻ると 0〜終了時間にスナップ）
  - `＋Y` / `−Y` ボタン ＝ 濃度軸の拡大 / 縮小（0 基準、全表示は「初期表示」で戻る）
- **パラメータ個別化**：年齢・身長・体重・性別・ASA 分類
- **折りたたみ式の詳細表示**：「時系列データ」と「鎮静深度の予測（MOAA/S）」カードは既定で折りたたまれ、ヘッダをクリックで展開
  - 時系列データ：ページング表示（ページサイズ 10/20/50 行切替・前後ページ移動）
  - 鎮静深度：スライダー/数値入力＋前後ボタンで任意の時刻を指定。時刻はシミュレーション時間を超えないよう自動で調整され、その時刻の Ce・MOAA/S≤1〜4 の確率・予測スコアを表示

---

## 使い方 / How to use

1. `index.html` を任意のブラウザ（最近の Chrome / Edge / Firefox / Safari など）で開く
2. 患者パラメータ（年齢・身長・体重・性別・ASA）とシミュレーション時間を入力
3. 投与プロトコルを設定
   - **静的投与**：`＋ 静的投与を追加` で時刻・量（mg/kg）を追加
   - **持続投与**：開始時刻と初期速度を指定。速度を変えたいたびに `＋ 速度変更を追加` で `{変更時刻, 新速度}` を追加（速度 0 で終了）
4. **シミュレーションを実行** を押す
5. チャート上でホイール/Shift+ホイール/ドラッグ/ピンチで表示を調整
6. 右側の「時系列データ」または「鎮静深度の予測（MOAA/S）」カードのヘッダをクリックすると詳細が展開される（既定は折りたたみ）
7. 必要なら `CSV出力` でデータをダウンロード

初期値は 60 歳・身長 165 cm・体重 65 kg・男性・ASA I・時間 120 分、
静的投与 0.6 mg/kg（t=0）+ 持続投与 1.0 mg/kg/h（t=0 開始）に設定されています。
`リセット` で初期値に戻ります。

### 持続投与の速度変更例

| 変更時刻（分） | 新速度（mg/kg/h） | 意味 |
|---|---|---|
| 0（開始） | 1.0 | 1.0 で開始 |
| 30 | 2.0 | 30 分時に 2.0 に増速 |
| 60 | 0 | 60 分時に投与終了 |

変更は時刻順に自動適用されます。入れ替えても問題ありません。

---

## 数値モデル / Model

### PK（3コンパートメント）
**Masui K, Stöhr T, Pesic M, et al.** A population pharmacokinetic model of remimazolam for general anesthesia and consideration of remimazolam dose in clinical practice. *J Anesth*. 2022;36:493–505. [DOI:10.1007/s00540-022-03079-y](https://doi.org/10.1007/s00540-022-03079-y)

- 中央コンパートメント V1、末梢コンパートメント V2 / V3
- 体重・年齢・性別・ASA 補正（ABW 基準）
- 分布係数 Q2 / Q3

### ke0（効果部位平衡定数）
**Masui K, Hagihira S.** Equilibration rate constant, ke0, to determine effect-site concentration for the Masui remimazolam population pharmacokinetic model. *J Anesth*. 2022;36:757–762. [DOI:10.1007/s00540-022-03099-8](https://doi.org/10.1007/s00540-022-03099-8)

- 年齢・体重・身長・性別・ASA を用いた重回帰式（既定）
- 代替：Chon 2024 の固定値 ke0 = 0.27 min⁻¹

### PD（MOAA/S）
**Chon JY, Seo KH, Lee J, et al.** Target-controlled infusion of remimazolam effect-site concentration for total intravenous anesthesia. *Front Med*. 2024;11:1364357. [DOI:10.3389/fmed.2024.1364357](https://doi.org/10.3389/fmed.2024.1364357)

- MOAA/S ≤4 / ≤3 / ≤2 / ≤1 の Ce50 = 0.302 / 0.397 / 0.483 / 0.654 µg/mL
- 各スコアに対する確率を合計して期待値（予測 MOAA/S）を算出

> 「効果部位濃度 Ce」は脳組織濃度の直接的測定ではなく、PK-PD の effect compartment（脳相当作用濃度）の標準的推定値です。

### 数値積分
RK4（Runge-Kutta 4 次）法、dt = 0.05 min。

---

## 技術メモ / Technical notes

- **完全自己完結**：HTML + CSS + JS が 1 ファイル（`index.html`）。外部依存・ビルド・サーバー不要
- チャートは **原生 `<canvas>` を自前**で描画（Chart.js 等のライブラリ不使用）
- ODE 解算は純 JS の RK4
- 動作確認：デスクトップ（Chrome / Edge / Firefox / Safari）およびタッチ（ピンチズーム）
- 検証：速度変更の増速/減速/終了・複数回変更・開始遅延・リセット、折りたたみカード・時刻選択・ページングを自動テスト（Puppeteer）で確認済み

---

## ファイル構成 / Repository

```
remimazolam-tci-simulator/
├── index.html      # 本体（HTML + CSS + JS、すべてこれ 1 ファイル）
└── README.md       # このファイル
```

---

## ライセンス / License

MIT License（詳細は `LICENSE` を参照）。
※ 引用元の論文・データはそれぞれの著作権者・出版社に帰属します。

---

*Personal project for research and education. Not for clinical use.*
