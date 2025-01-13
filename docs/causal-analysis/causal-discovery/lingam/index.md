# LiNGAM

## [SEM](/AI-Book/causal-analysis/#sem)における仮定

- 線形性
- 非ガウス連続誤差変数
- 非巡回性
- 未観測共通原因はない

## LiNGAMモデル

[SEM](/AI-Book/causal-analysis/#sem)は，LiNGAMの文脈では，観測変数ベクトル$\bm{x}$，未観測変数ベクトル$\bm{e}$，係数行列$\bm{B}$を使って

$$
\bm{x} = \bm{B} \bm{x} + \bm{e}
$$

と表せる．ただし，$\bm{e}$の各成分は独立でありかつ非ガウス連続分布に従い，$\bm{B}$を隣接行列とみなした有向グラフはDAGである．

## 手法

LiNGAMモデルを$\bm{x}$について解くと，

$$
\begin{aligned}
\bm{x} &= \bm{B} \bm{x} + \bm{e} \\
(\bm{I} - \bm{B}) \bm{x} &= \bm{e} \\
\bm{x} &= (\bm{I} - \bm{B})^{-1} \bm{e}
\end{aligned}
$$

となる．ここで，$\bm{A} = (\bm{I} - \bm{B})^{-1}$と置くと，

$$
\bm{x} = \bm{A} \bm{e}
$$

となる．ここで，$\bm{e}$の各成分は独立かつ非ガウス連続分布に従うため，[ICA](/AI-Book/other/ica/#ica)により復元行列$\hat{\bm{W}}_{ICA}$を推定することができる．

$$
\hat{\bm{W}}_{ICA} = \bm{P}_{ICA} \bm{D} \bm{A}^{-1} = \bm{P}_{ICA} \bm{D} (\bm{I} - \bm{B}) \label{a}\tag{1}
$$

ここで，この$\bm{P}_{ICA}$の推定を行う．

$\hat{\bm{W}}_{ICA}$の行の順序が$\bm{W}$と等しくない，つまり$\bm{P}_{ICA}$が単位行列ではない時，$\hat{\bm{W}}_{ICA}$の対角成分に必ず$0$が含まれることが知られている．従って，$\bm{W}_{ICA}$を置換して対角成分が全て非ゼロになるような置換行列$\bm{P}_{inv} = \bm{P}_{ICA}^{-1}$を推定すればよい．そこで，$\hat{\bm{W}}_{ICA}$の対角成分の絶対値が最大になるような$\hat{\bm{P}}_{inv}$を推定する．

$$
\hat{\bm{P}}_{inv} = \underset{\bm{P}_{inv}} {\operatorname{argmin}} \sum_{i=1}^{m} \frac{1}{|(\bm{P}_{inv} \hat{\bm{W}}_{ICA})_{ii}|}
$$

このような$\hat{\bm{P}}_{inv}$を探索する．

!!! Tip
    これを純粋に行うと計算量が$O(m!)$となり非常に大きくなるのでハンガリアン法で探索する．
 
 $\hat{\bm{P}}_{inv}$を式$\ref{a}$の両辺に左から掛けると，

$$
\hat{\bm{P}}_{inv} \hat{\bm{W}}_{ICA} = \hat{\bm{P}}_{inv} \hat{\bm{P}}_{ICA} \hat{\bm{D}} (\bm{I} - \hat{\bm{B}}) = \hat{\bm{D}} (\bm{I} - \hat{\bm{B}}) \label{b}\tag{2}
$$

ここで，$\bm{B}$を隣接行列とみなした有向グラフがDAGであることから，$\bm{B}$が (対角成分も$0$の) 下三角行列になるような$\bm{x}$の各成分の入れ替えが必ず存在するということが言える． (このような順序を因果的順序という．)

!!! Note
    アルゴリズムにおけるトポロジカルソートも，この「DAGの隣接行列に因果的順序が存在する」ことに基づいている．

また，因果順序の入れ替えでは対角成分の値は対角成分の値としか入れ替わらないことから，$\bm{B}$の対角成分は全て$0$である．よって$\rm{diag}({\bm{I} - \bm{B}}) = \bm{I}$が成り立つ．従って，$\hat{\bm{D}}$は

$$
\hat{\bm{D}} = \rm{diag}(\hat{\bm{P}}_{inv} \hat{\bm{W}}_{ICA})
$$

と推定できる．従って，$\hat{\bm{D}}^{-1}$を式$\ref{b}$の両辺に左から掛けると，

$$
\begin{aligned}
\hat{\bm{D}}^{-1} \hat{\bm{P}}_{inv} \hat{\bm{W}}_{ICA} &= \hat{\bm{D}}^{-1} \hat{\bm{D}} (\bm{I} - \hat{\bm{B}}) \\
\hat{\bm{D}}^{-1} \hat{\bm{P}}_{inv} \hat{\bm{W}}_{ICA} &= \bm{I} - \hat{\bm{B}} \\
\hat{\bm{B}} &= \bm{I} - \hat{\bm{D}}^{-1} \hat{\bm{P}}_{inv} \hat{\bm{W}}_{ICA}
\end{aligned}
$$

により$\hat{\bm{B}}$が推定できる．

しかし，$\hat{\bm{B}}$には推定誤差が含まれているため，$\hat{\bm{B}}$を隣接行列とみなした有向グラフはDAGになるとは限らない．

そこで，この$\hat{\bm{B}}$から$\bm{x}$の因果的順序を推測し，その順序を考慮した[線形回帰](/AI-Book/regression-analysis/linear-regression/#_1)を行うことで，その偏回帰係数によって$\bm{B}$を推定する．

$\bm{B}$が下三角行列になるように$\bm{x}$を置換する置換行列を$\bm{P}$と置く．LiNGAMモデルの両辺に$\bm{P}$を左から掛けると，

$$
\begin{aligned}
\bm{P} \bm{x} &= \bm{P} \bm{B} \bm{x} + \bm{P} \bm{e} \\
&= \bm{P} \bm{B} \bm{I} \bm{x} + \bm{P} \bm{e} \\
&= \bm{P} \bm{B} \bm{P}^\top \bm{P} \bm{x} + \bm{P} \bm{e} \\
&= (\bm{P} \bm{B} \bm{P}^\top) \bm{P} \bm{x} + \bm{P} \bm{e}
\end{aligned}
$$

となる．これは観測変数ベクトルが$\bm{P} \bm{x}$のLiNGAMモデルである．$\bm{P}$の定義から，$\bm{P} \bm{B} \bm{P}^\top$は下三角行列となる．そこで，$\bm{P} \hat{\bm{B}} \bm{P}^\top$が下三角行列になるような$\hat{\bm{P}}$を推定する．

$$
\hat{\bm{P}} = \underset{\bm{P}} {\operatorname{argmin}} \sum_{i \leq j} (\bm{P} \hat{\bm{B}} \bm{P}^\top)_{ij}^2
$$

このような$\hat{\bm{P}}$を探索する．

!!! Warning
    これを純粋に行うと計算量が$O(m!)$となり非常に大きくなるのでアルゴリズムを工夫する必要がある．

$\hat{\bm{P}}$により$\bm{x}$の因果的順序が推定されたので，$\bm{x}$の各成分について，自身より因果的順序が前だと推定された成分を説明変数として[線形回帰](/AI-Book/regression-analysis/linear-regression/#_1)を行うことで，その偏回帰係数によって$\bm{B}$を推定することができる．

この[線形回帰](/AI-Book/regression-analysis/linear-regression/#_1)に，[スパース回帰](/AI-Book/regression-analysis/linear-regression/sparse-regression/#_1)の一種である[適応型Lasso](/AI-Book/regression-analysis/linear-regression/sparse-regression/adaptive-lasso/#lasso)を用いることで冗長な有向辺を枝刈りすることができる．

## 論文

[1] SHIMIZU, Shohei, et al. A linear non-Gaussian acyclic model for causal discovery. Journal of Machine Learning Research, 2006, 7.10.