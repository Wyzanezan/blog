---
title: Python3中生成器和协程的使用
date: 2020-09-13 21:24:11
tags: python
categories: Python
---

今天整理下 Python3 中生成器和协程的使用。

<!--more-->

# yield

## 生成器输出数据

python 中可以使用 yield 关键字实现生成器，简单点说就是：一个函数中，如果包含 yield 关键字，那么这个函数就是一个生成器。下面看一个例子：

```python
def test():
    """生成器：yield的使用
    """
    flag = 0
    print("=======start=======")
    for i in range(5):
        print("=====flag:", flag)
        flag += 1
        yield i
        print("========end=========")
        
        
def main():
    """调用生成器
    """
    t = test()
    print(t)
    print(next(t))
    print(next(t))
    print(next(t))
       
    
if __name__ == "__main__":
    main() 
   
```

上面代码的输出结果是：

```txt
<generator object test at 0x000001726581C2B0>
=======start=======
=====flag: 0
0
========end=========
=====flag: 1
1
========end=========
=====flag: 2
2
```

从上面的输出结果中可以看出，每次调用next()，都会从上一次结束的地方执行。 相当于遇到 yield 时，会跳出当前代码的执行，再次调用next()时，会从上一次跳出的地方开始执行 (即print("========end=========")这行代码试下一次执行的开始)。调用生成器时可以使用 next() 函数，也可以使用生成器的 `__next__`() 方法来获取值。



## 生成器接收外部数据

在生成器中，yield 还可以接收外部传进来的值，例子如下：

```python
def test():
    """生成器：yield接收外部传入的值
    """
    for i in range(2):
        x = yield i
        print("=====x:", x)

    return "OK"


def main():
    t = test()

    try:
        # 第一次调用生成器的send()方法时，传入一个None值，用于启动生成器。
        # 调用 send()，相当于调用 next()，执行到 yield 时，会返回 yield 后面的值
        ret = t.send(None)
        print("=====ret1:", ret)

        # 第二次调用 send() 时，执行到 x = yield i 部分，相当于 x 接收到了通过 send() 传入的值
        ret = t.send(9)
        print("=====ret2:", ret)

        ret = t.send(8)
        print("=====ret3:", ret)

        ret = t.send(7)
        print("=====ret4:", ret)
    except StopIteration as exp:
        print("=====value:", exp.value)
            
    
if __name__ == "__main__":
    main()
```

上面代码的输出为：

```txt
=====ret1: 0
=====x: 9
=====ret2: 1
=====x: 8
=====value: OK
```

当生成器中 yield 后没有值返回时，再调用 send() 或者 next() 会抛出 StopIteration 异常，所以我们需要捕获该异常。若生成器中有返回值，可以在捕获异常时获取它。



## 使用for循环遍历生成器

生成器中的值也可以通过 for 循环获取，类似于调用 next() 方法，例子如下：

```python
def test():
    """生成器
    """
    for i in range(5):
        yield i
        
      
def main():
    """使用 for 循环调用生成器
    """
    t = test()
    for i in t:
        print(i)
          
    
if __name__ == "__main__":
    main()
```



协程，更像是一个过程，类似线程和进程，协程是由调用方来控制程序执行和切换的过程。python 中的协程是由生成器转化而来，即通过扩展生成器的功能来实现协程。

python中，生成器和协程一样，都是使用了 yield 关键字的函数，但是通常来说，生成器仅仅是向外部输出数据，并将代码的执行交给调用方，协程不仅可以向外部输出数据，还能通过 send() 方法接收外部传进来的数据，即 yield data 和 x = yield data。yield 类似于一种控制流程的方式。



# yield from

一个含有 yield from 的方法也是一个生成器，与 yield 不同的是，yield from 后面如果跟一个生成器，那么可以直接调用这个生成器，而不再需要 next() 函数。yield from 后面也可一跟一个可迭代对象，此时相比于 for 循环加 yield 输出数据要方便。例子如下：

```python
def test01():
    """yield from 后面可以跟一个可迭代对象
    """
    yield from [1, 2, 3, 4, 5]
    return "ok"


def test02():
    """yield from 后面跟一个生成器
    """
    print("=====start=====")
    ret = yield from test01()
    # 还可以获取子生成器的值
    print("=====ret:", ret)
    
  
def main():
    t = test02()
    try:
        print("=====t1:", next(t))
        print("=====t2:", next(t))
        print("=====t3:", next(t))
        print("=====t4:", next(t))
        print("=====t5:", next(t))
        print("=====t6:", next(t))
    except StopIteration:
        pass
    
    
if __name__ == "__main__":
    main()
```

上面到妈的输出结果为：

```txt
=====start=====
=====t1: 1
=====t2: 2
=====t3: 3
=====t4: 4
=====t5: 5
=====ret: ok
```

可以看出，生成器 test02() 里面嵌套了一个生成器 test01()。

