# CSP
## 1. 演算法
在 CSP 中，我們的目標是找到一個投影向量 $w$，使得兩個類別的信號變異數比例最大化。這可以表示為最大化瑞利商（Rayleigh Quotient）：

$$
\begin{aligned}
\argmax J(w)=\argmax \frac{w^T \Sigma_1 w}{w^T \Sigma_2 w}
\end{aligned}
$$

其中 $\Sigma_1, \Sigma_2$ 是正定（Positive Definite）的共變異數矩陣。為了求解這個問題，我們通常將其轉換為以下約束優化問題：

$$
\begin{aligned}
\argmax w^T \Sigma_1 w \text{ subject to } w^T \Sigma_2 w=1
\end{aligned}
$$

利用拉格朗日乘數法（Lagrange Multiplier），這會導向 廣義特徵值問題（GEP）：

$$
\begin{aligned}
\Sigma_1 w = \lambda \Sigma_2 w
\end{aligned}
$$

## 2. Problem
### Invariance Property
$$
\begin{aligned}
J(\alpha w^*) = \frac{(\alpha w^*)^T \Sigma_1 (\alpha w^*)}{(\alpha w^*)^T \Sigma_2 (\alpha w^*)} = \frac{\alpha^2 (w^{*T} \Sigma_1 w^*)}{\alpha^2 (w^{*T} \Sigma_2 w^*)} = J(w^*)
\end{aligned}
$$

- 解不唯一性： 這意味著如果 $w$ 是解，那麼整條通過原點的直線上的點都是解。在數學上，解的空間是一個射影空間（Projective Space），而不是歐幾里得空間中的一個孤立點。
- 正則化的衝突： 當你加上 $L_1$ 正則化 $\|w\|_1$ 時，這個正則項不具備尺度不變性。這會導致優化演算法在試圖縮小 $\|w\|_1$ 的同時，為了滿足 $w^T \Sigma_2 w = 1$ 的約束，必須不斷調整 $w$ 的長度，這使得搜索空間的拓撲結構變得很複雜。

### Intrinsic Nonconvexity
- 瑞利商 $J(w) = \frac{w^T \Sigma_1 w}{w^T \Sigma_2 w}$ 的二階導數存在很多「鞍點」或「局部極值」

## 3. Alternative CSP algorithm
我們一樣要去最大化Rayleigh quotient

$$
\begin{aligned}
\argmax J(w)=\argmax \frac{w^T \Sigma_1 w}{w^T \Sigma_2 w}
\end{aligned}
$$

經過Lagrange multipler可以轉換成廣義特徵值問題，在這邊有兩種等價的寫法。

$$
\begin{aligned}
& X^T_1 X_1 w = λ X^T_2 X_2 w \\
& X^T_1 X_1 w = λ (X^T_1 X_1 + X^T_2 X_2) w
\end{aligned}
$$

一次處理全部channel，寫成矩陣的形式就會變成對角化問題。

$$
\begin{aligned}
& \max \operatorname{trace}(\Lambda) \\
& W^T X^T_1 X_1 W = \Lambda, W^T X^T_2 X_2 W = I
\end{aligned}
$$

接著我們要使用Least Squares去解。

$$
\begin{aligned}
& \min \Sigma^N_{n=1}||x^T_n \beta - y_n||^2_2 = ||XW-Y||^2_F \\
& (X^T X) \beta = X^T Y
\end{aligned}
$$

首先我們先把Lagrange form第二種表示式子整理一下，左式代表總體Covariance的能兩大小，右邊是類別1的能量大小。

$$
\begin{aligned}
(X^T_1 X_1 + X^T_2 X_2)w &= \frac{1}{\lambda} X^T_1 X_1 w \\
(X^T X)w &= \frac{1}{\lambda} X^T_1 X_1 w \\
\end{aligned}
$$

接著設計一個LS problem，隔離出類別1的特徵，設計Y在類別一的時候有個投影$X_1 w$，類別2是0。在這裡我們就可以用LS Problem解Rayleigh Quotient Problem。

$$
\begin{aligned}
X^T Y &=
\left[
    \begin{matrix}
    X_1^T & X_2^T \\
    \end{matrix}
\right]
\left[
    \begin{matrix}
    X_1 w \\
    0 \\
    \end{matrix}
\right] \\
&=
X_1^T X_1 w
\end{aligned}
$$

$$
\begin{aligned}
(X^T X) \beta &= X^T Y \\
(X^T X) \beta &= X_1^T X_1 w \\
(X^T X) w &= \frac{1}{\lambda} X^T_1 X_1 w \\
\end{aligned}
$$


首先假設原始EEG訊號可以做SVD decomposition，其中$U$是正交矩陣。接著我們就可以得到Covariance的SVD form。

$$
\begin{aligned}
X &= U D V^T \\
\Sigma_i &= X^T X \\
&= (U D V^T)^T U D V^T \\
&= (V D^T U^T) U D V^T \\
&= V D^2 V^T \\
\end{aligned}
$$

我們就可以透過SVD來算他的pseudo inverse來避免掉Singular Matrix。

$$
\begin{aligned}
\beta &= \Sigma^{-1} X_1^T X_1 w \\
&= (V D^2 V^T)^{-1} X_1^T X_1 w \\
&= (V D^{-2} V^T) X_1^T X_1 w \\
\end{aligned}
$$

最後再去解特徵值問題就可以得到我們的濾波器w。

$$
\begin{aligned}
\beta &= (V D^{-2} V^T) X_1^T X_1 w \\
\lambda w &= (V D^{-2} V^T) X_1^T X_1 w \\
\end{aligned}
$$

最後總結一下

只要把矩陣 $M = (V D^{-2} V^{\top} X_1^{\top} X_1)$ 算出來，然後對它跑一次特徵值分解，得到的特徵向量就是你要的 $w$。

## 4. Paper

把傳統的Rayleigh Quotient Problem改寫成一個LS Problem，找到一個濾波器要讓$X_1$最接近它白化後的訊號。剛好$A$就會是空間濾波矩陣$W$。$B$是白化訊號 $Y$ 的前 $K$ 個右奇異向量。

$$
\begin{aligned}
\min_{A, B} ||X_1 A B^T - Y||^2_F
\end{aligned}
$$

## 4. Reference
https://www.webofscience.com/wos/woscc/full-record/WOS:000706832000024
