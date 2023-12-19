# atmaCup #16 6th Place Solution

[「atmaCup #16 in collaboration with RECRUIT」](https://www.guruguru.science/competitions/22/)の 6th place solution です。

## コード

- [atmacup_16_maruyama.ipynb](./atmacup_16_maruyama.ipynb)

このコードの予測精度は以下の通りです。

- CV: 0.3898
- Public LB: 0.4475 (6位)
- Private LB: 0.4439 (6位)

---

以下、[ディスカッション](https://www.guruguru.science/competitions/22/discussions/a922d9e0-dd5f-4d82-9e82-5e1afcc35a36/)へ投稿した内容を転載します。

## 概要

### 方針

- 「この宿をみた人は他にこんな宿をみています」方式でレコメンドすることにしました。
- 理由は……
    - セッション情報が乏しいため、宿情報を使ったコンテンツベースのレコメンドはあまり機能しないと思われたから。
    - じゃらんには既に「この宿をみた人は他にこんな宿をみています」方式のレコメンド機能が実装されているため、提供されているログデータも左記のレコメンド機能によってバイアスがかかっているはずだから。

### モデル

- Matrix factorizationによる協調フィルタリング
- ["Revisiting the Performance of iALS on Item Recommendation Benchmarks"](https://dl.acm.org/doi/10.1145/3523227.3548486) で提案されている改良版のiALSを使いました。
- 協調フィルタリングの結果をリランキングする方法も試しましたが、CVもLBも悪化したためやめました。

### データ

- 以下のデータを結合して、matrix factorizationにかけました。
    - train_log.csv
    - train_label.csv
    - test_log.csv (※)
    - test_log.csv のうち宿が2件以上あるセッション x9 (※)
- ※ test_log.csv を10倍に増やしているのは、学習データ-テストデータ間のドメインシフト対策です。テストデータの割合を増やすことで、データ全体の分布をテストデータに近づけています。ドメインシフトの存在に関しては[「訓練データとテストデータの差異について」](https://www.guruguru.science/competitions/22/discussions/9676ddb4-e07b-4b31-8315-da87fe99cf3b/)などで指摘されています。

### 後処理

- 各セッションのログの最後から2番目の宿をランキング1位で推薦するよう、後処理で推薦結果を修正しています。これは、matrix factorizationは宿の出現順序を加味できないためです。
- 後処理の実装にあたり[「train_logにおいてsessionの最後のseq_noと途中のseq_noの差分を取ったときに奇数番目の宿が選ばれやすい」](https://www.guruguru.science/competitions/22/discussions/b7abc605-9025-4a64-911e-2c760523db09/)を参考にしました。

### 備考

- Matrix factorizationにかけるデータを変えると、スコアは以下のようになります (括弧外はpublic LB、括弧内はprivate LB)。
    - テストログのみ：0.4300 (0.4266)
    - 学習ログ＋教師ラベル＋テストログ：0.4410 (0.4379)
    - 学習ログ＋教師ラベル＋テストログ10倍：0.4475 (0.4439)
- 今回のコンペでは使いませんでしたが、matrix factorizationのモデル内部ではセッションベクトル・宿ベクトルを作っていますので、協調フィルタリングではなくセッションや宿の分散表現を得る手法としても使えると思います。
