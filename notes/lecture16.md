# Lecture 16：NP 问题 II——P、NP、归约与完全性

> 依据：[hand16.pdf](../lectures/hand16.pdf)（26 页讲义）与 [algo16.pdf](../lectures/algo16.pdf)（60 页，含逐步展示的幻灯片）。下文页码按讲义页脚编号。
> 主线：集合覆盖与图同构 → P 与 NP → 补问题与 co-NP → 有界定理证明与指数时间假设（ETH） → 多项式时间归约 → NP 完全性 → co-NP 完全性与永真式 → 因数分解、质数判定 → NP 中间问题 → Quiz。
> 标为“补充”“辨析”“例题演算”或“讲义修正”的内容用于补全条件、证明或例子，不是讲义原文。输入长度记为 $\ell=|I|$，整数默认用二进制编码；图默认是有限简单无向图。讲义以搜索问题介绍 P、NP，本文保留这一动机，但讨论补集、完全性和 NPI 时统一使用标准的**判定问题**定义；除特别说明外，$\le_p$ 表示多项式时间多对一归约。

## 1. 本讲要回答什么？（第 2、5、13 页）

[Lecture 15](lecture15.md) 介绍了一批“给出候选解后容易验证，但尚未得到一般高效算法”的问题。本讲把这种直觉整理为三个层次：

1. **问题属于哪一类？** 能否多项式时间求解？YES 实例有没有可以快速验证的短证书？
2. **怎样比较两个问题的难度？** 能否把问题 $A$ 转成问题 $B$，再利用 $B$ 的算法解决 $A$？
3. **怎样证明一个新问题足够难？** 从已知 NP 完全问题归约到它。

讲义第 5 页回顾了以下对比。左栏的 NP 完全性应理解为相应的判定版本；右栏列出可高效解决的问题。

| NP 完全问题的判定版本 | 具有多项式时间算法的相关问题 |
| --- | --- |
| 3-SAT | 2-SAT、Horn-SAT |
| TSP：是否存在费用不超过 $K$ 的巡回路线？ | 最小生成树 |
| 最长简单路径：是否存在长度至少 $K$ 的简单路径？ | 最短路 |
| 三维匹配：是否存在指定大小的匹配？ | 二分图匹配 |
| 二进制编码的 0/1 背包判定 | 一元编码背包 |
| 独立集：是否存在大小至少 $K$ 的独立集？ | 树上的最大独立集 |
| 一般整数线性规划的可行性判定 | 有理数编码的线性规划 |
| Hamilton 路径（讲义称 Rudrata path） | Euler 路径 |
| 带割边数预算的平衡割判定 | 最小割 |

**辨析。** “NP 完全”并不是已经证明需要指数时间。它表示：在规定的归约下，这个问题至少和所有 NP 问题一样难。若 $\mathrm P\ne\mathrm{NP}$，它就没有多项式时间算法；但这仍不直接给出 $2^{\Omega(\ell)}$ 的时间下界。

## 2. 两个新例子：集合覆盖与图同构（第 3–4 页）

### 2.1 集合覆盖：不能漏掉预算或最小化目标

给定元素全集 $B$ 和子集

$$
S_1,S_2,\ldots,S_m\subseteq B,
$$

要选出一些集合，使其并集等于 $B$。

| 版本 | 要求 |
| --- | --- |
| 无预算的可行性判定 | 是否存在一些集合覆盖 $B$？ |
| 判定版本 | 给定 $K$，能否用至多 $K$ 个集合覆盖 $B$？ |
| 带预算的搜索版本 | 找出至多 $K$ 个集合构成的覆盖，或报告不存在 |
| 最优化版本 | 在所有覆盖中，最小化选中的集合数 |

判定版本可写为

$$
\exists J\subseteq\{1,\ldots,m\}:
\quad |J|\le K,
\qquad \bigcup_{j\in J}S_j=B.
$$

**关键区别。** 如果完全不限制集合数量，只需检查

$$
\bigcup_{i=1}^{m}S_i=B;
$$

成立就把所有集合都选上。因此困难来自“尽量少选”或“至多选 $K$ 个”，不是覆盖可行性本身。讲义的 “Cost: Number of sets picked” 正是在指定要最小化的量。

**为什么判定版属于 NP？** 证书是所选集合的编号列表 $J$。去重后最多含 $m$ 个编号；验证数量不超过 $K$，再扫描集合并标记被覆盖元素即可。若集合显式列出，所需时间相对于输入总长度是多项式。

**补充例子。** 设

$$
B=\{1,2,3,4\},\quad
S_1=\{1,2\},\quad S_2=\{2,3\},\quad S_3=\{3,4\}.
$$

选择 $S_1,S_3$ 就覆盖了 $B$，所以 $K=2$ 时答案为 YES；没有任何一个集合等于 $B$，所以 $K=1$ 时答案为 NO。

这也延续了 [Lecture 14](lecture14.md) 的集合覆盖模型：$x_i\in\{0,1\}$ 表示是否选择 $S_i$，最小化 $\sum_i x_i$。允许分数变量得到 LP 松弛，并不自动得到原来的最优整数覆盖。

### 2.2 图同构：顶点改名，邻接关系不变

图 $G,H$ 同构，是指存在双射

$$
f:V(G)\to V(H)
$$

满足对所有顶点 $u,v$：

$$
\boxed{
\{u,v\}\in E(G)
\iff
\{f(u),f(v)\}\in E(H).
}
$$

这里必须同时保留边与非边，而且每个顶点恰好对应另一个图中的一个顶点。图画在平面上的形状、线段交叉和顶点名称都不是同构判定的依据。

**为什么属于 NP？** 给出映射 $f$ 后，先检查它是双射，再检查所有顶点对的邻接关系。使用邻接矩阵可在 $O(n^2)$ 次检查内完成；映射本身只需 $O(n\log n)$ 位记录。

**例题演算：第 4 页的两幅图。** 左图的邻接关系为

$$
N(a)=\{g,h,i\},\quad N(b)=\{g,h,j\},\quad
N(c)=\{g,i,j\},\quad N(d)=\{h,i,j\}.
$$

一个可行的同构映射是：

| 左图顶点 | $a$ | $b$ | $c$ | $d$ | $g$ | $h$ | $i$ | $j$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 右图顶点 | 1 | 3 | 6 | 8 | 2 | 4 | 5 | 7 |

例如 $a$ 的邻居映到 $\{2,4,5\}$，恰好是右图中 1 的邻居。其余顶点也满足同样的对应。图中的颜色只是绘图元素，定义没有要求保持颜色；不能直接按相同颜色配对。

**辨析。** 相同的顶点数、边数、度数序列只是必要条件，不足以保证同构。例如六边形 $C_6$ 和两个互不相连的三角形都是 6 个顶点、6 条边、每点度数为 2，但前者连通，后者不连通。

