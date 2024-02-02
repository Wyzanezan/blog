---
title: Django处理请求流程
date: 2019-11-28 22:31:01
tags: django
categories: Python
---

Django是Python的一个web框架，Django应用通过wsgi服务器与客户端进行http通信。今天介绍下从Django接收客户端http请求到生成响应过程中的流程。

<!--more-->

## 1. 请求响应流程

整体流程如下：

```python
get_wsgi_application() --> WSGIHandler() --> self.load_middleware() --> __call__() --> self.get_response() --> self._get_response()
```

request 经过 self._get_response()后，就生成了 response。



## 2. 请求响应具体流程

### 2.1 WSGIHandler 初始化

WSGIHandler类如下：

```python
class WSGIHandler(base.BaseHandler):
    request_class = WSGIRequest

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.load_middleware()

    def __call__(self, environ, start_response):
        set_script_prefix(get_script_name(environ))
        signals.request_started.send(sender=self.__class__, environ=environ)
        request = self.request_class(environ)
        response = self.get_response(request)

        response._handler_class = self.__class__

        status = '%d %s' % (response.status_code, response.reason_phrase)
        response_headers = [
            *response.items(),
            *(('Set-Cookie', c.output(header='')) for c in response.cookies.values()),
        ]
        start_response(status, response_headers)
        if getattr(response, 'file_to_stream', None) is not None and environ.get('wsgi.file_wrapper'):
            response = environ['wsgi.file_wrapper'](response.file_to_stream)
        return response
```

wsgi服务器接收到请求后，会初始化 WSGIHandler类，并调用\_\_call\_\_()方法。初始化时，会调用BaseHandler中的 load_middleware()方法。

self.load_middleware()方法如下：

```python
    def load_middleware(self):
        """
        Populate middleware lists from settings.MIDDLEWARE.

        Must be called after the environment is fixed (see __call__ in subclasses).
        """
        self._view_middleware = []
        self._template_response_middleware = []
        self._exception_middleware = []

        handler = convert_exception_to_response(self._get_response)
        for middleware_path in reversed(settings.MIDDLEWARE):
            middleware = import_string(middleware_path)
            try:
                mw_instance = middleware(handler)
            except MiddlewareNotUsed as exc:
                if settings.DEBUG:
                    if str(exc):
                        logger.debug('MiddlewareNotUsed(%r): %s', middleware_path, exc)
                    else:
                        logger.debug('MiddlewareNotUsed: %r', middleware_path)
                continue

            if mw_instance is None:
                raise ImproperlyConfigured(
                    'Middleware factory %s returned None.' % middleware_path
                )

            if hasattr(mw_instance, 'process_view'):
                self._view_middleware.insert(0, mw_instance.process_view)
            if hasattr(mw_instance, 'process_template_response'):
                self._template_response_middleware.append(mw_instance.process_template_response)
            if hasattr(mw_instance, 'process_exception'):
                self._exception_middleware.append(mw_instance.process_exception)

            handler = convert_exception_to_response(mw_instance)

        # We only assign to this when initialization is complete as it is used
        # as a flag for initialization being complete.
        self._middleware_chain = handler
```

load_middleware()方法中主要做了一下几步操作：

1. 调用 convert_exception_to_response() 方法，返回一个可调用对象，该方法接收另一个方法（这个方法负责处理request）作为参数

   convert_exception_to_response() 方法如下：

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

   该方法中用到了装饰器。

2.  mw_instance = middleware(handler)， 这一行代码会创建中间件的实例对象，后面的几个if语句再逐个判断中间件对象中是否定义了 process_view()、process_exception()等方法。

3.  handler = convert_exception_to_response(mw_instance)，这一行代码将中间件实例作为参数传入方法中，返回一个可调用对象。这里将中间件对象作为参数，后面会调用中间件中的\_\_call\_\_()方法来处理request。Django中，中间件类MiddlewareMixin的实现如下：

   ```python
   class MiddlewareMixin:
       def __init__(self, get_response=None):
           self.get_response = get_response
           super().__init__()
   
       def __call__(self, request):
           response = None
           if hasattr(self, 'process_request'):
               response = self.process_request(request)
           response = response or self.get_response(request)
           if hasattr(self, 'process_response'):
               response = self.process_response(request, response)
           return response
   ```

4.  最后一行代码 self._middleware_chain = handler 是将handler赋值给 _middleware_chain 属性。



### 2.2 WSGIHandler call()方法

WSGIHandler初始化完成后，会调用\_\_call\_\_()方法，该方法用来处理 request，返回response，实现如下：

```python
    def __call__(self, environ, start_response):
        set_script_prefix(get_script_name(environ))
        signals.request_started.send(sender=self.__class__, environ=environ)
        request = self.request_class(environ)
        response = self.get_response(request)

        response._handler_class = self.__class__

        status = '%d %s' % (response.status_code, response.reason_phrase)
        response_headers = [
            *response.items(),
            *(('Set-Cookie', c.output(header='')) for c in response.cookies.values()),
        ]
        start_response(status, response_headers)
        if getattr(response, 'file_to_stream', None) is not None and environ.get('wsgi.file_wrapper'):
            response = environ['wsgi.file_wrapper'](response.file_to_stream)
        return response
```

