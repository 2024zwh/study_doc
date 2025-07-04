## 一、概念

### 1.1 flask概念

> flask是一款基于Werkzeug 和jinja2 的微服务框架。只保留了核心功能werkzeug和jinjia2功能，支持灵活第三方扩展组件，可以根据自己的需求搭建，非常的

### 1.2 Werkzeug介绍

> Werkzeug 是一个 **Python 的 WSGI（Web Server Gateway Interface）工具库**，它为开发者提供了构建 Web 应用所需的底层基础设施和实用工具。作为 Python Web 开发领域的重要工具，Werkzeug 以其灵活性、模块化设计和丰富的功能著称，广泛应用于 Flask 等知名框架的底层实现中

### 1.3 jinja介绍

> “Jinja” 是一个 **Python 的模板引擎**，通常指 **Jinja2**（即 Jinja 的第二代版本），它被广泛用于 Web 开发中动态生成 HTML、XML 或其他文本格式的文件。Jinja2 的设计灵感来源于 Django 模板语言，但功能更强大且更灵活，是 Flask 等 Python Web 框架的默认模板引擎

### 1.4 一些协议

1. **WSGI** (Web Server Gateway Interface)

  - **定义**：

    > WSGI是Python官方定义的同步接口标准，用于规范Web服务器与Web应用程序/框架之间的通信。

  - **核心作用**：

    - 解耦Web服务器（如Nginx）和Web框架（如Django、Flask）。

    - 提供统一的接口，使不同的服务器和框架可以自由组合。

  - **特点**：

    - 同步模式：每个请求按顺序处理，适合传统的HTTP请求-响应模型。

    - 中间件支持：通过WSGI中间件可以扩展功能（如身份验证、日志记录）。

  - 适用场景：

    - 同步框架（如Flask、Django的传统模式）。

    - 部署生产环境时，常与uWSGI或Gunicorn等服务器结合使用。

2. **uwsgi**

  - **定义**：

    > uwsgi是uWSGI服务器自定义的二进制通信协议，用于服务器与Web框架之间的高效数据传输。

  - **核心作用**：

    - 作为uWSGI服务器与前端服务器（如Nginx）之间的通信协议。

    - 相比HTTP协议，uwsgi更高效，支持更多功能（如进程间通信、文件传输）。

  - **特点**：

    - 二进制协议：比文本协议（如HTTP）更轻量，传输速度更快。

    - 灵活性：支持多种传输方式（TCP、Unix Socket等）。

  - **与uWSGI的关系**：

    - uWSGI是实现了WSGI协议的服务器软件。

    - uwsgi是uWSGI服务器内部使用的协议，用于与前端服务器通信。

3. **ASGI** (Asynchronous Server Gateway Interface)

  - **定义**：

    > ASGI是WSGI的异步扩展，用于处理现代Web中的高并发和实时通信需求（如WebSocket、HTTP/2）。

  - **核心作用**：

    - 支持异步编程模型（基于Python的asyncio）。

    - 兼容传统HTTP请求和实时通信协议（如WebSocket）。

  - **特点**：

    - 异步模式：可同时处理多个请求，适合高并发场景。

    - 协议兼容性：支持HTTP、HTTP/2、WebSocket等。

  - **适用场景**：

    - 异步框架（如FastAPI、Starlette）。

    - 实时应用（如聊天室、在线游戏、实时数据推送）。

4. **uWSGI**

  - **定义**：

    > uWSGI是一个高性能的Web服务器/应用服务器，支持WSGI、ASGI和uwsgi协议。

  - **核心作用**：

    - 运行Python Web应用（如Django、Flask）。

    - 通过uwsgi协议与Nginx等前端服务器通信。

  - **特点**：

    - 支持多种协议：WSGI、ASGI、HTTP、FastCGI等。

    - 高度可配置：支持进程/线程管理、缓存、负载均衡、动态扩展等。

    - 生产环境常用：适合部署高流量的Web应用。

## 二、核心使用

### 2.1 路由

>  一种是使用默认route， 一种是使用蓝图来注册理由

#### 2.1.1 **route**：

```python
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello, World'
```



