---
title: Python中装饰器与partial结合使用
date: 2020-01-01 16:31:44
tags: python
categories: Python
---

之前看 Django 框架源码的时候，发现装饰器与 partial() 函数能结合使用，今天介绍它们两个如何一起使用。

首先介绍 partial() 函数，然后再介绍装饰器与 partial() 结合使用的例子。

<!--more-->

## partial()

当调用一个有多个参数的函数时，我们可能想只传入一部分参数，另一部分参数使用默认值。此时可以直接调用这个函数，也可以使用 partial() 函数实现这一功能。例子如下：

```python
from functools import partial


def test(a, b, c):
    print(a)
    print(b)
    print(c)


if __name__ == '__main__':

    # 方式一：正常调用
    test(1, 2, 3)

    print("="*50)

    # 方式二：使用 partial 调用
    wrapped_func = partial(test, b=2, c=3)
    wrapped_func(1)
    
    print("="*50)
    # 再次给冻结的函数传参
    wrapped_func(4, b=5, c=6)
```

上面的例子中，先调用 partial() 函数，传入待执行函数和要冻结的参数。partial() 函数返回一个functools.partial对象，然后再调用它，同时传入剩余的参数即可。同时 partial() 函数还可以重新给冻结的参数传值。



下面介绍下 partial() 在装饰器中的使用。



## 装饰器与partial

首先下面是 Django 源码中的一个例子：

```python
def load_middleware(self):
    """
    Populate middleware lists from settings.MIDDLEWARE.
    Must be called after the environment is fixed (see __call__ in subclasses).
    """
    # ...

    handler = convert_exception_to_response(self._get_response)
    # ...
```

```python
def convert_exception_to_response(get_response):
    """
    Wrap the given get_response callable in exception-to-response conversion.

    All exceptions will be converted. All known 4xx exceptions (Http404,
    PermissionDenied, MultiPartParserError, SuspiciousOperation) will be
    converted to the appropriate response, and all other exceptions will be
    converted to 500 responses.

    This decorator is automatically applied to all middleware to ensure that
    no middleware leaks an exception and that the next middleware in the stack
    can rely on getting a response instead of an exception.
    """
    @wraps(get_response)
    def inner(request):
        try:
            response = get_response(request)
        except Exception as exc:
            response = response_for_exception(request, exc)
        return response
    return inner
```

```python
def wraps(wrapped,
          assigned = WRAPPER_ASSIGNMENTS,
          updated = WRAPPER_UPDATES):
    """Decorator factory to apply update_wrapper() to a wrapper function

       Returns a decorator that invokes update_wrapper() with the decorated
       function as the wrapper argument and the arguments to wraps() as the
       remaining arguments. Default arguments are as for update_wrapper().
       This is a convenience function to simplify applying partial() to
       update_wrapper().
    """
    return partial(update_wrapper, wrapped=wrapped,
                   assigned=assigned, updated=updated)
```

```python
def update_wrapper(wrapper,
                   wrapped,
                   assigned = WRAPPER_ASSIGNMENTS,
                   updated = WRAPPER_UPDATES):
    """Update a wrapper function to look like the wrapped function

       wrapper is the function to be updated
       wrapped is the original function
       assigned is a tuple naming the attributes assigned directly
       from the wrapped function to the wrapper function (defaults to
       functools.WRAPPER_ASSIGNMENTS)
       updated is a tuple naming the attributes of the wrapper that
       are updated with the corresponding attribute from the wrapped
       function (defaults to functools.WRAPPER_UPDATES)
    """
    for attr in assigned:
        try:
            value = getattr(wrapped, attr)
        except AttributeError:
            pass
        else:
            setattr(wrapper, attr, value)
    for attr in updated:
        getattr(wrapper, attr).update(getattr(wrapped, attr, {}))
    # Issue #17482: set __wrapped__ last so we don't inadvertently copy it
    # from the wrapped function when updating __dict__
    wrapper.__wrapped__ = wrapped
    # Return the wrapper so this can be used as a decorator via partial()
    return wrapper
```

上面发生在 Django 通过 request 返回 response 时候的一段代码调用。主要的功能是给 self._get_response() 函数增加了一些属性。

执行流程如下：

```txt
1. 调用 convert_exception_to_response() 时会先执行 wraps(get_response)
2. 执行 wraps(get_response) 时，会返回 partial(update_wrapper, wrapped=wrapped,               	
	assigned=assigned, updated=updated)，即函数 update_wrapper() 对应的一个 functools.partial 对
	象，此时 wrapped 就是 get_response()
3. @wraps装饰器返回一个 functools.partial 对象后，wrapper 参数就是 inner
4. 最后将 inner 赋值给了 handler
```

上面的整个功能就是将被包装函数(self\.\_get_response())的属性赋值给包装它的函数(inner())，即在执行包装函数 inner() 之前，会做一些额外的操作。



下面看一个例子：

```python
from functools import partial


def update_wrapper(wrapper,
                   wrapped):
    print("wrapper:", wrapper)
    print("wrapped:", wrapped)

    # 其它操作

    return wrapper


def wraps(wrapped):
    return partial(update_wrapper, wrapped=wrapped)


def get_response(num):
    print("get response num:", num)
    return num


def outer(get_response):

    @wraps(get_response)
    def inner(num):
        response = get_response(num)
        return response
    return inner


if __name__ == '__main__':
    ret = outer(get_response)
    ret(5)


"""
打印结果：
wrapper: <function outer.<locals>.inner at 0x0000020231A819D8>
wrapped: <function get_response at 0x0000020231A818C8>
get response num: 5
"""
```

从打印结果可以看出，wrapped 就是 get_response() 函数对象，wrapper 是 inner() 函数对象。

上面比较关键的一个函数是 update_wrapper()，因为在它里面会对最终的执行结果产生影响。



在实际项目开发中，如果有需要，可以通过装饰器 + partial()函数的方式实现一些需求。