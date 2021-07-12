# Jina-ai 文档翻译 Document & DocumentArray



Document、Executor 和 Flow 是 Jina 中的三个基本概念

+ Document 是 Jina 中的基本数据类型
+ Executor 是 Jina 处理 Document 的方式
+ Flow 是 JIna 精简和拓展 Executor 的方式



## 关于 Document / DocumentArray 2.0 Api 的文档说明

`Document` 是 **Jina** 操作的基本数据类型。可以存放：文字、图片、视频、音频、图像或者 3D mesh

`DocumentArray` 是 `Document` 的列表容器，它完全支持 `Executor`， 作为 `Executor`的输入和输出。

可以说`Document`对**Jina**来说就像`np.float`对于Numpy一样，并且`DocumentArray`类似于`np.ndarray`

目录

+ 最小工作示例
+ `Document`应用程序接口
  + `document` 属性
    + 设置和取消设置属性
  + 构造`Document`
    + 唯一属性 `doc.content`
    + `doc.content`之间的转换
    + 支持稀疏数组
    + 多属性构造
    + 从Dict或JSON字符串构造
    + 从另一个构建`Document`
    + 从JSON、CSV、ndarray和文件构建
  + `Document`序列化
  + 添加递归到`Document`
    + 递归属性
  + 表示`Document`为字典或JSON
  + 可视化`Document`
  + 将相关性添加到`Document`
    + 相关属性
+ `DocumentArray`应用程序接口
  + 构造`DocumentArray`
  + 持久化`save()` `load()`
  + 访问元素
  + 遍历元素
  + 排序元素
  + 筛选元素
  + 在`DocumentArray`中使用`itertools`
  + 批量获取属性
  + 从标签访问嵌套属性
+ `DocumentArrayMemmap`应用程序接口
  + 创建`DocumentArrayMemmap`对象
  + 将文档添加到`DocumentArrayMemmap`对象
  + 清除`DocumentArrayMemmap`对象
    + 修改
  + 具有“只读”元素的可变序列
  + 对比`DocumentArray`
  + `DocumentArray`和`DocumentArrayMemmap`之间的转换
  + 通过`.reload()`方法保持一致性



## 最小工作示例

```python
from jina import Document

d = Document() 
```

## `Document`应用程序接口

### `document` 属性

一个`Document`对象具有以下属性，可以分为以下几类：

| 内容属性 | `.buffer`, `.blob`, `.text`, `.uri`, `.content`,`.embedding` |
| -------- | ------------------------------------------------------------ |
| 元属性   | `.id`, `.weight`, `.mime_type`, `.location`, `.tags`, `.offset`, `.modality`,`siblings` |
| 递归属性 | `.chunks`, `.matches`, `.granularity`,`.adjacency`           |
| 相关属性 | `.score`, `.evaluations`                                     |

#### 设置和取消设置属性

设置一个属性：

```python
from jina import Document

d = Document()
d.text = 'hello world'
```

```shell
<jina.types.document.Document id=9badabb6-b9e9-11eb-993c-1e008a366d49 mime_type=text/plain text=hello world at 4444621648>
```

取消设置属性：

```python
d.pop('text')
```

```shell
<jina.types.document.Document id=cdf1dea8-b9e9-11eb-8fd8-1e008a366d49 mime_type=text/plain at 4490447504>
```

取消设置多个属性：

```python
d.pop('text', 'id', 'mime_type')
```

```shell
<jina.types.document.Document at 5668344144>
```



### 构造 `Document`

##### 内容属性

| `doc.buffer`    | 此文档的原始二进制内容                                       |
| --------------- | ------------------------------------------------------------ |
| `doc.blob`      | 所述`ndarray`图像/音频/视频文件的                            |
| `doc.text`      | 文档的文本信息                                               |
| `doc.uri`       | 文档的 uri 可以是：本地文件路径、以 http 或 https 开头的远程 url 或数据 URI 方案 |
| `doc.content`   | 上述非空字段之一                                             |
| `doc.embedding` | `ndarray`本文档的嵌入                                        |

