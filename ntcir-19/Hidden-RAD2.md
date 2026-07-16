# NTCIR-19 Hidden-RAD2 タスク仕様まとめ

> **調査日:** 2026-07-16
> **対象:** NTCIR-19 Hidden-RAD2 Task 1 / Task 2
> **注意:** 本文は、公式Webサイトおよび公式GitHubリポジトリで公開されている情報を整理したものです。正式な提出形式、実行回数、ファイル名、評価スクリプトなどは今後更新される可能性があります。参加時には必ず公式情報を再確認してください。

## 公式リンク

* Hidden-RAD2公式サイト: https://sites.google.com/view/hidden-rad2/
* タスク概要: https://sites.google.com/view/hidden-rad2/tasks
* Task 1 GitHub: https://github.com/hidden-rad2/Task1
* Task 2 GitHub: https://github.com/hidden-rad2/Task2
* MIMIC-CXR: https://physionet.org/content/mimic-cxr/2.1.0/

---

## 1. タスク全体の概要

Hidden-RAD2は、胸部X線画像と放射線診断レポートを対象とする2つのサブタスクから構成される。

| 項目    | Task 1                                       | Task 2                                                         |
| ----- | -------------------------------------------- | -------------------------------------------------------------- |
| 正式名称  | Structured Radiological Reasoning Prediction | Causal Verification and Correction                             |
| 主目的   | 胸部X線画像から、放射線科医が検証した構造化診断推論 A1–A5 を予測する       | AI生成の因果説明を検証し、誤りの位置・種類を特定して修正する                                |
| 主入力   | MIMIC-CXR胸部X線画像                              | 因果説明、MIMIC-CXRレポート参照、胸部X線画像参照                                  |
| 主出力   | A1、A2、A3、A4、A5                               | validity、errors、corrected explanation、confidence、evidence_used |
| 公開形式  | CSV                                          | JSONL                                                          |
| 公開量   | 合計1,000症例予定                                  | 件数未公表                                                          |
| 現在の状態 | 第1回300症例を公開済み                                | スキーマと合成例のみ公開、正式データは放射線科医レビュー中                                  |
| 主評価   | A1–A5を各20点、合計100点                            | 5評価項目の加点、合計100点                                                |

参加者はTask 1、Task 2の一方または両方に参加できる。

---

# 2. Task 1: Structured Radiological Reasoning Prediction

## 2.1 目的

胸部X線画像を入力として、単一の最終診断や自由記述レポートだけではなく、放射線科医の診断過程を表す次の構造化情報を予測する。

1. **A1: Initial Impressions**
2. **A2: Anatomical Location**
3. **A3: Thoracic Level**
4. **A4: Final Impressions**
5. **A5: Confirmation Checklist**

すなわち、Task 1は単純な画像分類ではなく、初期所見、部位、胸郭内レベル、最終所見、確認根拠を含む**構造化された放射線診断推論の予測**である。

## 2.2 データ量と公開スケジュール

| 公開回            |        公開日 |       症例数 |
| -------------- | ---------: | --------: |
| Data Release 1 | 2026-07-14 |       300 |
| Data Release 2 | 2026-07-20 |       400 |
| Data Release 3 | 2026-07-25 |       300 |
| **合計**         |            | **1,000** |

第1回公開CSVは約90.3 KBで、300症例を含む。画像ファイルは含まれないため、実際に必要となる総ストレージ容量は、MIMIC-CXRから取得するDICOM画像の枚数とサイズに依存する。

第1回300症例のうち、現在の規則では77症例がA4=`-`であり、最終結論が正常と扱われる。

## 2.3 入力データ

各症例の主入力は次のとおり。

* MIMIC-CXRの胸部X線画像
* 症例によっては1枚以上のDICOM画像
* 任意で利用可能な外部医学知識、事前学習済みモデル、その他の補助情報

対応する放射線診断レポートはアノテーション作成時の参照資料であるが、Task 1の主要なテスト入力ではない。

