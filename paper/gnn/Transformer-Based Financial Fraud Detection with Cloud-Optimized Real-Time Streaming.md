# Transformer-Based Financial Fraud Detection with Cloud-Optimized Real-Time Streaming

**Authors:** Tingting Deng, Shuochen Bi, Jue Xiao  
**Year:** 2024  
**Venue:** BDEIM 2024

**Paper:** [DOI](https://doi.org/10.1145/3724154.3724271)  
**PDF:** [ACM Digital Library](https://doi.org/10.1145/3724154.3724271)

---

## [まとめ]

> **一言でいうと:** クレジットカード取引を複雑なTransaction Graphとして表現し、Graph Attention Transformerで取引のトポロジーと時系列パターンを学習して、不正取引を検出するTGTNを提案する手法。

- **何をしたか:** クレジットカード取引をノードとするTransaction Graphを構築し、Graph Attention Transformerで取引間の関係と時間的特徴を学習。
- **新しい点:** 複雑な取引グラフから特徴を自動的に学習し、複雑なFeature Engineeringを行わずに不正確率を予測する。
- **クラウド活用:** AWS Kinesisなどによるリアルタイムデータストリーミングと、AWS SageMakerなどによるスケーラブルな学習・推論環境を想定。
- **結果:** クレジットカード取引データでSVM、RF、XGBoost、DNN、STAN、GCN、GATと比較し、TGTNが各月のAP・AUCで最も高い値を示した。

---

## [タスク概要]

**タスク:**  
クレジットカード取引データから、各取引が**不正取引かどうか**を予測する。

**入力:**

- クレジットカード取引の元データ
- 取引属性（merchant number、card number、transaction amountなど）
- 取引間の関係から構築したTransaction Graph

**出力:**  
各取引の不正確率。

---

## [目的]

### 背景・課題

モバイル決済、デジタルウォレット、Webベースの決済などの普及により、大量の決済データの中から少数の不正取引を正確に検出することが重要になっている。

従来の不正検知手法には、異常検知・分類・クラスタリングなどがあるが、論文では複雑なパターンや非線形な関係を扱う上での限界を指摘している。

一方、金融データには以下のような特徴がある。

- 長期的な依存関係が存在する
- 時系列上のパターンを持つ
- 複雑な取引関係が存在する
- 取引量が大きく、リアルタイム処理が求められる

### 本研究の目的

**Transaction GraphとGraph Attention Transformerを組み合わせ、取引のトポロジーと時間的パターンを利用して、高精度なクレジットカード不正検知を実現すること。**

---

## [手法]

### 全体像

提案手法 **TGTN (Transaction Graph Transformer Network)** は、主に以下の流れで不正取引を予測する。

```text
クレジットカード取引データ
        ↓
Transaction Graph構築
        ↓
Graph Attention Transformer
        ↓
トポロジー・時間的特徴の学習
        ↓
Fraud Prediction Network
        ↓
Fraud Probability
```

TGTNでは、元のクレジットカード取引データを複雑なTransaction Graphに変換し、Graph Attention Transformerによって取引間の関係と時系列的な特徴を学習する。

### Step 1. Transaction Graphを構築

元のクレジットカード取引レコードからTransaction Graphを構築する。

各ノードは取引に関係するEntityを表し、論文ではユーザーや加盟店などが例として挙げられている。

また、取引関係を表すEdgeを構築し、取引データに存在する潜在的な関係を抽出する。

```text
取引データ
   ↓
Graph Construction
   ↓
Nodes + Edges
   ↓
Transaction Graph
```

取引グラフ上では、通常・疑わしい・高リスクなど異なる種類の取引関係も区別して表現できる。

### Step 2. Transaction Nodeに取引属性を埋め込む

Transaction Graphの各ノードには、元の取引レコードに含まれる基本的な取引属性を持たせる。

論文では以下のような属性が例として挙げられている。

- Merchant Number
- Card Number
- Transaction Amount

これらの属性をAttribute Matrixとして表現し、Graph Transformerへの入力とする。

```text
Merchant Number
Card Number
Transaction Amount
        ↓
Transaction Node
        ↓
Attribute Matrix X
```

### Step 3. Graph Attention Transformerで特徴を学習

Transaction Graphに対してGraph Attention Transformerを適用する。

各ノードについて、

1. Neighborの情報を収集
2. Neighborの情報をAggregation
3. 集約した情報を用いてNode Featureを更新

という処理を行う。

```text
Neighbor Nodes
      ↓
Collection
      ↓
Aggregation
      ↓
Update
      ↓
Updated Node Representation
```

Attention機構によって、ノード間の関係性を考慮しながら取引の特徴表現を学習する。

さらに、取引のTiming InformationをNode Representationに組み込むことで、トポロジーだけでなく時間的な特徴も扱う。

### Step 4. 時系列パターンを学習

TransformerのSelf-Attentionは、入力系列内の要素間の関係を計算することで、離れた位置にある要素間の依存関係も捉えることができる。

金融時系列データでは、

- Non-stationary
- Long-term dependencies
- Cyclical / Seasonal patterns

などが存在するため、Transformerによって長期的な依存関係を捉えることを狙っている。

Transformer自体には再帰構造や畳み込みがないため、時系列の順序情報を扱うためにPositional Encodingを利用する。

### Step 5. Fraud Prediction

Graph Attention Transformerによって学習された取引のトポロジー・時間的特徴を、Fraud Prediction Networkに入力する。

```text
Transaction Graph
        ↓
Graph Attention Transformer
        ↓
Topological Features
        +
Temporal Features
        ↓
Fraud Prediction Network
        ↓
Fraud Probability
```

論文では、元の取引データに対して複雑なFeature Engineeringを行わず、Transaction Graphから特徴を自動的に学習し、End-to-Endで不正確率を出力することを特徴としている。

---

## [クラウド・リアルタイム処理]

### Real-Time Data Ingestion

金融取引では、取引が発生した時点に近いタイミングで不正を検出する必要がある。

論文では、AWS KinesisやAzure Stream Analyticsなどのストリーミング基盤を利用して、金融取引データをリアルタイムに取得・処理する構成を想定している。

```text
Financial Transaction
        ↓
Streaming Platform
        ↓
Transformer Model
        ↓
Fraud Detection
```

論文では、AWS Kinesisが大量のEvent Dataを処理でき、決済ネットワークや銀行取引データをリアルタイムに処理できる例を挙げている。

### Scalability / Elasticity

クラウドのElastic Scalabilityを利用することで、取引量の変動に応じて計算リソースを調整できる。

また、AWS SageMakerによる複数GPU・TPUインスタンスを利用した分散学習を例として挙げ、大規模金融データに対するTransformerの学習を高速化する構成を説明している。

---

## [実験]

### データセット

実験には、ある期間のクレジットカード取引データを使用している。

- 期間：2月1日〜9月30日
- 総取引数：141,861
- Fraud：33,858

学習・テストは時間で分割している。

```text
2/1 ───────── 6/30 | 7/1 ───────── 9/30
        Training    |        Test
```

学習データは2月1日〜6月30日、テストデータは7月1日〜9月30日としている。

学習時には5-fold Cross Validationによってパラメータを選択・最適化し、その後、7月・8月・9月について月単位でテストデータを評価している。

### 評価指標

以下の2つの指標を使用する。

- AP (Average Precision)
- AUC (Area Under the ROC Curve)

**AP:** 各ThresholdにおけるPrecisionを利用して評価する。

**AUC:** Fraud / Non-Fraudを識別するモデルの全体的な能力を評価する。

### 比較手法

以下の7つのBaseline Modelと比較している。

- SVM
- RF
- XGBoost
- DNN
- STAN
- GCN
- GAT

SVM、RF、XGBoost、DNN、STANについては、RFM Feature Engineeringによって90個のFinancial Business Featuresを手動で構築している。

一方、GCN、GAT、TGTNなどのGraph Neural Network系モデルでは、Transaction Graphから特徴を自動的に学習する。

---

## [結果]

### 定量結果

各月のAP・AUCは以下の通り。

| Model | July AP | July AUC | August AP | August AUC | September AP | September AUC |
|---|---:|---:|---:|---:|---:|---:|
| SVM | 0.1203 | 0.6599 | 0.1582 | 0.6467 | 0.1172 | 0.6842 |
| RF | 0.1547 | 0.7324 | 0.1658 | 0.6841 | 0.1258 | 0.6892 |
| XGBoost | 0.2074 | 0.8894 | 0.2099 | 0.8216 | 0.2570 | 0.8671 |
| DNN | 0.2512 | 0.8948 | 0.3483 | 0.9113 | 0.2509 | 0.8919 |
| STAN | 0.3021 | 0.9041 | 0.3963 | 0.9051 | 0.3315 | 0.9213 |
| GCN | 0.3473 | 0.9006 | 0.4293 | 0.8981 | 0.3275 | 0.8870 |
| GAT | 0.3884 | 0.9247 | 0.4275 | 0.9190 | 0.3985 | 0.9162 |
| **TGTN** | **0.4637** | **0.9471** | **0.5261** | **0.9446** | **0.4732** | **0.9429** |

### TGTNの性能

TGTNは7つのBaseline Modelとの比較において、7月・8月・9月のすべてでAPとAUCが最も高い値となっている。

GATとの比較では、論文のAbstractにおいて、平均APが20%、平均AUCが2.7%向上したと報告している。

また、本文ではBaseline Model全体との比較について、TGTNがAPで約6%、AUCで約4%改善したと説明している。

### Graph Modelとの比較

Graph-based Modelでは、GCN、GAT、TGTNを比較している。

```text
GCN
 ↓
GAT
 ↓
TGTN
```

TGTNでは、Graph Attentionに加えてTransformerを利用し、Transaction Graphのトポロジーと時間的な特徴を組み合わせている。

---

## [考察]

### 1. Transaction Graphによって複雑な取引関係を利用できる

通常の表形式の取引データだけでは、取引間に存在する複雑な関係を直接扱いにくい。

TGTNではTransaction Graphを構築することで、

```text
Transaction
     ↓
Graph
     ↓
Node / Edge Relationship
     ↓
Graph Representation
```

という形で取引間の関係をモデルに取り込む。

### 2. Feature Engineeringへの依存を減らせる

SVM、RF、XGBoost、DNN、STANでは90個のFinancial Business FeaturesをRFM Feature Engineeringによって構築している。

一方、TGTNでは元のTransaction RecordからTransaction Graphを構築し、Graph Attention Transformerによって特徴を自動的に学習する。

そのため、論文では複雑なFeature Engineeringを行わずにTransaction Graphから潜在的なFraud Featureを抽出できることを特徴としている。

### 3. トポロジーと時間的パターンを同時に扱う

TGTNはGraph構造だけではなく、Transformerによって取引の時間的なパターンも扱う。

```text
Transaction Graph
       +
Temporal Pattern
       ↓
Graph Attention Transformer
       ↓
Fraud Prediction
```

これにより、取引間の関係と時系列的な行動パターンを組み合わせた不正検知を行う。

### 4. クラウド環境との組み合わせ

論文では、TransformerによるFraud Detectionをクラウド上のリアルタイム処理基盤と組み合わせる構成を説明している。

```text
Real-Time Transactions
        ↓
Cloud Streaming
        ↓
Transaction Graph / Transformer
        ↓
Fraud Detection
```

大量の取引データを扱う場合に、クラウドのScalabilityとElasticityを利用することが想定されている。

---

## [Limitations]

- 実験データについて、本文では「ある年」の2月1日〜9月30日のクレジットカード取引データと説明されており、データセットの詳細な出所は本文からは明確ではない。
- Transaction Graphの構築方法やグラフ構造がモデル性能に影響する可能性がある。
- リアルタイム処理についてはAWS Kinesisなどを利用した構成が説明されているが、論文の実験結果として実際のEnd-to-Endレイテンシやスループットを詳細に評価しているわけではない。
- クラウド環境でのスケーラビリティについては具体的な実運用規模での検証よりも、クラウドサービスを利用した構成・可能性が中心に説明されている。

---

## [関連リンク]

- [Paper](https://doi.org/10.1145/3724154.3724271)
- [PDF](https://doi.org/10.1145/3724154.3724271)
