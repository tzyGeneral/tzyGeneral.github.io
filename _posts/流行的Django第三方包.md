# 流行的Django第三方包



### django-allauth - 用户注册登陆管理

django-allauth 是一个能够解决你的注册和认证需求的、可重用的 Django 应用。无论你需要构建本地注册系统还是社交账户注册系统，django-allauth 都能够帮你做到。



这个应用支持多种认证体系，比如用户名或电子邮件。一旦用户注册成功，它还可以提供从无需认证到电子邮件认证的多种账户验证的策略。同时，它也支持多种社交账户和电子邮件账户。它还支持插拔式注册表单，可让用户在注册时回答一些附加问题。



django-allauth 支持多于 20 种认证提供者，包括 Facebook、Google、微博 和 微信。如果你发现了一个它不支持的社交网站，很有可能通过第三方插件提供该网站的接入支持。这个项目还支持自定义后端，可以支持自定义的认证方式，对每个有定制认证需求的人来说这都很棒。



django-allauth 易于配置，且有完善的文档。该项目通过了很多测试，所以你可以相信它的所有部件都会正常运作。



### django-ckeditor - 富文本编辑器

django没有提供官方的富文本编辑器，而ckeditor恰好是内容型网站后台管理中不可或缺的控件。ckeditor是一款基于javascript，使用非常广泛的开源网页编辑器。它允许用户直接编写图文，插入列表和表格，并支持文本和HTML格式代码输入。



### django-imagekit - 自动化处理图像

现代网站开发一般免不了处理一些图片，例如头像、用户上传的图片等内容。django-imagekit 帮你配合 django 的 model 模块自动完成图片的裁剪、压缩、生成缩略图、加水印等一系列图片相关的操作。



### django-crispy-forms - 快速美化django表单首选

大大增强 Django 内置的表单功能，Django 内置的表单生成原生的 HTML 表单代码还可以，但为其设置样式是一个麻烦的事情。django-crispy-forms 帮助你使用一行代码渲染一个 Bootstrap 样式的表单，当然它还支持其它一些热门的 CSS 框架样式的渲染。



### django-constance - 常量管理



有时我们会在 django 的 settings 中设置一些常量，但是有可能会进行变更。利用这个包，只需简单的配置就可以自动生成 admin 管理后台可以修改管理常量。



### Django-filter

Django-filter允许用户根据模型字段进一步过滤从数据库查得到的queryset，从而筛选出用户想要的查询结果，这样避免了对数据库的再次查询，大大提升了效率。比如用户访问文章列表页面，已经看到了文章清单，但用户还希望通过标题关键词进一步在查询结果中筛选出自己感兴趣的文章清单(而不必重新查询数据库)，这时使用Django-filter就可以轻松帮你实现上述需求。下期文章我们将详细介绍如何使用和美化django-filter，欢迎关注。



### cookiecutter-django



使用cookiecutter-django创建一个新项目比使用django-admin.py的命令创建新项目有更多优势，比如它会给你提供一个更加合适的项目目录结构，并允许你在创建项目之初就选择需要安装配置的库比如(compressor, celery), 你只要选择yes或no就行了。小编我喜欢自行配置项目，所以把它放在了第二位。

它的主要特色包括：

- 默认支持SSL
- 优化开发和生产配置
- 默认集成django-allauth
- 默认使用自定义的User model
- 支持使用Anymail发送邮件
- 支持配置使用S3或者Google Storage Cloud做云存储
- 支持Docker，支持docker-compose
- 默认使用postgresql
- 支持celery



### **django-grappelli**

大家都知道django自带admin功能无比强大，学会[django admin的定制与使用](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMjM5OTMyODA4Nw%3D%3D%26mid%3D2247484126%26idx%3D1%26sn%3Ddb9e171b03c52aa6bad941ce84caf39c%26chksm%3Da73c62e6904bebf0aac2fd0f994bd5fc19b563a3e9dc1b9774b5b8bd2dd36d655aebda11a496%26scene%3D21%23wechat_redirect)后就根本不需要开发自己的后台了，然而其界面却是非常简陋的。虽然xadmin很美观，但安装使用复杂。只要你不是对后台界面的外观太过挑剔，使用django-grappeli可以迅速给你一个看上去专业的管理后台。



### **django-guardian**

在我们介绍[Django权限管理](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMjM5OTMyODA4Nw%3D%3D%26mid%3D2247484184%26idx%3D1%26sn%3D1e4845e1784db7d106565e6838f542aa%26chksm%3Da73c6320904bea36ffd83eaccfab028cc5e81fca8cd74aa1637f74ce084d01c1e4ec506fef5b%26scene%3D21%23wechat_redirect)时，我们提到过Django没有提供对象（Object）级别的权限控制。一个用户如果对Article模型有修改的权限，那么他将对所有Article对象有修改权限。使用django-guardian可以帮助我们实现对对象级别（比如某篇具体文章）的权限控制。



### **django-taggit**

当你在django项目中需要对任何对象（文章，用户）打上一个标签（tags）时，一个最好的方式就是使用django-taggit。你可以像使用django模型自带的一个字段一样使用它快速创立多对多的关系，而且可以对你的标签进行集中管理。



### **django-ompressor**

使用django-compressor可将页面中链接的以及直接编写的JavaScript和CSS打包到一个单一的缓存文件中，以减少页面对服务器的请求数，加快页面的加载速度。当您的用户很少时，你可能不用关注压缩文件提升性能这些小事，但当你用户非常多时，你就会发现任何能够改善性能的事都是值得做的。



### **django-activity-stream**

如果你要开发一个社交类网站，需要实现用户关注、收藏、点赞、用户动态等功能时，这个第三方库可以快速帮你实现并提供用户的活动流。出于学习目的话，该项目的源码也是一个很好的学习对象。

