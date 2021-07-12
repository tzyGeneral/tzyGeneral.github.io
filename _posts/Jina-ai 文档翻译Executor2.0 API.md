# Jina-ai 文档翻译`Executor`2.0 API



目录

+ 最小工作示例
  + 存python
  + 使用 YAML
+ 执行器API
  + 继承
  + `__init__` 构造函数
  + 方法命名
  + `@requests`装饰器
    + 默认绑定：`@requests`无`on=`
    + 多重绑定： `@requests(on=[...\])`
    + 无约束
  + 方法签名
  + 方法参数
  + 方法返回
    + 示例1：嵌入文档`blob`
    + 示例2：通过分割文档添加块
    + 示例3：`id`仅保留文档
  + YAML接口
  + 加载和保存`Executor`的YAML配置
  + 在流程外使用`Executor`
+ Executor 内置功能
  + 1.x 于 2.x
  + 工作区
  + 元
  + `.metas` & `.runtime_args`
  + 处理参数
+ 迁移实践
  + 输入`Tina hello fashion`
+ Executors in Action
  + Fastai
  + Pytorch Lightning
  + Paddle
  + Tensorflow
  + MindSpore
  + Scikit-learn
  + PyTorch
  + ONNX-Runtime

## 最小工作示例

### 纯Python

```python
from jina import Executor, Flow, Document, requests


class MyExecutor(Executor):

    @requests
    def foo(self, **kwargs):
        print(kwargs)


f = Flow().add(uses=MyExecutor)

with f:
    f.post(on='/random_work', inputs=Document(), on_done=print)
```

### 使用 YAML

`MyExecutor`如上所述。将其另存为`foo.py`.

`my.yml`：

```yaml
jtype: MyExecutor
metas:
  py_modules:
    - foo.py
  name: awesomeness
  description: my first awesome executor
requests:
  /random_work: foo
```

`Executor`从 YAML构建：

```python
from jina import Executor

my_exec = Executor.load_config('my.yml')
```

`Executor`来自 YAML 的Flow 使用：

```python
from jina import Flow, Document

f = Flow().add(uses='my.yml')

with f:
    f.post(on='/random_work', inputs=Document(), on_done=print)
```

## 执行器 API

`Executor`处理`DocumentArray`就地通过与装饰功能`@requests`。

- 一个`Executor`应直接从继承`jina.Executor`类。
- 一个`Executor`类是一组具有共享状态（通过`self`）的函数；它可以包含任意数量的具有任意名称的函数。
- 装饰的函数`@requests`将根据它们的`on=`端点被调用。

#### 继承

每个新的执行器都应该直接从`jina.Executor`.

1.x 继承树被移除。`Executor`不再具有多态性。

您可以自由命名您的执行程序类。

### `__init__` 构造函数

如果您的 executor 定义了`__init__`，则需要携带`**kwargs`签名并`super().__init__(**kwargs)` 在正文中调用：

```python
from jina import Executor


class MyExecutor(Executor):

    def __init__(self, foo: str, bar: int, **kwargs):
        super().__init__(**kwargs)
        self.bar = bar
        self.foo = foo
```



在这里，`kwargs`包含来自 YAML 配置并在启动时注入的`metas`和`requests`（代表请求到函数的映射）值`runtime_args`。请注意，您可以访问他们的价值观`__init__`通过身体`self.metas` / `self.requests`/ `self.runtime_args`，或发送到之前修改它们的值`super().__init__()`。

`__init__`如果您`Executor`不包含初始状态，则无需实施。



### 方法命名

`Executor`的方法可以自由命名。没有装饰的方法与`@requests`Jina无关。

### `@requests` 装饰器

`@requests`定义何时调用函数。它有一个关键字`on=`来定义端点。

要调用 Executor 的函数，请使用`Flow.post(on=..., ...)`. 例如，给定：

```python
f
```

