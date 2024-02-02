---
title: Python中如何查看对象的内存分配
date: 2020-04-11 22:05:23
tags: python
categories: Python
---



对于大多数开发者来说，项目运行时发生内存泄漏的情况可能都经历过，那么当Python项目发生这种情况时，我们如何排查呢？这时候就可以使用python中的 tracemalloc 和 objgraph 这两个库。下面介绍下它们的使用方法。

<!--more-->

## tracemalloc的使用

不多说，先上代码：

```python
import tracemalloc


class MemoryTrace:

    __slots__ = ('frame', 'mem_top', 'key_type', 'mapping_key_type')

    def __init__(self, frame=1, mem_top=10, key_type="lineno"):
        self.frame = frame
        self.mem_top = mem_top
        self.key_type = key_type
        self.mapping_key_type = {
            "lineno": self.lineno,
            "traceback": self.traceback,
            "deference": self.deference,
        }

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            result = (self.mapping_key_type
                      .get(self.key_type)
                      (func, *args, **kwargs))

            self.end()

            return result

        return wrapper

    def lineno(self, func, *args, **kwargs):
        """
        display the files that most top memory was allocated
        """
        print("========lineno=======")
        tracemalloc.start()
        result = func(*args, **kwargs)
        snapshot = tracemalloc.take_snapshot()
        top_stats = snapshot.statistics("lineno")

        for top in top_stats[0:self.mem_top]:
            print("mem-trace ", top)

        return result

    def traceback(self, func, *args, **kwargs):
        """
        display the traceback of the biggest memory block
        """
        print("========traceback=======")
        tracemalloc.start(self.frame)
        result = func(*args, **kwargs)
        snapshot = tracemalloc.take_snapshot()
        top_stats = snapshot.statistics("traceback")
        top_one = top_stats[0]

        print("blocks count: %s" % top_one.count)
        print("size: %.1f KiB" % (top_one.size / 1024))
        for top in top_one.traceback.format():
            print("mem-trace ", top)

        return result

    def deference(self, func, *args, **kwargs):
        """
        compute the deference between two snapshot
        """
        tracemalloc.start(self.frame)
        snapshot_start = tracemalloc.take_snapshot()
        result = func(*args, **kwargs)
        snapshot_end = tracemalloc.take_snapshot()
        top_stats = snapshot_end.compare_to(snapshot_start, "lineno")

        for top in top_stats[:self.mem_top]:
            print("mem-trace ", top)

        return result

    def end(self):
        trace_malloc_module_usage = tracemalloc.get_tracemalloc_memory()
        print("trace alloc module use memory: %.1f KiB" %
              (trace_malloc_module_usage / 1024))

        current_size, peak_size = tracemalloc.get_traced_memory()
        print("current size: %.1f KiB" % (current_size / 1024))
        print("peak size: %.1f KiB" % (peak_size / 1024))

    def write(self, snapshot, filename):
        pass
```

上面类主要有以下功能：

```txt
1.跟踪对象的内存分配
2.统计每个文件中已分配block信息，包括：行号、占用内存总大小、blocks数量、每个blocks平均占用内存大小
3.比较两个snapshot之间的不同，以便定位内存泄露问题
```

使用 tracemalloc 库，我们可以跟踪代码中对象的内存分配情况和不同时刻的内存变化情况。

调用例子如下：

```python
from MemoryTrace.memory_trace import MemoryTrace


@MemoryTrace(frame=20, key_type="lineno")
def func():
    d = [dict(zip('xy', (0, 1))) for i in range(1000000)]
    t = [tuple(zip('xy', (0, 1))) for i in range(1000000)]


if __name__ == '__main__':
    func()
```

上述例子的结果如下：

```txt
========lineno=======
mem-trace  memory_trace.py:104: size=126 KiB, count=2002, average=64 B
mem-trace  memory_trace.py:103: size=19.2 KiB, count=161, average=122 B
mem-trace  memory_trace.py:47: size=448 B, count=1, average=448 B
mem-trace  memory_trace.py:46: size=448 B, count=1, average=448 B
trace alloc module use memory: 216.2 KiB
current size: 155.2 KiB
peak size: 438862.8 KiB
```





## objgraph的使用

使用 objgraph 需要先安装：pip install objgraph



```python
import datetime
import objgraph


class ObjectTrace:

    __slots__ = ("limit", "graph_type", "growth_file")

    def __init__(self, limit=8,
                 graph_type="growth",
                 growth_file=None):
        self.limit = limit
        self.graph_type = graph_type
        self.growth_file = growth_file

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            result = None

            if self.graph_type == "growth":
                result = self.object_trace(func, *args, **kwargs)
            return result
        return wrapper

    def object_trace(self, func, *args, **kwargs):
        """
        display the growth information of objects
        """
        start = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        if self.growth_file:
            with open(self.growth_file, "w") as file:
                file.write("start:" + start + "\n")
                objgraph.show_growth(self.limit, file=file)
                result = func(*args, **kwargs)
                objgraph.show_growth(self.limit, file=file)

                end = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
                file.write("end:" + end + "\n")
        else:
            print("========object trace========")
            print("start:", start)
            objgraph.show_growth(self.limit)
            result = func(*args, **kwargs)
            objgraph.show_growth(self.limit)
            end = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
            print("end:", end)
        return result

    @staticmethod
    def object_backref(obj, max_depth=10,
                       backref_file="object_backref.dot"):
        print("========object backref========")
        trace_objects = objgraph.by_type(obj)
        try:
            objgraph.show_backrefs(trace_objects[0],
                                   max_depth,
                                   filename=backref_file)
        except IndexError:
            pass
```

使用 objgraph 库，可以展示python中每类对象的内存增长情况。它会生成一个 .dot 文件，展示代码中对象的引用情况，可以用来检查代码中是否存在循环引用。在 ubuntu 中，可以使用如下方式打开 .dot 文件：

```txt
1. sudo apt-get install xdot
2. 使用XDot方式打开文件
```



调用例子如下：

```python
from MemoryTrace.object_trace import ObjectTrace


seq = list()


class Test(object):
    pass


@ObjectTrace()
def func():
    o = Test()
    seq.append(o)
    return
    seq.remove(o)


if __name__ == '__main__':
    func()
    ObjectTrace.object_backref("Test")
```

上述例子的结果为：

```txt
========object trace========
start: 2020-04-11 22:30:51
function                       2155     +2155
dict                           1173     +1173
wrapper_descriptor             1116     +1116
tuple                           924      +924
weakref                         819      +819
method_descriptor               818      +818
builtin_function_or_method      685      +685
getset_descriptor               439      +439
Test        1        +1
end: 2020-04-11 22:30:51
========object backref========
Graph written to object_backref.dot (24 nodes)
```



上面介绍了 python 中 tracemalloc 和 objgraph 库的使用，它们在排查内存泄漏时非常有用，感兴趣的同学可以继续深入了解。