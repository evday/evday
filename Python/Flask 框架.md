# Flask 框架:four_leaf_clover:

Flask 是一个基于Python 开发，依赖于`jinjia2` 模板引擎 和 `Werkzeug WSGI`  服务的微型web开发框架。

## 启动Flask :deciduous_tree:

```python
pip install flask # 安装

from flask import Flask  # 新建flask 项目自动生成，妈个鸡的我居然没写出来
app = Flask(__name__) # 实例化Flask 类
app.route("/") # 路由
def hello_world(): # 视图
  	return "Hello World"
if __name__ == "__main__":
  	app.run()
```

## 配置文件:deciduous_tree:

### 设置配置文件的4种方式:palm_tree:

#### 方式一，直接写入脚本:leaves:

```python 
from flask import Flask
app = Flask(__name__)
app.Config["DEBUG"] = True
app.Config["SECRTE_key"] = "lift if short you need python"

#可以自典的update
app.config.update(
  	DEBUG = True,
  	SECRTE_key = "lift if short you need python"
)
```

#### 方式二，写入环境变量:leaves:

```python
set YOURAPPLICATION_SETTINGS=\path\to\settings.cfg  # 将配置文件路径写入到环境变量
from flask import Flask
app = Flask(__name__)
app.Config.from_envvar("YOURAPPLICATION_SETTINGS")
```

#### 方式三，写入配置文件:leaves:

```python 
# config.py
SECRET_KEY = "life is short you need python"
DEBUG = True
# 使用方式一，导入类
import config
app = Flask(__name__)
app.Config.from_object(config)
# 使用方式二
app.Config.from_object("config.py")
```

#### 方式四，创建不同的类:leaves:

```python
# config.py

class Config(object):
    DEBUG = False
    TESTING = False
    DATABASE_URI = 'sqlite://:memory:'

class ProductionConfig(Config):
    DATABASE_URI = 'mysql://user@localhost/foo'

class DevelopmentConfig(Config):
    DEBUG = True

class TestingConfig(Config):
    TESTING = True
    
config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,

    'default': DevelopmentConfig
}

# 使用
from flask import Flask 
app = Flask(__name__)
app.config.from_obj(Config["development"])  
```

## 路由系统:deciduous_tree:

### 添加路由的两种方式:palm_tree:

```python
# 装饰器方法
from flask import Flask 
@app.route("/")
def hello_world():
  	return "hello world"
# 方式二，add_url_rule 装饰器方法在源码中也是用的add_url_rule
def hello_world():
  	return "hello world"
app.add_url_rule(rule="/",endpoint="index",view_func=hello_world,methods=["GET","POST"])
```

### url 传参:palm_tree:

```python
@app.route('/user/<username>')  # 匹配字符串
@app.route('/book/<int:book_id>') # 数字
@app.route('/price/<float:price>') # 浮点数
```

### `app.route()&app.add_url_rule()` 参数 :palm_tree:

```python
rule # url
view_func # 视图函数
defaults = None # 默认值，url需要参数时，使用defaults = {"k","v"} 提供参数
endpoint = None # 用于反向生成url, url_for("名称")
methods = None  #允许请求的方式
strict_slashes = None # 对url 后的“/” 是否严格要求
redirect_to  # 重定向地址
```

### 自定义路由匹配规则:palm_tree:

```python
from flask import Flask
app = Flask(__name__)
from werkzeug.routing import BaseConverter 
class RegexConverter(BaseConverter):
  	"""自定义url匹配正则表达式"""
    def __init__(self, map,regex):
        super(RegexConverter,self).__init__(map)
        self.regex = regex
    def to_python(self, value):
      	 """
         路由匹配时，匹配成功后传递给视图函数中参数的值
         :param value: 
         :return: 
         """
        return int(value)
    def to_url(self, value):
      	"""
        使用url_for反向生成URL时，传递的参数经过该方法处理，返回的值用于生成URL中的参数
        :param value: 
        :return: 
        """
      	val = super(RegexConverter,self).to_url(value)
        return val
 # 添加到converters
 app.url_map.converters['regex'] = RegexConverter
 # 进行使用
 app.route('/index/<regex("\d+"):nid>',endpoint="index")
 def index(nid):
 	 url_for("index",nid=123) # 反向生成，就会去执行to_url方法
     return "Index"
    
 if __name__ == "__main__":
  	app.run()
```

## 视图:deciduous_tree:

### CBV:palm_tree:

```python
from flask import Flask,render_template,redirect,request,session,views
app = Flask(__name__)
app.secrect_key = "hello world"

def wrapper(func):  # 登录验证装饰器
  	def inner(*args,**kwargs):
      	if not session.get("user_info"):
          	return redirect("/login")
        ret = func(*args,**kwargs)
        return ret
    return inner

class IndexView(views.MethosView):
  	methods = ["GET","POST"]
    decorators = [wrapper,] # 给所有方法加上登录验证的装饰器
    def get(self):   #如果是get请求需要执行的代码
                v = url_for('index')
                print(v)
                return "GET"

            def post(self):  #如果是post请求执行的代码
                return "POST"
app.add_url_rule('/index',view_func = IndexView.as_view(name='index'))m #name 相当于endpoint
if __name__ == "__main__":
	app.run()
```

### FBV:palm_tree:

```python 
# 装饰器方法
from flask import Flask 
@app.route("/")
def hello_world():
  	return "hello world"
# 方式二，add_url_rule 装饰器方法在源码中也是用的add_url_rule
def hello_world():
  	return "hello world"
app.add_url_rule(rule="/",endpoint="index",view_func=hello_world,methods=["GET","POST"])
```

## 请求与响应:deciduous_tree:

```python 
from flask import Flask
    from flask import request
    from flask import render_template
    from flask import redirect
    from flask import make_response

    app = Flask(__name__)


    @app.route('/login.html', methods=['GET', "POST"])
    def login():

        # 请求相关信息
        # request.method
        # request.args
        # request.form
        # request.values
        # request.cookies
        # request.headers
        # request.path
        # request.full_path
        # request.script_root
        # request.url
        # request.base_url
        # request.url_root
        # request.host_url
        # request.host
        # request.files
        # obj = request.files['the_file_name']
        # obj.save('/var/www/uploads/' + secure_filename(f.filename))

        # 响应相关信息
        # return "字符串"
        # return render_template('html模板路径',**{})
        # return redirect('/index.html')

        # response = make_response(render_template('index.html'))
        # response是flask.wrappers.Response类型
        # response.delete_cookie('key')
        # response.set_cookie('key', 'value')
        # response.headers['X-Something'] = 'A value'
        # return response


        return "内容"

    if __name__ == '__main__':
        app.run()
```





















