# レミフェンタニル併用モデル — 実装完了報告（2026-09-30）

## やったこと
- **新規タブ `vellinga.html` を追加**（別ファイル、依存ゼロ・オフライン動作）
- **`index.html` のヘッダーに「→ レミフェンタニル併用モデルを開く（Vellinga 2025）」リンク**を追加（Masui 2022 本体は変更なし）
- レミフェンタニルは **TCI方式**（ターゲット血中濃度 ng/mL 指定、試験と同様）＋負荷量/開始/終了時刻
- レミマゾラムも TCI（ターゲット µg/mL）＋負荷量

## 実装したモデル（Vellinga 2025 補遺 D790/D791 の exact NONMEM 値）
| モデル | 内容 |
|---|---|
| レミマゾラム PK | 3コンパートメント refit 固定値: V1=5.31 / V2=12.81 / V3=40.04 L, CL=1.15, Q2=0.91, Q3=0.29 L/min, 代謝分率 FM=0.80 |
| 代謝物 CNS7054 | 見かけの CL=0.05 L/min, V5≈7.8 L, ktr≈0.30 |
| 相互作用（急性耐性） | remifentanil が代謝物分解を競合抑制: `INH = CR^0.6/(CR^0.6 + 8.0^0.6)`（IC50=8.0 ng/mL, Hill=0.6）→ 代謝物 CL = `K50·(1−INH)` |
| レミフェンタニル PK | Eleveld 2017 の3コンパートメント allometric モデル（D790 の固定 THETA、年齢/身長/体重/性で算出） |
| MOAAS PD | 補遺 D791 の**順位（categorical）モデル**: 基準 logits LLE0–LLE4（BASE0=2.1, BASE1=0.5, BASE2=−0.1, BASE3=−0.4, BASE4=0.6）+ DRUGM → P0–P5 → 期待 MOAAS。DRUGM = EMAXM·(CAM/(1+CAM+CBM))·(1+CCM), EMAXM=e^2.4, EC50M=5.3, EC50MM=8.0, GAMMAM=e^0.5, EC50MR=e^−0.3 |
| BIS PD | `BIS = 100·expit(LBASE − DRUGB)`, LBASE=log(BASEB/(100−BASEB)), BASEB=e^4.54, EMAXB=e^1.2, EC50B=e^5.2, EC50MB=e^9.0, EC50RB=e^2.08 |

## 検算結果（node で実行・すべて正常）
- JS syntax check: **OK**
- 統合ODE（両TCI + INH）: レミフェンタニルがターゲット到達（例: 目標10 → 9.98 ng/mL @ t=120分）
- MOAAS/BIS 挙動（例、70歳相当設定）:
  - 投与なし: MOAAS≈4.3, BIS≈94
  - remimazolam 1 µg/mL + 代謝物蓄積 + remifentanil 10 ng/mL: MOAAS≈0.2, BIS≈70 → 鎮静が適切に深まる

## 注記
- 旧タブの EC50B=35.79 / EC50M=43.03 は旧 dose-finding 論文（1期のみ）の値。新タブは3期間横断 refit 値を採用し、旧値は脚注として残している
- 研究・教育・学習用。臨床投与判断に使用しないこと（タブ内に明示）

## 使い方
1. `index.html` をブラウザで開く → ヘッダーの「→ レミフェンタニル併用モデルを開く」リンク
2. 直接 `vellinga.html` を開くことも可
3. 患者情報・両 TCI を設定 → 「▶ シミュレーション実行」→ グラフ・時刻別表・CSV出力
