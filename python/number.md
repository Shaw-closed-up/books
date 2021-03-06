# Python3 数据类型-数字(number)

Python 数字数据类型用于存储数值。

数据类型是不允许改变的,这就意味着如果改变数字数据类型的值，将重新分配内存空间。

以下实例在变量赋值时 Number 对象将被创建：

```Python
var1 = 1
var2 = 10
var1+var2
```

您也可以使用del语句删除一些数字对象的引用。

del语句的语法是：`del var1[,var2[,var3[....,varN]]]`

您可以通过使用del语句删除单个或多个对象的引用，例如：`del var`,或`del var_a, var_b`

Python 支持三种不同的数值类型：

- **整型(Int)** - 通常被称为是整型或整数，是正或负整数，不带小数点。Python3 整型是没有限制大小的，可以当作 Long 类型使用，所以 Python3 没有 Python2 的 Long 类型。
- **浮点型(float)** - 浮点型由整数部分与小数部分组成，浮点型也可以使用科学计数法表示（2.5e2 = 2.5 x 102 = 250）
- **复数( (complex))** - 复数由实数部分和虚数部分构成，可以用a + bj,或者complex(a,b)表示， 复数的实部a和虚部b都是浮点型。

#### 练习：使用十六进制和八进制来代表整数：

```Python
number = 0xA0FF # 十六进制
print(number)

number = 0b0101011 # 二进制
print(number)

number = 0o16 # 八进制
print(number)
```

| int    | float      | complex    |
| :----- | :--- | :--- |
| 10     | 0.0        | 3.14j      |
| 100    | 15.20      | 45.j       |
| -786   | -21.9      | 9.322e-36j |
| 080    | 32.3e+18   | .876j      |
| -0490  | -90.       | -.6545+0J  |
| -0x260 | -32.54e100 | 3e+26J     |
| 0x69   | 70.2E-12   | 4.53e-7j   |

- Python支持复数，复数由实数部分和虚数部分构成，可以用`a + bj`,或者`complex(a,b)`表示， 复数的实部`a`和虚部`b`都是浮点型。

## Python 数字类型转换

有时候，我们需要对数据内置的类型进行转换，数据类型的转换，你只需要将数据类型作为函数名即可。

- `int(x)` 将x转换为一个整数。
- `float(x)`将x转换到一个浮点数。
- `complex(x)` 将x转换到一个复数，实数部分为 x，虚数部分为 0。
- `complex(x, y)` 将 x 和 y 转换到一个复数，实数部分为 x，虚数部分为 y。x 和 y 是数字表达式。

#### 练习：将浮点数变量 a 转换为整数

```Python
a = 1.0
int(a)
a
```

## Python 数字运算

Python 解释器可以作为一个简单的计算器，您可以在解释器里输入一个表达式，它将输出表达式的值。

表达式的语法很直白： `+`, `-`, `*` 和 `/`, 和其它语言（如Pascal或C）里一样。

#### 练习：Python数字运算

```Python
print(2 + 2)

print(50 - 5*6)

print((50 - 5*6) / 4)

print(8 / 5)  # 总是返回一个浮点数
```

**注意：**在不同的机器上浮点运算的结果可能会不一样。

在整数除法中，除法`/`总是返回一个浮点数，如果只想得到整数的结果，丢弃可能的分数部分，可以使用运算符 `//` ：

```Python
print(17 / 3)  # 整数除法返回浮点型

print(17 // 3)  # 整数除法返回向下取整后的结果

print(17 % 3)  # ％操作符返回除法的余数

print(5 * 3 + 2)
```

**注意：`//`得到的并不一定是整数类型的数，它与分母分子的数据类型有关系。**

```Python
print(7//2)

print(7.0//2)

print(7//2.0)
```

等号`=`用于给变量赋值。赋值之后，除了下一个提示符，解释器不会显示任何结果。

```Python
width = 20
height = 5*9
width * height
```

Python 可以使用 `**` 操作来进行幂运算：

```Python
5 ** 2  # 5 的平方

2 ** 7  # 2的7次方
```

变量在使用前必须先"定义"（即赋予变量一个值），否则会出现错误：

```Python
n   # 尝试访问一个未定义的变量
```

不同类型的数混合运算时会将整数转换为浮点数：

```Python
3 * 3.75 / 1.5
```

