# Credit Card Fraud Detection Using Advanced Transformer Model

**Authors:** Chang Yu, Yongshun Xu, Jin Cao, Ye Zhang, Yixin Jin, Mengran Zhu  
**Year:** 2024  
**Venue:** arXiv preprint, arXiv:2406.03733v4  
**Paper:** https://arxiv.org/abs/2406.03733  
**PDF:** https://arxiv.org/pdf/2406.03733v4.pdf  
**Code:** 記載なし

---

## [まとめ]

> **一言でいうと:**  
> クレジットカード取引データを前処理・サンプリングしたうえで、**通常のTransformer EncoderのSelf-Attentionを使った分類モデル**を構築し、Logistic Regression、KNN、SVM、Decision Tree、Neural Network、XGBoost、TabNetと比較した研究。

- **何をしたか:** 欧州のクレジットカード取引データを対象に、データのクラス不均衡への対応、相関分析、外れ値処理、次元削減を行い、その後Transformerを分類器として適用した。
- **入力:** 表形式のクレジットカード取引データ。論文ではV1〜V28の特徴量が使われ、それらは処理済み・float型に変換されている。各V1〜V28が具体的に何を意味するかは論文中では説明されていない。
- **分類単位:** 実験上は、各クレジットカード取引についてfraud / non-fraudを判定する取引単位の二値分類と理解できる。ただし、「1取引＝Transformerの1 token」とするか、「1取引内のV1〜V28をsequenceとして扱う」かなど、Transformerへの具体的な入力tensorの構成は論文に明記されていない。
- **Transformerの役割:** Self-Attentionにより入力sequence内の依存関係を学習し、Feed-Forward Networkと組み合わせて分類を行う。
- **顧客ごとの取引sequence:** **論文では確認できない。** 顧客ごとに過去取引を時系列に並べてTransformerへ入力したとは記載されていない。
- **グラフ構築:** **行っていない。** Transaction Graph / GNN / Graph Attentionは使わず、表形式データをTransformerへ入力する。
- **不均衡対応:** fraud / non-fraud が同数になるようにランダムサンプリングしてbalanced datasetを作成し、シャッフルした。
- **結果:** 2023年データではTransformerがPrecision 0.998、Recall 0.998、F1 0.998、ROC AUC 0.99を達成し、比較モデルを上回った。
- **追加検証:** 2013年データでもTransformerはPrecision 0.998、Recall 0.998、F1 0.998、ROC AUC 0.98を達成し、異なる年代のデータでも高い性能を示した。

---

## [タスク概要]

### タスク

クレジットカード取引が**fraudulent（不正）か non-fraudulent（正常）かを分類する二値分類タスク**。

### データの特徴

不正取引は全取引の中で非常に少なく、クラス不均衡が大きな課題となる。

論文で説明されているデータでは、284,804件の取引のうち492件がfraudで、fraud率は0.172%だった。

---

## [目的]

### 背景・課題

クレジットカード取引では不正取引が非常に少ないため、通常の機械学習モデルでは不正クラスを十分に捉えにくい。

また、従来の機械学習手法は、複雑な高次元データや長距離依存関係の扱いに課題を持つ場合がある。

### 本研究の目的

TransformerのSelf-Attentionを利用して、

- 複雑な特徴間の関係
- 長距離依存関係
- 非線形なパターン

を学習し、従来モデルより高性能なfraud detectionを実現できるかを検証する。

---

## [手法]

### 全体像

```text
クレジットカード取引データ
        ↓
クラス分布確認・サンプリング
        ↓
特徴量間の相関分析
        ↓
外れ値検出・除去
        ↓
次元削減の比較
(T-SNE / PCA / Truncated SVD)
        ↓
Transformer Encoder
        ↓
Self-Attention
        ↓
Feed-Forward Network
        ↓
分類
        ↓
Fraud / Non-Fraud
```

### Step 1: データセット

欧州のクレジットカード取引データを利用。

論文では、2023年までのデータを使用した実験について「55万件超の取引」と説明する一方、実験説明では284,804件の取引と492件のfraudを扱っている。

特徴量はV1〜V28で構成され、float型へ変換されている。

また、2013年の欧州payment dataも追加し、別時期のデータによる比較・検証を行っている。

### Step 2: クラス不均衡への対応

fraudが極端に少ないため、ランダムサンプリングとマージによって、

```text
Fraud      → 同数
Non-Fraud  → 同数
```

となるbalanced datasetを作成する。