### 画像ファイル

MIMIC-CXRの画像はDICOM形式で提供される。Hidden-RAD2の公開CSVには画像本体ではなく、画像を特定するためのディレクトリとDICOMファイル名が格納される。

## 2.4 公開CSVの形式

1行が1症例に対応する。

| 列名            | 役割    | 内容                  |
| ------------- | ----- | ------------------- |
| `Dir`         | 入力参照  | MIMIC-CXR内の対象ディレクトリ |
| `DICOM_name`  | 入力参照  | 1個以上のDICOMファイル名     |
| `A1`          | 正解ラベル | 初期所見候補              |
| `A2`          | 正解ラベル | 解剖学的位置、左右情報         |
| `A3`          | 正解ラベル | 胸椎レベルまたは肺野レベル       |
| `A4`          | 正解ラベル | 最終所見。`-`は正常な最終結論    |
| `A5_1`–`A5_4` | 正解ラベル | 最終所見を支持する確認チェック項目番号 |

### CSV利用上の注意

* `-`は「なし」または「該当なし」を表す。
* 複数値、リスト、辞書形式に見える構造化文字列が引用符付きで格納されることがある。
* 1症例に複数のDICOMファイルが対応する場合がある。
* 単純な文字列分割ではなく、Pythonの`csv`、`pandas.read_csv`など標準準拠のCSVパーサを使う。
* 公開データ中には`Parenchyme`などのレガシー表記が含まれる可能性があるため、独自に綴りを修正せず、公式の正規化規則に従う。

### 説明用のCSV例

以下はスキーマを説明するための**合成例**であり、実際のMIMIC-CXR症例ではない。

```csv
Dir,DICOM_name,A1,A2,A3,A4,A5_1,A5_2,A5_3,A5_4
pXX/pXXXXXXX/sXXXXXXXX,"['synthetic-frontal.dcm']","['Pneumonia','Atelectasis']","['Left lower lung']","[{'begin':6,'end':11}]","['Pneumonia']","[8]","[14]","-","-"
```

## 2.5 出力データ

### A1 — Initial Impressions

初期画像確認時に考慮した、合理的な所見・診断候補をすべて出力する。複数ラベルを許す。

A1は最終診断ではないため、A4に残らない候補が含まれてよい。

<details>
<summary>現在公開されている36ラベル</summary>

* Abscess
* Ascites
* Aspergillosis
* Atelectasis
* Atherosclerosis
* Bronchiectasis
* Bulla
* Cancer
* Cardiomegaly
* Chronic obstructive pulmonary disease
* Congestion
* Congestive heart failure
* Contusion
* Edema
* Effusion
* Emphysema
* Fibrosis
* Fungus
* Granuloma
* Hemothorax
* Infection
* Interstitial Lung Disease
* Lymphadenopathy
* Mass
* Metastasis
* Nodule
* Perforation
* Pleural effusion
* Pleurisy
* Pneumomediastinum
* Pneumonia
* Pneumothorax
* Pulmonary hypertension
* Thromboembolism
* Tuberculosis
* Tumor

</details>

### A2 — Anatomical Location

各所見の解剖学的位置を出力する。必要に応じて左右を区別する。

対象には、肺実質・肺葉、気管・気管支、肺血管、胸膜、縦隔、心腔、横隔膜、肋骨、骨構造、皮下組織、肺門、気管分岐部周辺などが含まれる。

### A3 — Thoracic Level

所見に対応する胸椎レベルまたは肺野レベルを出力する。

現在のCSVでは、T1–T12を整数1–12で表す`begin`と`end`の組として表現される。

```text
{'begin': 6, 'end': 11}
```

これはT6からT11までを表す。

### A4 — Final Impressions

画像所見により最終的に支持される診断・所見を出力する。A1と同じ36ラベル語彙を使用する。

現在の公開データでは、正常性は次のように表される。

