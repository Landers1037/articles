---
title: 源码剖析-HTTPAuth
name: vue-flask4
date: 2020-01-15
id: 0
tags: [flask,python]
categories: []
abstract: ""
---


源码剖析-HTTPAuth认证的实现


<!--more-->


源码剖析-HTTPAuth认证的实现

<!--more-->

# HTTPAuth

## 实现功能

提供密码验证，token验证，多重验证等方式的前后端验证服务

### 提供的类

```python
HTTPAuth
HTTPBasicAuth
HTTPDigestAuth
HTTPTokenAuth
MultiAuth
```

### HTTPAuth

基础类，完成基础的密码认证

```python
class HTTPAuth(object):
    def __init__(self, scheme=None, realm=None):
        self.scheme = scheme
        self.realm = realm or "Authentication Required"
        self.get_password_callback = None
        self.auth_error_callback = None

        def default_get_password(username):
            return None

        def default_auth_error():
            return "Unauthorized Access"

        self.get_password(default_get_password)
        self.error_handler(default_auth_error)

    def get_password(self, f):
        self.get_password_callback = f
        return f

    def error_handler(self, f):
        @wraps(f)
        def decorated(*args, **kwargs):
            res = f(*args, **kwargs)
            res = make_response(res)
            if res.status_code == 200:
                # if user didn't set status code, use 401
                res.status_code = 401
            if 'WWW-Authenticate' not in res.headers.keys():
                res.headers['WWW-Authenticate'] = self.authenticate_header()
            return res
        self.auth_error_callback = decorated
        return decorated

    def authenticate_header(self):
        return '{0} realm="{1}"'.format(self.scheme, self.realm)

    def get_auth(self):
        auth = request.authorization
        if auth is None and 'Authorization' in request.headers:
            # Flask/Werkzeug do not recognize any authentication types
            # other than Basic or Digest, so here we parse the header by
            # hand
            try:
                auth_type, token = request.headers['Authorization'].split(
                    None, 1)
                auth = Authorization(auth_type, {'token': token})
            except ValueError:
                # The Authorization header is either empty or has no token
                pass

        # if the auth type does not match, we act as if there is no auth
        # this is better than failing directly, as it allows the callback
        # to handle special cases, like supporting multiple auth types
        if auth is not None and auth.type.lower() != self.scheme.lower():
            auth = None

        return auth

    def get_auth_password(self, auth):
        password = None

        if auth and auth.username:
            password = self.get_password_callback(auth.username)

        return password

    def login_required(self, f):
        @wraps(f)
        def decorated(*args, **kwargs):
            auth = self.get_auth()

            # Flask normally handles OPTIONS requests on its own, but in the
            # case it is configured to forward those to the application, we
            # need to ignore authentication headers and let the request through
            # to avoid unwanted interactions with CORS.
            if request.method != 'OPTIONS':  # pragma: no cover
                password = self.get_auth_password(auth)

                if not self.authenticate(auth, password):
                    # Clear TCP receive buffer of any pending data
                    request.data
                    return self.auth_error_callback()

            return f(*args, **kwargs)
        return decorated

    def username(self):
        if not request.authorization:
            return ""
        return request.authorization.username
```

核心装饰器`login_required`，用户登录验证函数，置于路由前时用于在请求到来时判断用户的登录状态，访问请求头里的用户token信息。如果登录状态则允许继续执行访问路由，否则返回状态错误信息

### HTTPBasicAuth

基础认证方式，用户名密码

```python
class HTTPBasicAuth(HTTPAuth):
    def __init__(self, scheme=None, realm=None):
        super(HTTPBasicAuth, self).__init__(scheme or 'Basic', realm)

        self.hash_password_callback = None
        self.verify_password_callback = None

    def hash_password(self, f):
        self.hash_password_callback = f
        return f

    def verify_password(self, f):
        self.verify_password_callback = f
        return f

    def authenticate(self, auth, stored_password):
        if auth:
            username = auth.username
            client_password = auth.password
        else:
            username = ""
            client_password = ""
        if self.verify_password_callback:
            return self.verify_password_callback(username, client_password)
        if not auth:
            return False
        if self.hash_password_callback:
            try:
                client_password = self.hash_password_callback(client_password)
            except TypeError:
                client_password = self.hash_password_callback(username,
                                                              client_password)
        return client_password is not None and \
            stored_password is not None and \
            safe_str_cmp(client_password, stored_password)
```

继承自HTTPAuth类，核心装饰器函数`verify_password`，`authenticate`，`login_required`

使用`@get_password`回调函数或者`verify_password`回调函数，对于明文密码可以直接使用`@get_password`来验证，装饰器从请求信息中获取auth信息，获取用户名和密码，验证用户的登录状态，验证成功后由`@login_required`修饰的路由即可以访问



### HTTPTokenAuth

基于token请求头的认证

```python
class HTTPTokenAuth(HTTPAuth):
    def __init__(self, scheme='Bearer', realm=None):
        super(HTTPTokenAuth, self).__init__(scheme, realm)

        self.verify_token_callback = None

    def verify_token(self, f):
        self.verify_token_callback = f
        return f

    def authenticate(self, auth, stored_password):
        if auth:
            token = auth['token']
        else:
            token = ""
        if self.verify_token_callback:
            return self.verify_token_callback(token)
        return False

```

核心装饰器函数`verify_token`，`login_required`

路由前添加`@login_required`装饰器，在每次请求路由前都会进行登录验证，定义`@verify_token`装饰器，用于校验请求头里的token是否和服务器上保存的token一致。每一次`login_required`被调用都会调用`verify_token`并返回验证结果，验证通过则可以继续访问路由

### MultiAuth

多重认证

```python
class MultiAuth(object):
    def __init__(self, main_auth, *args):
        self.main_auth = main_auth
        self.additional_auth = args

    def login_required(self, f):
        @wraps(f)
        def decorated(*args, **kwargs):
            selected_auth = None
            if 'Authorization' in request.headers:
                try:
                    scheme, creds = request.headers['Authorization'].split(
                        None, 1)
                except ValueError:
                    # malformed Authorization header
                    pass
                else:
                    for auth in self.additional_auth:
                        if auth.scheme == scheme:
                            selected_auth = auth
                            break
            if selected_auth is None:
                selected_auth = self.main_auth
            return selected_auth.login_required(f)(*args, **kwargs)
        return decorated
```

使用时传入多个认证实例

```python
basic_auth = HTTPBasicAuth()
token_auth = HTTPTokenAuth(scheme='Bearer')
multi_auth = MultiAuth(basic_auth, token_auth)
```

### 令牌保存

对于令牌token信息的保存可以使用`itsdangerous`库来管理

```python
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
serializer = Serializer("一串用于加密的字符串", expires_in=1800)
token = serializer.dumps({'username': 'mm'})
```

这样就可以生成用户名为mm的token信息