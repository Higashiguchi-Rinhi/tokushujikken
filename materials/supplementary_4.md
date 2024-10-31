第4回 補足資料
================
Higashiguchi Rinhi
2024-10-31

# 質問・コメント

<div class="callout-box">

## 推定方法によってパラメータが大きく異なることがあるのか

</div>

ここでは一般的に使用される最尤推定法及びベイズ推定法の推定値の数値シミュレーションの結果を比較します。

今回取り扱る仮想的なデータでは
患者50人の血圧値を測定し、そのデータを基に病院が患者全体の血圧の平均値とばらつき（分散）を推定したいとします。

数値シミュレーションというのでデータを生成する所から行います。

### データ生成(50人の血圧データ)

(実際はわからないですが、)真の血圧平均120 mmHg、真の標準偏差 50
mmHgとなる正規分布から50人のデータをサンプリングしたと考える

``` r
set.seed(4747) #再現可能性を担保するためシードを決める
true_mean <- 120  # 真の血圧平均
true_sd <- 15     # 真の標準偏差
n <- 50           # サンプルサイズ
data <- rnorm(n, mean = true_mean, sd = true_sd)  # サンプリング

hist(data, breaks = 10, probability = TRUE, col = "lightblue", main = "Simulated Data")
```

![](supplementary_4_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

**推定方法の違い**

- MLEでは、集めたサンプル（50人の血圧値）のみを元に、血圧の「平均」と「分散」を推定します。

- ベイズ推定では、例えば「健康な成人の血圧の平均は120
  mmHg付近、標準偏差は15
  mmHg付近」という**事前情報**(後述)を加味します。これにより、サンプルが少なくても事前分布の情報を反映し、推定の安定性が増します。

### MLEを用いた推定

正規分布の平均および分散を、MLEによる点推定を行う場合、
母平均は**標本平均**と同じ値、母分散は**標本分散**と同じ値を取ることが尤度方程式を解くことでわかる。

導出を追いたい場合は[こちら](https://stats.biopapyrus.jp/stats/estimation.html)を参照されたい。

``` r
mle_mean <- mean(data)                       # 平均を推定
mle_sd <- sqrt(sum((data - mle_mean)^2) / n) # 標準偏差を推定
```

    ## === 真のパラメータ ===

    ## 平均: 120 標準偏差: 15

    ## === 最尤推定 (MLE) の結果 ===

    ## 推定平均: 120.9362 推定標準偏差: 12.14296

MLEを用いる際新しい患者のデータが増えると推定精度が高まりますが、サンプルが少ないと結果が不安定になる可能性があります。これは、サンプルサイズが小さい場合MLEを使用することが望ましくないことを示唆します。

また、第二回の回帰分析でもあったようにMLEは推定値が**正規分布**であることを仮定するので、正規分布にならないパラメータの信頼区間を正確に推定できないです。無理に正規分布を当てはめると相関係数が1以上になったり、分散が負になるといった**不敵解**が生じやすい。

### ベイズ推定を用いた推定

ベイズ推定では**真値(パラメータ)を確率分布**として考えます。

また、データを確率的なもの(第二回では回帰分析の従属変数$Y$は正規分布に従うという仮定をおいて)と考えず、あくまで情報でそれを基に真値の分布を**更新**している発想です。

事後分布は簡単に言えば以下のようになります
`事後分布　=　（事前分布　×　尤度） / データの分布`

今回は、「健康な成人の血圧の平均は120mmHg付近、標準偏差は15mmHg付近」という事前分布(事後情報)で分析してみましょう。

``` r
prior_mean <- 120  # 健康な成人の平均血圧（事前情報）
prior_sd <- 15     # 健康な成人の血圧のばらつき（事前情報）

# ベイズ推定（事後分布の解析解）
posterior_mean <- (sum(data) + (prior_mean / prior_sd^2)) / (n + 1 / prior_sd^2)
posterior_sd <- sqrt(1 / (n / true_sd^2 + 1 / prior_sd^2))
```

    ## === 真のパラメータ ===

    ## 平均: 120 標準偏差: 15

    ## === ベイズ推定の結果 ===

    ## 推定平均: 120.9362 推定標準偏差: 2.10042

両方の推定法の違いを可視化する:

``` r
hist(data, breaks = 10, probability = TRUE, col = "lightblue", main = "MLE vs Bayesian Estimation", xlim = c(80, 180), ylim = c(0, 0.2))
curve(dnorm(x, mean = mle_mean, sd = mle_sd), col = "red", lwd = 2, add = TRUE)
curve(dnorm(x, mean = posterior_mean, sd = posterior_sd), col = "blue", lwd = 2, add = TRUE)
legend("topright", legend = c("MLE", "Bayesian"), col = c("red", "blue"), lwd = 2)
```

![](supplementary_4_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

ベイズ推定において**事前分布**を真の分布と全然違う「健康な成人の血圧の平均は1mmHg付近、標準偏差は1mmHg付近」とした場合どうなるでしょう。

``` r
prior_mean <- 1  # 健康な成人の平均血圧（事前情報）
prior_sd <- 1  # 健康な成人の血圧のばらつき（事前情報）

# ベイズ推定（事後分布の解析解）
posterior_mean <- (sum(data) + (prior_mean / prior_sd^2)) / (n + 1 / prior_sd^2)
posterior_sd <- sqrt(1 / (n / true_sd^2 + 1 / prior_sd^2))
```

    ## === 真のパラメータ ===

    ## 平均: 120 標準偏差: 15

    ## === 新しいベイズ推定の結果 ===

    ## 推定平均: 118.5846 推定標準偏差: 0.904534

``` r
hist(data, breaks = 10, probability = TRUE, col = "lightblue", main = "MLE vs Bayesian Estimation", xlim = c(80, 150), ylim = c(0, 0.2))
curve(dnorm(x, mean = mle_mean, sd = mle_sd), col = "red", lwd = 2, add = TRUE)
curve(dnorm(x, mean = posterior_mean, sd = posterior_sd), col = "blue", lwd = 2, add = TRUE)
legend("topright", legend = c("MLE", "Bayesian"), col = c("red", "blue"), lwd = 2)
```

![](supplementary_4_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

なんかやってみたらデータのおかげで、割と真値に近い推定値になってこまったのですが(良い例ではない)、
このように、ベイズ推定では事前分布の設定が分析者にゆだねられる点と、設定する事前分布について未だ議論がある点が欠点として挙げられます。
また、推定時間がMLEより長いのも難点です。計算のために特殊なソフトを使用する必要があるのでそれもハードルだと聞きます。

参考までですが、私が使っているのは [Stan](https://mc-stan.org/)
(C++を学ぶ必要がある)及び [JAGS](https://mcmc-jags.sourceforge.io/)
ですが最近は頻度論的な分析も、ベイズ推定もやりやすい[JASP](https://jasp-stats.org/)が注目されている気がします。

まだベイズ推定・統計については質問があるので来週続きをやります。

参考文献: <https://norimune.net/708>

<div class="callout-box">

## パラメータを推定する方法がなぜ複数あってどう使い分けるのか

</div>

パラメータ推定法が複数明確な理由はわからないのですが(きっと誰かが増やそうといって増えたものではなく、パラメータ推定について最適な方法を色んな人が考えた結果だと思うのですが)、
下記の特徴を踏まえて推定法を使い分けるのが分析者として良いのかなと思います。

1.データが豊富な場合はMLE、少ない場合はベイズ推定

→事前分布を上手く設定すればサンプルサイズが小さい場合でも妥当な推定が可能

→データを次々足していけば推定精度が上がる

2.事前情報があればベイズ推定を用いることで過去の経験(事前情報)を反映した推定が可能になる

→その現象・分析に対して事前情報があるとは限らない

3.複雑なモデルはMLEの方が推奨される。ベイズ推定では事前分布を設定したり、計算自体に時間がかかる場合がある。

# 演習

<div class="callout-box">

psychパッケージに含まれるbfiというデータセットを解析し、因子数を特定し、解を推定、推定された解を回転し、因子を解釈してみましょう

1.BICを基準に因子数を決定しましょう

2.回転を行うことで解釈がしやすくなることを確認してみましょう

</div>

## データセット

**bfiとは** 国際パーソナリティ項目プール（International Personality Item
Pool
ipip.ori.org）で提供されている、パーソナリティの5因子モデル（IPIP-NEO）についての質問紙への回答（2800人分）のデータです。
28列のうち、最初の25列が質問文への回答(6件法)、最後の3列はそれぞれ

- 性別（1=男, 2=女）

- 学歴（1=高校入学, 2=高校卒業, 3=大学入学, 4=大学卒業, 5=大学院卒業）

- 年齢

参考: <https://www.personality-project.org/r/html/bfi.html>

まず、psychパッケージに含まれるbfiというデータセットを読み込みこむ
<details>
<summary>
❔Code
</summary>

``` r
library(psych)
library(GPArotation)
```

    ## 
    ## Attaching package: 'GPArotation'

    ## The following objects are masked from 'package:psych':
    ## 
    ##     equamax, varimin

``` r
orig_df<-bfi[1:25] #最初の25列のみを分析する
```

</details>

### bfiの逆転項目の特定・対処

1.逆転項目はどれかを質問紙の説明(元論文やインターネット上)を基に特定する

<details>
<summary>
❔Hint
</summary>

``` r
bfi.keys
```

    ## $agree
    ## [1] "-A1" "A2"  "A3"  "A4"  "A5" 
    ## 
    ## $conscientious
    ## [1] "C1"  "C2"  "C3"  "-C4" "-C5"
    ## 
    ## $extraversion
    ## [1] "-E1" "-E2" "E3"  "E4"  "E5" 
    ## 
    ## $neuroticism
    ## [1] "N1" "N2" "N3" "N4" "N5"
    ## 
    ## $openness
    ## [1] "O1"  "-O2" "O3"  "O4"  "-O5"

「-A1」のよう変数名の頭にマイナスがついているものが逆転項目

</details>

2.逆転項目の処理を行う

リッカート尺度の場合，逆転項目に対する処理は「（最大値
+1）から回答の値を引く」を行います。

<details>
<summary>
❔Code
</summary>

ここでは1, 9, 10, 11, 12, 22, 25 番目の項目が逆転項目。

``` r
gyakuten <- c(1, 9:12, 22, 25) #後から編集しやすいように逆転項目の列番号をベクトルにする
df <- bfi[1:25] #編集するデータを作る
df[, gyakuten] <- 7 - orig_df[, gyakuten] 

#確認
head(orig_df[,gyakuten]) #元のbfiデータ
```

    ##       A1 C4 C5 E1 E2 O2 O5
    ## 61617  2  4  4  3  3  6  3
    ## 61618  2  3  4  1  1  2  3
    ## 61620  5  2  5  2  4  2  2
    ## 61621  4  5  5  5  3  3  5
    ## 61622  2  3  2  2  2  3  3
    ## 61623  6  1  3  2  1  3  1

``` r
head(df[, gyakuten]) #逆転項目の処理後のデータ
```

    ##       A1 C4 C5 E1 E2 O2 O5
    ## 61617  5  3  3  4  4  1  4
    ## 61618  5  4  3  6  6  5  4
    ## 61620  2  5  2  5  3  5  5
    ## 61621  3  2  2  2  4  4  2
    ## 61622  5  4  5  5  5  4  4
    ## 61623  1  6  4  5  6  4  6

</details>

## ①データが因子分析に適しているか確認する/bfiデータセットを見る

今回はないがデータの入力ミスなどがないか見る。これは**データクリーニング**というステップで、実データを用いる際は必ず踏むべきである。

<details>
<summary>
❔Code
</summary>

``` r
summary(df)
```

    ##        A1              A2              A3              A4            A5      
    ##  Min.   :1.000   Min.   :1.000   Min.   :1.000   Min.   :1.0   Min.   :1.00  
    ##  1st Qu.:4.000   1st Qu.:4.000   1st Qu.:4.000   1st Qu.:4.0   1st Qu.:4.00  
    ##  Median :5.000   Median :5.000   Median :5.000   Median :5.0   Median :5.00  
    ##  Mean   :4.587   Mean   :4.802   Mean   :4.604   Mean   :4.7   Mean   :4.56  
    ##  3rd Qu.:6.000   3rd Qu.:6.000   3rd Qu.:6.000   3rd Qu.:6.0   3rd Qu.:5.00  
    ##  Max.   :6.000   Max.   :6.000   Max.   :6.000   Max.   :6.0   Max.   :6.00  
    ##  NA's   :16      NA's   :27      NA's   :26      NA's   :19    NA's   :16    
    ##        C1              C2             C3              C4              C5       
    ##  Min.   :1.000   Min.   :1.00   Min.   :1.000   Min.   :1.000   Min.   :1.000  
    ##  1st Qu.:4.000   1st Qu.:4.00   1st Qu.:4.000   1st Qu.:3.000   1st Qu.:2.000  
    ##  Median :5.000   Median :5.00   Median :5.000   Median :5.000   Median :4.000  
    ##  Mean   :4.502   Mean   :4.37   Mean   :4.304   Mean   :4.447   Mean   :3.703  
    ##  3rd Qu.:5.000   3rd Qu.:5.00   3rd Qu.:5.000   3rd Qu.:6.000   3rd Qu.:5.000  
    ##  Max.   :6.000   Max.   :6.00   Max.   :6.000   Max.   :6.000   Max.   :6.000  
    ##  NA's   :21      NA's   :24     NA's   :20      NA's   :26      NA's   :16     
    ##        E1              E2              E3              E4       
    ##  Min.   :1.000   Min.   :1.000   Min.   :1.000   Min.   :1.000  
    ##  1st Qu.:3.000   1st Qu.:3.000   1st Qu.:3.000   1st Qu.:4.000  
    ##  Median :4.000   Median :4.000   Median :4.000   Median :5.000  
    ##  Mean   :4.026   Mean   :3.858   Mean   :4.001   Mean   :4.422  
    ##  3rd Qu.:5.000   3rd Qu.:5.000   3rd Qu.:5.000   3rd Qu.:6.000  
    ##  Max.   :6.000   Max.   :6.000   Max.   :6.000   Max.   :6.000  
    ##  NA's   :23      NA's   :16      NA's   :25      NA's   :9      
    ##        E5              N1              N2              N3       
    ##  Min.   :1.000   Min.   :1.000   Min.   :1.000   Min.   :1.000  
    ##  1st Qu.:4.000   1st Qu.:2.000   1st Qu.:2.000   1st Qu.:2.000  
    ##  Median :5.000   Median :3.000   Median :4.000   Median :3.000  
    ##  Mean   :4.416   Mean   :2.929   Mean   :3.508   Mean   :3.217  
    ##  3rd Qu.:5.000   3rd Qu.:4.000   3rd Qu.:5.000   3rd Qu.:4.000  
    ##  Max.   :6.000   Max.   :6.000   Max.   :6.000   Max.   :6.000  
    ##  NA's   :21      NA's   :22      NA's   :21      NA's   :11     
    ##        N4              N5             O1              O2              O3       
    ##  Min.   :1.000   Min.   :1.00   Min.   :1.000   Min.   :1.000   Min.   :1.000  
    ##  1st Qu.:2.000   1st Qu.:2.00   1st Qu.:4.000   1st Qu.:3.000   1st Qu.:4.000  
    ##  Median :3.000   Median :3.00   Median :5.000   Median :5.000   Median :5.000  
    ##  Mean   :3.186   Mean   :2.97   Mean   :4.816   Mean   :4.287   Mean   :4.438  
    ##  3rd Qu.:4.000   3rd Qu.:4.00   3rd Qu.:6.000   3rd Qu.:6.000   3rd Qu.:5.000  
    ##  Max.   :6.000   Max.   :6.00   Max.   :6.000   Max.   :6.000   Max.   :6.000  
    ##  NA's   :36      NA's   :29     NA's   :22                      NA's   :28     
    ##        O4              O5      
    ##  Min.   :1.000   Min.   :1.00  
    ##  1st Qu.:4.000   1st Qu.:4.00  
    ##  Median :5.000   Median :5.00  
    ##  Mean   :4.892   Mean   :4.51  
    ##  3rd Qu.:6.000   3rd Qu.:6.00  
    ##  Max.   :6.000   Max.   :6.00  
    ##  NA's   :14      NA's   :20

他にどの観測変数(項目)のNAが多いか、回答が偏っている項目(ほぼ全員が1を回答)があるかを確認・対処することがある。

</details>

**サンプルサイズ確認**

・10~15人/項目

    The exact number required is disputed, with estimates ranging from 3 to 20 subjects or items per variable (Gorsuch, 1983, 1990, 2003; Pett, Lackey,& Sullivan, 2003; Tabachnick & Fidell, 2013; Thompson, 2004). That being said, 10 to 15 is the most common suggestion (Field, 2009).

**項目間の相関を見る** \[参考文献4\]

・多くの項目間に適度な相関が見られる

・0.9以上の相関を持つ項目はほぼ同一なのでそのような項目があるかどうか

→ある場合は削除するか否かの問題が発生する

<details>
<summary>
❔Code
</summary>

``` r
library(corrplot)
```

    ## corrplot 0.95 loaded

``` r
library(RColorBrewer)
M <-cor(df, use="na.or.complete")
# head(round(M,2)) # これで数字を見れる
corrplot(M, type="upper", order = 'alphabet',
         col=brewer.pal(n=8, name="RdYlBu"))
```

![](supplementary_4_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

</details>

## ②因子の抽出方法の特定

今回は基本的な最尤法にしましょう。fa関数での最尤法の選択はどうするか調べましょう。

## ③因子数の決定

今回はBICによる因子数の決定を行います。

<details>
<summary>
❔Code
</summary>

``` r
for(i in 1:10){ #とりあえず10回分回します
  factor_res <- fa(df, nfactors = i, fm= "ml")
  print(paste0("no. of factor: ", i, "; BIC: ", round(factor_res$BIC,3)))
}
```

    ## [1] "no. of factor: 1; BIC: 9646.409"
    ## [1] "no. of factor: 2; BIC: 5286.187"
    ## [1] "no. of factor: 3; BIC: 3198.764"
    ## [1] "no. of factor: 4; BIC: 1729.551"
    ## [1] "no. of factor: 5; BIC: 281.469"
    ## [1] "no. of factor: 6; BIC: -295.882"
    ## [1] "no. of factor: 7; BIC: -462.708"
    ## [1] "no. of factor: 8; BIC: -524.265"
    ## [1] "no. of factor: 9; BIC: -521.28"

    ## Warning in GPFoblq(A, Tmat = Tmat, normalize = normalize, eps = eps, maxit =
    ## maxit, : convergence not obtained in GPFoblq. 1000 iterations used.

    ## [1] "no. of factor: 10; BIC: -499.504"

</details>

この場合因子数を10にすると収束できなかったという警告が出ますが、これは因子数が多く、モデルが複雑になったため収束できなかったと考えます。25項目の質問紙で10以上の因子があるとは考えにくいので警告に対処せず、因子数の検討は1~9の中で検討しようと思います。

</details>

今回は因子数がビッグファイブで5なのでは?と考えてしなうかもしれませんが、先入観を捨ててBICが最小になる8因子で分析を続けてみましょう。

因子の解釈ができない場合因子数を変えてみるこのステップから再スタートすることにしましょう。

また、モデル適合度の検討の際に、因子の回転はまだ考える必要ないです。因子の回法は推定をした後の話で、データの適合度には影響しません。つまり、どの**回転法を選んでもデータとの適合は変わらない**です。

## ④因子の回転を色々試してみる

以下[清水先生のブログ](https://norimune.net/706)より抜粋:

    知能テストなどは、一つのテストが複数の知能と関連するのはほぼ常識です。そのような実質的科学的知見を無視して、単純構造を目指すのはやや疑問が残ります。
    そのような場合は、プロマックス回転よりも**ジェオミン回転やオブリミン回転**がオススメです。どちらかというとジェオミン回転のほうが複雑さを許容した回転を行います。

今回はgeomin + quartimin回転を行ってみます。

`GPA rotation`パッケージで行える回転は[こちら](https://cran.r-project.org/web/packages/GPArotation/GPArotation.pdf)を参照。

<details>
<summary>
❔Code
</summary>

``` r
res_geo = fa(df, nfactors = 8, fm = "ml", rotate = "geominQ" )
fa.diagram(res_geo)
```

![](supplementary_4_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

``` r
res_qmin = fa(df, nfactors = 8, fm = "ml", rotate = "quartimin" )
fa.diagram(res_qmin)
```

![](supplementary_4_files/figure-gfm/unnamed-chunk-17-2.png)<!-- -->

こちらのパス図では0.3以下の因子負荷量は表示されません。
基本的に各因子**最低3項目**あるのが識別性の条件なので、今回8因子にしたことは間違いでしたね。

因子を減らしてみましょう。
以下は5因子の分析結果です。より単純構造っぽい/解釈しやすい回転法を選び解釈しましょう。

``` r
res_geo = fa(df, nfactors = 5, fm = "ml", rotate = "geominQ" )
res_geo
```

    ## Factor Analysis using method =  ml
    ## Call: fa(r = df, nfactors = 5, rotate = "geominQ", fm = "ml")
    ## Standardized loadings (pattern matrix) based upon correlation matrix
    ##      ML2   ML1   ML5   ML3   ML4   h2   u2 com
    ## A1 -0.16 -0.12  0.39 -0.02  0.03 0.15 0.85 1.6
    ## A2  0.03  0.02  0.59  0.11  0.03 0.40 0.60 1.1
    ## A3  0.02  0.11  0.65  0.05  0.03 0.51 0.49 1.1
    ## A4 -0.03  0.05  0.45  0.21 -0.15 0.29 0.71 1.7
    ## A5 -0.09  0.21  0.58  0.01  0.04 0.48 0.52 1.3
    ## C1  0.01 -0.03 -0.02  0.52  0.18 0.32 0.68 1.2
    ## C2  0.09 -0.10  0.07  0.63  0.09 0.43 0.57 1.2
    ## C3  0.01 -0.06  0.09  0.56 -0.04 0.32 0.68 1.1
    ## C4 -0.18  0.06 -0.05  0.62  0.05 0.47 0.53 1.2
    ## C5 -0.19  0.18  0.00  0.55 -0.08 0.43 0.57 1.5
    ## E1  0.07  0.58  0.07 -0.12  0.09 0.37 0.63 1.2
    ## E2 -0.10  0.69  0.08  0.00  0.04 0.55 0.45 1.1
    ## E3  0.05  0.38  0.27 -0.02  0.30 0.44 0.56 2.8
    ## E4  0.00  0.55  0.33  0.00 -0.07 0.52 0.48 1.7
    ## E5  0.13  0.43  0.03  0.25  0.22 0.40 0.60 2.4
    ## N1  0.83  0.07 -0.24  0.02 -0.04 0.71 0.29 1.2
    ## N2  0.80  0.02 -0.23  0.03  0.02 0.66 0.34 1.2
    ## N3  0.68 -0.15 -0.01 -0.03  0.03 0.53 0.47 1.1
    ## N4  0.46 -0.43  0.03 -0.12  0.09 0.48 0.52 2.2
    ## N5  0.48 -0.26  0.15  0.01 -0.13 0.34 0.66 1.9
    ## O1 -0.03  0.10  0.01  0.04  0.53 0.32 0.68 1.1
    ## O2 -0.20  0.00 -0.18  0.07  0.44 0.24 0.76 1.8
    ## O3 -0.01  0.15  0.07 -0.01  0.62 0.47 0.53 1.1
    ## O4  0.11 -0.33  0.13 -0.02  0.39 0.26 0.74 2.4
    ## O5 -0.14 -0.04 -0.08  0.03  0.52 0.27 0.73 1.2
    ## 
    ##                        ML2  ML1  ML5  ML3  ML4
    ## SS loadings           2.50 2.24 2.05 1.95 1.61
    ## Proportion Var        0.10 0.09 0.08 0.08 0.06
    ## Cumulative Var        0.10 0.19 0.27 0.35 0.41
    ## Proportion Explained  0.24 0.22 0.20 0.19 0.16
    ## Cumulative Proportion 0.24 0.46 0.66 0.84 1.00
    ## 
    ##  With factor correlations of 
    ##       ML2   ML1  ML5   ML3  ML4
    ## ML2  1.00 -0.11 0.06 -0.12 0.06
    ## ML1 -0.11  1.00 0.34  0.22 0.16
    ## ML5  0.06  0.34 1.00  0.18 0.23
    ## ML3 -0.12  0.22 0.18  1.00 0.17
    ## ML4  0.06  0.16 0.23  0.17 1.00
    ## 
    ## Mean item complexity =  1.5
    ## Test of the hypothesis that 5 factors are sufficient.
    ## 
    ## df null model =  300  with the objective function =  7.23 with Chi Square =  20163.79
    ## df of  the model are 185  and the objective function was  0.63 
    ## 
    ## The root mean square of the residuals (RMSR) is  0.03 
    ## The df corrected root mean square of the residuals is  0.04 
    ## 
    ## The harmonic n.obs is  2762 with the empirical chi square  1474.7  with prob <  1.3e-199 
    ## The total n.obs was  2800  with Likelihood Chi Square =  1749.88  with prob <  1.4e-252 
    ## 
    ## Tucker Lewis Index of factoring reliability =  0.872
    ## RMSEA index =  0.055  and the 90 % confidence intervals are  0.053 0.057
    ## BIC =  281.47
    ## Fit based upon off diagonal values = 0.98
    ## Measures of factor score adequacy             
    ##                                                    ML2  ML1  ML5  ML3  ML4
    ## Correlation of (regression) scores with factors   0.93 0.89 0.88 0.87 0.84
    ## Multiple R square of scores with factors          0.86 0.79 0.77 0.76 0.71
    ## Minimum correlation of possible factor scores     0.71 0.58 0.54 0.52 0.43

``` r
res_qmin = fa(df, nfactors = 5, fm = "ml", rotate = "quartimin" )
res_qmin
```

    ## Factor Analysis using method =  ml
    ## Call: fa(r = df, nfactors = 5, rotate = "quartimin", fm = "ml")
    ## Standardized loadings (pattern matrix) based upon correlation matrix
    ##      ML2   ML5   ML1   ML3   ML4   h2   u2 com
    ## A1 -0.20  0.36 -0.14 -0.04  0.04 0.15 0.85 2.0
    ## A2 -0.02  0.60 -0.01  0.09  0.03 0.40 0.60 1.1
    ## A3 -0.03  0.67  0.07  0.03  0.04 0.51 0.49 1.0
    ## A4 -0.06  0.46  0.04  0.20 -0.15 0.29 0.71 1.7
    ## A5 -0.14  0.58  0.17  0.00  0.06 0.48 0.52 1.3
    ## C1  0.06  0.00 -0.05  0.53  0.16 0.32 0.68 1.2
    ## C2  0.13  0.11 -0.13  0.64  0.06 0.43 0.57 1.2
    ## C3  0.04  0.11 -0.08  0.56 -0.06 0.32 0.68 1.1
    ## C4 -0.12 -0.06  0.04  0.64  0.03 0.47 0.53 1.1
    ## C5 -0.14 -0.01  0.16  0.57 -0.10 0.43 0.57 1.4
    ## E1  0.09  0.10  0.56 -0.10  0.11 0.37 0.63 1.3
    ## E2 -0.06  0.09  0.67  0.03  0.07 0.55 0.45 1.1
    ## E3  0.06  0.30  0.34 -0.02  0.31 0.44 0.56 3.0
    ## E4  0.00  0.36  0.53  0.01 -0.05 0.52 0.48 1.8
    ## E5  0.18  0.08  0.39  0.27  0.22 0.40 0.60 3.1
    ## N1  0.85 -0.09  0.09  0.01 -0.05 0.71 0.29 1.1
    ## N2  0.82 -0.08  0.04  0.02  0.01 0.66 0.34 1.0
    ## N3  0.67  0.10 -0.14 -0.06  0.03 0.53 0.47 1.2
    ## N4  0.41  0.09 -0.42 -0.16  0.08 0.48 0.52 2.4
    ## N5  0.44  0.22 -0.25 -0.02 -0.14 0.34 0.66 2.4
    ## O1 -0.01  0.02  0.06  0.06  0.53 0.32 0.68 1.1
    ## O2 -0.16 -0.21 -0.03  0.10  0.44 0.24 0.76 1.9
    ## O3  0.01  0.09  0.10  0.00  0.63 0.47 0.53 1.1
    ## O4  0.08  0.14 -0.36 -0.04  0.38 0.26 0.74 2.4
    ## O5 -0.11 -0.10 -0.07  0.05  0.52 0.27 0.73 1.2
    ## 
    ##                        ML2  ML5  ML1  ML3  ML4
    ## SS loadings           2.49 2.10 2.07 2.05 1.64
    ## Proportion Var        0.10 0.08 0.08 0.08 0.07
    ## Cumulative Var        0.10 0.18 0.27 0.35 0.41
    ## Proportion Explained  0.24 0.20 0.20 0.20 0.16
    ## Cumulative Proportion 0.24 0.44 0.64 0.84 1.00
    ## 
    ##  With factor correlations of 
    ##       ML2   ML5   ML1   ML3   ML4
    ## ML2  1.00 -0.03 -0.23 -0.21 -0.01
    ## ML5 -0.03  1.00  0.31  0.20  0.23
    ## ML1 -0.23  0.31  1.00  0.22  0.17
    ## ML3 -0.21  0.20  0.22  1.00  0.20
    ## ML4 -0.01  0.23  0.17  0.20  1.00
    ## 
    ## Mean item complexity =  1.6
    ## Test of the hypothesis that 5 factors are sufficient.
    ## 
    ## df null model =  300  with the objective function =  7.23 with Chi Square =  20163.79
    ## df of  the model are 185  and the objective function was  0.63 
    ## 
    ## The root mean square of the residuals (RMSR) is  0.03 
    ## The df corrected root mean square of the residuals is  0.04 
    ## 
    ## The harmonic n.obs is  2762 with the empirical chi square  1474.7  with prob <  1.3e-199 
    ## The total n.obs was  2800  with Likelihood Chi Square =  1749.88  with prob <  1.4e-252 
    ## 
    ## Tucker Lewis Index of factoring reliability =  0.872
    ## RMSEA index =  0.055  and the 90 % confidence intervals are  0.053 0.057
    ## BIC =  281.47
    ## Fit based upon off diagonal values = 0.98
    ## Measures of factor score adequacy             
    ##                                                    ML2  ML5  ML1  ML3  ML4
    ## Correlation of (regression) scores with factors   0.93 0.88 0.88 0.88 0.85
    ## Multiple R square of scores with factors          0.86 0.78 0.78 0.77 0.72
    ## Minimum correlation of possible factor scores     0.73 0.56 0.56 0.54 0.44

</details>

## ⑤因子の解釈

観測変数
(ここではアンケート項目)と検出された因子との関係を因子負荷量パラメータの解から読み取る

<details>
<summary>
❔Code
</summary>

5因子の場合のパス図が仮定していたbig fiveの因子に対応している。
実際はこうならないかもしれないが、今回はこれでめでたし?

``` r
fa.diagram(res_geo)
```

![](supplementary_4_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->
</details>

## bfi分析における参考サイト

1.  <https://necostat.hatenablog.jp/entry/2022/07/02/065934>
2.  <https://htsuda.net/stats/factor-analysis.html>
3.  <https://m-clark.github.io/posts/2020-04-10-psych-explained/>

# 参考文献

1.  分寺先生の資料 <https://www2.kobe-u.ac.jp/~bunji/resource.html>

2.  清水先生のブログ <https://norimune.net/psychmetrics>

3.  足立 浩平・山本 倫生 (2024).主成分分析と因子分析
    特異値分解を出発点として.
    <https://www.kyoritsu-pub.co.jp/book/b10085699.html>

4.  Plonsky, L. (2015). Advancing quantitative methods in second
    language research. Chapter 9.