| 状態             | 表現                |
| -------------- | ----------------- |
| 異常             | A4に1個以上の最終所見      |
| 初期疑いはあるが最終的に正常 | A4=`-`、A1–A3に所見あり |
| 完全正常           | A1–A5がすべて`-`      |

A4が`-`の場合、A5もすべて`-`でなければならない。

### A5 — Confirmation Checklist

A4の最終所見を確認・支持する画像所見を、ABCDE方式に基づく28種類のチェック項目番号で出力する。

公開CSVでは最大4列の`A5_1`–`A5_4`があり、各列に複数のチェック番号が含まれる場合がある。

チェック項目は、例えば次を対象とする。

* 気管偏位、気管径、気管分岐角
* 気管支狭窄・途絶
* 左右肺野の対称性
* 異常陰影
* 肺門形状・肺血管分枝
* 肋骨横隔膜角
* 心陰影、心胸郭比
* 縦隔拡大
* 骨折・骨病変
* 横隔膜下遊離ガス
* 皮下気腫
* 食道裂孔ヘルニア

完全な28項目は公式`DATA_SCHEMA.md`を参照する。

## 2.6 Task 1の評価方法

各要素は20点で、合計100点。

| 要素     |      配点 | 主な評価指標                                   |
| ------ | ------: | ---------------------------------------- |
| A1     |      20 | Precision、Recall、集合ベースF1                 |
| A2     |      20 | 正規化Exact Matchまたは集合ベースF1                 |
| A3     |      20 | 正規化Exact Match、複数回答時はPrecision・Recall・F1 |
| A4     |      20 | Precision、Recall、集合ベースF1                 |
| A5     |      20 | Micro-F1、Macro-F1                        |
| **合計** | **100** |                                          |

### 正規化

評価前に次の正規化が行われる可能性がある。

* 大文字・小文字
* 前後空白
* 句読点
* 公式略語
* 単数・複数
* 公式同義語

公式語彙と正規化マッピングは評価スクリプトとともに公開予定である。

### 構造的一貫性

所見、解剖学的位置、胸椎レベルの対応関係も評価される可能性がある。

例えば、正しい所見・部位・レベルを個別に出力していても、組合せを誤っている場合には満点にならない。

## 2.7 Task 1で未確定・要確認の事項

2026-07-16時点では、次の詳細は未確定または未公開である。

* 正式な提出ファイル形式
* 正式な正常ラベルの提出時表現
* ファイル名と提出方法
* 提出可能なrun数
* A4の各所見と`A5_1`–`A5_4`の対応規則
* 重複ラベルの扱い
* A1とA2/A3の対応付け方法
* 評価スクリプト
* 同義語・正規化辞書

したがって、現時点では公開CSVを内部表現へ正しく変換できる前処理を先に構築し、正式提出仕様の公開後に出力層を調整するのが安全である。

---

# 3. Task 2: Causal Verification and Correction

## 3.1 目的

胸部X線症例について生成された因果説明を検証し、次を実行する。

1. 説明全体が妥当か判定する。
2. 誤りの正確な文字範囲を特定する。
3. 誤りに公式のhallucination typeを付与する。
4. 根拠に整合する完全な修正版を生成する。
5. 判定のconfidenceを0–1で出力する。

すべての説明に誤りがあるとは限らず、データには次の両方が含まれる。

* 誤りのないvalidな説明
* 1個または少数の制御されたhallucinationを挿入したinvalidな説明

## 3.2 データ量と公開状況

* Task 2正式データの症例数は、2026-07-16時点では公表されていない。
* 正式データ公開予定日は2026-07-22だが、放射線科医レビューの進捗により変更される可能性がある。
* GitHubでは、ドキュメント、`0.1-draft`スキーマ、合成JSONL例が公開されている。
* 正式公開時にスキーマを`1.0`として固定する予定である。

## 3.3 入力データ

各入力レコードは次を含む。

