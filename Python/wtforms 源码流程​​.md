# wtforms 源码流程:four_leaf_clover:

## Table of contents

<extoc></extoc>

## 源码流程第一步创建类:deciduous_tree:

```python 
from flask_wtf import FlaskForm   # 导入文件的第一步就已经开始执行代码
Class Form(with_metaclass(FormMeta,BaseForm))  # wtforms/form.py 中执行这样一句

def with_metaclass(meta,base=object): # meta 继承了type类，动态创建类的过程
  return meta("NewBase",(base,),{}) # 创建了一个NewBase 类，这个类的基类是BaseForm

# meta 加括号会执行 FormMeta 的 __init__ 方法
class FormMeta(type):  # 执行 __init__ 方法后,NewBase类 以及它的派生类中都增加了两个静态字段
    def __init__(cls, name, bases, attrs):
        type.__init__(cls, name, bases, attrs)
        cls._unbound_fields = None  # 这里的cls 就是 NewBase这个类
        cls._wtforms_meta = None

```

## 源码流程第二步创建字段:deciduous_tree:

```python
from wtforms import StringField,SubmitField
from wtforms.validators import DataRequired

class MenuForm (FlaskForm):
    '''菜单表单'''
    name = StringField (label = "菜单名称",validators = [DataRequired ("菜单不能为空!")],
                        render_kw = {"class":"form-control","placeholder":"请输入菜单名称!"})
    submit = SubmitField ("提交",render_kw = {"class":"btn btn-primary"})
    
# 创建字段的时候都发生了什么，以StringField 为例
class Field(object):  # 实例化StringField会先执行 __init__ 方法，在执行init方法之前会执行new方法
    def __new__(cls, *args, **kwargs):
      if '_form' in kwargs and '_name' in kwargs:
        return super(Field, cls).__new__(cls)
      else:
        return UnboundField(cls, *args, **kwargs) # 这里得到的是一个UnboundField的对象
# 再看看 UnboundField 类实例化的时候都做了什么
class UnboundField(object):
    _formfield = True
    creation_counter = 0
    def __init__(self, field_class, *args, **kwargs):
        UnboundField.creation_counter += 1
        self.field_class = field_class
        self.args = args
        self.kwargs = kwargs
        self.creation_counter = UnboundField.creation_counter
        # 这里 得到一个 Creation_counter 计数器，每实例化一次这个技术器都会加一
```

## 源码流程第三步，执行`FormMeta` 类的 `__call__` 方法:deciduous_tree:

```python
form = MenuForm()  # 实例化MenuForm() 类，MenuForm 类继承的NewBase 这个类 而 NewBase 是由 FormMeta 创建的，所以 对象加括号执执行 FormMeta 类的__call__ 方法
    def __call__(cls, *args, **kwargs):
    	#  _unbound_fields   _wtforms_meta 就是刚创建类的时候添加的两个静态字段 
        if cls._unbound_fields is None:
            fields = []
            for name in dir(cls): # dir 会取得类里所有的属性和方法
                if not name.startswith('_'): # 取出 下划线开头的
                    unbound_field = getattr(cls, name) # 获取自定义字段(就是上面MenuForm中的name字段)
                    if hasattr(unbound_field, '_formfield'): # 如果这个对象里面有_formfield 这个方法或者属性 在UnboundFied 这个类里面定义了一个_formField = True
                        fields.append((name, unbound_field))# 将 字段名和字段对象封装到一个元组中添加到fields 列表中
            fields.sort(key=lambda x: (x[1].creation_counter, x[0])) # 按照刚才的计数器进行排序，这里控制的是页面上input 框的显示顺序
            cls._unbound_fields = fields # 将排序好的列表赋值给 cls.unbound_fields
        if cls._wtforms_meta is None:
            bases = []
            for mro_class in cls.__mro__: # __mro__ 获取类之间所有的继承关系
                if 'Meta' in mro_class.__dict__:#__dict__ 可以获取类中所有的成员，判断有没有“Meta”
                    bases.append(mro_class.Meta)# 将这些Meta 添加到列表中
            cls._wtforms_meta = type('Meta', tuple(bases), {})#动态创建类，继承所有的Meta类 这个时候_wtforms_meta 就是一个继承了所有Meta类的 Meta类了
        return type.__call__(cls, *args, **kwargs)
```

## 源码流程第四步，执行Form 的 `__init__` 方法:deciduous_tree:

