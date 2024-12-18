第2回 補足資料
================
Higashiguchi Rinhi
2024-10-17

# 演習

データは[ここから](https://github.com/Higashiguchi-Rinhi/tokushujikken/blob/dc9812d63186b8dc9685103218a305e9bb848661/dataset/simulated_height_data_n100.csv)ダウンロードしてください。
父親と息子の身長に関する生成データです。

<div class="callout-box">

最尤法により、回帰分析のパラメータを推定するコードを実装してみましょう。

入力: $x=(x_1,..,x_N), y=(y_1,...,y_N)$

出力: $a,b,\sigma^2$, log-likelihood

</div>

## Rで関数を作る

データ $(y_1,...,y_N)$ と $(x_1,...,x_N)$ を入力すると各パラメータ及び対数尤度を出力する関数を作ります。

Rでの関数の作り方を調べましょう。

<details>
<summary>
❔Hint
</summary>

入力: 複数のデータ

例: 2,3,4

出力: その半分の値　

例: 1,1.5,2

``` r
hint_function <- function(data) { #"関数の名前" <- function("引数"){
  new_data <- data/2  #操作
  return(new_data) #return("出力")}
}
```

実行してみる:

``` r
hint_function(c(2:4))
```

    [1] 1.0 1.5 2.0

このとき、引数(data)は数値1個ではなくベクトルである。
関数はベクトルの各要素に対しどのような処理を行い、出力しているだろうか。

また、複数の引数を指定するには(この場合 $x$ と $y$ 2つのベクトル)どうすればいいのか。

</details>

## 解を構成するのに必要な要素を考える

$a$, $b$,
$\sigma^2$及び対数尤度関数はどのように表せられるかをスライドなどで調べましょう。

<details>
<summary>
❔Hint
</summary>

``` r
b <- cov(x,y)/var(x)
a <- ybar-b*xbar
```

Rでは分散、共分散を出力する関数があるので簡潔に表現できます。

</details>

## 関数を組み立てる

複数の出力をするには?

<details>
<summary>
❔Hint
</summary>

`list()`というデータの形を使いましょう。

</details>

# 線形代数の基礎

## 線形代数とは

- データ=数値の表をそのまま扱うことのできる数学
- スカラーを使うよりもシンプルに分析法を記述できる
- 直感的な分析法の理解に役立つ
- 様々な分析法の数理的な側面を理解するのに必須

## ベクトルと行列

数値(**スカラー**;scalar)
$a,b,c$ を考える。スカラーを複数並べ、1つのまとまりとしたものを**ベクトル**(vector)と呼ぶ。ベクトルを構成するスカラーを**要素**(element),
その数をベクトルの**次元数**(dimensionality)と呼ぶ。

スカラーを縦方向に並べたベクトルは**列ベクトル**、 

$$
\begin{bmatrix}
a \\
b \\
c
\end{bmatrix}
$$

横方向に並べたベクトルを**行ベクトル**と呼ぶ。

$$
\begin{bmatrix}
a & b & c
\end{bmatrix}
$$

同じ次元数のベクトルを複数束ねたものを**行列**(matrix)と呼ぶ。ここで、縦方向の広がりを**行**(row)、横方向の広がりを**列**(column)と呼ぶ。行の数と列の数で表される行列のサイズを**次元数**と呼び、下の例であれば行数は3、列数は2になり、この行列の次元数は「 $2 \times 3$ の行列」と呼ぶ。

$$
\begin{bmatrix}
a & d\\
b & e\\
c & f
\end{bmatrix}
$$

行列の $X$ の $i$ 番目の行かつ $j$ 番目の列に位置する要素は第 $(i,j)$ 要素と表記され、
行列の特定の要素の位置を示す $i$ と $j$
の組み合わせをその要素の**インデックス**(index)と呼ぶ。また、この要素を「 $x_{ij}$ 」と、行列のアルファベットまたはギリシャ文字の小文字のイタリック体に、インデックスの添え字を「行番号,列番号」の順で付した形で表記する。このことを $X = \{x_{ij}\}$ と表すこともある。

## 行列の足し算

ベクトル同様、行列は次元数が一致する任意の二つの行列に対して行列の和を定義することができる。

$$
A=\begin{bmatrix}
a_{11} & a_{12} \\
a_{21} & a_{22} \\
a_{31} & a_{32} 
\end{bmatrix}, \
B= \begin{bmatrix}
b_{11} & b_{12} \\
b_{21} & b_{22} \\
b_{31} & b_{32} 
\end{bmatrix}
$$ 

と定義すると、

$$ 
A+B = \begin{bmatrix}
a_{11} & a_{12} \\
a_{21} & a_{22} \\
a_{31} & a_{32} 
\end{bmatrix} +
\begin{bmatrix}
b_{11} & b_{12} \\
b_{21} & b_{22} \\
b_{31} & b_{32} 
\end{bmatrix} =
\begin{bmatrix}
a_{11}+b_{11} & a_{12}+b_{12} \\
a_{21}+b_{21} & a_{21}+b_{22} \\
a_{21}+b_{31} & a_{31}+b_{32} 
\end{bmatrix}
$$

行列の和の性質は以下にまとめられる:

1.  $A+B=B+A$
2.  $(A+B)+C=A+(B+C)$

## 行列の掛け算

まず、行列 $X$ と $Y$ の積 $XY$ が計算できるためには、 $X$ の列数と $Y$ の行数が一致していなければならない。
$Z=XY=\{z_{ij}\}$ とすると $z_{ij}$ は 

$$
z_{ij}=\sum_k^n x_{ik} y_{kj}
$$

例えば


$$
\begin{bmatrix}
1 & 2 \\
5 & -7 \\
-4 & 3 
\end{bmatrix} 
\begin{bmatrix}
-3 & 4 & -2 \\
1 & 5 & -1
\end{bmatrix} \\ =
\begin{bmatrix}
1\times(-3)+2\times1 & 1\times4+2\times5 & 1\times(-2)+2\times(-1) \\
5\times(-3)-7\times1 & 5\times4-7\times5 & 5\times(-2)-7\times(-1) \\
-4\times(-3)+3\times1 & -4\times4+3\times5 & -4\times(-2)+3\times(-1)
\end{bmatrix} \\ =
\begin{bmatrix}
-1 & 14 & -4 \\
-22 & -15 & -3 \\
15 & -1 & 5
\end{bmatrix}
$$

行列の積に対する注意点:

1.  $XY$ が定義できても、両者を入れ替えた $YX$ が定義できるとは限らない。
2.  一般に、$XY \neq YX$ である。行列の和の性質と異なることに注意する。

## 行列の転置

ある行列やベクトルの行と列を入れ替えたものを,その行列の**転置**(transpose)と呼ぶ。転置された行列は元の行列の右上に’や $^T$ をつけて表記する。

$$
X=\begin{bmatrix}
a & d\\
b & e\\
c & f
\end{bmatrix}
$$ 

$$
X'=\begin{bmatrix}
a & b & c\\
d & e & f\\
\end{bmatrix}
$$

## 参考文献

- 豊田(1998)『共分散構造分析\[入門編\]:構造方程式モデリング』朝倉書店

- 2024年Sセメ開講『教育学・心理学のための多変量解析の基礎』山下直人先生の講義資料

# 付録: ギリシャ文字一覧

| 大文字 | 小文字 | 読み方               |
|--------|--------|----------------------|
| Α      | α      | アルファ (alpha)     |
| Β      | β      | ベータ (beta)        |
| Γ      | γ      | ガンマ (gamma)       |
| Δ      | δ      | デルタ (delta)       |
| Ε      | ε      | イプシロン (epsilon) |
| Ζ      | ζ      | ゼータ (zeta)        |
| Η      | η      | イータ (eta)         |
| Θ      | θ      | シータ (theta)       |
| Ι      | ι      | イオタ (iota)        |
| Κ      | κ      | カッパ (kappa)       |
| Λ      | λ      | ラムダ (lambda)      |
| Μ      | μ      | ミュー (mu)          |
| Ν      | ν      | ニュー (nu)          |
| Ξ      | ξ      | クシー (xi)          |
| Ο      | ο      | オミクロン (omicron) |
| Π      | π      | パイ (pi)            |
| Ρ      | ρ      | ロー (rho)           |
| Σ      | σ/ς    | シグマ (sigma)       |
| Τ      | τ      | タウ (tau)           |
| Υ      | υ      | ユプシロン (upsilon) |
| Φ      | φ      | ファイ (phi)         |
| Χ      | χ      | カイ (chi)           |
| Ψ      | ψ      | プサイ (psi)         |
| Ω      | ω      | オメガ (omega)       |