その後、データセット全体をシャッフルして、サンプル順序による偏りを避ける。

### Step 3: 相関分析

元のimbalanced datasetと、サブサンプリング後のbalanced datasetについて特徴量間の相関を比較する。

論文では、データをbalancedにすることで、

- model performanceの向上
- overfittingへの感受性の低下
- predictive stabilityの向上
- interpretabilityの向上

が得られたと説明している。

### Step 4: 外れ値処理

V14、V12、V10について外れ値を分析する。

まず分布を可視化した後、IQR（Interquartile Range）法によって外れ値を判定する。

$$
IQR = Q3 - Q1
$$

外れ値の範囲は、

$$
Lower = Q1 - 1.5 \times IQR
$$

$$
Upper = Q3 + 1.5 \times IQR
$$

として定義し、この範囲を超える値を外れ値として除去する。

その後、box plotなどを用いて、処理前後の分布を確認している。

### Step 5: 次元削減の比較

3種類の次元削減手法を比較する。

#### T-SNE

非線形な構造や局所的な関係を保持しながら低次元空間へ写像する手法。

主にデータ分布やクラスタリング傾向の可視化に利用する。

#### PCA

主成分分析によって、データの分散をできるだけ保持しながら低次元化する。

#### Truncated SVD

行列を分解して低次元表現を得る手法。

PCAと異なり、事前にデータを中心化しない点が説明されている。

論文では、これらを2次元へ写像して比較し、データ構造を可視化している。

---

### Step 6: Transformer Encoder

本研究ではTransformerのEncoderを利用する。

主要な構成要素は、

```text
Input Sequence X
      ↓
Multi-Head Self-Attention
      ↓
Feed-Forward Network
      ↓
Residual Connection
      ↓
Layer Normalization
```

である。

論文では、Transformerの入力を一般的なsequenceとして

\[
X = \{x_1,x_2,\ldots,x_n\}
\]

と定義している。

### Step 7: Transformerへの入力形成

ここは論文中で詳細な実装方法が説明されていない。

論文から確認できるのは、

1. データにV1〜V28の特徴量が存在する
2. それらは処理済みでfloat型に変換されている
3. Transformerでは入力をsequence `X` として扱う
4. `X` からQuery / Key / Valueを計算する

という点である。

したがって、概念的には、

```text
1件のクレジットカード取引
[V1, V2, ..., V28]
        ↓
  前処理済み特徴量
        ↓
    Input X
        ↓
Transformer Encoder
```

と考えられる。

ただし、以下は論文からは特定できない。

- V1〜V28の各特徴量をsequenceの各positionとして扱ったのか
- 1取引全体を1つのtokenとして扱ったのか
- 複数の取引を1つのsequenceとしてまとめたのか
- 顧客ごとの過去取引を時系列sequenceとして入力したのか
- Transformer入力前にどのようなembedding / linear projectionを行ったのか
- input tensorの具体的なshape
- positional encodingを実際のfraud detectionモデルで使用したか

特に、論文には**「顧客ごとに過去取引を並べてTransformerへ入力した」という記述はない**。

したがって、本研究を「顧客ごとの取引sequenceをTransformerに入力する時系列モデル」と断定することはできない。

### Step 8: Self-Attention

入力sequence `X` からQuery、Key、Valueを計算する。

\[
Q = XW^Q
\]

\[
K = XW^K
\]

\[
V = XW^V
\]

Attention weightは、

\[
A = softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)
\]

で計算し、その後、

\[
Attention(X)=AV
\]

として出力を得る。

これによって、input sequence中の各positionが他のpositionをどの程度参照すべきかを学習する。

### Step 9: Transformer Encoder Layer

Self-Attentionの出力をFeed-Forward Networkへ渡し、Residual ConnectionとLayer Normalizationを利用する。

論文では概ね、

\[
EncoderLayer(X)
=
LayerNorm(X + Attention(X))
+
FeedForward(X)
\]

と表現されている。

### Step 10: Fraud Classification

最終的には各クレジットカード取引について、fraud / non-fraudを分類する。

実験では、284,804件の取引のうち492件がfraudとして扱われ、Precision、Recall、F1-score、ROC AUCによって評価している。

したがって、**タスクとしては取引単位の二値分類**と理解するのが妥当である。

ただし、1取引のV1〜V28を具体的にどのようなtensor shape・token構造へ変換してTransformerへ入力したかは、論文には明記されていない。

