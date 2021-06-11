# SymPy符号计算-让Python帮我们推公式

> via: https://zhuanlan.zhihu.com/p/83822118

作者: 阿凯

Email: xingshunkai@qq.com

## 概要

像我这种粗心的小孩, 在推导一些复杂的公式(尤其是矩阵运算)的时候, 经常容易算错数, 一步推错,步步错。

万能的Python有什么方法可以帮助我们节省时间， 减少出错率呢? 有一个包叫做**SymPy**, 它可以帮我们自动的进行**符号化计算**. 所谓符号化计算的含义是指, 带入运算的不是某个具体的数值, 而是抽象的数学符号, 并且还可以帮我们将最终得到的结果进行归并简化(例如sin cos函数的合并).

这篇文章会用案例的方式, 给大家展示一下sympy的常用的功能.

## 安装工具包

```python
sudo pip3 install sympy
```

## 导入工具包

```python
import sympy as sym
from sympy import sin,cos
```

## 求解二元一次方程组

```python
x,y = sym.symbols('x, y')
sym.solve([x + y - 1,x - y -3],[x,y])
```

输出日志:

```text
{x: 2, y: -1}
```

## 测试不定积分

```python
x = sym.symbols('x')
a = sym.Integral(cos(x))
# 积分之后的结果
a.doit()
```

输出日志:

(缺失的图片)

```python
# 显示等式
sym.Eq(a, a.doit())
```

输出日志:

(缺失的图片)

## 测试定积分

```python
e = sym.Integral(cos(x), (x, 0, sym.pi/2))
e
```

输出日志:

(缺失的图片)

```python
# 计算得到结果
e.doit()
```

输出日志:

(缺失的图片)

## 求极限

```python
n = sym.Symbol('n')
s = ((n+3)/(n+2))**n

#无穷为两个小写o
sym.limit(s, x, sym.oo)
```

输出日志:

(缺失的图片)

```python
print(sym.limit(s, x, sym.oo))
((n + 3)/(n + 2))**n
```

## 测试三角函数合并

sympy支持latex表达式

```python
theta = symbols('theta')
a = cos(theta) * cos(theta) - sin(theta)*sin(theta)
Eq(a)
```

输出日志:

(缺失的图片)

## 三角函数简化

可以调用`simplify` 函数进行结果的简化, 简直是太好用了!!!!

```python
sym.simplify(a)
```

输出日志:

(缺失的图片)

```python
x = sym.symbols('x')
f = sym.simplify('x/x+1')
f
```

输出日志:

(缺失的图片)

## 多元表达式

```python
x, y = sym.symbols('x y')
f = (x + 2)**2 + 5*y
sym.Eq(f)
```

输出日志:

(缺失的图片)

## 传入数值

```python
f.evalf(subs = {x:1,y:2})
```

输出日志:

(缺失的图片)

## 拓展Latex格式

[https://docs.sympy.org/0.7.2/modules/galgebra/latex_ex/latex_ex.html](https://link.zhihu.com/?target=https%3A//docs.sympy.org/0.7.2/modules/galgebra/latex_ex/latex_ex.html)

例如想要显示 (缺失的图片)

```python
delta_t = sym.symbols('delta_t')
```

## 测试行列式求解

```python
dt = sym.symbols('delta_t')
# 定义矩阵T
T = sym.Matrix(
    [[1, 0, 0, 0], 
     [1, dt, dt**2, dt**3], 
     [0, 1, 0, 0],
     [0, 1, 2*dt, 3*dt**2]])
T
```

输出日志:

(缺失的图片)

```python
# 计算行列式
sym.det(T)
```

输出日志:

(缺失的图片)

```python
# 矩阵求逆
T_inverse = T.inv()
# 逆矩阵
T_inverse
```

输出日志:

(缺失的图片)

## 运动学公式自动推导

下面演示使用sympy自动推导运动学公式

定义一些符号

```python
# 定义符号
theta_1, theta_2,theta_3, l_2, l_3 = sym.symbols('theta_1, theta_2, theta_3, l_2, l_3')
theta_1
```

输出日志:

(缺失的图片)

下面是机械臂DH空间变换用到的矩阵

```python
def RZ(theta):
    '''绕Z轴旋转'''
    return sym.Matrix(
    [[cos(theta), -sin(theta), 0, 0], 
     [sin(theta), cos(theta), 0, 0], 
     [0, 0, 1, 0],
     [0, 0, 0, 1]])

def RX(gamma):
    '''绕X轴旋转'''
    return sym.Matrix([
        [1, 0, 0, 0],
        [0, cos(gamma), -sin(gamma), 0],
        [0, sin(gamma), cos(gamma), 0],
        [0, 0, 0, 1]])

def DX(l):
    '''绕X轴平移'''
    return sym.Matrix(
    [[1, 0, 0, l], 
     [0, 1, 0, 0], 
     [0, 0, 1, 0],
     [0, 0, 0, 1]])

def DZ(l):
    '''绕Z轴'''
    return sym.Matrix(
    [[1, 0, 0, 0], 
     [0, 1, 0, 0], 
     [0, 0, 1, l],
     [0, 0, 0, 1]])
```

按照变换顺序, 依次相乘,得到总的变换矩阵

```python
T04 = RZ(theta_1)*RX(-sym.pi/2)*RZ(theta_2)*DX(l_2)*RZ(theta_3)*DX(l_3)
```

公式简化, 最终得到了三自由度机械臂， 正向运动学的结果

```python
T04 = sym.simplify(T04)
T04
```

输出日志:

(缺失的图片)

## 导出Latex

运算得到的结果可以直接插到论文里面, 不用自己再手敲一遍latex.

直接导出结果的latex字符

```python
sym.print_latex(T_inverse)
```

输出日志:

```text
\left[\begin{matrix}1 & 0 & 0 & 0\\0 & 0 & 1 & 0\\- \frac{3}{\delta_{t}^{2}} & \frac{3}{\delta_{t}^{2}} & - \frac{2}{\delta_{t}} & - \frac{1}{\delta_{t}}\\\frac{2}{\delta_{t}^{3}} & - \frac{2}{\delta_{t}^{3}} & \frac{1}{\delta_{t}^{2}} & \frac{1}{\delta_{t}^{2}}\end{matrix}\right]
```

可以copy， 插入到markdown正文中

```text
$$
这里插入刚才导出的字符串
$$
```