**补充。** Babai 给出了图同构的拟多项式时间算法，时间界为 $\exp((\log n)^{O(1)})$；这项结果不能直接当作多项式时间算法，也没有把图同构证明为 NP 完全问题。参见 [Babai 的原始论文](https://arxiv.org/abs/1512.03547)。本讲引入图同构，正是提醒我们不能把“容易验证”自动等同于“NP 完全”。

## 3. P 与 NP：求解和验证的区别（第 6–8 页）

### 3.1 先区分判定、搜索和最优化

以 SAT 为例：

- **判定：** 给定公式，是否存在满足赋值？输出 YES 或 NO。
- **搜索：** 找出一个满足赋值；若不存在，则报告无解。
- **验证：** 给定公式和赋值，检查该赋值是否满足公式。

对 TSP 还可以问最优化问题：“最短巡回路线的费用是多少？”这与“是否存在费用不超过 $K$ 的路线”是不同的输入输出约定。

讲义把可高效求解的搜索问题称作 P，把有高效验证器的搜索问题称作 NP。标准复杂性理论中的 P、NP 是**判定问题类**；对应的函数／搜索问题通常使用 FP、FNP 等术语。下文涉及补集与完全性时，采用判定定义，避免把“输出一个解”和“回答是否有解”混为一谈。

### 3.2 P：确定性多项式时间判定

把一个判定问题表示成它的所有 YES 实例组成的集合 $L$。若存在确定性算法，对每个输入 $I$ 都能在 $\operatorname{poly}(|I|)$ 时间内正确判断 $I\in L$，则

$$
L\in\mathrm P.
$$

算法必须在 YES 和 NO 实例上都按时结束并回答正确。不能只承诺“有解时最终能找到”。

这里的多项式相对于**编码长度**。例如整数 $N$ 的输入长度为 $\Theta(\log N)$，试除到 $\sqrt N$ 不能据此称作多项式时间算法。

### 3.3 NP：YES 实例有短证书，可以快速验证

$L\in\mathrm{NP}$，当且仅当存在多项式 $p$ 和确定性多项式时间验证器 $C$，使得

$$
\boxed{
I\in L
\iff
\exists S,\ |S|\le p(|I|),\ C(I,S)=1.
}
$$

它包含三项要求：

1. **短证书：** 证书长度至多为输入长度的某个固定多项式。
2. **完整性：** YES 实例至少有一个证书会被接受。
3. **可靠性：** NO 实例没有任何允许长度的证书会被接受。

验证器的时间通常写成 $\operatorname{poly}(|I|+|S|)$；结合证书长度上界，就得到 $\operatorname{poly}(|I|)$。

**讲义补全。** 第 6 页强调验证时间由 $|I|$ 的多项式控制。正式写定义时，应把证书长度约束一起写出，不能允许任意长的解而仍宣称其验证时间关于原实例是多项式。

| 判定问题 | YES 证书 | 验证内容 |
| --- | --- | --- |
| SAT | 变量赋值 | 每个子句至少一个文字为真 |
| TSP，预算 $K$ | 城市排列 | 每城一次、返回起点、总费用不超过 $K$ |
| 集合覆盖，预算 $K$ | 集合编号列表 | 集合数不超过 $K$，并集为全集 |
| 图同构 | 顶点双射 | 所有顶点对的邻接关系保持不变 |
| 独立集，阈值 $K$ | 顶点集合 | 至少 $K$ 个点，任意两点之间无边 |
| 合数判定 | 非平凡因子 $d$ | $1<d<N$ 且 $d\mid N$ |

验证一条路线费用不超过 $K$，只验证其**可行性**；它不等于证明这条路线是全局最优的。

### 3.4 NP 中的 N 是 nondeterministic

$$
\mathrm P=\text{polynomial time},\qquad
\mathrm{NP}=\text{nondeterministic polynomial time}.
$$

“非确定性”的直观描述是：猜一个多项式长度的证书，然后验证；只要有一个猜测分支成功，实例就是 YES。

它不是随机算法，也不是“non-polynomial”。NP 中当然可以包含容易的问题。

### 3.5 为什么 $\mathrm P\subseteq\mathrm{NP}$？

**补充证明。** 若 $L\in\mathrm P$，验证器可以忽略证书，直接运行 $L$ 的多项式时间判定算法。空字符串就足以作为 YES 实例的证书。因此

$$
\boxed{\mathrm P\subseteq\mathrm{NP}.}
$$

反方向是否成立就是 P vs NP 问题：容易验证是否意味着容易判定存在性？这一问题仍列在 Clay 的未解决问题中，参见 [P vs NP 官方说明](https://www.claymath.org/millennium/p-vs-np/)。

**补充：若 P = NP，能否找出证书？** 对多项式长度、可高效验证的证书关系，可以定义辅助判定问题：“是否存在一个以当前前缀开头的合法证书？”这个辅助问题属于 NP。若 P = NP，就能在多项式时间判定，并逐位固定证书，最终找出一个解。这里使用的是 P = NP 对**所有这些辅助问题**的保证；不是说任意搜索问题都自动能用它自身的原始判定器恢复解。

## 4. 补问题与 co-NP（第 9–10 页）

### 4.1 补问题翻转的是实例的 YES／NO

在同一个输入全集 $\Sigma^*$ 上，问题 $L$ 的补问题为

$$
\overline L=\Sigma^*\setminus L.
$$

讨论具体例子时，默认输入编码合法；若把问题扩展到所有字符串，需统一规定非法编码的处理。

以 TSP 判定为例：

$$
\begin{aligned}
\mathrm{TSP}:&\quad\exists\text{ 巡回路线 }T,\ c(T)\le K;\\
\overline{\mathrm{TSP}}:&\quad\nexists\text{ 巡回路线 }T,\ c(T)\le K.
\end{aligned}
$$

在完整距离图中，第二行也可写成“每条巡回路线的费用都大于 $K$”。

**常见错误：** 给出一条费用大于 $K$ 的路线，只说明这一条不合格。它不是“没有任何合格路线”的证明。

### 4.2 P 对补运算封闭

若 $L\in\mathrm P$，先运行其判定算法，再把 YES 和 NO 交换，就能判定 $\overline L$。运行时间只增加常数，所以

$$
L\in\mathrm P\Longrightarrow\overline L\in\mathrm P,
\qquad \boxed{\mathrm{coP}=\mathrm P.}
$$

为什么不能直接对 NP 验证器做同样的操作？因为

$$
\neg\bigl(\exists S:C(I,S)=1\bigr)
\iff
\forall S:C(I,S)=0.
$$

把一个候选证书的接受结果翻转，并不会把“存在一个成功证书”变成“所有证书都失败”。

### 4.3 co-NP 的定义

$$
\boxed{
\mathrm{coNP}=\{L:\overline L\in\mathrm{NP}\}.
}
$$

因此：

- $L\in\mathrm{NP}$：$L$ 的 YES 实例有可快速验证的短证书。
- $L\in\mathrm{coNP}$：$L$ 的 NO 实例有可快速验证的短证书，即 $\overline L$ 的 YES 证书。

例如 SAT 的满足赋值是 SAT 的 YES 证书，也是 UNSAT 的 NO 证书，所以

$$
\mathrm{SAT}\in\mathrm{NP},\qquad
\mathrm{UNSAT}\in\mathrm{coNP}.
$$

**辨析。** co-NP 不是“NP 以外的所有问题”，也不是 $\mathrm{NP}\setminus\mathrm P$。它是逐个取补问题后得到的问题类。

### 4.4 已知包含关系与四种示意情况

因为 P 对补封闭，且 $\mathrm P\subseteq\mathrm{NP}$，可得

$$
\boxed{\mathrm P\subseteq\mathrm{NP}\cap\mathrm{coNP}.}
$$

讲义第 10 页画出了四种关系。应把它们理解为对未知关系的分类，而不是已经证明的复杂性分布图。

| 情形 | 关系 |
| --- | --- |
| 1 | $\mathrm P=\mathrm{NP}=\mathrm{coNP}$ |
| 2 | $\mathrm P\subsetneq\mathrm{NP}=\mathrm{coNP}$ |
| 3 | $\mathrm{NP}\ne\mathrm{coNP}$，且 $\mathrm P=\mathrm{NP}\cap\mathrm{coNP}$ |
| 4 | $\mathrm{NP}\ne\mathrm{coNP}$，且 $\mathrm P\subsetneq\mathrm{NP}\cap\mathrm{coNP}$ |

**补充。** 若 $\mathrm{NP}\subseteq\mathrm{coNP}$，对两边的问题逐个取补就得到反向包含，所以两者相等。故 NP 与 co-NP 若不相等，就不可能一个真包含另一个。

需要牢牢记住的推论是

$$
\mathrm P=\mathrm{NP}
\Longrightarrow
\mathrm P=\mathrm{NP}=\mathrm{coNP}.
$$

而 $\mathrm{NP}=\mathrm{coNP}$ 并不能据此推出 $\mathrm P=\mathrm{NP}$；表中的情形 2 没有被这一等式排除。

## 5. 有界定理证明与指数时间假设（第 11–12 页）

### 5.1 P = NP 对定理证明意味着什么？（第 11 页）

讲义给出一个动机：输入数学命题 $\varphi$ 和长度上界 $n$，寻找一个长度至多 $n$ 的形式化证明。

**讲义补全：必须限定编码与证明系统。** 固定一个可以多项式时间检查证明的形式系统。若输入写成

$$
(\varphi,1^n),
$$

即 $n$ 使用一元编码，那么输入长度至少是 $n$，长度不超过 $n$ 的证明就是多项式长度证书。于是“是否存在这样的证明”的判定问题属于 NP。

若 $n$ 使用二进制编码，$n$ 可能相对于输入长度指数大，不能仅凭“证明可快速检查”就断言这个有界证明问题属于 NP。另一种足够的条件，是预先把证明长度限制为 $|\varphi|$ 的某个固定多项式。

如果 P = NP，就能高效判定并搜索**在给定界内存在的证明**。但讲义“能证明任何定理、因而不再需要数学家”的说法只是动机性的夸张：

- 时间界依赖允许的证明长度，不仅依赖命题本身的长度。
- 定理不一定有短证明，也不一定在固定形式系统中可证。
- 多项式算法的次数和常数仍可能使实际计算不可行。

第 11 页标题“P $\ne$ NP”表达讨论背景，并不是一个已经完成的分离证明。

### 5.2 3-SAT 的指数时间假设（第 12 页）

设 $n$ 是 3-CNF 公式的变量数，$\ell$ 是公式的编码长度。**指数时间假设（Exponential Time Hypothesis，ETH）** 断言，3-SAT 没有最坏情况运行时间为 $2^{o(n)}\operatorname{poly}(\ell)$ 的确定性判定算法。讲义省略关于输入长度的多项式因子，简写为 $2^{o(n)}$。

在 $2^{g(n)}$ 中，$g(n)=o(n)$ 表示 $g(n)/n\to 0$。把枚举全部赋值的 $2^n$ 降到 $2^{n/2}$，指数与 $n$ 的比值仍为 $1/2$。若 3-SAT 有 $2^{\sqrt n}\operatorname{poly}(\ell)$ 时间算法，由于 $\sqrt n/n\to 0$，就足以反驳 ETH。

**补充辨析。** 若 $\mathrm P=\mathrm{NP}$，3-SAT 就有 $\operatorname{poly}(\ell)$ 时间算法，已经达到 ETH 所排除的时间界。因此 $\mathrm{ETH}\Rightarrow\mathrm P\ne\mathrm{NP}$。上例中的 $2^{\sqrt n}$ 比任意固定次数的 $n^k$ 增长得都快。要由 3-SAT 推出 P = NP，仍需证明它有关于 $\ell$ 的多项式时间算法。

**讲义中的研究进展。** 2026 年 10 月，OpenAI 和 Anthropic 分别报告，其内部大语言模型找到了 3-SAT 的次指数时间算法，并据此声称反驳了 ETH。Russell Impagliazzo、Ramamohan Paturi 和 Ryan Williams 邀请学界独立核查反证主张，其中前两人参与提出了 ETH。[OpenAI 公告](https://openai.com/index/building-scientific-understanding-with-academic-communities/) · [Anthropic 预印本](https://github.com/anthropics/formal-math/blob/16a5ce41304fcdaea4e0be7c132e1c361dc26da5/eth/paper.pdf) · [核查小组说明](https://ethdiscussion.blog/2026/10/26/eth-refutation-examination-and-discussion/)

## 6. 多项式时间归约：把 B 的算法变成 A 的算法（第 13–16 页）

### 6.1 讲义中的搜索归约

要把搜索问题 $A$ 归约到搜索问题 $B$，需要两种翻译：

1. **实例翻译 $f$：** 把 $A$ 的实例 $I$ 转成 $B$ 的实例 $f(I)$。
2. **解的翻译 $h$：** 把 $f(I)$ 的任意合法解 $S$ 转成 $I$ 的合法解。

两种翻译都必须在多项式时间内完成，还必须保证

$$
f(I)\text{ 无解}\Longrightarrow I\text{ 无解}.
$$

更一般地，把解的翻译写成 $h(I,S)$，因为恢复原问题的解时可以使用原实例。讲义简写为 $h(S)$。

```text
SOLVE_A(I):
    J = f(I)
    result = SOLVE_B(J)
    if result is NO_SOLUTION:
        return NO_SOLUTION
    return h(I, result)
```

上述条件实际上给出两个方向：

- 若 $I$ 有解，$f(I)$ 必须有解；否则由无解条件产生矛盾。
- 若 $f(I)$ 有解，任取其合法解，经 $h$ 就得到 $I$ 的解。

因此必须有

$$
\boxed{I\text{ 有解}\iff f(I)\text{ 有解}.}
$$

不能只保证“原问题有解时，新问题也有解”，却放任原本无解的实例被转换成有解的实例。

### 6.2 判定版本：多项式时间多对一归约

对判定问题 $A,B$，记

$$
A\le_p B
$$

表示存在可在多项式时间计算的函数 $f$，使对所有实例 $I$：

$$
\boxed{I\in A\iff f(I)\in B.}
$$

此时只需构造一个 $B$ 实例，再把 $B$ 的 YES／NO 答案作为 $A$ 的答案。它不要求输出或恢复具体证书；若讨论搜索版本，则还要给出解的翻译。

可以用下面的流程理解：

```text
A 的实例 I --多项式时间 f--> B 的实例 f(I) --B 的判定器--> YES / NO
```

归约本身不能把答案当作已知条件。例如“先解决 $A$，再根据答案输出一个固定的 B-YES 或 B-NO 实例”，只有在先解决 $A$ 本来就能多项式完成时，才是合法的多项式归约。

**辨析。** Lecture 15 中用多次 SAT 判定调用恢复赋值，是更一般的多项式时间图灵归约／自归约，不应不加说明地当作这里“一次实例转换”的多对一归约。

### 6.3 为什么输出规模也是关键？

若 $f$ 在 $O(\ell^a)$ 时间内运行，就只能显式输出多项式多个符号，因此

$$
|f(I)|\le O(\ell^a).
$$

若 $B$ 的判定器耗时 $O(|J|^b)$，则解决 $A$ 的总时间至多为

$$
O(\ell^a)+O((\ell^a)^b)
=O(\ell^a+\ell^{ab}),
$$

仍然是多项式。搜索版本还需加上 $h$ 的多项式开销。

例如把一个二进制整数 $N$ 展开为长度 $N$ 的一元串，可能产生指数长输出，因而不是关于原输入位数的多项式时间转换。

### 6.4 归约的方向：容易向左，困难向右

$A\le_p B$ 的含义是：**如果能解决 B，就能解决 A；B 至少能够承担 A 的计算任务。**

| 已知信息 | 可以推出 |
| --- | --- |
| $A\le_p B$ 且 $B\in\mathrm P$ | $A\in\mathrm P$ |
| $A\le_p B$ 且 $A\notin\mathrm P$ | $B\notin\mathrm P$ |
| $A\le_p B$ 且 $A$ 为 NP-hard | $B$ 为 NP-hard |
| $A\le_p B$ 且 $A\in\mathrm P$ | 不能仅据此判断 $B$ 是否容易 |
| $A\le_p B$ 且 $B$ 很难 | 不能仅据此判断 $A$ 是否很难 |

例如 2-SAT 可以直接看作 SAT 的特殊情况，因此 $\mathrm{2SAT}\le_p\mathrm{SAT}$。这不妨碍 2-SAT 属于 P，也不会因为 SAT 是 NP 完全就使 2-SAT 自动成为 NP 完全。

### 6.5 归约具有传递性

若

$$
A\le_p B\quad\text{通过 }f,
\qquad
B\le_p C\quad\text{通过 }g,
$$

则

$$
I\in A
\iff f(I)\in B
\iff g(f(I))\in C.
$$

多项式时间函数的复合仍可在多项式时间内计算，所以

$$
\boxed{A\le_p B,\ B\le_p C\Longrightarrow A\le_p C.}
$$

### 6.6 补充例子：独立集归约到顶点覆盖

定义两个判定问题：

- $\mathrm{IS}(G,K)$：是否存在大小至少 $K$ 的独立集？
- $\mathrm{VC}(G,k)$：是否存在大小至多 $k$ 的顶点覆盖？

对 $0\le K\le n=|V|$，构造

$$
f(G,K)=(G,n-K).
$$

关键等价关系是

$$
S\text{ 是独立集}
\iff
V\setminus S\text{ 是顶点覆盖}.
$$

**证明。** $S$ 中不存在两端都在 $S$ 的边，等价于每条边至少有一个端点落在 $V\setminus S$ 中。因此

$$
\begin{aligned}
G\text{ 有大小至少 }K\text{ 的独立集}
\iff
G\text{ 有大小至多 }n-K\text{ 的顶点覆盖}.
\end{aligned}
$$

若得到顶点覆盖 $C$，则输出 $V\setminus C$，就恢复了原实例的独立集。这里实例转换和解恢复都很简单，真正需要证明的是那个“当且仅当”。

## 7. NP-hard 与 NP-complete（第 17–19 页）

### 7.1 定义中有两个独立条件

一个判定问题 $B$ 是 **NP-hard（NP 困难）**，若

$$
\forall A\in\mathrm{NP},\quad A\le_p B.
$$

它是 **NP-complete（NP 完全，NPC）**，若同时满足

$$
\boxed{
B\in\mathrm{NP}
\quad\text{且}\quad
\forall A\in\mathrm{NP},\ A\le_p B.
}
$$

即

$$
\boxed{\text{NP-complete}=\text{属于 NP}+\text{NP-hard}.}
$$

NP-hard 不要求问题本身属于 NP。最优化 TSP 通常称为 NP-hard；称其“NP 完全”时，标准含义应是带费用阈值的判定版本。

**讲义修正。** 第 19 页 “all other search problems” 应结合上下文理解为“所有 NP 中的问题”，不能扩大成所有可计算问题，甚至所有不可判定问题。

### 7.2 为什么一个 NP 完全问题进 P，就全部解决？

设 $B$ 为 NP 完全，且 $B\in\mathrm P$。

任取 $A\in\mathrm{NP}$。由完全性，$A\le_p B$；由归约的算法用途，$A\in\mathrm P$。于是

$$
\mathrm{NP}\subseteq\mathrm P.
$$

结合本来就成立的 $\mathrm P\subseteq\mathrm{NP}$，得到

$$
\boxed{
B\text{ 为 NP 完全且 }B\in\mathrm P
\Longrightarrow\mathrm P=\mathrm{NP}.
}
$$

这就是讲义“Solve One and All Solved”的精确含义。它不是说所有 NP 问题的定义相同，而是说它们的求解能力由归约联系起来。

### 7.3 证明新问题 NP 完全的标准模板

目标是证明新问题 $B$ 为 NP 完全：

1. **证明 $B\in\mathrm{NP}$。** 写出证书、长度上界、验证步骤及其时间。
2. **选择已知 NP 完全的问题 $A$。** 明确其输入和 YES 条件。
3. **构造 $A\le_p B$。** 把任意 $A$ 实例 $I$ 转换为 $f(I)$。
4. **证明双向正确性。** 分别证明 $I\in A\Rightarrow f(I)\in B$ 和 $f(I)\in B\Rightarrow I\in A$。
5. **证明构造规模与时间为多项式。** 然后由传递性得到 $B$ 为 NP-hard，再结合第 1 步得 NP 完全。

方向必须是

$$
\boxed{\text{已知困难问题}\le_p\text{待证明困难的新问题}.}
$$

只证明 $B\le_p A$，说明 $A$ 的算法可用于 $B$，不能证明 $B$ 足够难。只完成困难性归约而没有证明 $B\in\mathrm{NP}$，则只证明了 NP-hard。

**补充：第一个 NP 完全问题从哪里来？** 不能无限地依赖“另一个已知 NP 完全问题”。Cook–Levin 定理直接证明 SAT 的 NP 完全性：把任意非确定性多项式时间计算编码成可满足性实例。本讲没有展开这个基础定理的构造；后续具体归约可以从已建立的完全性结果出发。

### 7.4 补充例题：由顶点覆盖证明集合覆盖的困难性

以下以“顶点覆盖判定是 NP 完全问题”为已知前提，演示完整归约。

给定顶点覆盖实例 $(G=(V,E),k)$，构造集合覆盖实例：

- 全集为 $U=E$，每一条原图边成为一个待覆盖元素。
- 对每个顶点 $v$，创建集合

$$
S_v=\{e\in E:e\text{ 与 }v\text{ 关联}\}.
$$

- 集合数预算保持为 $k$。

**正方向。** 若 $C\subseteq V$ 是大小不超过 $k$ 的顶点覆盖，则每条边都与某个 $v\in C$ 关联。因此集合族 $\{S_v:v\in C\}$ 覆盖 $U$，使用的集合数不超过 $k$。

**反方向。** 若选取不超过 $k$ 个集合就覆盖 $U$，取这些集合对应的顶点构成 $C$。每条边作为全集元素都被某个选中集合覆盖，故至少一个端点在 $C$ 中。因此 $C$ 是大小不超过 $k$ 的顶点覆盖。

构造包含 $|V|$ 个集合，每条边在两个端点的集合中各出现一次，集合元素出现总数为 $2|E|$，所以规模和构造时间都为多项式。于是

$$
\mathrm{VERTEX\ COVER}\le_p\mathrm{SET\ COVER}.
$$

结合第 2.1 节的 NP 成员资格，得到集合覆盖判定版为 NP 完全。这也解释了为什么不能省掉预算 $k$。

## 8. co-NP 完全性及其与 NP 完全性的联系（第 20 页）

### 8.1 定义

$B$ 为 co-NP 完全，当且仅当

$$
B\in\mathrm{coNP}
\quad\text{且}\quad
\forall A\in\mathrm{coNP},\ A\le_p B.
$$

与 NP 完全性完全对应：先属于这个类，再证明这个类中的所有问题都能归约到它。

### 8.2 对两端同时取补，归约仍成立

若 $A\le_p B$ 通过 $f$ 成立，则

$$
I\in A\iff f(I)\in B
$$

两侧同时取否定，得到

$$
I\in\overline A\iff f(I)\in\overline B.
$$

因此同一个 $f$ 证明

$$
\boxed{A\le_p B\Longrightarrow\overline A\le_p\overline B.}
$$

注意是**两端同时**取补；它没有自动给出 $A\le_p\overline B$。

### 8.3 NP 完全与 co-NP 完全互为补问题

$$
\boxed{
L\text{ 是 NP 完全}
\iff
\overline L\text{ 是 co-NP 完全}.
}
$$

**补充证明。** 假设 $L$ 为 NP 完全：

1. $L\in\mathrm{NP}$，所以 $\overline L\in\mathrm{coNP}$。
2. 任取 $A\in\mathrm{coNP}$，则 $\overline A\in\mathrm{NP}$。
3. 由 $L$ 的完全性，$\overline A\le_p L$。
4. 对两端取补，得到 $A\le_p\overline L$。

这就证明 $\overline L$ 为 co-NP 完全。反方向同理。特别地，CNF-SAT 为 NP 完全，所以 CNF-UNSAT 为 co-NP 完全。

### 8.4 若一个问题及其补问题都 NP 完全，会发生什么？

设 $L$ 和 $\overline L$ 都为 NP 完全。因为 $\overline L$ 为 NP 完全，上一结论说明 $L$ 为 co-NP 完全；而 $L$ 本身又属于 NP。

任取 $A\in\mathrm{coNP}$，有 $A\le_p L$。先做归约，再使用 $L$ 的验证器，就给出了 $A$ 的 NP 验证器，因此

$$
\mathrm{coNP}\subseteq\mathrm{NP}.
$$

对补问题使用同样关系可得反向包含，故

$$
\boxed{\mathrm{NP}=\mathrm{coNP}.}
$$

这里用到了一个基本事实：若 $A\le_p B$ 且 $B\in\mathrm{NP}$，则 $A\in\mathrm{NP}$。因为 $f(I)$ 的长度是 $|I|$ 的多项式，$B$ 的短证书也是 $A$ 的多项式长度证书。

**常用的更一般结论。** 只要存在一个 NP 完全问题属于 co-NP，就足以推出 NP = co-NP；对称地，一个 co-NP 完全问题若属于 NP，也会推出两类相等。这不直接推出 P = NP。

## 9. TAUTOLOGY：为什么输入是 DNF 很重要？（第 21 页）

### 9.1 可满足、不可满足与永真

| 性质 | 量词形式 |
| --- | --- |
| 可满足 | 存在赋值 $a$，使 $f(a)=1$ |
| 不可满足 | 对所有赋值 $a$，都有 $f(a)=0$ |
| 永真（tautology） | 对所有赋值 $a$，都有 $f(a)=1$ |

直接得到

$$
\boxed{f\text{ 不可满足}\iff\neg f\text{ 永真}.}
$$

“不是永真式”的证书非常简单：给出一个使公式为假的赋值即可。所以永真式判定属于 co-NP。

### 9.2 否定一个 CNF，得到线性大小的 DNF

设

$$
f=\bigwedge_{i=1}^{m}\left(\bigvee_j\ell_{ij}\right).
$$

应用德摩根律：

$$
\boxed{
\neg f
=\bigvee_{i=1}^{m}\left(\bigwedge_j\neg\ell_{ij}\right).
}
$$

原来的每个“或子句”变成一个“与项”，外层合取变成析取，文字出现次数保持不变，因此转换规模为线性。

**辨析。** 这里是把 CNF 的**否定**写成 DNF；不是把同一个 CNF 公式等价展开成 DNF。后一种操作可能因为分配律而导致指数级膨胀。

**补充例子。** 令

$$
f=(x\lor y)\land(\neg x)\land(\neg y).
$$

后两个子句迫使 $x=y=0$，与第一个子句矛盾，故 $f$ 不可满足。其否定为

$$
g=(\neg x\land\neg y)\lor x\lor y.
$$

若 $x,y$ 至少一个为真，则 $x\lor y$ 为真；若两者都为假，则第一项为真。因此 $g$ 永真。

### 9.3 DNF-TAUTOLOGY 为 co-NP 完全

本讲的 TAUTOLOGY 输入约定是：**给定一个 DNF 公式，判断它是否永真。**

证明分两步：

1. **属于 co-NP：** 一个使公式为假的赋值是 NO 实例的短证书，可以逐项验证。
2. **co-NP-hard：** 将 CNF-UNSAT 的实例 $f$ 转成 DNF 公式 $\neg f$；由上面的等价式，

$$
f\in\mathrm{CNF\text{-}UNSAT}
\iff
\neg f\in\mathrm{DNF\text{-}TAUTOLOGY}.
$$

转换是多项式时间的，故

$$
\boxed{\mathrm{DNF\text{-}TAUTOLOGY}\text{ 是 co-NP 完全问题}.}
$$

### 9.4 推导讲义中的两个等价关系

记 $T=\mathrm{DNF\text{-}TAUTOLOGY}$。

**结论一：**

$$
\boxed{T\in\mathrm P\iff\mathrm{coNP}=\mathrm P.}
$$

若 $T\in\mathrm P$，任何 co-NP 问题都归约到 $T$，所以 $\mathrm{coNP}\subseteq\mathrm P$；结合 $\mathrm P\subseteq\mathrm{coNP}$ 得到相等。反方向由 $T\in\mathrm{coNP}$ 立即成立。又因为 P 对补封闭，这也等价于 P = NP。

**结论二：**

$$
\boxed{T\in\mathrm{NP}\iff\mathrm{coNP}=\mathrm{NP}.}
$$

若 $T\in\mathrm{NP}$，同样利用完全性和 NP 对多项式归约的向下封闭性，得 $\mathrm{coNP}\subseteq\mathrm{NP}$，再取补得相等。反方向显然。

### 9.5 补充对比：交换 CNF／DNF 会改变难度

| 输入形式 | 问“可满足吗？” | 问“永真吗？” |
| --- | --- | --- |
| CNF | NP 完全 | 属于 P |
| DNF | 属于 P | co-NP 完全 |

原因如下：

- **DNF 可满足：** 只要某个与项内部不同时包含 $x$ 与 $\neg x$，就能令该项为真。
- **CNF 永真：** 当且仅当每个或子句都永真；对纯文字子句，这要求每个子句都含某对 $x,\neg x$。

这里使用标准约定：空合取为真、空析取为假；若公式中显式出现布尔常量，可先化简。考试中看到 TAUTOLOGY 时，要先确认题目规定的公式表示形式。

## 10. 因数分解、质数与合数：三个不同任务（第 22–23 页）

### 10.1 先明确输入输出

| 问题 | 输入 | 输出要求 |
| --- | --- | --- |
| PRIMALITY／PRIMES | 整数 $N\ge2$ | 判断 $N$ 是否为质数 |
| COMPOSITE | 整数 $N\ge2$ | 判断是否存在 $p,q\ge2$，使 $N=pq$ |
| FACTORING | 整数 $N\ge2$ | 实际输出 $N$ 的完整质因数分解 |

**讲义修正。** 第 23 页的 “Primarily” 应为 **Primality**，即质数判定。讲义对 COMPOSITE 写 $N\ge4$；为统一补问题的定义，这里让两者都接收 $N\ge2$，并把 2、3 在 COMPOSITE 中判为 NO。

在这个共同输入域上，质数判定与合数判定互为补问题。对合数 $N$，一个非平凡因子 $d$ 就是短证书，检查整除关系可在关于 $\log N$ 的多项式时间内完成。

### 10.2 质数判定已经属于 P

Agrawal、Kayal、Saxena 的 AKS 算法给出了确定性的多项式时间质数判定算法。因此

$$
\boxed{\mathrm{PRIMES}\in\mathrm P,\qquad\mathrm{COMPOSITE}\in\mathrm P.}
$$

第二个结论也可由 P 对补封闭直接推出。这里“多项式时间”指关于输入位数 $\log N$ 的多项式。参见 [AKS 原始论文《PRIMES is in P》](https://cse.iitk.ac.in/users/manindra/algebra/primality_original.pdf)。

**重要区别：** 判断一个数是合数，只需回答 YES；因数分解要求实际找出因子。上述判定算法不自动提供因数分解算法。

### 10.3 “总有质因数分解”说明了什么？

每个整数 $N\ge2$ 都有完整质因数分解；若 $N$ 是质数，其分解就是 $N$ 本身。因此完整因数分解是一个**总有输出的搜索任务**。

但这不意味着输出很容易找到，也不能仅凭“总有解”就证明它不是 NP-hard。若把它机械地改成“是否存在完整质因数分解”，则所有合法输入的答案都为 YES，丢失了原来的搜索内容。

讲义说 FACTORING 与 SAT、TSP 的难度性质不同，应理解为在提醒我们区分搜索任务和相应的判定表述。“没有人相信它是 NP 完全”是直觉性的表述，不是完全性分类的证明。

### 10.4 补充：一个适合讨论 NP 与 co-NP 的因数判定版本

定义带界的因数判定问题，输入 $N\ge2$ 和 $1\le K<N$：

$$
\mathrm{FACTOR}_{\le}
=\{(N,K):\exists d,\ 1<d\le K,\ d\mid N\}.
$$

它问的是“有没有**不超过指定界**的非平凡因子”，比单纯问“是不是合数”更细。

**属于 NP。** YES 证书为因子 $d$，验证大小和整除即可。

**属于 co-NP。** NO 证书可以取完整质因数列表

$$
N=p_1p_2\cdots p_r,
$$

其中允许重复。验证以下条件：

1. 每个 $p_i$ 都是质数，可用多项式时间质数判定。
2. 所有 $p_i$ 的乘积等于 $N$。
3. 每个 $p_i>K$。

若存在 $1<d\le K$ 且 $d\mid N$，$d$ 必有一个质因子不超过 $K$，与第 3 条矛盾。反之，若不存在这样的 $d$，则 $N$ 的每个质因子都大于 $K$，所以完整分解确实能通过验证。

由于 $r\le\log_2N$，且因子列表的编码总长度为 $\log N$ 的多项式，得到

$$
\boxed{\mathrm{FACTOR}_{\le}\in\mathrm{NP}\cap\mathrm{coNP}.}
$$

于是有一个严谨的条件结论：

$$
\mathrm{FACTOR}_{\le}\text{ 若为 NP 完全}
\Longrightarrow
\mathrm{NP}=\mathrm{coNP}.
$$

它比“因为总有解，所以不可能 NP 完全”更准确，但仍不是无条件的非完全性证明。

**与搜索的联系。** 对固定 $N$，上述判定随 $K$ 增大具有单调性。若能高效判定，就可二分寻找最小非平凡因子，除去该因子后继续处理剩余整数，恢复完整分解。二分调用次数为 $O(\log N)$，重复分解次数也至多 $O(\log N)$，因而这是多项式时间的图灵归约。若已有完整分解，检查最小质因子是否不超过 $K$ 则立即给出判定答案。

### 10.5 量子算法不改变这里 P 的定义

Shor 给出了量子计算模型下的多项式时间因数分解算法，时间相对于整数输入位数计量，参见 [Shor 的原始论文](https://arxiv.org/abs/quant-ph/9508027)。

本讲的 P 指**经典确定性计算模型**中的多项式时间。因此量子算法的存在不等于证明经典因数分解属于 P。Shor 算法也没有给出 SAT、TSP 的多项式算法，更不能由此推出它们不可能有量子多项式算法。

## 11. NP-intermediate 与讲义中的关系图（第 24–25 页）

### 11.1 NPI 的标准定义

NP 中间问题（NP-intermediate）是：属于 NP，但既不属于 P，也不是 NP 完全的问题。

记 NPC 为所有 NP 完全判定问题的集合，则

$$
\boxed{
\mathrm{NPI}
=\mathrm{NP}\setminus(\mathrm P\cup\mathrm{NPC}).
}
$$

因此从定义就知道

$$
\mathrm{NPI}\cap\mathrm P=\varnothing,
\qquad
\mathrm{NPI}\cap\mathrm{NPC}=\varnothing.
$$

**辨析。** “目前不知道一个问题在 P，也不知道它是否 NP 完全”只是我们知识上的缺口。要证明它属于 NPI，必须证明它**不在 P**，并证明它**不是 NP 完全**；不能用“尚未找到算法／归约”代替这两项证明。

### 11.2 Ladner 定理：NP 内不一定只有容易和最难两档

讲义第 25 页的 “Lander Theorem” 应为 **Ladner's theorem（Ladner 定理）**。

$$
\boxed{
\mathrm P\ne\mathrm{NP}
\Longrightarrow
\mathrm{NPI}\ne\varnothing.
}
$$

也就是说，如果 P 与 NP 确实不同，那么 NP 中必然存在既不能多项式时间求解、又不足以代表整个 NP 难度的问题。原始结果为 Richard Ladner 的 *On the Structure of Polynomial Time Reducibility*，可核对其 [作者论文目录](https://homes.cs.washington.edu/~ladner/papers.html)。本讲使用定理结论，没有给出构造证明。

反方向由定义立即成立：若有一个问题属于 NPI，它就在 NP 中而不在 P，因此 P $\ne$ NP。故也可写成

$$
\mathrm P\ne\mathrm{NP}
\iff
\mathrm{NPI}\ne\varnothing.
$$

当 P $\ne$ NP 时，NP 内可按下表分类：

| 类别 | 可多项式时间求解？ | NP 完全？ |
| --- | --- | --- |
| P | 是 | 否，在 P $\ne$ NP 的假设下 |
| NPI | 否 | 否 |
| NPC | 否，在 P $\ne$ NP 的假设下 | 是 |

这是一种**条件下的分类**，不是已经证明 P $\ne$ NP。

### 11.3 为什么不能直接照抄第 24 页的图？

第 24 页标题已经注明 “A Problematic Category”。其中嵌入的旧图及文字存在几个需要辨析的地方：

1. **图中的 NPI 覆盖了 NP 与 co-NP 的交集，并包含 P。** 这与下一页“既不在 P、也不是 NP 完全”的 NPI 定义不同。按标准 NP-intermediate 定义，$\mathrm P\subseteq\mathrm{NPI}$ 是错误的。
2. **图把 PRIME、COMPOSITE 放在 P 外。** AKS 定理已经证明它们都在 P 中，不能将这一位置当作有效分类。
3. **不能由图推出 NPI 对补封闭。** 对补封闭的是 $\mathrm{NP}\cap\mathrm{coNP}$：若一个问题及其补问题都在 NP 中，交换二者后仍然如此。标准 NPI 不是这个交集。
4. **图把 NP 完全区与 co-NP 完全区分开。** 这应视为某种猜想关系的示意，不是已经证明 NP $\ne$ co-NP。

应分别记忆下面两个式子，而不要混用其名称：

$$
\underbrace{\mathrm P\subseteq\mathrm{NP}\cap\mathrm{coNP}}_{\text{交集包含 P}},
\qquad
\underbrace{\mathrm{NPI}=\mathrm{NP}\setminus(\mathrm P\cup\mathrm{NPC})}_{\text{NPI 排除 P}}.
$$

**进一步辨析。** 若假设 NP $\ne$ co-NP，那么 $(\mathrm{NP}\cap\mathrm{coNP})\setminus\mathrm P$ 中的每个问题都属于 NPI：它已经不在 P，若再是 NP 完全，就会由它属于 co-NP 推出 NP = co-NP，矛盾。但这并不说明所有 NPI 问题都落在 NP 与 co-NP 的交集中。

## 12. Quiz：若 NP ≠ co-NP，则 P ≠ NP（第 26 页）

题目要求证明

$$
\mathrm{NP}\ne\mathrm{coNP}
\Longrightarrow
\mathrm P\ne\mathrm{NP}.
$$

最直接的方法是证明逆否命题：

$$
\mathrm P=\mathrm{NP}
\Longrightarrow
\mathrm{NP}=\mathrm{coNP}.
$$

**证明。** 假设 P = NP。任取 $L\in\mathrm{NP}$，由假设有 $L\in\mathrm P$。P 对补封闭，因此

$$
\overline L\in\mathrm P=\mathrm{NP},
$$

按 co-NP 的定义，$L\in\mathrm{coNP}$，从而 $\mathrm{NP}\subseteq\mathrm{coNP}$。

反过来，任取 $L\in\mathrm{coNP}$，则 $\overline L\in\mathrm{NP}=\mathrm P$。再由 P 对补封闭，得到 $L\in\mathrm P=\mathrm{NP}$。因此 $\mathrm{coNP}\subseteq\mathrm{NP}$。

综上 NP = co-NP，逆否命题成立，原命题得证。

**考试简写：**

$$
\mathrm P=\mathrm{NP}
\Longrightarrow
\mathrm{coNP}=\mathrm{coP}=\mathrm P=\mathrm{NP},
$$

其中 $\mathrm{coP}=\mathrm P$ 使用了 P 对补运算封闭。

**不要写反。** 题目没有要求、也没有证明

$$
\mathrm P\ne\mathrm{NP}\Longrightarrow\mathrm{NP}\ne\mathrm{coNP}.
$$

P 严格小于 NP = co-NP 的情形仍与这个 Quiz 的结论相容。

## 13. 核心结论与易错点速查

### 13.1 必须掌握的公式

| 内容 | 结论 |
| --- | --- |
| NP 证书定义 | $I\in L\iff\exists S,\ \lvert S\rvert\le p(\lvert I\rvert),\ C(I,S)=1$ |
| 基本包含 | $\mathrm P\subseteq\mathrm{NP}\cap\mathrm{coNP}$ |
| co-NP 定义 | $L\in\mathrm{coNP}\iff\overline L\in\mathrm{NP}$ |
| ETH 若成立 | 3-SAT 没有 $2^{o(n)}\operatorname{poly}(\ell)$ 时间的确定性算法，因此 $\mathrm P\ne\mathrm{NP}$（$n$ 为变量数） |
| 多对一归约 | $A\le_p B\iff\exists$ 多项式时间 $f$，使 $I\in A\iff f(I)\in B$ |
| 易解性传递 | $A\le_p B,\ B\in\mathrm P\Rightarrow A\in\mathrm P$ |
| 归约传递性 | $A\le_p B,\ B\le_p C\Rightarrow A\le_p C$ |
| 对补保持归约 | $A\le_p B\Rightarrow\overline A\le_p\overline B$ |
| NP 完全 | 属于 NP，并且所有 NP 问题都归约到它 |
| NP 完全问题进入 P | 推出 P = NP |
| 完全性与补问题 | $L$ NP 完全 $\iff\overline L$ co-NP 完全 |
| 完全问题跨入另一类 | NP 完全问题属于 co-NP $\Rightarrow$ NP = co-NP |
| DNF-TAUTOLOGY | co-NP 完全；属于 P $\iff$ P = NP；属于 NP $\iff$ NP = co-NP |
| NPI | $\mathrm{NP}\setminus(\mathrm P\cup\mathrm{NPC})$ |
| Ladner 定理 | P $\ne$ NP $\Rightarrow$ NPI 非空 |
| Quiz | NP $\ne$ co-NP $\Rightarrow$ P $\ne$ NP |

### 13.2 易错说法与修正

| 容易误记的说法 | 正确理解 |
| --- | --- |
| NP 就是“非多项式” | NP 是非确定性多项式时间，且包含 P |
| 验证器能快速运行就够了 | 还要保证 YES 证书长度为输入长度的多项式 |
| 求最优覆盖只是找一组能覆盖全集的集合 | 不限制数量时可选所有集合；困难来自预算或最小化 |
| 图同构就是按图中同色顶点对应 | 本讲定义只要求双射保持邻接关系，不要求保持绘图颜色 |
| 给出一个坏解就证明无解 | 只排除了这一个候选解 |
| co-NP 是 NP 的集合补集 | co-NP 是 NP 中各判定问题取补后形成的问题类 |
| NP $\cap$ co-NP 就是 P | 只知道 P 包含于该交集，不知道是否相等 |
| 3-SAT 有次指数时间算法就能推出 P = NP | 这样的算法足以反驳 ETH。P = NP 要求 3-SAT 有多项式时间算法 |
| 证明新问题难，应把它归约到 SAT | 应把已知 NP 完全问题归约到新问题 |
| 只证明归约一个方向即可 | 必须保证原实例 YES 当且仅当新实例 YES |
| 构造规则短，就一定是多项式归约 | 还要检查实际输出规模和执行时间 |
| NP-hard 一定属于 NP | 不要求；NP 完全才同时要求属于 NP |
| NP 完全意味着已证明不存在多项式算法 | 需要 P $\ne$ NP 这一尚未证明的假设 |
| 任意公式的 CNF、DNF 都能无代价互换 | 一般等价展开可能指数膨胀；本讲用的是对 CNF 取否定 |
| TAUTOLOGY 不论输入形式都同样难 | DNF 永真是 co-NP 完全，CNF 永真可多项式判定 |
| 质数判定和找质因子是同一个任务 | 前者输出 YES／NO，后者必须输出实际因子 |
| 总有解就一定容易、一定不 NP-hard | 存在性不自动给出构造算法，也不是非困难性的证明 |
| NPI 就是 NP $\cap$ co-NP，包含 P | 标准 NPI 排除 P，也排除 NP 完全问题 |
| 没证明在 P，也没证明 NP 完全，所以属于 NPI | “不知道”不能代替“不属于”的证明 |
| Ladner 定理已经证明 P $\ne$ NP | 它是以 P $\ne$ NP 为前提的条件定理 |
| NP = co-NP 就能推出 P = NP | 本讲只证明 P = NP $\Rightarrow$ NP = co-NP |

## 14. 复习检查清单

- [ ] 能区分集合覆盖的无预算可行性、带预算判定、搜索与最优化版本。
- [ ] 能写出图同构的双射条件，并解释度数序列相同为何还不够。
- [ ] 能完整写出 NP 的证书长度、验证器和“当且仅当”条件。
- [ ] 能说明 NP 的 N 为什么不是“non-polynomial”。
- [ ] 能证明 P $\subseteq$ NP，以及 P 对补运算封闭。
- [ ] 能用量词解释为什么翻转 NP 验证器的答案不能证明 NP 对补封闭。
- [ ] 能准确区分 co-NP、NP 的集合补集、NP $\cap$ co-NP 和 NPI。
- [ ] 能说明有界定理证明为何需要一元长度界或多项式长度限制。
- [ ] 能按变量数 $n$ 陈述 ETH，并用 $2^{\sqrt n}$ 解释反驳 ETH 与证明 P = NP 的区别。
- [ ] 能写出搜索归约的 $f,h$，并分别解释有解与无解时的正确性。
- [ ] 能说明多对一归约与多次调用判定器的图灵归约有什么区别。
- [ ] 能用 $A\le_p B$ 正确传递易解性和困难性，不把箭头写反。
- [ ] 能给出独立集到顶点覆盖的阈值转换和解恢复方法。
- [ ] 能按“属于 NP + 从已知 NP 完全问题归约”证明新问题 NP 完全。
- [ ] 能复述顶点覆盖到集合覆盖的构造、双向证明与规模分析。
- [ ] 能证明 NP 完全问题进入 P 会导致 P = NP。
- [ ] 能证明一个问题 NP 完全当且仅当其补问题 co-NP 完全。
- [ ] 能解释 NP 完全问题若属于 co-NP，为什么会导致 NP = co-NP。
- [ ] 能把 CNF 的否定写成线性大小的 DNF，并证明 DNF-TAUTOLOGY 的 co-NP 完全性。
- [ ] 能推导 TAUTOLOGY 属于 P／NP 分别意味着哪些复杂性类相等。
- [ ] 能区分质数判定、合数判定和因数分解，并指出第 24 页旧图的问题。
- [ ] 能解释带界因数判定为何同时属于 NP 与 co-NP。
- [ ] 能准确陈述 Ladner 定理，并区分“已证明是 NPI”与“分类尚不明确”。
- [ ] 能独立完成 Quiz 的逆否证明，并指出不能反推的方向。
