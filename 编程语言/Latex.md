# 公式

## 公式基本语法

- **行内公式**：用 `$...$` 包围公式。
- **块级公式**：用 `$$...$$` 或 `\[...\]` 包围公式。

## 数学符号

### 基本符号

| 描述     | 符号                | LaTeX 代码             |
| -------- | ------------------- | ---------------------- |
| 加号     | $+$                 | `+`                    |
| 减号     | $-$                 | `-`                    |
| 正负号   | $\pm$               | `\pm`                  |
| 乘号     | $\times$            | `\times`               |
| 除号     | \                   | `\`                    |
| 根号     | $\sqrt{x}$          | `\sqrt{}`              |
| 分数     | $\frac{a}{b}$       | `\frac{a}{b}`          |
| 等于     | $=$                 | `=`                    |
| 不等于   | $\neq$              | `\neq`                 |
| 小于     | $<$                 | `<`                    |
| 大于     | $>$                 | `>`                    |
| 小于等于 | $\leq$              | `\leq`                 |
| 大于等于 | $\geq$              | `\geq`                 |
| 向下取整 | $\lfloor x \rfloor$ | `\lfloor`<br>`\rfloor` |
| 向上取整 | $\lceil x \rceil$   | `\lceil`<br>`\rceil`   |

### 集合符号
| 描述           | 符号         | LaTeX 代码   |
| -------------- | ------------ | ------------ |
| 属于           | $\in$        | `\in`        |
| 不属于         | $\notin$     | `\notin`     |
| 包含（真子集） | $\subset$    | `\subset`    |
| 包含等于       | $\subseteq$  | `\subseteq`  |
| 并集           | $\cup$       | `\cup`       |
| 交集           | $\cap$       | `\cap`       |
| 空集           | $\emptyset$  | `\emptyset`  |
| 所有实数集合   | $\mathbb{R}$ | `\mathbb{R}` |
| 所有自然数集合 | $\mathbb{N}$ | `\mathbb{N}` |
| 所有整数集合   | $\mathbb{Z}$ | `\mathbb{Z}` |

---

### **运算符**
| 描述   | 符号            | LaTeX 代码      |
| ------ | --------------- | --------------- |
| 求和   | $\sum$          | `\sum`          |
| 积分   | $\int$          | `\int`          |
| 重积分 | $\iint$         | `\iint`         |
| 极限   | $\lim$          | `\lim`          |
| 微分   | $\frac{dy}{dx}$ | `\frac{dy}{dx}` |

---

### **逻辑符号**
| 描述 | 符号       | LaTeX 代码 |
| ---- | ---------- | ---------- |
| 与   | $\land$    | `\land`    |
| 或   | $\lor$     | `\lor`     |
| 非   | $\neg$     | `\neg`     |
| 蕴含 | $\implies$ | `\implies` |
| 等价 | $\iff$     | `\iff`     |

---

### **希腊字母**
| 描述      | 符号     | LaTeX 代码 |
| --------- | -------- | ---------- |
| α (Alpha) | $\alpha$ | `\alpha`   |
| β (Beta)  | $\beta$  | `\beta`    |
| γ (Gamma) | $\gamma$ | `\gamma`   |
| Δ (Delta) | $\Delta$ | `\Delta`   |
| π (Pi)    | $\pi$    | `\pi`      |
| Ω (Omega) | $\Omega$ | `\Omega`   |

---

### **上下标**
| 描述             | 符号            | LaTeX 代码      |
| ---------------- | --------------- | --------------- |
| 上标（幂）       | $x^2$           | `x^2`           |
| 下标             | $x_i$           | `x_i`           |
| 同时有上标和下标 | $x_i^2$         | `x_i^2`         |
| 顶部加`^`        | $\hat{a}$       | `\hat{1}`       |
| 顶部加横线       | $\overline{a}$  | `\overline{a}$` |
| 顶部加波浪线     | $\widetilde{a}$ | `\widetilde{a}` |
| 顶部加点         | $\dot{a}$       | `\dot{a}`       |
| 顶部加两点       | $\ddot{a}$      | `\ddot{a}$`     |
| 顶部加箭头       | $\vec{a}$       | `\vec{a}`       |



### 大括号

为了表示一组内容，可以使用 `\left` 和 `\right`，配合不同类型的括号：

$$
\left( \frac{a}{b} \right)
$$
也可以使用 `\{ \}` 来显示大括号：

$$
\{ x \mid x > 0 \}
$$

### 矩阵

使用 `\begin{matrix} ... \end{matrix}`，或者带括号的 `\begin{pmatrix} ... \end{pmatrix}`：

$$
\begin{matrix}
a & b \\
c & d
\end{matrix}
$$

$$
\begin{pmatrix}
a & b \\
c & d
\end{pmatrix}
$$

#### 方程组

使用 `\begin{array} ... \end{array}`，配合 `\left` 和 `\right`：
$$
\left\{
\begin{array}{l}
x + y = 1 \\
x - y = 2
\end{array}
\right.
$$

