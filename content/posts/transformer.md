---
title: "Transformer"
date: 2026-09-15
math: true
---
# 什么是注意力机制？

$$
\mathrm{Attention}(Q,K,V) =
\mathrm{softmax}
\left(
\frac{QK^T}{\sqrt{d_k}}
\right)V
$$


注意力机制就是根据 Query 和 Key 的匹配程度，计算每个 Value 应该被关注多少，然后加权求和得到新的表示。

$$
QK^T =
\begin{bmatrix}
q_1\\
q_2\\
q_3
\end{bmatrix}
\begin{bmatrix}
k_1^T & k_2^T & k_3^T
\end{bmatrix}
{}={}
\begin{bmatrix}
q_1k_1^T & q_1k_2^T & q_1k_3^T\\
q_2k_1^T & q_2k_2^T & q_2k_3^T\\
q_3k_1^T & q_3k_2^T & q_3k_3^T
\end{bmatrix}
$$

# Q、K、V从哪里来？

Transformer里面有三个可训练矩阵：

$$
W_Q,\ W_K,\ W_V
$$

然后：

$$
Q=XW_Q
$$

$$
K=XW_K
$$

$$
V=XW_V
$$

$X$是原始信息，$W_K、W_V、W_Q$是三个不同观察方式，$Q$：我想找什么特征、$K$：我有什么特征可以被别人找到、$V$：我要传递什么信息。

$$
\mathrm{Attention}(Q,K,V) =
\mathrm{softmax}
\left(
\frac{(XW_Q)(XW_K)^T}{\sqrt{d_k}}
\right)V = 
\mathrm{softmax}
\left(
\frac{XW_Q{W_K}^TX^T}{\sqrt{d_k}}
\right)V 
$$

$d_k$是特征的维度