```python
class Form(with_metaclass(FormMeta, BaseForm)): 
  	Meta = DefaultMeta # 这个Meta是DefaultMeta 这个类，在上篇的type动态创建Meta 这个类的时候用到 坑
    def __init__(self, formdata=None, obj=None, prefix='', data=None, meta=None, **kwargs):
        meta_obj = self._wtforms_meta() # Meta 类实例化，执行DefaulMeta的__init__ 方法
        if meta is not None and isinstance(meta, dict):	# 这里说我们在实例化的时候还可以传一个meta = {"name","xiang"} 这样参数
            meta_obj.update_values(meta) # 这里会执行DefaultMeta 中的update_value 方法，将meta传过来的字典更新到self的属性中 self 也就是form这个对象
        super(Form, self).__init__(self._unbound_fields, meta=meta_obj, prefix=prefix)#这里要执行BaseForm的__init__ 方法 
        # 执行BaseForm __init__ 方法得到 self._fields[name] = StringField()
        for name, field in iteritems(self._fields): # 这里紧接着就循环了 self._fields 这个是从BaseForm 里面弄到的
            setattr(self, name, field) #这里给self设定了一个属性， self.name = StringField()
        self.process(formdata, obj, data=data, **kwargs) # 执行Form对象的process方法
```

## 源码流程第五步，执行BaseForm的 `__init__` 方法:deciduous_tree:

```python
class BaseForm(object):
    def __init__(self, fields, prefix='', meta=DefaultMeta()):
        if prefix and prefix[-1] not in '-_;:/.':  # 处理前缀
            prefix += '-'

        self.meta = meta  
        self._prefix = prefix
        self._errors = None
        self._fields = OrderedDict()
		# 如果这些字段中含有items 方法， 说明fields 中应该会有字典格式的字段对象
        if hasattr(fields, 'items'):
            fields = fields.items()
		# 这里是处理本地化的，用作翻译错误信息提示
        translations = self._get_translations()
        extra_fields = []
        if meta.csrf:# DefaultMeta 中默认 csrf = False 可以设置为True 会给我们自动生成一个[(field_name,unbound_field)]
            self._csrf = meta.build_csrf(self) # 这里实例化了一个 SessionCSRF()对象
            extra_fields.extend(self._csrf.setup_form(self)) #SessionCRSF 对象调用了setup_form()这个方法 这里面给form_meat赋值了，然后还执行了父类的setup_form 方法  return[(field_name,unbound_field)]  这个就是给我自动生成了一个csrf字段  最终是执行了Field 的__call__方法，生成了一个隐藏的input 标签 
        for name, unbound_field in itertools.chain(fields, extra_fields):
          	# 循环 _unbound_fields，[(field_name,unbound_field)]  注意chain 的用法
            options = dict(name=name, prefix=prefix, translations=translations)#这个options字典中封装了字段名，前缀，和translations
            field = meta.bind_field(self, unbound_field, options)
            # 这里会执行unbound_field 的bind 方法 return self.field_class(*self.args, **kw) 会得到一个StringField的对象
            self._fields[name] = field # 这里创建了一个有序字典 以name 为健，StringField对象为value 如 {"username":StringField()}  在上一步中循环
     
class DefaultMeta(object): # 上面的bind_field返回了一个unboundfield 执行 bind 方法后的结果
    def bind_field(self, form, unbound_field, options):
        """
        :return: A bound field
        """
        return unbound_field.bind(form=form, **options) # 执行 UnboundField的bind 方法
      
      
class UnboundField(object):     
    def bind(self, form, name, prefix='', translations=None, **kwargs):
        kw = dict(
            self.kwargs,
            _form=form,
            _prefix=prefix,
            _name=name,
            _translations=translations,
            **kwargs
        )
        return self.field_class(*self.args, **kw) # 这个fields_class 就是 StringField 类，返回了这么一个StringField 对象，一直到这里才执行了真正实例化了StringField 类
```

## 源码流程第六步，执行BaseForm 的 `process` 方法:deciduous_tree:

