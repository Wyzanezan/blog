---
title: Python标准库之contextlib
date: 2019-12-01 09:44:38
tags: python
categories: Python
---

contextlib 库提供了一些接口，使我们能更容易实现 with 语法。下面主要介绍下 contextmanager 和 suppress 的用法。

<!--more-->

## with语法的实现

平时我们实现读取文件的操作时，会使用 with 语法，像下面这样：

```python
with open("data.txt") as f:
    f.read()
```

这样读取完文件后，会自动执行 f.close() 操作，关闭文件对象。

实际上，我们也可以用过实现魔法函数 \_\_enter\_\_() 和 \_\_exit\_\_() 来自定义 with 语法，像下面这样：

```python
class MyOpen():
    def __init__(self, file, flag):
        self.f = open(file, flag)
        
    def __enter__(self):
        """with ... as中as后面的内容
        """
        return self.f
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """with语句块执行完成后执行
        """
        self.f.close()
        
        
if __name__ == "__main__":
    with MyOpen("test.txt", "w") as f:
        f.write("hello jupyter!")
```

其中， \_\_enter\_\_() 函数的返回值是 with ... as 后面的对象，\_\_exit\_\_()函数中定义with语句块执行完成后的操作。



## contextmanager

contextmanager 函数是一个装饰器，可以使用它来实现 with 语法，而不用实现 \_\_enter\_\_() 和 \_\_exit\_\_() 。上文中实现 with 语法的例子可以写成下面这样：

```python
from contextlib import contextmanager

@contextmanager
def MyOpen(name, state):
    try:
        f = open(name, state)
        yield f
    finally:
        f.close()
      

if __name__ == "__main__":
    with MyOpen("test.txt", "w") as f:
        f.write("hello jupyter notebook!")
```

使用 contextmanager 实现 with 语法时，被修饰的方法必须是一个生成器，生成器返回的值就作为 as 后面的值。

下面再看一个例子：

```python
from contextlib import contextmanager
 
@contextmanager
def example():
    l = [1,2,3,4]
    print('start')
    try:
        # raise Exception('test')
        yield l
    finally:
        print('end')

with example() as msg:
    for i in msg:
        print(i)
```

如果在 contextmanager 修饰的函数中发生异常时，我们可以在函数中捕获该异常，像下面这样：

```python
@contextmanager
def example():
    l = [1, 2, 3, 4]
    print('start')
    try:
        yield l
        raise Exception('test')
    except:
        print("exception")
    finally:
        print('end')
```

也可以在 with 语句块之外捕获该异常：

```python
@contextmanager
def example():
    l = [1, 2, 3, 4]
    print('start')
    try:
        yield l
        raise Exception('test')
    finally:
        print('end')


try:
    with example() as msg:
        for i in msg:
            print(i)
except:
    pass
```



如果异常发生在 yeild 之前，则 yield 不会返回到 with 语句中，with语句块会抛出RuntimeError异常：

```python
@contextmanager
def example():
    l = [1, 2, 3, 4]
    print('start')
    try:
        raise Exception('test')
        yield l
    except:
        print("exception")
    finally:
        print('end')


with example() as msg:
    for i in msg:
        print(i)
```

上面的代码会抛出如下异常：

```python
raceback (most recent call last):
  File "tmp.py", line 17, in <module>
    with example() as msg:
  File "D:\software\Python36\lib\contextlib.py", line 83, in __enter__
    raise RuntimeError("generator didn't yield") from None
RuntimeError: generator didn't yield
```



contextmanager 的源码如下：

```python
def contextmanager(func):
    """@contextmanager decorator.

    Typical usage:

        @contextmanager
        def some_generator(<arguments>):
            <setup>
            try:
                yield <value>
            finally:
                <cleanup>

    This makes this:

        with some_generator(<arguments>) as <variable>:
            <body>

    equivalent to this:

        <setup>
        try:
            <variable> = <value>
            <body>
        finally:
            <cleanup>

    """
    @wraps(func)
    def helper(*args, **kwds):
        return _GeneratorContextManager(func, args, kwds)
    return helper
```

contextmanager 会返回一个_GeneratorContextManager 对象。 _GeneratorContextManager 类中实现了 \_\_enter\_\_() 和 \_\_exit\_\_() 方法，如下：

```python
    def __init__(self, func, args, kwds):
        self.gen = func(*args, **kwds)
        self.func, self.args, self.kwds = func, args, kwds
        # Issue 19330: ensure context manager instances have good docstrings
        doc = getattr(func, "__doc__", None)
        if doc is None:
            doc = type(self).__doc__
        self.__doc__ = doc
    
    def __enter__(self):
        try:
            return next(self.gen)
        except StopIteration:
            raise RuntimeError("generator didn't yield") from None

    def __exit__(self, type, value, traceback):
        if type is None:
            try:
                next(self.gen)
            except StopIteration:
                return False
            else:
                raise RuntimeError("generator didn't stop")
        else:
            if value is None:
                # Need to force instantiation so we can reliably
                # tell if we get the same exception back
                value = type()
            try:
                self.gen.throw(type, value, traceback)
            except StopIteration as exc:
                # Suppress StopIteration *unless* it's the same exception that
                # was passed to throw().  This prevents a StopIteration
                # raised inside the "with" statement from being suppressed.
                return exc is not value
            except RuntimeError as exc:
                # Don't re-raise the passed in exception. (issue27122)
                if exc is value:
                    return False
                # Likewise, avoid suppressing if a StopIteration exception
                # was passed to throw() and later wrapped into a RuntimeError
                # (see PEP 479).
                if type is StopIteration and exc.__cause__ is value:
                    return False
                raise
            except:
                # only re-raise if it's *not* the exception that was
                # passed to throw(), because __exit__() must not raise
                # an exception unless __exit__() itself failed.  But throw()
                # has to raise the exception to signal propagation, so this
                # fixes the impedance mismatch between the throw() protocol
                # and the __exit__() protocol.
                #
                if sys.exc_info()[1] is value:
                    return False
                raise
            raise RuntimeError("generator didn't stop after throw()")
```

_GeneratorContextManager初始化时，会接收一个生成器函数，\_\_enter\_\_()方法中会返回该生成器函数的值。



## suppress的使用

suppress会返回一个上下文管理器对象，如果with语句块中出现异常时，会忽略 suppress 中指定的异常，并且退出程序。例子如下：

```python
try:
    os.remove('somefile.tmp')
except FileNotFoundError:
    pass

try:
    os.remove('someotherfile.tmp')
except FileNotFoundError:
    pass
    
# 上面的异常捕获可以写成下面这样

from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove('somefile.tmp')

with suppress(FileNotFoundError):
    os.remove('someotherfile.tmp')
```

```python
import contextlib


class NonFatalError(Exception):
    pass


def non_idempotent_operation():
    raise NonFatalError(
        'The operation failed because of existing state'
    )


with contextlib.suppress(NonFatalError):
    print('trying non-idempotent operation')
    non_idempotent_operation()
    print('succeeded!')

print('done')

# output
"""
trying non-idempotent operation
done
"""
```

supress中需要传入异常名称，不能为空。



以上就是 contextlib中 contextmanager 和 suppress 的用法。