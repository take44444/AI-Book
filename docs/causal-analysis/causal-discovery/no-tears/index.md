# NO TEARS[^1]

## [SEM](/AI-Book/causal-analysis/#sem)における仮定

- 線形性
- 非巡回性

## 手法

因果探索の問題が難しい大きな原因は，推定するグラフにDAGであるという制約があること．この制約により，因果探索は探索空間が超指数関数的な組み合わせ最適化問題となっている．因果探索の多くの方法は，このDAG制約を満たすためのヒューリスティックに依存している．NO TEARSは，この組み合わせ最適化問題を数理（連続）最適化問題として定式化する手法．これにより，因果探索にL-BFGS等の既存の標準的なアルゴリズムを活用できるようになる．

因果探索におけるDAG制約を$h(\bm{W}) = 0$という等式制約に置き換えたい．この時，グラフのノード数を$d$として，関数$h:\mathbb{R}^{d \times d} \to \mathbb{R}$が以下を満たす関数であると理想的である．

1. $\bm{W}$がDAGである時，かつその時に限り$h(\bm{W}) = 0$となる．
2. $h(\bm{W})$でグラフの"DAG度"が定量化される．
3. $h$は連続で微分可能である．
4. $h$とその導関数は容易に計算可能である．

こんな関数があれば，因果探索は数理最適化問題として扱えるため，ラグランジュ乗数法等の既存の等式制約付き最適化手法が使える．

1~4を全て満たす関数として，

$$
h(\bm{W}) = \mathrm{tr}(e^{\bm{W} \circ \bm{W}}) - d
$$

という関数を発見したというのがNO TEARSの内容．勾配は

$$
\nabla h(\bm{W}) = (e^{\bm{W} \circ \bm{W}})^\top \circ 2 \bm{W}
$$

で計算可能．$e^{\bm{X}}$は$X$の行列指数関数．

証明は意外に簡単．

$\bm{W} \in \mathbb{R}^{d \times d}$なので，$\bm{W}$を隣接行列とするグラフがDAGになることは，任意の$x \in \mathbb{N}$について$\mathrm{tr}(\bm{W}^x) = 0$となることと同値である．

!!! Tip
    このテクニックは競技プログラミングでもグラフ問題に対する行列累乗での解法としてよく登場する．

この式を言い換えると，

$$
\begin{aligned}
\mathrm{tr}(\bm{W}) + \mathrm{tr}(\bm{W}^2) + \cdots &= 0 \\
\mathrm{tr}(\bm{W} \circ \bm{W}) + \mathrm{tr}((\bm{W} \circ \bm{W})^2) + \cdots &= 0 \\
\mathrm{tr}(\bm{I}) + \mathrm{tr}(\bm{W} \circ \bm{W}) + \mathrm{tr}((\bm{W} \circ \bm{W})^2) + \cdots &= d
\end{aligned}
$$

と変形でき，さらにマクローリン展開を利用して

$$
\begin{aligned}
\mathrm{tr}(\bm{I}) + \mathrm{tr}(\bm{W} \circ \bm{W}) + \frac{1}{2!} \mathrm{tr}((\bm{W} \circ \bm{W})^2) + \cdots &= d \\
\mathrm{tr}(e^{\bm{W} \circ \bm{W}}) &= d \\
\mathrm{tr}(e^{\bm{W} \circ \bm{W}}) - d &= 0
\end{aligned}
$$

と変形できる．■

また，$h(\bm{W}) = \mathrm{tr}(e^{\bm{W} \circ \bm{W}}) - d \geq 0$なので$h(\bm{W})$が大きい程"DAG度"が低いと捉えることができる．

DAG制約を等式制約に置き換えることができたため，あとは滑らかな損失関数 (スコア関数)を用意すれば因果探索は等式制約付き最適化問題に帰着される．

滑らかな損失関数は何でもいいが，ここでは，最小二乗損失$l(\bm{W} ; \bm{X}) = \frac{1}{2n} ||\bm{X}-\bm{X}\bm{W}||_F^2$を採用し，これをL1正規化した

$$
\begin{aligned}
F(\bm{W}) &= l(\bm{W} ; \bm{X}) + \lambda||\bm{W}||_1 \\
&= \frac{1}{2n} ||\bm{X}-\bm{X}\bm{W}||_F^2 + \lambda||\bm{W}||_1
\end{aligned}
$$

を損失関数として使用する．

これにより，因果探索は

$$
\begin{aligned}
&\min_{\bm{W} \in \mathbb{R}^{d \times d}} F(\bm{W}) \\
&\mathrm{s.t.} \quad h(\bm{W}) = 0
\end{aligned}
$$

の等式制約付き最適化問題となる．これにペナルティを追加した

$$
\begin{aligned}
&\min_{\bm{W} \in \mathbb{R}^{d \times d}} F(\bm{W}) + \frac{\rho}{2}|h(\bm{W})|^2 \\
&\mathrm{s.t.} \quad h(\bm{W}) = 0
\end{aligned}
$$

を拡張ラグランジュ法により解く．拡張ラグランジュ関数を

$$
L_\rho(\bm{W}, \alpha) = F(\bm{W}) + \frac{\rho}{2}|h(\bm{W})|^2 + \alpha h(\bm{W})
$$

として

$$
\begin{aligned}
\bm{W} &\leftarrow \underset{\bm{W}} {\operatorname{argmin}} L_\rho(\bm{W}, \alpha) \\
\bm{\alpha} &\leftarrow \alpha + \rho h(\bm{W})
\end{aligned}
$$

と置き換えることを，許容値$\epsilon$に対して$h(\bm{W}) < \epsilon$を満たすまで繰り返し，その$\bm{W}$を推定値$\hat{\bm{W}}_\mathrm{ECP}$とする．

最後に，$\hat{\bm{W}}_\mathrm{ECP}$の成分の内，$\hat{\bm{W}}_\mathrm{ECP}$を隣接行列とする有向グラフに閉路がなくなるのに十分な閾値$\omega$に対して，$\omega$よりも絶対値が小さいものを$0$に置き換えて$\hat{\bm{W}}$とする．

[^1]: Zheng, Xun, et al. "Dags with no tears: Continuous optimization for structure learning." Advances in neural information processing systems 31 (2018).