---
title: "Python 做机试题目技巧"
date: 2023-08-19T10:30:35+08:00
tags: [Python]
---

## 排序

使用`sorted()`来做, **不修改原来的变量**, 而是返回一个新的。 [自定义cmp函数的例子]()

>被排序的类型必须是iterable的。


# 字符串

## 无重复字符的最长子串
```py
def lengthOfLongestSubstring(self, s: str) -> int:
    mp = {}
    left, right = 0, 0
    max_len = 0

    while right < len(s):
        if s[right] not in mp:
            mp[s[right]] = 1
        else :
            mp[s[right]] += 1
        
        while mp[s[right]] > 1:
            mp[s[left]] -= 1
            left += 1
        max_len = max(max_len, right-left+1)
        right += 1
    return max_len

```


## 牛客网处理输入

https://blog.nowcoder.net/n/0632a788b94b4923976b7c82c45eca95

## 写递归

遇到需要用递归的题目中, 常常需要传值出来. 比如写一个求 sum(1..n)的函数

这样写是**错误的**, 因为每次对 `Sum` 的更新都是局部的, 并不会影响到
parent frame 的值. 所以最后传出的值一定是 0

```py
def func(n, Sum):
  if n == 0:
    return
  else:
    Sum+= n
    func(n-1, Sum)

result = 0
func(3, result)
```

有两个方案解决:

1. 使用全局变量来保存结果, 见[全局变量](#全局变量的使用)章节
2. 将传出的参数用**可修改的**类型来保存, 比如说 List. 以上代码可以修改为:

```py
def func(n, Sum):
  if n == 0:
    return
  else:
    Sum[0] += n
    func(n-1, Sum)

result = [0]
func(3, result)
```

## 全局变量的使用

在写机试题时, 一般不会出现函数嵌套, 所以用不到`nonlocal`关键字.

反而需要使用`global`关键字, 使用的方法为:

```py
max_len = 0

def dfs(..):
    global max_len
    # 在dfs()中修改或者访问的max_len就是全局的了
    ...
```

## 进制以及 ascii 码转换

ascii 和字符相互转换, `ord()`和`chr()`互为逆函数:

```py
c = 'a'
ac = ord(c) # 65
c = chr(ac) # 'a'
```

进制转换
- 如果要生成整型数据, 不管什么进制, 统一使用`int()`在最外层
- 如果要生成字符串, 统一使用`format()`在最外层, 当然它和`bin()`, `hex()`这类是等价的

下面给出几个常用的例子:

```py
# '1010' -> 10
int('1010', 2) # 10
# 10 -> '1010' 二进制字符串
format(10, 'b') # '1010'
format(10, '#b') # '0b1010' , 等价于bin(10)的结果
# 10 -> '12' 八进制的字符串
format(10, 'o') # '12'
format(10, '#o') # '0o12' , 等价于 oct(10)的结果
```

## 浮点数的处理

浮点数打印精度(四舍五入)

```py
x = 1.23
print("{:.1f}".format(x))
print(format(x, '.1f'))   # 我自己习惯统一使用此方法
```

## 字符串/字符的常用处理

- 字符串中含有非字母转小写/大写: 可直接调用`.lower()`, `.upper()`
  非字符会默认保持不变

string 和 list 相互转换

```py
# list to str
a = ['h', 'e', 'l', 'l', 'o']
s = ''.join(a)

# str ot list
s = 'hello'
a = list(s)
```

字符串是多个数字字段的组合, 如何分割每个字段转成`int`并存入 list:

```py
s = "10 20 30"
s = s.split()
l = [int(x) for x in s]
```

字符和整数之间的转换

```py
# c = 'b'
c = chr(ord('a')+1)

# c = '2'
c = chr(ord('1')+1)
```

字符串回文判断

```py
s == s[::-1]
```

统计字符出现的次数: **不一定要借用字典**

string 和元组一样, 有`.count(x)`方法统计 x 的出现次数

```py
s = input()

for i in s:
    if s.count(i) == 1:
        print(i)
        break
else:
    print('-1')
```

搜索一个字符串中是否包含另一字符串, 不要再使用`.find() != -1`了
直接使用`in`关键字即可

```py
if s1 in s2:
    print("s1 is included in s2")
```

字符串删除指定字符:

```py
s = "hello"
#删除所有 l
s = s.replace('l', '')
#删除前一个 l
s = s.replace('l', '', 1)
```

## 字典的常用处理

字典按照 value 排序:

```py
# 返回降序的value list
d = {}
l = list(sorted(d.values(), reverse=True))
# 返回按value升序排序的的字典
d_sorted = dict(sorted(d.items(),key=lambda x: x[1]))
# 返回value降序的字典
d_sorted = dict(sorted(d.items(), key=lambda x: -x[1]))
# 返回按value降序,如果value相同时按key升序排序的字典
d_sorted = dict(sorted(d.items(), key=lambda x: (-x[1], x[0])))
```

## 集合

```py
# 创建一个空集合
s = set()
# 集合中添加元素
s.add(123)
```

## 列表的常用处理

建立非重复的列表, C++中的集合(set)

```py
s = "abcdabcd"  # 想要得到['a', 'b', 'c', 'd']
set_list = []
for c in s:
    if c not in set_list:
        set_list.append(c)
```

列表删除元素

```py
l = [1, 2, 3, 3]

l.remove(3) # 删除第一个3

# 删除所有3
for c in l:
    if c == 3:
        l.remove(c)
```


## 工具模块

lru_cache 装饰器优化递归

注意, 用此装饰器的函数参数必须为**hashable**的, 例如 list 就是 unhashable 的,
需要转为元组: `func(tuple(lst))`

```py
import functools

@functools.lru_cache(2**20)
def fibonacci(n):
    ...
```

## 典型问题整理

### [称砝码](https://www.nowcoder.com/practice/f9a4c19050fc477e9e27eb75f3bfd49c?tpId=37&rp=1&ru=%2Fexam%2Foj%2Fta&qru=%2Fexam%2Foj%2Fta&sourceUrl=%2Fexam%2Foj%2Fta%3Fpage%3D3%26tpId%3D37%26type%3D37&difficulty=3&judgeStatus=3&tags=&title=&gioEnter=menu)

以小见大, 假设我们有三个砝码, 要统计可能出现的所有重要, 我们的方法是:

1. 先放 A, 统计所有可能 [0, A]
2. 再放 B, 与之前统计的所有可能相加, 去重后加入所有可能 [0, A, B, A+B]
3. 再放 C, 与之前统计的所有可能相加, 去重后加入所有可能 [0, A, B, A+B, C, A+C, B+C, A+B+C]