#### 2.1.2 **蓝图**

- **为啥使用蓝图**
  - 拆分应用，可以分模块注册，非常的灵活，是一个针对大模型的理想方案
  - 可以使用同一个URL前缀或者子域，不需要手动添加
  - 蓝图支持定义请求钩子（如 `before_request`）和错误处理函数（如 `@bp.errorhandler(404)`

- **使用**

  - **创建蓝图**

    ```python
    from flask import Blueprint, render_template, redirect, url_for
    
    # 创建蓝图对象，名称为 'auth'
    blog_bp = Blueprint('auth', __name__,
                        template_folder='templates',  # 模板路径（相对于当前文件）
                        static_folder='static') # 静态文件路径
    
    @auth_bp.route('/login')
    def login():
        return render_template('auth/login.html')
    
    @auth_bp.route('/logout')
    def logout():
        return redirect(url_for('auth.login'))
    
    @auth_bp.route('/register')
    def register():
        return render_template('auth/register.html')
    ```

    

  - **注册蓝图**

    ```python
    # 文件: app.py
    from flask import Flask
    from auth.routes import auth_bp
    
    app = Flask(__name__)
    
    # 注册蓝图，设置 URL 前缀为 '/auth'
    app.register_blueprint(auth_bp, url_prefix='/auth')
    
    if __name__ == '__main__':
        app.run(debug=True)
    ```

  - **错误处理**

    ```python
    @auth_bp.errorhandler(404)
    def auth_not_found(e):
        return render_template('auth/404.html')
    ```

#### 2.1.3 **路由变量规则**

- **概念**

  > 通过把 URL 的一部分标记为 `<variable_name>` 就可以在 URL 中添加变量。标记的 部分会作为关键字参数传递给函数。通过使用 `<type:variable_name>` ，可以 选择性的加上一个转换器，为变量指定规则

- **例子**

  ```python
  @app.route('/user/<username>')
  def show_user_profile(username):
      # show the user profile for that user
      return 'User %s' % escape(username)
  
  @app.route('/post/<int:post_id>')
  def show_post(post_id):
      # show the post with the given id, the id is an integer
      return 'Post %d' % post_id
  
  @app.route('/path/<path:subpath>')
  def show_subpath(subpath):
      # show the subpath after /path/
      return 'Subpath %s' % escape(subpath)
  ```

- **规则类型**

  | 类型     | 含义                                |
  | -------- | ----------------------------------- |
  | `string` | （缺省值） 接受任何不包含斜杠的文本 |
  | `int`    | 接受正整数                          |
  | `float`  | 接受正浮点数                        |
  | `path`   | 类似 `string` ，但可以包含斜杠      |
  | `uuid`   | 接受 UUID 字符串                    |

#### 2.1.4 **设置HTTP方法**

> 通过route函数methods 方法指定

```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```



#### 2.1.5 **URL重定向**

- **最基本重定向**

  > 使用 `redirect()` 函数可以将用户重定向到指定的 URL

  ```python
  from flask import Flask, redirect
  
  app = Flask(__name__)
  
  # 外部路由
  @app.route('/external')
  def external_redirect():
      return redirect('https://www.baidu.com')
      
  # 内部路由
  @app.route('/')
  def index():
      return 'Hello, World!'
  
  @app.route('/internal')
  def internal_redirect():
      return redirect(url_for('index'))  # 重定向到 index 路由
  ```

- **动态传参**

  ```python
  @app.route('/user/<username>')
  def user_profile(username):
      return f'User: {username}'
  
  @app.route('/redirect-to-user')
  def redirect_to_user():
      return redirect(url_for('user_profile', username='Alice'))
  ```

- **重定向到上一个页面**

  ```python
  from flask import request
  
  @app.route('/back')
  def go_back():
      return redirect(request.referrer or url_for('index'))
  ```

