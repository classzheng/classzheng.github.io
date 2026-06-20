# 退治化学方程式的配平  
> 想法来自 [支持向量机](https://baike.baidu.com/item/%E6%94%AF%E6%8C%81%E5%90%91%E9%87%8F%E6%9C%BA/9683835) 中的 序列最小优化 (Sequential Minimal Optimization, SMO)算法，即通过每次只选择并针对含有两种元素的项进行配平，事实上，SMO也是每次选择两个拉格朗日算子进行迭代式优化——这不重要，SMO确实是C-SVM优化问题的优秀算法，也确实轻松退治了一般算法（内点法或是QP优化）每次全局更新时打破s.t.条件的问题。这是不是说，这样 ~~迫害~~ 化学方程式是不是太狠了？  

以上暂且作为此篇blog的序，然而足可见分治思想的优越性。  

0. 在化学方程式的符号系统中引入必要的约定
1. 一个比较有效的配平策略
2. 从元素守恒定律引出优化问题，乃至其对偶形式
3. 拉格朗日乘子法轻松斩杀你的化学方程式

## 0. 在化学方程式的符号系统中引入必要的约定  
虽然我们有非常完备的配平算法了，比如矩阵法；或者利用各种各样化学反应理论进行配平的算法，例如氧化数法、零价法。不过，对于一个更加完备的符号系统，不免存在一些力所不能及之处，因而存在引入数学符号的约定的必要。  

- 根据括号代数的约定，
$$
\overset{\smile}{\boldsymbol{\mathrm{X}}}
$$
用来表示不含X的组合。在这里，引申为不含元素X的化学式，并且通过连写表示其化合物。  

- 因为元素周期表中不含字母J, Q，这里作为未知的元素（或元素的组合）表示。  

- 不重要的数值统一由x,y,z等表示，比如糖类化合物表示为
$$
\mathrm{C}_x\mathrm{H}_y\mathrm{O}_z
$$
- 对于重要的参数/数值/变量，用希腊字母表示。  
- 杨氏表序 (tableaux order) 的约定出自论文 [《仿射括号代数理论与算法及其在几何定理机器证明中的应用》](https://doi.org/10.1360/za2007-37-5-523)，这里只浅显地引用了部分符号。规定如果A的序小于B的序，记为
$$
A \prec B
$$
- 对于左右式元素不守恒的式子，中间用 `~` 连接。  
- 默认化学方程式中的加号可以被认为是数学含义上的加号，并且可以用 `\sum` Σ 表示求和。

## 1. 一个比较有效的配平策略  
从【序】中可知，我们的策略是每次选择两种元素并针对含有元素的项进行配平。也就是说，我们需要先选择两个不同的元素，记为{J, Q}。  
那么，我们将化学方程式中的化学计量数记为 `\xi` ξ 或者 `\zeta` ζ 表示，同时将右下角标的系数用`\iota` ι表示，则一般的化学方程式总能被写为  
$$
\sum^{}_i \xi_0[i]\overset{\smile}{\mathrm{JQ}}\mathrm{J}_{\iota_0[i]}+\sum^{}_j  \xi_1[j]\overset{\smile}{\mathrm{JQ}}\mathrm{Q}_{\iota_1[j]}+\sum^{}_k  \xi_2[k]\overset{\smile}{\mathrm{JQ}}\mathrm{J}_{\iota_2[k]}\mathrm{Q}_{\iota_3[k]}+\sum^{}x\overset{\smile}{\mathrm{JQ}}

\xlongequal{}

\sum^{}_i \zeta_0[i]\overset{\smile}{\mathrm{JQ}}\mathrm{J}_{\vartheta_0[i]}+\sum^{}_j  \zeta_1[j]\overset{\smile}{\mathrm{JQ}}\mathrm{Q}_{\vartheta_1[j]}+\sum^{}_k  \zeta_2[k]\overset{\smile}{\mathrm{JQ}}\mathrm{J}_{\vartheta_2[k]}\mathrm{Q}_{\vartheta_3[k]}+\sum^{}y\overset{\smile}{\mathrm{JQ}}
$$
即，左式总能分成【J的化合物】，【Q的化合物】，【含J和Q的化合物】和【不含J或Q的其它项】。  

当然，这里未必是化合物，单质也可以的，不过是
$$
\overset{\smile}{\mathrm{JQ}}:=\varnothing
$$
罢了，我们的约定允许这种情况的存在。  

显然，如果我们先配平出现频率最高 (最大occurrences) 的元素，那么配平的效率会高出很多，因为若除去第四者【不含J或Q的其它项】以外前三者尽可能地多，则配平局部的方程式后回代时能确定更多前三者类型的项的化学计量数。  
因为以上的通项形式 ~~又臭又长~~，所以我们不妨去掉第四类项，然后方程式用~连接，有
$$
\sum^{}_i \xi_0[i]\overset{\smile}{\mathrm{JQ}}\mathrm{J}_{\iota_0[i]}+\sum^{}_j  \xi_1[j]\overset{\smile}{\mathrm{JQ}}\mathrm{Q}_{\iota_1[j]}+\sum^{}_k  \xi_2[k]\overset{\smile}{\mathrm{JQ}}\mathrm{J}_{\iota_2[k]}\mathrm{Q}_{\iota_3[k]}

\rightarrow

\sum^{}_i \zeta_0[i]\overset{\smile}{\mathrm{JQ}}\mathrm{J}_{\vartheta_0[i]}+\sum^{}_j  \zeta_1[j]\overset{\smile}{\mathrm{JQ}}\mathrm{Q}_{\vartheta_1[j]}+\sum^{}_k  \zeta_2[k]\overset{\smile}{\mathrm{JQ}}\mathrm{J}_{\vartheta_2[k]}\mathrm{Q}_{\vartheta_3[k]}
$$

然而，我们的策略就是依据表序
$$
ord: \mathrm{A} \prec \mathrm{B} \prec \mathrm{C} \prec ...
$$
逆序依次选择两个元素，抽出局部的化学方程式并配平，配平只需要满足左右式的J, Q元素守恒即可，然后就得出了ξ和ζ。  
将ξ和ζ回代到原化学方程式，我们暂时将系数写在对于项的正下方，
因为显然有时候某些化学方程式的局部算出来同一项的系数不同，其实只要算完后将所有系数都扩大几倍使得两次算出来的系数成为最小公倍数即可。  

## 2. 从元素守恒定律引出优化问题，乃至其对偶形式  
对于含J, Q的局部化学方程式，我们配平的目标是使含J/Q的项满足元素守恒，然而这些项中的其它元素我们并不过问。因而，我们有
$$
\begin{cases}
\sum^{}_i \xi_0[i] + \sum^{}_j \xi_2[j] = \sum^{}_i \zeta_0[i] + \sum^{}_j \zeta_2[j];\\

\sum^{}_j \xi_1[j] + \sum^{}_k \xi_2[k] = \sum^{}_j \zeta_1[j] + \sum^{}_k \zeta_2[k];
\end{cases} \quad(1)
$$

同时，我们对化学计量数也有特殊的要求，至少是
$$
\forall\xi\in\mathbb{Z}, \zeta\in\mathbb{Z}
$$
吧。也就是，我们有
$$
\begin{cases}
\xi>0;\\
\zeta>0;
\end{cases}\quad(2)
$$


此外，为了避免出现化学方程式的系数不是既约的，我们还需要令ξ和ζ取最小值，也就是
$$
\min\limits{} \left(\sum^{}_i (\xi_0[i])^2+\sum^{}_j (\xi_1[j])^2+\sum^{}_k (\xi_2[k])^2+\sum^{}_i (\zeta_0[i])^2+\sum^{}_j (\zeta_1[j])^2+\sum^{}_k (\zeta_2[k])^2\right)
$$
整理，我们就得出了带约束的 **优化问题**：
$$
\min\limits{} \left(\sum^{}_i (\xi_0[i])^2+\sum^{}_j (\xi_1[j])^2+\sum^{}_k (\xi_2[k])^2+\sum^{}_i (\zeta_0[i])^2+\sum^{}_j (\zeta_1[j])^2+\sum^{}_k (\zeta_2[k])^2\right)\\
\text{s.t.}\quad
\xi>0,
\zeta>0,\quad
\begin{cases}
\sum^{}_i \xi_0[i] + \sum^{}_j \xi_2[j] = \sum^{}_i \zeta_0[i] + \sum^{}_j \zeta_2[j]\\

\sum^{}_j \xi_1[j] + \sum^{}_k \xi_2[k] = \sum^{}_j \zeta_1[j] + \sum^{}_k \zeta_2[k]
\end{cases}
$$
对 (1) 进行移项，便得到
$$
\min\limits{} \left(\sum^{}_i (\xi_0[i])^2+\sum^{}_j (\xi_1[j])^2+\sum^{}_k (\xi_2[k])^2+\sum^{}_i (\zeta_0[i])^2+\sum^{}_j (\zeta_1[j])^2+\sum^{}_k (\zeta_2[k])^2\right)\\
\text{s.t.}\quad
\xi>0,
\zeta>0,\quad
\begin{cases}
\sum^{}_i \xi_0[i] + \sum^{}_j \xi_2[j] - \sum^{}_i \zeta_0[i] - \sum^{}_j \zeta_2[j]=0\\

\sum^{}_j \xi_1[j] + \sum^{}_k \xi_2[k] - \sum^{}_j \zeta_1[j] - \sum^{}_k \zeta_2[k]=0
\end{cases}
$$
这是我们的 **原始优化问题(Primary Optimization)** ，我们先用拉格朗日乘子法将其转化为拉格朗日对偶问题吧。

> 定义拉格朗日函数：
> $$
> \mathcal{L}(\xi_0,\xi_1,\xi_2,\zeta_0,\zeta_1,\zeta_2,\alpha,\beta)=(\sum^{}_i (\xi_0[i])^2+\sum^{}_j (\xi_1[j])^2+\sum^{}_k (\xi_2[k])^2+\sum^{}_i (\zeta_0[i])^2+\sum^{}_j (\zeta_1[j])^2+\sum^{}_k (\zeta_2[k])^2)\\
> +\sum^{}_i \alpha[0][i]\xi_0[i] + \sum^{}_j \alpha[1][j]\xi_1[j] + \sum^{}_k \alpha[2][k]\xi_2[k]\\
> +\sum^{}_i \beta[0][i]\zeta_0[i] + \sum^{}_j \beta[1][j]\zeta_1[j] + \sum^{}_k \beta[2][k]\zeta_2[k]
> $$
> 
