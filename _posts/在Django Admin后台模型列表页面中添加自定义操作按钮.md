## 在Django Admin后台模型列表页面中添加自定义操作按钮



在Django Admin页面中有`add Models`这样的一个按钮，假如有需求，可以新增按钮，一键更改所有模型的某个字段，或者一键删除所有的数据



首先，我们要更改Admin管理模型使用的模板文件，以便我们可以添加两个按钮

```python
@admin.register(UserModel)
class UserModelAdmin(admin.ModelAdmin):
  change_list_template = "user_changelist.html"
```



然后，我们需要覆盖`get_urls`方法，并在管理模型上添加`set_immortal`和`set_motal`方法。它们将作用于两种view视图



```python
def get_urls(self):
  urls = super().get_urls()
  my_urls = [
    path('immortal/', self.set_immortal),
    path('mortal/', self.set_mortal)
  ]

def set_immortal(self. request):
  self.model.objects.all().update(is_immortal=True)
  self.message_user(request, "All heroes are now immortal")
  return HttpResponseRedirect("../")

def set_mortal(self, request):
    self.model.objects.all().update(is_immortal=False)
    self.message_user(request, "All heroes are now mortal")
    return HttpResponseRedirect("../")
```



最后，我们通过拓展admin/change_list.html来创建模板文件`user_changelist.html`

```html
{% extends "admin/change_list.html" %}

{% block object-tools-items %}
  <li>
    <a href="immortal/" class="addlink">
     上传Cookie
    </a>
  </li>
  <li>
    <a href="mortal/" class="addlink">
     上传Cookie2
    </a>
  </li>
  {{ block.super }}
{% endblock %}

```



