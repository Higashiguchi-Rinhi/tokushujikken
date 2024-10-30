第2回 補足資料
================
Higashiguchi Rinhi
2024-10-30

<style type="text/css">
&#10;body{ /* Normal  */
      font-size: 14px;
  }
td {  /* Table  */
  font-size: 14px;
}
&#10;h1.title {
  font-size: 34px;
  color: Black;
}
h1 { /* Header 1 */
  font-size: 20px;
  color: SteelBlue;
}
h2 { /* Header 2 */
    font-size: 18px;
  color: Black;
}
h3 { /* Header 3 */
  font-size: 16px;
  color: Black;
}
code.r{ /* Code block */
    font-size: 12px;
}
pre { /* Code block - determines code spacing between lines */
    font-size: 14px;
}
</style>
<style>
.callout-box {
    border: 1px solid #007bff;
    background-color: #e9f5ff;
    padding: 10px;
    border-radius: 5px;
}
</style>
&#10;<style>
.hint-box {
    border: 1px solid #007bff;
    background-color: white;
    padding: 10px;
    border-radius: 5px;
}
</style>

# 演習①

データは[ここから](https://github.com/Higashiguchi-Rinhi/tokushujikken/blob/dc9812d63186b8dc9685103218a305e9bb848661/dataset/simulated_height_data_n100.csv)ダウンロードしてください。
父親と息子の身長に関する生成データです。

<div class="callout-box">

先週のデータセット及び最尤推定法の関数を用いてAICを算出してください

もちろん、先週の以下の関数にAICを出力するようにする形でも問題ございません

入力: $x=(x_1,..,x_N), y=(y_1,...,y_N)$

出力: $a,b,\sigma^2$, 最大化されたlog-likelihood, AIC \[new!\]

</div>

AICの定義

$$
AIC = -2(最大化された対数尤度)+ 2×(パラメータ数)
$$ 回帰分析の最尤推定法では
以下の対数尤度関数が最大になる$\{a,b,\sigma^2\}$を求める

$$
log𝐿(𝑎,𝑏,𝜎^2 )=∑_𝑖(−\frac{1}{2} log⁡(2𝜋𝜎^2 )−\frac{(𝑌_𝑖−𝑎−𝑏𝑥_𝑖 )^2}{2𝜎^2})
$$

<details>
<summary>
❔Hint 1
</summary>

対数尤度関数が最大になるのは対数尤度関数にパラメータ推定値$\{a,b,\sigma^2\}$を代入したもの

</details>
<details>
<summary>
❔Hint 2
</summary>

パラメータ数は$\{a,b,\sigma^2\}$の3つです

</details>

→答え合わせは`lm()`の結果を`AIC()`に入れることでできる

``` r
results <-lm(data1$sons..height~data1$fathers..height)
AIC(results)
```

# おまけ演習: 重回帰分析

<div class="callout-box">

実際に Boston housing dataset を用いて、重回帰分析してみましょう

</div>

## データセットについて

### データの読み込み

``` r
library(MASS)
data(Boston) #データの名前はBoston
head(Boston) #データセットの最初の数行を表示
```

         crim zn indus chas   nox    rm  age    dis rad tax ptratio  black lstat
    1 0.00632 18  2.31    0 0.538 6.575 65.2 4.0900   1 296    15.3 396.90  4.98
    2 0.02731  0  7.07    0 0.469 6.421 78.9 4.9671   2 242    17.8 396.90  9.14
    3 0.02729  0  7.07    0 0.469 7.185 61.1 4.9671   2 242    17.8 392.83  4.03
    4 0.03237  0  2.18    0 0.458 6.998 45.8 6.0622   3 222    18.7 394.63  2.94
    5 0.06905  0  2.18    0 0.458 7.147 54.2 6.0622   3 222    18.7 396.90  5.33
    6 0.02985  0  2.18    0 0.458 6.430 58.7 6.0622   3 222    18.7 394.12  5.21
      medv
    1 24.0
    2 21.6
    3 34.7
    4 33.4
    5 36.2
    6 28.7

### データの変数について

データの各行の名前

``` r
colnames(Boston)
```

     [1] "crim"    "zn"      "indus"   "chas"    "nox"     "rm"      "age"    
     [8] "dis"     "rad"     "tax"     "ptratio" "black"   "lstat"   "medv"   

従属変数(1個):

- `medv`
  :「住宅価格」（1000ドル単位）の中央値。通常はこの数値が目的変数として使われる

独立変数(13個):

- `crim`: 町別の「犯罪率」
- `ZN`:
  25,000平方フィートを超える区画に分類される住宅地の割合＝「広い家の割合」
- `indus`： 町別の「非小売業の割合」
- `chas`：
  チャールズ川のダミー変数（区画が川に接している場合は**1**、そうでない場合は**0**）＝「川の隣か」
- `nox`： 「NOx濃度（0.1ppm単位）」＝一酸化窒素濃度（parts per 10
  million単位）。この項目を目的変数とする場合もある
- `rm`： 1戸当たりの「平均部屋数」
- `age`： 1940年より前に建てられた持ち家の割合＝「古い家の割合」
- `dis`：
  5つあるボストン雇用センターまでの加重距離＝「主要施設への距離」
- `rad`： 「主要高速道路へのアクセス性」の指数
- `tax`： 10,000ドル当たりの「固定資産税率」
- `ptratio`： 町別の「生徒と先生の比率」
- `b`： 「1000(Bk - 0.63)」の二乗値。Bk＝「町ごとの黒人の割合」を指す
- `lstat`： 「低所得者人口の割合」

## 分析手順

1.このデータはスケールがそろっていないので、まず標準化を行う

<details>
<summary>
❔Hint 1
</summary>

`scale()`関数を用いることで各変数を標準化します。つまり、各変数からその平均を引き、標準偏差で割る処理が自動的に行われます。

``` r
options(scipen=999)
boston_scaled <- as.data.frame(scale(Boston))
```

注: 理由について気になる方は心理統計学の基礎p234を参照

</details>

2.従属変数には住宅価格を、説明変数にはそれ以外を用いる

<details>
<summary>
❔Hint 2
</summary>

`lm`関数を用います。
`lm`関数における従属変数と説明変数の指定はどのように行うか調べましょう

``` r
# 目的変数medv（住宅価格）を、すべての説明変数で説明する場合「.」を用いる
model <- lm(medv ~ ., data = boston_scaled)
```

</details>

3.重回帰分析において主に興味があるのは係数パラメータ

係数パラメータの解から、どの説明変数が従属変数の説明
(予測)に大きく寄与しているのかを読み取る

→解の絶対値の大きなものに注目する

<details>
<summary>
❔Hint 3
</summary>

``` r
summary(model)
```


    Call:
    lm(formula = medv ~ ., data = boston_scaled)

    Residuals:
         Min       1Q   Median       3Q      Max 
    -1.69559 -0.29680 -0.05633  0.19322  2.84864 

    Coefficients:
                              Estimate             Std. Error t value
    (Intercept) -0.0000000000000001628  0.0229370281451951111   0.000
    crim        -0.1010170764073737709  0.0307368196733816892  -3.287
    zn           0.1177152012154594751  0.0348107484188725294   3.382
    indus        0.0153352002454430758  0.0458711923391040485   0.334
    chas         0.0741988319690506104  0.0237940249602185995   3.118
    nox         -0.2238480280384697874  0.0481263469733852131  -4.651
    rm           0.2910564654888393443  0.0319275986444202409   9.116
    age          0.0021186381363495457  0.0404301526590648275   0.052
    dis         -0.3378363471562726428  0.0456658803233316105  -7.398
    rad          0.2897490528123856102  0.0628127788186940150   4.613
    tax         -0.2260316798263767302  0.0689119054706839734  -3.280
    ptratio     -0.2242712309134744786  0.0307958675712234099  -7.283
    black        0.0924322323192746376  0.0266621757055152363   3.467
    lstat       -0.4074469332149451284  0.0393777125678079140 -10.347
                            Pr(>|t|)    
    (Intercept)             1.000000    
    crim                    0.001087 ** 
    zn                      0.000778 ***
    indus                   0.738288    
    chas                    0.001925 ** 
    nox            0.000004245643808 ***
    rm          < 0.0000000000000002 ***
    age                     0.958229    
    dis            0.000000000000601 ***
    rad            0.000005070529023 ***
    tax                     0.001112 ** 
    ptratio        0.000000000001309 ***
    black                   0.000573 ***
    lstat       < 0.0000000000000002 ***
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    Residual standard error: 0.516 on 492 degrees of freedom
    Multiple R-squared:  0.7406,    Adjusted R-squared:  0.7338 
    F-statistic: 108.1 on 13 and 492 DF,  p-value: < 0.00000000000000022

参考文献2では各変数と従属変数との相関係数を見て、用いる独立変数を決めている。参考文献2の前半を是非参照されたい。

“From correlation matrix, we see TAX and RAD are highly correlated
features. The columns LSTAT, INDUS, RM, TAX, NOX, PTRAIO has a
correlation score above 0.5 with MEDV which is a good indication of
using as predictors.”

一応こちらでも変数間の相関係数を図示する

``` r
library(corrplot)
```

    corrplot 0.95 loaded

``` r
library(RColorBrewer)
M <-cor(boston_scaled)
corrplot(M, type="upper", order="hclust",
         col=brewer.pal(n=8, name="RdYlBu"))
```

![](supplementary_3_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

今回の結果において、標準偏回帰係数が大きい独立変数と対応していることが確認できる。
実際は先に相関係数を確認することで多重共線性の問題を避けて重回帰分析を行うことが推奨される。

</details>

## 参考文献

1.  <https://atmarkit.itmedia.co.jp/ait/articles/2006/24/news033.html>

2.  <https://www.kaggle.com/code/prasadperera/the-boston-housing-dataset>

# 演習②

<div class="callout-box">

第1回で使用した5教科のテストのデータを$m=2$で解析してみましょう

また、実際にプロットしてみましょう

</div>

データは[ここから](https://github.com/Higashiguchi-Rinhi/tokushujikken/blob/8115d54838df05579f27cbaa0baea8fa8fe3cd98/dataset/five_subject_score_n20p5.csv)ダウンロードしてください。
第1回で使用した5教科のテストのデータです。

主成分分析の手順:

1.主成分数$m$を決定 →今回は$m=2$

2.解析を実行

<details>
<summary>
❔Hint 1
</summary>

- Rの関数`prcomp()`を用いる

→`prcomp()`：主成分分析を実行してくれる関数

- データの標準化を行うにはどうすれば良いか

</details>

3.主成分負荷量の解を読み取り、主成分が何を表すものなのかを特定
(何がデータを背後から説明しているのかを考える)

<details>
<summary>
❔Hint 2
</summary>

- Rの関数`biplot()`を用いる

→`prcomp()`の結果を入力すると、プロット結果を図示してくれる

</details>
