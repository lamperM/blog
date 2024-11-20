---
title: "Python：Pytest框架学习"
tags: ["Python", "Pytest"]
categories: ["Python"]
date: 2024-10-06T19:28:12+08:00
---

## Config

## Marker

### 使用 Marker 对用例分类

Maker 帮助我们对用例进行分类，区分不同的场景和平台。

```python

# 使用 @pytest.mark.smoke 标记一个冒烟测试
@pytest.mark.smoke
def test_login():
    ...

# 使用 @pytest.mark.integration 标记一个集成测试
@pytest.mark.integration
class TestPaymentSystem:
```

一个用例支持有多个 Marker，比如说某个用例可能 smoke 和 dailybuild 都要跑，就可以为它指定两个 maker。

Pytest 运行时，通过 `-m`/ `--maker` 参数来指定本次测试的 marker 表达式。支持与或非的组合。

```sh

# 执行所有标记为 smoke 的测试
pytest -m smoke

# 执行标记为 smoke 或 integration 的测试
pytest -m "smoke or integration"

# 执行既非 slow 也非 integration 的测试
pytest -m "not slow and not integration"

# 运行同时带有 unit 和 integration 的测试
pytest -m "unit and integration"

# 复杂组合筛选
pytest -m "(system or integration) and not unit"  #运行system或integration，且不运行带有unit的测试
```

代码中也可以修改 Marker。比如说在 config 里加了一个属性，有这个属性的情况下就自动加上一个 marker，这样就不用在运行命令参数里写两遍了。

### 使用 Marker 配置 Skip 用例

分类按理说用 marker 正向的指定就行，但是 Pytest 也提供了反向过滤的方法，更加的灵活。特别是`skipif` 我觉得更加适合于对运行时变量的条件过滤。

```python
# @pytest.mark.skip(跳过原因)
# @pytest.mark.skipif(跳过条件,跳过原因)

# 示例
class TestDemo:

    workage2 = 5
    workage3 = 20

    @pytest.mark.skip(reason="无理由跳过")
    def test_demo1(self):
        print("我被跳过了")

    @pytest.mark.skipif(workage2<10,reason="工作经验少于10年跳过")
    def test_demo2(self):
        print("由于经验不足，我被跳过了")

    @pytest.mark.skipif(workage3<10,reason="工作经验少于10年跳过")
    def test_demo3(self):
        print("由于经验过关，我被执行了")

    def test_demo3(self):
        print("我没有跳过条件，所以我被执行了")
```

## 测试函数的参数

在使用中我看到每个测试函数的参数都不是固定的，而且这种自动化的测试框架找不到每个函数的调用点，🤔 这些参数是怎么就可以随便指定，在传入的时候又能正确呢？
