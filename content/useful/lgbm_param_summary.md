---
title: "LightGBM(gbdt)のパラメータ,Tuningの個人的まとめ"
date: 2020-09-26T14:19:41+09:00
draft: false
tags: ["データサイエンス","まとめ記事"]
categories: [ "技術" ]
---

初手LightGBMは機械学習系だと割とやると思うんですが、いざobjectiveとかパラメータTuningをするたびにドキュメントを読むことになっているので、まとめようと思いました。

基本は[ドキュメント](https://lightgbm.readthedocs.io/en/latest/Parameters.html)を抜粋した日本語訳に近くなると思います。
<!--more-->

# Objective
よく使うのだけにします。

+ "regression" : mean_squared_errorを誤差関数として使う回帰です。
+ "regression_l1" : mean_absolute_errorを誤差関数として使う回帰です。
+ "binary" : 二値分類のloglossです。
+ "multiclass" : 多クラス分類、softmaxです。"num_class"でクラス数も一緒に与えて上げる必要があります。
+ "multiclassova" : 多クラス分類をOneVsAllでときます。同様に"num_class"でクラス数も一緒に与えて上げる必要があります。
+ "cross_entropy" : クロスエントロピーです。

# 学習に関わるパラメータ
+ "num_iterations" : 学習のイテレーション回数を指定します。defalut 100です。
+ "learning_rate" : learning_rateです。default 0.1です。
+ "num_leaves" : **木の複雑度をコントロールするメインパラメータです**分岐の終着点の数を決めます。default 31です。
+ "seed" : random seedです。**これを決めれば色々ある他のseedがすべて決まります** default Noneなのでか必ず設定しましょう。
+ "max_depth" : 木の最大深さを決めます。defaultの-1は上限無しなので、ここも必ず設定したほうがいいと思います。
+ "min_data_in_leaf" : 葉に行き着くデータがこれより少なくならない用になります。default 20です。
+ "bagging_fraction" : baggingで選択されるサンプルの割合です。default 1.0で、baggingは無効化されています。また、**baggingするには"bagging_freq"も正の値にしなくてはなりません。**
+ "bagging_freq" : 何回に一回baggingするかです。こちらだけセットしてもやはりbaggingは出来ず、**baggingするには"bagging_fraction"が1未満である必要があります。**
+ "feature_fraction" : 1.0未満の値にすると、特徴量の一部を削減して学習を行う用になります。default 1.0です。
+ "early_stopping_round" : early stoppingのpatienceです。するときは設定しましょう。
+ "lambda_l1" : L1正則化です。
+ "lambda_l2" : L2正則化です。
+ "verbosity" : < 0: Fatal, = 0: Error (Warning), = 1: Info, > 1: Debug　だそうです。default 1です。

# parameter Tuning

おおよその探索範囲表です(まずはこのあたりをGridSearchする)
**個人的な体感によるものですので、当然ベストではないですし、betterですらない可能性があることをご承知ください**

個人的優先度順になってるので、計算時間が無いときはこの上からN個だけとかで探索します。

| パラメータ名 | 探索範囲 | 備考 |
| ---- | ---- | ---- |
| num_leaves | 7,15,31 | 大ほどOverFit, 2^(mad_depth)* 0.7程度に決め打ちもあり。 |
| max_depth | 5,7,9 | 大ほどOverFit |
| min_data_in_leaf | 20,30,50 | 小ほどOverFit |
| bagging_fraction | 0.8,0.9 | 大ほどOverFit? |
| bagging_freq | 1,3 | 小ほどOverFit? |
| feature_fraction | 0.9,1.0  | 大ほどOverFit? |
| lambda_l1 | 0,2,5 | 小ほどOverFit。正直どのくらいがいいのか全くわからないので、下手に弄らないことも多し。|
| lambda_l2 | 0,2,5 | 小ほどOverFit。正直どのくらいがいいのか全くわからないので、下手に弄らないことも多し。 |

bagging_fractionや,feature_fractionを設定しないとrandom_stateを変えても結果が変わらなくなるので、パラメータTuningの時間が無い時+ensemble(or random seed average)する前提のときはこのあたりはtuning出来なくてもに0.8〜0.9にとりあえずしといた方が、混ぜた結果は強くなるのではという気がします。

その他learning_rateを下げてiterationを伸ばすと当然良くなると思われますが、自明なのでやるにしても最後です。