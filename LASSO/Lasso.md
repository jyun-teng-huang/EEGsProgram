# Lasso
## 1. 演算法
Loss function由MSE跟L1-panelty組合而成，總共有N trials, P feature.

$$
\begin{aligned}
L&=\frac{1}{2}|y-\hat{y}|^2_2+\alpha|W|_1 \\
L&=\frac{\Sigma^N_{i=1}(y_i-\hat{y_i})^2}{2N}+\alpha|W|_1 \\
\end{aligned}
$$

由於Lasso的L1-panelty絕對值不能微分，如果使用Gradient Descent會導致特徵沒辦法拉到0，因此我們是採用Coordinate Desent搭配Soft_threshold，這個方法一次只會調整一個feature，下面我們需要做一些前置作業改變原有的模型結構。

$$
\begin{aligned}
\hat{y}&=Xw \\
\hat{y_i}&=X_iw = \Sigma_{k!=j} X_{ik}w_k + X_{ij}w_j \\
\hat{y}&=
    \begin{bmatrix}
    \Sigma_{k!=j} X_{1k}w_k + X_{1j}w_j \\
    \Sigma_{k!=j} X_{2k}w_k + X_{2j}w_j \\
    \vdots\\
    \Sigma_{k!=j} X_{Nk}w_k + X_{Nj}w_j \\
    \end{bmatrix}
\end{aligned}
$$

接著把剛剛改寫後的架構放進Loss Funtion裡面。這邊我們定義一個partial residual代表去掉第j個feature所造成的loss有多少。
$$residual = y_i-\Sigma_{k!=j} X_{ik}w_k$$
$$residual = y_i-\hat{y_i}+X_{ij}w_j$$
$$
\begin{aligned}
L&=\frac{\Sigma^N_{i=1}(y_i-(\Sigma_{k!=j} X_{ik}w_k + X_{ij}w_j))^2}{2N}+\alpha|W|_1 \\
L&=\frac{\Sigma^N_{i=1}(y_i-\Sigma_{k!=j} X_{ik}w_k - X_{ij}w_j)^2}{2N}+\alpha|W|_1 \\
L&=\frac{\Sigma^N_{i=1}(residual - X_{ij}w_j)^2}{2N}+\alpha|W|_1
\end{aligned}
$$

在使用Soft threshold的時候我們先不考慮$\alpha|W|_1$，專注在MSE本身的Loss有多少，再根據這個大小來決定我們的L1-panelty要怎麼運作。\
那我們就可以針對一個feature去做partial derivative，來算出沿著$w_j$方向Loss怎麼樣會最小。

$$
\begin{aligned}
\frac{\partial L}{\partial w_j}=\frac{\Sigma^N_{i=1}(residual-X_{ij}*w_j)(-X_{ij})}{N}=0 \\
\end{aligned}
$$

調整式子，我們可以得到新的$w_j$，讓MSE會最小。

$$
% \begin{aligned}
\Sigma^N_{i=1} X_{ij} w_j X_{ij}=\Sigma^N_{i=1}residual*X_{ij} \\
w_j = \frac{\Sigma^N_{i=1}residual*X_{ij}}{\Sigma^N_{i=1} X_{ij}^2}
% \end{aligned}
$$

接著就要使用Soft threshold來把L1-panelty加進來Loss Function裡面討論微分結果。Soft threshold的方程式如下。

$$
\begin{aligned}
S(\rho,\lambda) &=
    \left
        \{\begin{aligned}
        &\rho-\lambda \text{, if $\rho$ > $\lambda$} \\
        &\rho+\lambda \text{, if $\rho$ < $-\lambda$} \\
        &0 \text{, if $|\rho|$ < $\lambda$} \\
        \end{aligned}
    \right.
\end{aligned}
$$

剛剛微分後得到的分子剛好就是$\rho$，$\lambda$也剛好是$N$倍的$\alpha$。我們就可以透過Soft threshold得到加入L1-panelty的新權重，也就是更新後feature的權重。

$$
\begin{aligned}
&\rho = \Sigma^N_{i=1}residual*X_{ij} \\
&\lambda = \alpha N \\
&z_j = \Sigma^N_{i=1} X_{ij}^2 \\
&w_{j,new}=\frac{S(\rho,\lambda)}{z_j}
\end{aligned}
$$

最後我們做個總結：
1. 由於我們前面的Loss function有乘0.5因此我們的$\lambda$才會等於$\alpha N$，沒有乘0.5的話那$\lambda$才會等於$\frac{\alpha N}{2}$。
2. 在算Loss的時候，我們是使用MSE，也就是平均去算那個$\lambda$。但是我們在算$\rho$的時候是算Sum，這兩個數量等級不一樣，為了讓Soft threshold有可比性我們要將$\alpha$乘上N。
3. Lasso主要在做特徵的刪除，讓weight變成0代表該特徵比較不重要。我們需要ground truth，輸入特徵就可以做了。

## 2. Reference
1. https://www.sas.upenn.edu/~fdiebold/NoHesitations/BookAdvanced.pdf