您可以分配`str`，`ndarray`，`buffer`或`uri`到`Document`。

```python
from jina import Document
import numpy as np

d1 = Document(content='hello')
d2 = Document(content=b'\f1')
d3 = Document(content=np.array([1, 2, 3]))
d4 = Document(content='https://static.jina.ai/logo/core/notext/light/logo.png')
```

```shell
<jina.types.document.Document id=2ca74b98-aed9-11eb-b791-1e008a366d48 mimeType=text/plain text=hello at 6247702096>
<jina.types.document.Document id=2ca74f1c-aed9-11eb-b791-1e008a366d48 buffer=DDE= mimeType=text/plain at 6247702160>
<jina.types.document.Document id=2caab594-aed9-11eb-b791-1e008a366d48 blob={'dense': {'buffer': 'AQAAAAAAAAACAAAAAAAAAAMAAAAAAAAA', 'shape': [3], 'dtype': '<i8'}} at 6247702416>
<jina.types.document.Document id=4c008c40-af9f-11eb-bb84-1e008a366d49 uri=https://static.jina.ai/logo/core/notext/light/logo.png mimeType=image/png at 6252395600>
```



内容将被自动分配到的`text`，`buffer`，`blob`，或`uri`领域。`id`并且`mime_type`在未给出时自动生成。

您可以`Document`在 Jupyter Notebook 中或通过调用`.plot()`.



#### 唯一属性 `doc.content`

需要注意的是一个`Document`只能包含一种类型的`content`：它要么是`text`，`buffer`，`blob`或`uri`。`text`先设置再设置`uri`将清除该`text`字段。

```python
d = Document(text='hello world')
d.uri = 'https://jina.ai/'
assert not d.text  # True

d = Document(content='https://jina.ai')
assert d.uri == 'https://jina.ai'  # True
assert not d.text  # True
d.text = 'hello world'

assert d.content == 'hello world'  # True
assert not d.uri  # True
```



#### `doc.content`之间的转换

您可以使用以下方法之间进行转换`.uri`，`.text`，`.buffer`和`.blob`：

```python
doc.convert_buffer_to_blob()
doc.convert_blob_to_buffer()
doc.convert_uri_to_buffer()
doc.convert_buffer_to_uri()
doc.convert_text_to_uri()
doc.convert_uri_to_text()
```

您可以使用 将 URI 转换为数据 URI（数据内联 URI 方案）`doc.convert_uri_to_datauri()`。这将获取资源并使其内联。

特别是，当您使用 image 时`Document`，有一些额外的助手可以实现更多转换：

```python
doc.convert_image_buffer_to_blob()
doc.convert_image_blob_to_uri()
doc.convert_image_uri_to_blob()
doc.convert_image_datauri_to_blob()
```

##### 设置嵌入

嵌入是 的高维表示`Document`。您可以将任何 Numpy 分配`ndarray`为 a`Document`的嵌入。

```python
import numpy as np
from jina import Document

d1 = Document(embedding=np.array([1, 2, 3]))
d2 = Document(embedding=np.array([[1, 2, 3], [4, 5, 6]]))
```



#### 支持稀疏数组

Scipy 稀疏数组 ( `coo_matrix, bsr_matrix, csr_matrix, csc_matrix`) 支持两者`embedding`或`blob`：

```python
import scipy.sparse as sp 

d1 = Document(embedding=sp.coo_matrix([0,0,0,1,0]))
d2 = Document(embedding=sp.csr_matrix([0,0,0,1,0]))
d3 = Document(embedding=sp.bsr_matrix([0,0,0,1,0]))
d4 = Document(embedding=sp.csc_matrix([0,0,0,1,0]))

d5 = Document(blob=sp.coo_matrix([0,0,0,1,0]))
d6 = Document(blob=sp.csr_matrix([0,0,0,1,0]))
d7 = Document(blob=sp.bsr_matrix([0,0,0,1,0]))
d8 = Document(blob=sp.csc_matrix([0,0,0,1,0]))
```