#### 练习： 交互模式下的`_`

在交互模式中，最后被输出的表达式结果被赋值给变量`_` 。例如：

```Python
tax = 12.5 / 100
price = 100.50
print(price * tax)
print('_:',_)#注意此时的_的类型为str,不能直接与数字进行运算
print(price + float(_))
```
此处，`_` 变量应被用户视为只读变量，且类型为`str`


## 数学函数

| 函数              | 返回值 ( 描述 )                                              |
| :---- | :----- |
| `abs(x)` |                                                              |
| `ceil(x)`       | 返回数字的上入整数，如`math.ceil(4.1)` 返回 5                |
| `cmp(x, y)`       | 如果 x < y 返回 -1, 如果 x == y 返回 0, 如果 x > y 返回 1。 **Python 3 已废弃，使用 (x>y)-(x<y) 替换**。 |
| `exp(x)`        | 返回e的x次幂(ex),如math.exp(1) 返回2.718281828459045         |
| `fabs(x)`       | 返回数字的绝对值，如math.fabs(-10) 返回10.0                  |
| `floor(x)`     | 返回数字的下舍整数，如math.floor(4.9)返回 4                  |
| `log(x)`        | 如math.log(math.e)返回1.0,math.log(100,10)返回2.0            |
| `log10(x)`      | 返回以10为基数的x的对数，如math.log10(100)返回 2.0           |
| `max(x1, x2,...)` | 返回给定参数的最大值，参数可以为序列。                       |
| `min(x1, x2,...)` | 返回给定参数的最小值，参数可以为序列。                       |
| `modf(x)`       | 返回x的整数部分与小数部分，两部分的数值符号与x相同，整数部分以浮点型表示。 |
| `pow(x, y)`     | x**y 运算后的值。                                            |
| `round(x [,n])` | 返回浮点数 x 的四舍五入值，如给出 n 值，则代表舍入到小数点后的位数。**其实准确的说是保留值将保留到离上一位更近的一端。** |
| `sqrt(x)`       | 返回数字x的平方根。                                          |

## 随机数函数

随机数可以用于数学，游戏，安全等领域中，还经常被嵌入到算法中，用以提高算法效率，并提高程序的安全性。

Python包含以下常用随机数函数：

| 函数                                 | 描述                                                         |
| :----- | :----- |
| `choice(seq)`                      | 从序列的元素中随机挑选一个元素，比如random.choice(range(10))，从0到9中随机挑选一个整数。 |
| `randrange ([start,\]stop [,step])` | 从指定范围内，按指定基数递增的集合中获取一个随机数，基数默认值为 1 |
| `random()`                         | 随机生成下一个实数，它在[0,1)范围内。                        |
| `seed([x\])`                       | 改变随机数生成器的种子seed。如果你不了解其原理，你不必特别去设定seed，Python会帮你选择seed。 |
| `shuffle(lst)`                     | 将序列的所有元素随机排序                                     |
| `uniform(x, y)`                    | 随机生成下一个实数，它在[x,y]范围内。                        |

## 三角函数

| 函数          | 描述                                              |
| : | : |
| `acos(x)`   | 返回x的反余弦弧度值。                             |
| `asin(x)`   | 返回x的反正弦弧度值。                             |
| `atan(x)`   | 返回x的反正切弧度值。                             |
| `atan2(y, x)` | 返回给定的 X 及 Y 坐标值的反正切值。              |
| `cos(x)`    | 返回x的弧度的余弦值。                             |
| `hypot(x, y)` | 返回欧几里德范数 sqrt(x*x + y*y)。                |
| `sin(x)`    | 返回的x弧度的正弦值。                             |
| `tan(x)`    | 返回x弧度的正切值。                               |
| `degrees(x)` | 将弧度转换为角度,如degrees(math.pi/2) ， 返回90.0 |
| `radians(x)` | 将角度转换为弧度                                  |

## 数学常量

| 常量 | 描述                                  |
| :--- | : |
| `pi` | 数学常量 pi（圆周率，一般以π来表示）  |
| `e`  | 数学常量 e，e即自然常数（自然常数）。 |

#### 练习：计算半径为5的圆的面积

根据公式$S={\pi}r^2$即可求出

```Python
import math
r=5
print("半径为5的圆的面积:",math.pi*r**2)
```