本研究では、Transaction GraphやGraph Attentionなどのグラフ構造は用いず、表形式の取引特徴量をTransformerで処理している。

---

## [実験]

### データセット

| 項目 | 内容 |
|---|---|
| データ | European credit card transaction data |
| 主な特徴量 | V1〜V28 |
| 取引件数として記載 | 284,804 |
| Fraud件数 | 492 |
| Fraud率 | 0.172% |
| 追加検証 | 2013年データ |
| 主実験 | 2023年データ |

### 前処理

| 処理 | 内容 |
|---|---|
| クラスバランス | Fraud / Non-Fraudを同数化 |
| シャッフル | Balanced datasetをランダムにshuffle |
| 相関分析 | Original / Sub-sampledを比較 |
| 外れ値処理 | IQR法 |
| 対象特徴量 | V14, V12, V10 |
| 次元削減 | T-SNE / PCA / Truncated SVD |

### 評価指標

- Precision
- Recall
- F1-score
- ROC AUC

### 比較手法

- Logistic Regression
- KNN
- SVM / SVC
- Decision Tree
- Neural Network
- XGBoost
- TabNet
- Transformer

---

## [結果]

### 定量結果：2023年データ

| Model | Precision | Recall | F1 Score | ROC AUC |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.93 | 0.93 | 0.93 | 0.98 |
| KNN | 0.93 | 0.93 | 0.93 | 0.98 |
| SVM | 0.91 | 0.91 | 0.91 | 0.99 |
| Decision Tree | 0.93 | 0.93 | 0.93 | 0.93 |
| Neural Network | 0.92 | 0.91 | 0.91 | 0.96 |
| XGBoost | 0.95 | 0.95 | 0.95 | 0.99 |
| TabNet | 0.93 | 0.93 | 0.93 | 0.98 |
| **Transformer** | **0.998** | **0.998** | **0.998** | **0.99** |

### 定量結果：2013年データ

| Model | Precision | Recall | F1 Score | ROC AUC |
|---|---:|---:|---:|---:|
| KNN | 0.93 | 0.92 | 0.92 | 0.95 |
| SVM | 0.93 | 0.93 | 0.93 | 0.95 |
| Decision Tree | 0.91 | 0.90 | 0.89 | 0.85 |
| Logistic Regression | 0.96 | 0.96 | 0.96 | 0.95 |
| Neural Network | 0.975 | 0.999 | 0.988 | 0.85 |
| XGBoost | 0.96 | 0.90 | 0.92 | 0.96 |
| TabNet | 0.59 | 0.77 | 0.67 | 0.48 |
| **Transformer** | **0.998** | **0.998** | **0.998** | **0.98** |

### 結果のポイント

- 2023年データではTransformerがPrecision / Recall / F1で0.998を達成。
- 2013年データでもPrecision / Recall / F1が0.998となり、異なる時期のデータに対しても高い性能を示した。
- ROC AUCについては、2023年が0.99、2013年が0.98。
- 論文では、TransformerのSelf-Attentionとpretraining-fine-tuning paradigmが高性能と安定性に寄与したと説明している。

---

## [考察]

### 1. Transformerによる特徴間の依存関係の学習

Self-Attentionにより、入力系列中の異なる位置同士の依存関係を直接モデル化できる点を利点としている。

### 2. 不均衡データへの対応

Fraudが非常に少ないという問題に対して、balanced datasetを作成してからモデルを学習する構成を採用している。

### 3. 異なる年代への一般化

2023年データだけでなく2013年データでも評価を行うことで、時間の異なるデータに対する性能の安定性を検証している。

### 4. 従来モデルとの比較

XGBoostやTabNetなどの代表的なモデルと比較し、Transformerが主要指標で高い性能を示したことを報告している。

---

## [Limitations]

論文で明確に制約として列挙されているわけではないため、以下は**論文の記述から確認できる範囲での注意点**。

- **Transformer入力の形成方法が十分に記載されていない。** V1〜V28をどのようにsequence / tokenへ変換したか、顧客単位の取引履歴sequenceを使ったか、具体的なinput tensor shapeが何かは論文から確認できない。(おそらく単一取引情報を入力したと考えられる)
- **Pre-trainingの実装詳細が十分ではない。** まとめではpretraining-fine-tuning paradigmに言及しているが、この論文内での具体的なpretraining手順は十分には説明されていない。

---

## [関連リンク]

- Paper: https://arxiv.org/abs/2406.03733
- PDF: https://arxiv.org/pdf/2406.03733v4.pdf
