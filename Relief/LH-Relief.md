# LH-Relief
LH-Relief優化Relief，解決Outlier影響結果的問題。以及高維度所造成雜訊影響很大，每個點的2-norm算起來會差不多。

## 1. 演算法
假設我們找到了k個最近鄰居$H_i$，接著我們假設一個重構鄰居是由著k個鄰居做線性組合。

$$
\begin{aligned}
\hat{x} = H_i\alpha_i
\end{aligned}
$$

$$
\begin{aligned}
& H_i \in R^{k,P} \\
& \alpha_i \in R^{k,1} \\
& k: \text{number of selected neighbors} \\
& P: \text{number of features} \\
\end{aligned}
$$