| フィールド                | 型      | 内容              |
| -------------------- | ------ | --------------- |
| `schema_version`     | string | 現在は`0.1-draft`  |
| `case_id`            | string | 症例ID            |
| `report_ref`         | object | MIMIC-CXRレポート参照 |
| `image_refs`         | array  | 1個以上の画像参照       |
| `causal_explanation` | string | 検証対象の因果説明       |

### `report_ref`

| フィールド       | 型           | 内容            |
| ----------- | ----------- | ------------- |
| `source`    | string      | 通常`MIMIC-CXR` |
| `report_id` | string      | レポートID        |
| `url`       | string/null | 認可アクセス用URL    |

### `image_refs[]`

| フィールド      | 型           | 内容            |
| ---------- | ----------- | ------------- |
| `source`   | string      | 通常`MIMIC-CXR` |
| `dicom_id` | string      | DICOM画像ID     |
| `url`      | string/null | 認可アクセス用URL    |

### 入力JSONLの合成例

```json
{"schema_version":"0.1-draft","case_id":"HR2-T2-TEST-000001","report_ref":{"source":"SYNTHETIC","report_id":"synthetic-report-001","url":null},"image_refs":[{"source":"SYNTHETIC","dicom_id":"synthetic-image-001","url":null}],"causal_explanation":"A focal opacity is seen in the left lower lung, supporting right lower lobe pneumonia."}
```

## 3.4 利用する根拠の区分

各runは推論時に実際に使用した根拠を指定する。

| コード  | 区分               | 推論に使用する情報     |
| ---- | ---------------- | ------------- |
| `R`  | Report-only      | 因果説明と元レポート    |
| `I`  | Image-only       | 因果説明と画像       |
| `RI` | Report-and-Image | 因果説明、元レポート、画像 |

これらは別タスクではなく、同一Task 2内のevidence-use categoryである。評価方式は共通だが、結果がカテゴリ別に集計される場合がある。

## 3.5 出力データ

提出はUTF-8のJSONLを予定しており、1行に1症例を格納する。

| フィールド                          | 型      |  必須 | 内容                  |
| ------------------------------ | ------ | --: | ------------------- |
| `schema_version`               | string | Yes | 提出スキーマ版             |
| `run_id`                       | string | Yes | run識別子              |
| `case_id`                      | string | Yes | 入力と同じ症例ID           |
| `evidence_used`                | enum   | Yes | `R`、`I`、`RI`        |
| `validity`                     | enum   | Yes | `valid`または`invalid` |
| `errors`                       | array  | Yes | 予測した誤り              |
| `corrected_causal_explanation` | string | Yes | 完全な修正済み説明           |
| `confidence`                   | number | Yes | 0以上1以下              |

### `errors[]`

| フィールド   | 型       | 内容                   |
| ------- | ------- | -------------------- |
| `start` | integer | 0始まり、inclusive       |
| `end`   | integer | 0始まり、exclusive       |
| `text`  | string  | 入力説明中の完全一致文字列        |
| `type`  | enum    | 公式hallucination type |

### 文字オフセットの規則

* UTF-8バイト位置やトークン位置ではなく、**Unicode文字位置**を使用する。
* `start`は含む。
* `end`は含まない。
* 次の条件を満たす必要がある。

```python
causal_explanation[start:end] == text
```

* エラーは`start`、次に`end`の昇順に並べる。
* `0.1-draft`では重複spanを許さない。
* `EVID-OMIT`と`DX-OMIT`は欠落の挿入位置を示すため、`start == end`かつ`text == ""`を許す。

### valid判定時

```json
{
  "validity": "valid",
  "errors": [],
  "corrected_causal_explanation": "入力のcausal_explanationを変更せず複製"
}
```

### invalid判定時

* `errors`に1件以上を格納する。
* 各`text`は入力中の対象spanと完全一致させる。
* `corrected_causal_explanation`には差分だけでなく、完全な修正済み文章を格納する。

### 出力JSONLの合成例

下の例では、入力文中の`right lower lobe pneumonia`が位置・左右誤りであり、文字位置は`[59, 85)`である。

