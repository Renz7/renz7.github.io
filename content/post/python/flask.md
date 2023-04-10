---
title: "Flask"
date: 2023-04-10T17:37:08+08:00
draft: false
---

## what is wsgi
Web服务器网关接口 (Web Server Gateway Interface).Web服务器网关接口是为Python语言定义的Web服务器和Web应用程序或框架之间的一种简单而通用的接口
`app.py`
```
    def wsgi_app(self, environ: dict, start_response: t.Callable) -> t.Any:
        """The actual WSGI application. This is not implemented in
        :meth:`__call__` so that middlewares can be applied without
        losing a reference to the app object. Instead of doing this::

            app = MyMiddleware(app)

        It's a better idea to do this instead::

            app.wsgi_app = MyMiddleware(app.wsgi_app)

        Then you still have the original application object around and
        can continue to call methods on it.

        .. versionchanged:: 0.7
            Teardown events for the request and app contexts are called
            even if an unhandled error occurs. Other events may not be
            called depending on when an error occurs during dispatch.
            See :ref:`callbacks-and-errors`.

        :param environ: A WSGI environment.
        :param start_response: A callable accepting a status code,
            a list of headers, and an optional exception context to
            start the response.
        """
        ctx = self.request_context(environ)
        error: t.Optional[BaseException] = None
        try:
            try:
                ctx.push()
                response = self.full_dispatch_request()
            except Exception as e:
                error = e
                response = self.handle_exception(e)
            except:  # noqa: B001
                error = sys.exc_info()[1]
                raise
            return response(environ, start_response)
        finally:
            if "werkzeug.debug.preserve_context" in environ:
                environ["werkzeug.debug.preserve_context"](_cv_app.get())
                environ["werkzeug.debug.preserve_context"](_cv_request.get())

            if error is not None and self.should_ignore_error(error):
                error = None

            ctx.pop(error)

    def __call__(self, environ: dict, start_response: t.Callable) -> t.Any:
        """The WSGI server calls the Flask application object as the
        WSGI application. This calls :meth:`wsgi_app`, which can be
        wrapped to apply middleware.
        """
        return self.wsgi_app(environ, start_response)
```
## flask 中的上下文变量
1. g
2. session
3. request
4. app


## what and why stack and contextVar
`global.py`摘录
```
_cv_app: ContextVar["AppContext"] = ContextVar("flask.app_ctx")
__app_ctx_stack = _FakeStack("app", _cv_app)
app_ctx: "AppContext" = LocalProxy(  # type: ignore[assignment]
    _cv_app, unbound_message=_no_app_msg
)
current_app: "Flask" = LocalProxy(  # type: ignore[assignment]
    _cv_app, "app", unbound_message=_no_app_msg
)
g: "_AppCtxGlobals" = LocalProxy(  # type: ignore[assignment]
    _cv_app, "g", unbound_message=_no_app_msg
)

_no_req_msg = """\
Working outside of request context.

This typically means that you attempted to use functionality that needed
an active HTTP request. Consult the documentation on testing for
information about how to avoid this problem.\
"""
_cv_request: ContextVar["RequestContext"] = ContextVar("flask.request_ctx")
__request_ctx_stack = _FakeStack("request", _cv_request)
request_ctx: "RequestContext" = LocalProxy(  # type: ignore[assignment]
    _cv_request, unbound_message=_no_req_msg
)
request: "Request" = LocalProxy(  # type: ignore[assignment]
    _cv_request, "request", unbound_message=_no_req_msg
)
session: "SessionMixin" = LocalProxy(  # type: ignore[assignment]
    _cv_request, "session", unbound_message=_no_req_msg
)
```
隔离多个app环境使用
1. contextVar
2. LocalProxy 

存在两个stack
`__app_ctx_stack` 应用上下文
`__request_ctx_stack` 请求上下文