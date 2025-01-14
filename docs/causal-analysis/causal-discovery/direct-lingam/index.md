# DirectLiNGAM[^1]

LiNGAMと同じ問題に対する別の手法．

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

## 準備

手法の説明に入る前に，以下の補題1を証明しておく．

### 補題1

$\bm{x}$がLiNGAMモデルに従うとして，$\bm{x}$の成分$x_i$を$x_j$に線形回帰した時の残差を$r_i^{(j)}$とする．つまり，

$$
r_i^{(j)} = x_i - \frac{\mathrm{cov}(x_i, x_j)}{\mathrm{var}(x_j)} x_j \qquad (i,j = 1, \cdots, m; i \neq j)
$$

とすると，$x_j$と$r_i^{(j)}$が独立であることと$x_j$が外生変数であることは同値である．

### 証明

後日加筆
■

## 手法

補題1から，$x_i$が任意の$j$に対する$r_j^{(i)}$と独立であるならば$x_i$は外生変数であると言える．独立性は相互情報量$I(x_i, r_j^{(i)})$で指標できるため，$I(x_i, r_j^{(i)})$と$I(x_j, r_i^{(j)})$を比較して小さい方がより[因果的順序](/AI-Book/causal-analysis/causal-discovery/lingam/#_1)が早いと推定することができる．

外生変数であることは因果的順序において一番最初になれることと同義であるため，以下の操作を$\bm{x}$の成分数が$1$になるまで繰り返すことで．$\bm{x}$の因果的順序を推定することができる．

1. $r_j^{(i)} \qquad (i,j = 1, \cdots, m; i \neq j)$を全て求める．
2. $first = \underset{\bm{i}} {\operatorname{argmin}} \sum_j \min(0, I(x_j, r_i^{(j)}) - I(x_i, r_j^{(i)}))^2$を求める．
3. $x_{first}$が最も因果的順序が早いと推定できる．ただし，1ループ前に求まっている$x_{first}$がある場合は，その次に因果的順序が早いことになる．
4. $\bm{x}$から$x_{first}$を取り除き，さらに$x_i \quad (i \neq first)$から$\frac{\mathrm{cov}(x_i, x_{first})}{\mathrm{var}(x_{first})}$を引いたものを$\bm{x}$とする．

これにより$\bm{x}$の因果的順序を推定することができる．

!!! Tip
    2番の$I(x_j, r_i^{(j)}) - I(x_i, r_j^{(i)})$は，以下のようにしてエントロピーの次元を落とすことで高速に計算することが可能である[^2]．

    $$
    \begin{aligned}
    I(x_j, r_i^{(j)}) - I(x_i, r_j^{(i)}) =& (H(x_j) + H(r_i^{(j)}) - H([x_j \; r_i^{(j)}]^\top)) \\
    & - (H(x_i) + H(r_j^{(i)}) - H([x_i \; r_j^{(i)}]^\top)) \\
    =& (H(x_j) + H(r_i^{(j)})) - (H(x_i) + H(r_j^{(i)})) \\
    =& H(x_j) + H(r_i^{(j)}) - H(x_i) - H(r_j^{(i)})
    \end{aligned}
    $$

    ただし$H$はエントロピー関数．

$\bm{x}$の因果的順序が推定されたので，$\bm{x}$の各成分について，自身より因果的順序が前だと推定された成分を説明変数として[線形回帰](/AI-Book/regression-analysis/linear-regression/#_1)を行うことで，その偏回帰係数によって$\bm{B}$を推定することができる．

この[線形回帰](/AI-Book/regression-analysis/linear-regression/#_1)に，[スパース回帰](/AI-Book/regression-analysis/linear-regression/sparse-regression/#_1)の一種である[適応型Lasso](/AI-Book/regression-analysis/linear-regression/sparse-regression/adaptive-lasso/#lasso)を用いることで冗長な有向辺を枝刈りすることができる．

[^1]: Shimizu, Shohei, et al. "DirectLiNGAM: A direct method for learning a linear non-Gaussian structural equation model." Journal of Machine Learning Research-JMLR 12.Apr (2011): 1225-1248.
[^2]: Hyvärinen, Aapo, and Stephen M. Smith. "Pairwise likelihood ratios for estimation of non-Gaussian structural equation models." The Journal of Machine Learning Research 14.1 (2013): 111-152.