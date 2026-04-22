```
Author: Jyun Teng Huang
Date: 2026/03/18
```
# H-ERL
## Concept
Covariance Matrix是屬於黎曼流形 (Riemannian manifold) 空間的一個點，傳統是透過CSP把它轉換成歐式空間再進行LDA等分類問題。

這篇則是使用TSP把Convariance Matrix轉成歐式空間之後接著使用ERL來進一步得到特徵後接著使用SVM做分類問題。

## Riemannian manifold
假設所有cov構成一個space，這個space有一個規則就是要是對稱正定矩陣。但是減法可能會不成立，那這個空間就不是vector space。我們透過TSM可以讓這個空間可以符合vector space的規則

## Reference
- https://www.sciencedirect.com/science/article/pii/S1746809426000583