```json
{"schema_version":"0.1-draft","run_id":"example-ri-run-01","case_id":"HR2-T2-TEST-000001","evidence_used":"RI","validity":"invalid","errors":[{"start":59,"end":85,"text":"right lower lobe pneumonia","type":"LOC-LAT"}],"corrected_causal_explanation":"A focal opacity is seen in the left lower lung, supporting left lower lobe pneumonia.","confidence":0.96}
```

## 3.6 Hallucination taxonomy

GitHubの`0.1-draft`では14種類が定義されている。

| コード           | 意味                |
| ------------- | ----------------- |
| `ADD-PATH`    | 根拠のない病変・診断・所見の追加  |
| `ADD-DEVICE`  | 根拠のない医療機器の追加      |
| `ADD-REC`     | 根拠のない推奨事項の追加      |
| `LOC-LAT`     | 部位・肺野・左右の誤り       |
| `SEV-CHG`     | 重症度の誤った増減         |
| `NEG-FLIP`    | 存在・不存在の反転         |
| `CAUSE-WRONG` | 誤った因果関係           |
| `DDX-WRONG`   | 根拠のない、または矛盾する鑑別診断 |
| `EVID-OMIT`   | 重要な支持根拠の欠落        |
| `DX-OMIT`     | 診断の欠落             |
| `TERM-MINOR`  | 軽微な用語置換           |
| `CONTRA`      | 内部矛盾              |
| `FLUENCY`     | 流暢性・文法の劣化         |
| `CERT-FLIP`   | 確実性の変更            |

この14分類はdraftであり、正式公開前に変更される可能性がある。公式Webの評価説明では一部の主要分類のみが例示されているため、最終的には正式リリースのtaxonomyを使用する。

## 3.7 Task 2の評価方法

合計100点。

| 評価項目                              |      配点 | 主な指標・内容                                                         |
| --------------------------------- | ------: | --------------------------------------------------------------- |
| Validity Classification           |      10 | Precision、Recall、F1、Balanced Accuracy。主指標はF1                    |
| Error-Span Detection              |      20 | Strict span-level Precision、Recall、F1                           |
| Hallucination-Type Classification |      20 | Typed span-level F1、type別Macro-F1                               |
| Correction Quality                |      45 | Edit correctness、clinical factuality、新規hallucination回避、最小修正、流暢性 |
| Confidence Calibration            |       5 | Brier Score。補助的にECE、reliability diagram                         |
| **合計**                            | **100** |                                                                 |

### Error-span detection

開始位置と終了位置がgold spanに完全一致した場合のみstrict matchとなる。

境界が完全一致しなくても重なりがある場合、relaxed overlap scoreが分析用に報告される可能性がある。ただし、正式得点ではstrict span F1が主指標である。

### Hallucination-type classification

span境界とtypeの両方が正しい場合にtyped spanが正解となる。

### Correction Qualityの内訳

| 項目                                  |     配点 | 評価例                                                             |
| ----------------------------------- | -----: | --------------------------------------------------------------- |
| Edit correctness                    |     15 | Exact edit match、修正span内token-F1、edit-level semantic similarity |
| Clinical factuality                 |     20 | 画像・レポートとの整合、部位・左右、診断、因果関係、重要根拠                                  |
| No-new-hallucination and minimality |      5 | 新規誤りを追加しない、不要な改変を抑える                                            |
| Fluency and readability             |      5 | 文法、首尾一貫性、臨床説明としての可読性                                            |
| **合計**                              | **45** |                                                                 |

修正箇所または修正文を対象に、BERTScoreや医学特化sentence similarityが使われる可能性がある。上位runについては、放射線科医によるexpert reviewが検証またはtie-breakに使われる可能性がある。

### Confidence

公式評価ページでは、confidenceは「validity判定が正しいとシステムが見積もる確率」と説明されている。

一方、GitHubのdraft提出仕様では、validity判定と関連する検証結果を含む症例全体の出力に対するconfidenceと説明されている。正式リリース時の定義を再確認する必要がある。

