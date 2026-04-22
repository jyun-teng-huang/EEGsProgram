# ADALINE
## 1. 演算法
ADALINE就是透過Gradient Descent的方法來解線性回歸模型。我們可以把線性回歸模型寫成下面的方程式。

$$
\begin{aligned}
\hat{Y} = Xw
\end{aligned}
$$

$$
\begin{aligned}
& \hat{Y} \subseteq R^{N \times 1}\\
& X \subseteq R^{N \times P}\\
& w \subseteq R^{P \times 1}\\
& N\text{是實驗次數} \\
& P\text{是特徵個數} \\
\end{aligned}
$$

模型是使用MSE來當作Loss function，我們要最小化這個Loss function。

$$
\begin{aligned}
\text{MSE} &= \frac{||Y - \hat{Y}||^2_2}{N} \\
            &= \frac{||Y - (Xw+b)||^2_2}{N} \\
           &= \frac{\Sigma^N_{i=1} (y_i-(x_i w + b))^2}{N} \\
\end{aligned}
$$

```
def MSE(y, x, w):
    N = np.shape(y)[0]
    y_hat = x@w
    error = y - y_hat
    return np.sum(error ** 2)/N
```

接著我們要使用Gradient Descent來更新權重，先把Loss function 改寫成element-wise。

$$
\begin{aligned}
L &= \frac{\Sigma^N_{i=1} (y_i-\hat{y_i})^2}{N} \\
L &= \frac{\Sigma^N_{i=1} (y_i- \Sigma^P_{j=1} x_{ij}w_j)^2}{N} \\
\end{aligned}
$$

所以我們要對Loss function微分。

$$
\begin{aligned}
\frac{\partial L}{\partial w_k} &= \frac{\partial L}{\partial \hat{y_i}}\frac{\partial \hat{y_i}}{\partial w_k}\\
&=\frac{1}{N}(2 \cdot \Sigma^N_{i=1} (y_i- \hat{y_i})(-x_{ik}))\\
\end{aligned}
$$

$$
\begin{aligned}
& \frac{\partial L}{\partial \hat{y_i}}= \frac{-2 \cdot \Sigma^N_{i=1} (y_i- \hat{y_i})}{N}\\
&\frac{\partial \hat{y_i}}{\partial w_k}= x_{ik}\\
\end{aligned}
$$

我們用矩陣運算重新計算一次，一次性對所有$w$更新，來加速程式。這邊會用到矩陣的微分法則$\frac{\partial}{\partial z} \|{a} - {B}{z}\|^2 = -2{B}^T({a} - {B}{z})$。

$$
\begin{aligned}
L &= \frac{||Y - \hat{Y}||^2_2}{N} \\
  &= \frac{(Y - \hat{Y})^T(Y - \hat{Y})}{N} \\
  &= \frac{1}{N}(Y^T Y - Y^T \hat{Y}- \hat{Y}^T Y - \hat{Y}^T \hat{Y}) \\
  &= \frac{1}{N}(Y^T Y - Y^T X w- w^T X^T Y - w^T X^T X w) \\
  &= \frac{1}{N}(Y^T Y - 2w^T X^T Y - w^T X^T X w) \\
\end{aligned}
$$

$$
\begin{aligned}
& \frac{\partial (Y^T Y)}{\partial w} = 0 \\
& \frac{\partial (-2w^T X^T Y)}{\partial w} = -2 X^T Y \\
& \frac{\partial (- w^T X^T X w)}{\partial w} = 2X^T X w \\
\end{aligned}
$$

$$
\begin{aligned}
 \frac{\partial L}{\partial w} &= \frac{1}{N}(-2 X^T Y)(2X^T X w) \\
&= -\frac{2}{N}X^T(Y - X w) \\
&= -\frac{2}{N}X^T(Y - \hat{Y}) \\
\end{aligned}
$$

最後更新權重。
$$
\begin{aligned}
& w_{new} = w_{old} - \eta \cdot \frac{\partial L}{\partial w} \\
& b_{new} = b_{old} - \eta \cdot \frac{\partial L}{\partial w} \\
\end{aligned}
$$

## 2. Reference
1. https://isl.stanford.edu/~widrow/papers/c1960adaptiveswitching.pdf