\_\_call\_\_()方法主要做了下面一些操作: 

1. self.request_class(environ) 这一行将 http request对象 转换为 wsgi request对象
2. self.get_response(request) 这一行代码会调用BaseHandler的get_response()方法，生成 http response对象。

下面具体介绍下self.get_response() 这个方法。

### 2.3 get_response() 方法

BaseHandler的get_response()方法用来处理request，并生成response。get_response()的源码如下：

```python
    def get_response(self, request):
        """Return an HttpResponse object for the given HttpRequest."""
        # Setup default url resolver for this thread
        set_urlconf(settings.ROOT_URLCONF)
        response = self._middleware_chain(request)
        response._closable_objects.append(request)
        if response.status_code >= 400:
            log_response(
                '%s: %s', response.reason_phrase, request.path,
                response=response,
                request=request,
            )
        return response
```

主要是 response = self.\_middleware_chain(request) 这一行代码，将 request 作为参数传入上文说到的 self.\_middleware_chain中，输出response。具体执行流程如下：

1. 上文说到，在 self.load_middleware()中会执行下面的代码：

   handler = convert_exception_to_response(self._get_response)

   convert_exception_to_response()方法的实现如下：

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

   所以，self.\_middleware_chain(request) 最终是调用了 self.\_get_response(request) 这个方法。

2. self.\_get_response()的实现如下：

   ```python
       def _get_response(self, request):
           """
           Resolve and call the view, then apply view, exception, and
           template_response middleware. This method is everything that happens
           inside the request/response middleware.
           """
           response = None
   
           if hasattr(request, 'urlconf'):
               urlconf = request.urlconf
               set_urlconf(urlconf)
               resolver = get_resolver(urlconf)
           else:
               resolver = get_resolver()
   
           resolver_match = resolver.resolve(request.path_info)
           callback, callback_args, callback_kwargs = resolver_match
           request.resolver_match = resolver_match
   
           # Apply view middleware
           for middleware_method in self._view_middleware:
               response = middleware_method(request, callback, callback_args, callback_kwargs)
               if response:
                   break
   
           if response is None:
               wrapped_callback = self.make_view_atomic(callback)
               try:
                   response = wrapped_callback(request, *callback_args, **callback_kwargs)
               except Exception as e:
                   response = self.process_exception_by_middleware(e, request)
   
           # Complain if the view returned None (a common error).
           if response is None:
               if isinstance(callback, types.FunctionType):    # FBV
                   view_name = callback.__name__
               else:                                           # CBV
                   view_name = callback.__class__.__name__ + '.__call__'
   
               raise ValueError(
                   "The view %s.%s didn't return an HttpResponse object. It "
                   "returned None instead." % (callback.__module__, view_name)
               )
   
           # If the response supports deferred rendering, apply template
           # response middleware and then render the response
           elif hasattr(response, 'render') and callable(response.render):
               for middleware_method in self._template_response_middleware:
                   response = middleware_method(request, response)
                   # Complain if the template response middleware returned None (a common error).
                   if response is None:
                       raise ValueError(
                           "%s.process_template_response didn't return an "
                           "HttpResponse object. It returned None instead."
                           % (middleware_method.__self__.__class__.__name__)
                       )
   
               try:
                   response = response.render()
               except Exception as e:
                   response = self.process_exception_by_middleware(e, request)
   
           return response
   ```

   该方法中，主要进行了以下几步操作：路由解析 --> 中间件校验 --> 调用视图函数获取response