## 3.8 Task 2で未確定・要確認の事項

* 正式データ件数
* 正式スキーマ`1.0`
* 最終的なhallucination taxonomy
* 提出ファイル名
* 配布・提出方法
* 1チームあたりの最大run数
* mixed evidence usageを1run内で許すか
* 公式validator
* correction qualityの最終自動評価器
* expert reviewの対象範囲
* 正式なconfidenceの意味

---

# 4. MIMIC-CXRへのアクセス条件

Hidden-RAD2の主催者は、MIMIC-CXRの画像やレポート本体を再配布しない。参加者は各自でPhysioNetから取得する。

必要な手続きは概ね次のとおり。

1. PhysioNetアカウントを作成する。
2. credentialed userの申請を行う。
3. 指定されたCITI Programの研究倫理・データプライバシー研修を修了する。
4. MIMIC-CXRのData Use Agreementに同意する。
5. 許可された本人のアカウントでデータを取得する。

Task 2では、MIMIC-CXRデータへアクセスするチームメンバー全員が個別に認可を受ける必要がある。

## データ取扱い上の制約

* MIMIC-CXR画像・レポートを第三者へ再配布しない。
* PhysioNetのID、パスワード、アクセストークンを共有しない。
* 非公開リポジトリで配布される参加者限定データを再配布しない。
* 再識別を試みない。
* MIMIC-CXRを利用した公開物では、指定されたデータセット・論文を引用する。
* restricted dataを外部APIや第三者クラウドへ送信する場合は、MIMIC-CXRのライセンス、DUA、所属機関の規程に適合するかを事前に確認する。

---

# 5. 実行環境と計算資源

2026-07-16時点の公開資料では、次のような主催者指定の実行環境は確認できない。

* 主催者提供GPU
* Dockerイメージ
* 固定OSまたはPythonバージョン
* 推論時間制限
* メモリ上限
* インターネット接続制限
* 外部LLM APIの一律禁止
* モデルサイズ上限
* 学習時間上限

現時点では、参加者が自身の環境で推論し、所定形式のファイルを提出する**ファイル提出型タスク**として準備するのが妥当である。ただし、最終的なrun数、外部資源、API利用、提出方法は正式ルール公開後に確認する。

Task 1では、外部医学知識、事前学習済みモデル、補助情報の利用が明示的に想定されている。ただし、MIMIC-CXRのrestricted dataを扱うため、第三者サービスへのデータ送信可否は別途確認が必要である。

---

# 6. 推奨する実装構成

```text
hidden-rad2-system/
├── README.md
├── pyproject.toml
├── requirements.txt
├── configs/
│   ├── task1.yaml
│   └── task2.yaml
├── data/
│   ├── task1/
│   │   ├── public_csv/
│   │   └── processed/
│   └── task2/
│       ├── input/
│       └── processed/
├── src/
│   ├── common/
│   │   ├── io.py
│   │   ├── logging.py
│   │   └── validation.py
│   ├── task1/
│   │   ├── dataset.py
│   │   ├── model.py
│   │   ├── decode.py
│   │   └── submit.py
│   └── task2/
│       ├── dataset.py
│       ├── verifier.py
│       ├── span_detector.py
│       ├── corrector.py
│       └── submit.py
├── scripts/
│   ├── download_mimic.py
│   ├── validate_task1.py
│   ├── validate_task2.py
│   └── make_submission.py
└── tests/
    ├── test_task1_schema.py
    └── test_task2_offsets.py
```

## Task 1の最低限の前処理確認

```python
import pandas as pd

df = pd.read_csv("NTCIR19_Task1_Sample_1st.csv")

required = {
    "Dir",
    "DICOM_name",
    "A1",
    "A2",
    "A3",
    "A4",
    "A5_1",
    "A5_2",
    "A5_3",
    "A5_4",
}

missing = required - set(df.columns)
if missing:
    raise ValueError(f"Missing columns: {sorted(missing)}")

print(f"cases: {len(df)}")
print(df.head())
```

