# Django order by 高级用法



一个业务需求，推荐的网站安装一定的网址id来排序，剩下的按照默认id排序

### 准备

+ 定义model

  ```python
  class WebSetUrl(model.Model):
    sid = model.IntegerField()  # 网站id
    url = model.CharField(max_length=255)
  ```



+ 生成测试数据

  ```python
  from model import WebSetUrl
  
  data_list = []
  for num in range(1000):
    url = 'url:{}'.format(num)
    data_list.append(WebSetUrl(sid=num, url=url))
  
  WebSetUrl.objects.bulk_create(data_list)
  ```

  

### 需求

+ 现在有这样一个排序列表 `sid_list = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 1000]`
+ 现在要让 WebSetUrl 的 queryset 先按照 sid_list 中的 id 位置进行排序， 不在 sid_list 的按照自增id排序



### 分析

+ 普通的 order_by('sid') 肯定无法解决这个需求
+ 把 query_set 转换为列表花销太大

### 解决

+ ```python
  from django.db.models import Case, When
  
  sid_list = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 1000]
  order_rule = Case(*[When(sid=id, then=pos) for pos, id in enumerate(sid_list)], default=len(sid_list))
  
  qs = WebSetUrl.objects.all().order_by(order_rule, 'sid')
  ```

  