3. 下面介绍下 “路由解析” 的流程：

   1）读取 settings 配置文件中的 ROOT_URLCONF 信息获取路由配置，获取 URLResolver 对象，即执行了下面的方法：

   ```python
   @functools.lru_cache(maxsize=None)
   def get_resolver(urlconf=None):
       if urlconf is None:
           urlconf = settings.ROOT_URLCONF
       return URLResolver(RegexPattern(r'^/'), urlconf)
   ```

   2）然后再执行 resolver_match = resolver.resolve(request.path_info) 这行代码，resolve()方法的实现如下：

   ```python
       def resolve(self, path):
           path = str(path)  # path may be a reverse_lazy object
           tried = []
           match = self.pattern.match(path)
           if match:
               new_path, args, kwargs = match
               for pattern in self.url_patterns:
                   try:
                       sub_match = pattern.resolve(new_path)
                   except Resolver404 as e:
                       sub_tried = e.args[0].get('tried')
                       if sub_tried is not None:
                           tried.extend([pattern] + t for t in sub_tried)
                       else:
                           tried.append([pattern])
                   else:
                       if sub_match:
                           # Merge captured arguments in match with submatch
                           sub_match_dict = {**kwargs, **self.default_kwargs}
                           # Update the sub_match_dict with the kwargs from the sub_match.
                           sub_match_dict.update(sub_match.kwargs)
                           # If there are *any* named groups, ignore all non-named groups.
                           # Otherwise, pass all non-named arguments as positional arguments.
                           sub_match_args = sub_match.args
                           if not sub_match_dict:
                               sub_match_args = args + sub_match.args
                           current_route = '' if isinstance(pattern, URLPattern) else str(pattern.pattern)
                           return ResolverMatch(
                               sub_match.func,
                               sub_match_args,
                               sub_match_dict,
                               sub_match.url_name,
                               [self.app_name] + sub_match.app_names,
                               [self.namespace] + sub_match.namespaces,
                               self._join_route(current_route, sub_match.route),
                           )
                       tried.append([pattern])
               raise Resolver404({'tried': tried, 'path': new_path})
           raise Resolver404({'path': path})
   ```

   函数返回了一个 ResolverMatch类的实例，后面通过这个实例，在获取处理请求的试图方法、请求参数等信息。该函数中，有下面一行代码 sub_match = pattern.resolve(new_path)，这行代码会返回一个 ResolverMatch() 对象，就是通过这个对象，来决定执行哪个视图的哪个方法。

   3）我们先来看看配置 url 时的三个方法：path, include, as_view。在下面的配置中会使用这三个方法：

   ```python
   from django.contrib import admin
   from django.urls import path, include
   
   
   urlpatterns = [
       path('admin/', admin.site.urls),
       path('test01/', include('cron.urls')),
       path("test02/", include('cron02.urls')),
       path("middleware/", include('test_middleware.urls')),
   ]
   
   
   from django.urls import path
   
   from .views import TestView
   
   urlpatterns = [
       path("cron/", TestView.as_view()),
   ]
   ```

   其中，path方法实现如下：

   ```python
   def _path(route, view, kwargs=None, name=None, Pattern=None):
       if isinstance(view, (list, tuple)):
           # For include(...) processing.
           pattern = Pattern(route, is_endpoint=False)
           urlconf_module, app_name, namespace = view
           return URLResolver(
               pattern,
               urlconf_module,
               kwargs,
               app_name=app_name,
               namespace=namespace,
           )
       elif callable(view):
           pattern = Pattern(route, name=name, is_endpoint=True)
           return URLPattern(pattern, view, kwargs, name)
       else:
           raise TypeError('view must be a callable or a list/tuple in the case of include().')
   
   
   path = partial(_path, Pattern=RoutePattern)
   ```

   根据view参数的类型不同，会执行不同的代码块。当path的第二个参数是TestView.as_view()时，会满足callable(view)这个条件，返回一个URLPattern类的对象，注意此时 view是可调用的。as_view()的实现如下：

   ```python
   	@classonlymethod
       def as_view(cls, **initkwargs):
           """Main entry point for a request-response process."""
           for key in initkwargs:
               if key in cls.http_method_names:
                   raise TypeError("You tried to pass in the %s method name as a "
                                   "keyword argument to %s(). Don't do that."
                                   % (key, cls.__name__))
               if not hasattr(cls, key):
                   raise TypeError("%s() received an invalid keyword %r. as_view "
                                   "only accepts arguments that are already "
                                   "attributes of the class." % (cls.__name__, key))
   
           def view(request, *args, **kwargs):
               self = cls(**initkwargs)
               if hasattr(self, 'get') and not hasattr(self, 'head'):
                   self.head = self.get
               self.setup(request, *args, **kwargs)
               if not hasattr(self, 'request'):
                   raise AttributeError(
                       "%s instance has no 'request' attribute. Did you override "
                       "setup() and forget to call super()?" % cls.__name__
                   )
               return self.dispatch(request, *args, **kwargs)
           view.view_class = cls
           view.view_initkwargs = initkwargs
   
           # take name and docstring from class
           update_wrapper(view, cls, updated=())
   
           # and possible attributes set by decorators
           # like csrf_exempt from dispatch
           update_wrapper(view, cls.dispatch, assigned=())
           return view
   ```

   as_view() 方法的主要功能就是根据 request 参数，确定要执行 view 中的哪个方法(post, get, put等)，当有代码调用 view 时，就会执行相应的 http 方法并返回 response。

   4）然后我们在回到 2）步结束的地方，当执行 sub_match = pattern.resolve(new_path) 这行代码时，会执行 URLPattern 中的 resolve()方法，返回 ResolverMatch对象，ResolverMatch对象初始化时的参数 func 的值就是 3）中 path()方法中的 view，后面会调用 view 这个函数获取response。

   5）最后，在 \_get\_response() 中会调用上面的 view函数，这部分代码为：

   ```python
           if response is None:
               wrapped_callback = self.make_view_atomic(callback)
               try:
                   response = wrapped_callback(request, *callback_args, **callback_kwargs)
               except Exception as e:
                   response = self.process_exception_by_middleware(e, request)
   ```

   首先判断是否需要添加事务，然后再执行 callback 函数，最后返回 response。

   

   以上就是 Django 处理 http 请求的大致流程，从源码中我们要能学习到写 python 代码的一些技巧。

    