## Task 2のspan検証

```python
def validate_error_span(causal_explanation: str, error: dict) -> None:
    start = error["start"]
    end = error["end"]
    text = error["text"]

    if not (0 <= start <= end <= len(causal_explanation)):
        raise ValueError("Invalid span range")

    if causal_explanation[start:end] != text:
        raise ValueError(
            f"Span mismatch: expected={causal_explanation[start:end]!r}, "
            f"submitted={text!r}"
        )
```

---

# 7. 主要日程

| 日付            | 内容                           |
| ------------- | ---------------------------- |
| 2026-07-14    | Task 1 Data Release 1: 300症例 |
| 2026-07-20    | Task 1 Data Release 2: 400症例 |
| 2026-07-22    | Task 2 Dataset Release予定     |
| 2026-07-25    | Task 1 Data Release 3: 300症例 |
| 2026-07-30    | Formal Run開始                 |
| 2026-08-17    | Run提出締切                      |
| 2026-08-22頃   | 評価結果返却                       |
| 2026-09-01    | Participant Paper Draft締切    |
| 2026-11-01    | Camera-ready締切               |
| 2026-12-08〜10 | NTCIR-19 Conference          |

原則として締切は午後11時59分、Anywhere on Earth（AoE）である。

---

# 8. 現時点での実務上の結論

## Task 1

すぐ着手できる。

* 公開300症例のCSVを取得する。
* PhysioNet認可済み環境からDICOM画像を対応付ける。
* CSVの複数値・辞書形式を壊さずに読み込む。
* 画像からA1–A5を個別またはマルチタスクで予測する。
* 特にA4の正常判定とA5のmulti-label予測を独立に検証する。
* 正式提出形式公開後にsubmission writerを確定する。

## Task 2

現在は実装準備が中心となる。

* JSONL reader/writerを作る。
* Unicode文字offsetを厳密に扱う。
* valid/invalid分類、span検出、type分類、全文修正を分離して設計する。
* R、I、RIの各条件を切り替えられるようにする。
* synthetic exampleでsubmission validatorを作る。
* 正式データ公開後、スキーマ`1.0`との差分を確認する。

---

# 9. 参考資料

* Hidden-RAD2 Tasks
  https://sites.google.com/view/hidden-rad2/tasks

* Task 1 Definition
  https://sites.google.com/view/hidden-rad2/tasks/task-1-definition

* Task 1 Evaluation
  https://sites.google.com/view/hidden-rad2/evaluation/task-1-evaluation

* Task 1 GitHub
  https://github.com/hidden-rad2/Task1

* Task 1 Data Schema
  https://github.com/hidden-rad2/Task1/blob/main/DATA_SCHEMA.md

* Task 2 Definition
  https://sites.google.com/view/hidden-rad2/tasks/task-2-definition

* Task 2 Evaluation
  https://sites.google.com/view/hidden-rad2/evaluation/task-2-evaluation

* Task 2 Span and Type Evaluation
  https://sites.google.com/view/hidden-rad2/evaluation/task-2-evaluation/span-and-type-evaluation

* Task 2 Correction Evaluation
  https://sites.google.com/view/hidden-rad2/evaluation/task-2-evaluation/correction-evaluation

* Task 2 GitHub
  https://github.com/hidden-rad2/Task2

* Task 2 JSONL Schema
  https://github.com/hidden-rad2/Task2/blob/main/SCHEMA.md

* Task 2 Submission Format
  https://github.com/hidden-rad2/Task2/blob/main/SUBMISSION_FORMAT.md

* Task 2 Hallucination Taxonomy
  https://github.com/hidden-rad2/Task2/blob/main/HALLUCINATION_TAXONOMY.md

* Official Schedule
  https://sites.google.com/view/hidden-rad2/schedule

* MIMIC-CXR Database
  https://physionet.org/content/mimic-cxr/2.1.0/

* PhysioNet CITI Course Instructions
  https://physionet.org/about/citi-course/