- **注意**

  > 使用蓝图的时候重定向，url_for('bpname.viewfunc'),bpname注册蓝图名称，viewfunc指该蓝图的视图函数

  **示例**

  ```python
  from flask import Blueprint, render_template, redirect, url_for
  
  # 创建蓝图对象，名称为 'auth'
  blog_bp = Blueprint('auth', __name__,
                      template_folder='templates',  # 模板路径（相对于当前文件）
                      static_folder='static') # 静态文件路径
  
  @auth_bp.route('/login')
  def login():
      return render_template('auth/login.html')
  
  @auth_bp.route('/logout')
  def logout():
      return redirect(url_for('auth.login'))
  
  ```

#### 2.1.6 **错误处理**

> 使用 redirect()函数可以重定向到错误页面。也可以使用 abort()可以 更早退出请求，并返回错误代码

```python
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
```



### 2.2 操作请求数据

> Flask 中由全局 对象 request来提供请求信息

#### 2.2.1 request全局对象是怎么保证线程安全的？

> Flask 的全局对象（如 `request`、`g`）并不是真正的全局变量，而是 **`LocalProxy` 代理对象**

- **Local，LocalStack，LocalProxy介绍**

  - **Local**

    > local 是一个线程/上下文隔离的容器，用于存储于当前线程或上下文相关的数据。每个flask的请求上下文都有自己打的独立副本，不会相互干扰

    ```python
    from werkzeug.local import Local
    
    local = Local()
    local.user = "Alice"  # 当前线程/上下文的 user 值为 "Alice"
    
    def another_context():
        local.user = "Bob"  # 在另一个线程/上下文中，user 值为 "Bob"
        print(local.user)  # 输出: Bob
    
    another_context()
    print(local.user)  # 输出: Alice（主线程的值未变）
    ```

  - **LocalStack**

    > LocalStack 是一个基于Local的栈结构，运行咋系统一个线程/上下文中管理多个值。通过LIFO（后进先出）规则管理数据

    ```python
    from werkzeug.local import LocalStack
    
    stack = LocalStack()
    stack.push("A")
    stack.push("B")
    print(stack.top)  # 输出: B
    
    stack.pop()
    print(stack.top)  # 输出: A
    ```

  - **LocalProxy**

    > LocalProxy 是一个代理对象，用于透明访问Local或LocalStack中的数据。简化了对隔离数据的访问，避免显示操作Local和LocalStack

    ```python
    from werkzeug.local import Local, LocalProxy
    
    local = Local()
    local.user = "Alice"
    
    # 创建 LocalProxy 代理
    user = LocalProxy(lambda: local.user)
    
    print(user)  # 输出: Alice
    ```

- **总结**

  > 每次有新的请求到达时，Flask 会创建一个新的请求上下文，并将这个上下文推入一个栈中。由于使用了 `LocalStack`，每个线程都有自己的栈实例，因此每个线程只能访问到属于自己的那个请求上下文。这样就确保了不同线程之间的请求不会相互干扰

#### 2.2.2 request的具体操作

- **获取请求方法和查询参数**

  ```python
  @app.route('/method', methods=['GET', 'POST'])
  def show_method():
      return f'This request was made using {request.method}'
  
  @app.route('/query')
  def get_query():
      name = request.args.get('name', 'Anonymous')  # 获取参数，若不存在则返回默认值
      age = request.args.get('age', 0, type=int)    # 指定类型转换
      return f'Hello, {name}! Your age is {age}.'
  ```

- **获取表单数据**

  ```python
  @app.route('/form', methods=['POST'])
  def handle_form():
      username = request.form.get('username')
      password = request.form.get('password')
      return f'Username: {username}, Password: {password}'
  ```

- **获取 JSON 数据**

  > 客户端发送的是 JSON 数据（`Content-Type: application/json`），可以通过 `request.get_json()` 或 `request.json` 获取

  ```python
  @app.route('/api/data', methods=['POST'])
  def api_data():
      data = request.get_json()  # 或直接使用 request.json
      return f'Received data: {data["name"]}, {data["age"]}'
  ```

-  **处理文件上传**

  > 文件上传通过 `multipart/form-data` 编码实现

  ```python
  @app.route('/upload', methods=['POST'])
  def upload_file():
      if 'file' not in request.files:
          return 'No file uploaded.'
      file = request.files['file']
      if file.filename == '':
          return 'No file selected.'
      file.save(f'uploads/{file.filename}')  # 保存文件到指定路径
      return 'File uploaded successfully!'
  ```