还支持 Tensorflow 和 Pytorch 稀疏数组

```python
import torch
import tensorflow as tf

indices = [[0, 0], [1, 2]]
values = [1, 2]
dense_shape = [3, 4]

d1 = Document(embedding=torch.sparse_coo_tensor(indices, values, dense_shape))
d2 = Document(embedding=tf.SparseTensor(indices, values, dense_shape))
d3 = Document(blob=torch.sparse_coo_tensor(indices, values, dense_shape))
d4 = Document(blob=tf.SparseTensor(indices, values, dense_shape))
```



#### 多属性构造

##### 元属性

| `doc.tags`      | 结构化数据值，由映射到动态类型值的字段组成                   |
| --------------- | ------------------------------------------------------------ |
| `doc.id`        | 表示唯一文档 ID 的十六进制摘要                               |
| `doc.weight`    | 文件重量                                                     |
| `doc.mime_type` | 文档的 MIME 类型                                             |
| `doc.location`  | 文档的位置。这可能是字符串的开始和结束索引；图像裁剪的 x,y（上、左）坐标；音频剪辑的时间戳等 |
| `doc.offset`    | Document在上一个粒度Document中的偏移量                       |
| `doc.modality`  | 文档所属模式的标识符                                         |

您可以通过以下方式在构造函数中分配多个属性：

```python
from jina import Document

d = Document(uri='https://jina.ai',
             mime_type='text/plain',
             granularity=1,
             adjacency=3,
             tags={'foo': 'bar'})
```

```shell
<jina.types.document.Document id=e01a53bc-aedb-11eb-88e6-1e008a366d48 uri=https://jina.ai mimeType=text/plain tags={'foo': 'bar'} granularity=1 adjacency=3 at 6317309200>
```



#### 从Dict或JSON字符串构造

您可以`Document`从 a`dict`或 JSON 字符串构建一个：

```python
from jina import Document
import json

d = {'id': 'hello123', 'content': 'world'}
d1 = Document(d)

d = json.dumps({'id': 'hello123', 'content': 'world'})
d2 = Document(d)
```

##### 解析无法识别的字段

`dict`/JSON 字符串中无法识别的字段会自动放入文档的`.tags`字段中：

```python
from jina import Document

d1 = Document({'id': 'hello123', 'foo': 'bar'})
```

```shell
<jina.types.document.Document id=hello123 tags={'foo': 'bar'} at 6320791056>
```

您可以使用`field_resolver`将外部字段名称映射到`Document`属性：

```python
from jina import Document

d1 = Document({'id': 'hello123', 'foo': 'bar'}, field_resolver={'foo': 'content'})
```

```	shell
<jina.types.document.Document id=hello123 mimeType=text/plain text=bar at 6246985488>
```



#### 从另一个构建 `Document`

将一个`Document`对象分配给另一个`Document`对象将进行浅拷贝：

```python
from jina import Document

d = Document(content='hello, world!')
d1 = d

assert id(d) == id(d1)  # True
```

要进行深层复制，请使用`copy=True`：

```python
d1 = Document(d, copy=True)

assert id(d) == id(d1)  # False
```

您可以`Document`根据另一个来源部分更新 a `Document`：

```python
from jina import Document

s = Document(
    id='🐲',
    content='hello-world',
    tags={'a': 'b'},
    chunks=[Document(id='🐢')],
)
d = Document(
    id='🐦',
    content='goodbye-world',
    tags={'c': 'd'},
    chunks=[Document(id='🐯')],
)

# only update `id` field
d.update(s, fields=['id'])

# update all fields. `tags` field as `dict` will be merged.
d.update(s)
```