```python
    def process(self, formdata=None, obj=None, data=None, **kwargs):
    	# 这里是对传进来的formdata 做了一个审核，要求这个formdata必须有getlist 方法，要不然就会报错
        formdata = self.meta.wrap_formdata(self, formdata)
        if data is not None:
            kwargs = dict(data, **kwargs)
        for name, field, in iteritems(self._fields):# 这里又循环了self._fields 分别拿到 name 和StringFields()对象
            if obj is not None and hasattr(obj, name):# 	这里传了obj,并且name(字段)在obj 这个对象中，这一步没有怎么细看，最后得到的结果是self.object_data 和self.data 里面赋值，这是在页面上显示默认值的  到这里Form的__init__ 方法已经执行完了
                field.process(formdata, getattr(obj, name))
            elif name in kwargs:
                field.process(formdata, kwargs[name])
            else:
                field.process(formdata)
```

## 源码流程第七步，模板渲染:deciduous_tree:

```python
# 页面显示标签，需要调用Field 的__str__ 方法
 def __str__(self):
     return self() # 对象加括号执行 Field 的__call__ 方法
 def __call__(self, **kwargs): 
     return self.meta.render_field(self, kwargs) # 执行meta 的render_field 方法
 ########## meta 类的 render_field 方法
 def render_field(self, field, render_kw):
      other_kw = getattr(field, 'render_kw', None) # 获取 render_kw
      if other_kw is not None:
          render_kw = dict(other_kw, **render_kw) # 组成字典
      return field.widget(field, **render_kw) # 执行每一个field 的 widget 方法 这里的widget是TexInput 的对象，对象加括号会执行类的__call__ 方法
    
 widget = widgets.TextInput()  # 以StringField 为例
# 这里渲染了页面上的标签
 def __call__(self, field, **kwargs):
        kwargs.setdefault('id', field.id)
        kwargs.setdefault('type', self.input_type)
        if 'value' not in kwargs:
            kwargs['value'] = field._value()
        return HTMLString('<input %s>' % self.html_params(name=field.name, **kwargs))
```



## 源码流程第八步，执行Form 的 `validate` 方法 :deciduous_tree:

```python
    def validate(self):
        extra = {} # 这个存放我们定义的钩子函数
        for name in self._fields: # 循环字段得到name
            inline = getattr(self.__class__, 'validate_%s' % name, None)# 从MenuForm 中获取validate_name 方法
            if inline is not None:
                extra[name] = [inline] # 写入到字典中
        return super(Form, self).validate(extra) # 执行 BaseForm 的validate 方法
      #################
       def validate(self, extra_validators=None):# extra_validators就是包含钩子函数的字典
        self._errors = None
        success = True
        for name, field in iteritems(self._fields): # 查看我们定义的钩子函数是否是合法的
            if extra_validators is not None and name in extra_validators:# 自定义的钩子函数必须是类中的字段，不能是其它随便的东西
                extra = extra_validators[name] # 取得钩子函数
            else:
                extra = tuple() # 如果不是就赋值一个空的元组
            if not field.validate(self, extra): # 这里执行的是 field 对象的validate 方法
                success = False
        return success
```

## 源码流程第九步，执行Field 的 `validate` 方法:deciduous_tree:

```python 
    def validate(self, form, extra_validators=tuple()):
        self.errors = list(self.process_errors) # 默认是一个空元组
        stop_validation = False

        # 预验证，这里可以扩展
        try:
            self.pre_validate(form)
        except StopValidation as e:
            if e.args and e.args[0]:
                self.errors.append(e.args[0]) # 添加错误信息
            stop_validation = True
        except ValueError as e:
            self.errors.append(e.args[0])

        # 执行验证函数
        if not stop_validation:
            # 这里将 validates,和钩子函数 联到一起
            chain = itertools.chain(self.validators, extra_validators)
            stop_validation = self._run_validation_chain(form, chain) # 执行_run_validation_chain

        # Call post_validate
        try:
            self.post_validate(form, stop_validation) # 执行 post_validate 方法
        except ValueError as e:
            self.errors.append(e.args[0]) # 写错误信息

        return len(self.errors) == 0
      
      
    def _run_validation_chain(self, form, validators):
        for validator in validators: # 循环验证函数
            try:
                validator(form, self) # 执行 验证函数
            except StopValidation as e:
                if e.args and e.args[0]:
                    self.errors.append(e.args[0])
                return True
            except ValueError as e:
                self.errors.append(e.args[0]) # 添加错误信息

        return False
    def post_validate(self, form, validation_stopped):
        """
        Override if you need to run any field-level validation tasks after
        normal validation. This shouldn't be needed in most cases.
        这句话是说在执行完上面所有的验证后，你还想写字段级别的验证可以重写这个方法，但是一般情况下你不需这样
        """
        pass
```