- **获取请求头**

  ```python
  @app.route('/')
  def index():
      user_agent = request.headers.get('User-Agent')  # 获取 User-Agent
      return f'Your User-Agent is {user_agent}'
  ```

- **获取cookie**

  > 户端发送的 Cookie 存储在 `request.cookies` 中

  ```python
  from flask import request
  
  @app.route('/')
  def index():
      username = request.cookies.get('username')
      return {}
  ```

#### 2.2.3 请求上下文生命周期

- **生命周期**

  - **创建**：当HTTP请求到达应用，Flask自动创建请求上下文
  - **销毁**：请求处理完成后自动销毁

- **触发时机**

  - **自动触发**：HTTP请求到达时Flask自动创建
  - **手动触发**：通过 `app.test_request_context()` 模拟请求（如测试脚本）

- **核心对象**：

  request，session

### 2.3 操作响应数据

> 在flask中，响应对象(Response)是通过视图函数返回给客户端的数据结构，flask会根据视图函数的返回值自动将其转换为合法的HTTP响应对象

**响应对象的类型**

| 返回值类型       | `Content-Type`       | 示例                                                 |
| ---------------- | -------------------- | ---------------------------------------------------- |
| 字符串           | `text/html`          | `return "Hello"`                                     |
| 字典             | `application/json`   | `return {"key": "value"}`                            |
| `jsonify` 返回值 | `application/json`   | `return jsonify({"key": "value"})`                   |
| HTML 模板        | `text/html`          | `return render_template("index.html")`               |
| 流式数据         | 指定的 `mimetype`    | `return Response(generate(), mimetype='text/plain')` |
| 文件             | 根据文件类型自动推断 | `return send_file("file.txt")`                       |

### 2.4 session（会话）

#### 2.4.1 核心机制

- **基于 Cookie 的实现**：
  - **Session ID**：Flask 为每个用户生成唯一的 Session ID（通过 `os.urandom` 或 `secrets` 模块生成），存储在客户端的 Cookie 中。
  - **加密存储**：Session 数据（如字典）在服务器端被加密后，以 Base64 编码的形式嵌入 Cookie 中。例如，`session['user_id'] = 123` 会被加密为类似 `abc123xyz` 的字符串存储在 Cookie 中。
  - **签名验证**：每次请求时，Flask 会验证 Cookie 中的 Session 数据是否被篡改（通过 `secret_key` 加密签名）
- **服务器端存储**（可选）：
  - 默认情况下，Session 数据加密后存储在客户端的 Cookie 中。但可以通过 **Flask-Session** 扩展，将 Session 数据存储在服务器端（如 Redis、数据库等），进一步提升安全性。

#### 2.4.2 基本操作

```python
from flask import Flask, session

# 初始化
app = Flask(__name__)
app.secret_key = 'slsdaljdljdlj'  # 设置加密密钥（必须）

# 设置Session
@app.route('/login')
def login():
    session['user_id'] = 123  # 存储用户ID
    session['username'] = 'john_doe'  # 存储用户名
    return 'Logged in!'

# 获取Session
@app.route('/profile')
def profile():
    user_id = session.get('user_id')  # 安全获取数据
    if user_id:
        return f'Welcome, user {user_id}!'
    return 'Not logged in.', 403

#删除Session
@app.route('/logout')
def logout():
    session.pop('user_id', None)  # 删除指定键
    session.clear()  # 清除所有 Session 数据
    return 'Logged out!'
```



#### 2.4.3 session的高级配置

- **设置session持久化时间**

  ```python
  from datetime import timedelta
  
  app.permanent_session_lifetime = timedelta(days=7)  # 设置为7天
  ```

- 使用flas-sesion存储到服务端

  ```python
  from flask import Flask
  from flask_session import Session
  import redis
  
  app = Flask(__name__)
  app.config['SESSION_TYPE'] = 'redis'  # 使用 Redis 存储
  app.config['SESSION_REDIS'] = redis.from_url('redis://localhost:6379')  # Redis 连接
  Session(app)
  ```

  