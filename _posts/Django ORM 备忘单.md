# Django ORM 备忘单



### 一些常用的orm功能

首先定义一下数据库模型

```python
from datetime import date

from django.db import models


class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name


class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name


class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField(default=date.today)
    authors = models.ManyToManyField(Author)
    number_of_comments = models.IntegerField(default=0)
    number_of_pingbacks = models.IntegerField(default=0)
    rating = models.IntegerField(default=5)

    def __str__(self):
        return self.headline
```

### 开始使用

```django
Entry.objects.all()

# SELECT "blog_entry"."id", "blog_entry"."blog_id", ...
# FROM "blog_entry" ...;

# [<Entry: Off foot official kitchen another turn.>,
#  <Entry: Yet year marriage yes her she.>,
#  <Entry: Special mission in who son sort.>,
#  ...
# ]
```

### 如何减少查询

#### select_related()

在 一对一，一对多 关系中使用

+ 在没有使用 `select_related`情况

  ```python
  entry = Entry.objects.first()
  # SELECT ... FROM "blog_entry" ...;
  
  # 此属性访问将运行第二个查询
  blog = entry.blog
  # SELECT ... FROM "blog_blog" WHERE ...;
  ```

+ 使用 `select_related`

  ```python
  entry = Entry.objects.select_related("blog").first()
  # SELECT "blog_entry"."id", ... "blog_blog"."id", ...
  # FROM "blog_entry"
  # INNER JOIN "blog_blog" ...;
  
  blog = entry.blog
  # 这样就没有查询运行，因为我们连接了上面的博客表 Blog 模型
  ```

#### prefetch_related()

在 一对多，多对多 关系中使用

+ 不使用 `prefetch_related`

  ```python
  entry = Entry.objects.first()
  # SELECT ... FROM "blog_entry" ...;
  
  # 这个相关查询再次访问数据库
  authors = list(entry.authors.all())
  # SELECT ...
  # FROM "blog_author" INNER JOIN "blog_entry_authors"...
  # WHERE "blog_entry_authors"."entry_id" = 4137;
  ```

+ 使用 `prefetch_related`

  ```python
  entry = Entry.objects.prefetch_related("authors").first()
  # SELECT ... FROM "blog_entry" ...;
  # SELECT ...
  # FROM "blog_author" INNER JOIN "blog_entry_authors" ...
  # WHERE "blog_entry_authors"."entry_id" IN (4137);
  
  authors = list(entry.authors.all())
  # 没有查询运行，因为我们在内存中有一个
  # 从上面查找相关作者
  ```

#### update()

+ 不使用 `update`情况

  ```python
  david = (
      Author.objects.filter(name__startswith="David")
      .prefetch_related("entry_set")
      .first()
  )
  # SELECT ... FROM "blog_author" WHERE "blog_author"."name" LIKE 'David%' ...;
  # SELECT ... FROM "blog_entry" INNER JOIN "blog_entry_authors" ...;
  # 使用for循环更新该作者的博客分数
  for entry in david.entry_set.all():
      entry.rating = 5
      entry.save()
  ```

+ 使用 `update`

  ```python
  Entry.objects.filter(authors__name__startswith="David").update(rating=5)
  # UPDATE "blog_entry" SET "rating" = 5 WHERE "blog_entry"."id" IN ...;
  ```

  

### 查询数据列以外的东西，在数据库中运行，不使用python

#### annotate()

`annotate()`向结果的每一行添加一个属性。与F()一起使用，可以从相关对象中添加一个字段

`annotate()`非常强大，在很多情况下适用，例如下面这个假想情况

```python
entries = Entry.objects.annotate(blog_name=F("blog__name"))[:5]
# SELECT "blog_entry"."id", ... "blog_blog"."name" AS "blog_name"
# FROM "blog_entry" INNER JOIN "blog_blog" ...;

# 现在我们有了带有一个额外属性的Entry对象: blog_name
[entry.blog_name for entry in entries]
# ['Hunter-Rhodes',
#  'Mcneil PLC',
#  'Banks, Hicks and Carpenter',
#  'Anderson PLC',
#  'George-Bray']
```

#### filter() 和 Q()

使用 `Q()` 在数据库中更多的过滤，尽量少在python中做

```python
low_engagement_posts = Entry.objects.filter(
    Q(number_of_comments__lt=20) | Q(number_of_pingbacks__lt=20)
)

list(low_engagement_posts)
# SELECT ... FROM "blog_entry"
# WHERE
#   ("blog_entry"."number_of_comments" < 20 OR
#   "blog_entry"."number_of_pingbacks" < 20);
```

 #### Query 表达式

Django ORM的很多部分都需要查询表达式。查询表达式包括

+ 对字段的引用(可能在相关对象上) `F()`
+ SQL 函数 `CASE`, `NOW`等
+ `Subquery()` 子查询

#### F()

经典的例子是对字段进行增量加减，但是请记住，F()可以在任何需要查询表达式的地方使用

```python
Entry.objects.filter(authors__name__startswith="David").first().rating
# 5

Entry.objects.filter(authors__name__startswith="David").update(
    rating=F("rating") - 1
)
# UPDATE "blog_entry"
#   SET "rating" = ("blog_entry"."rating" - 1) WHERE ...;

Entry.objects.filter(authors__name__startswith="David").first().rating
# 4
```

#### CASE 和 条件逻辑

`Value()`是一个String、integer、bool等字面值的查询表达式, 可以使用`F()`或其他表达式作为`then`值，甚至代替`rating=`xxx

```python
entries = Entry.objects.annotate(
    coolness=Case(
        When(rating=5, then=Value("super cool")),
        When(rating=4, then=Value("pretty cool")),
        default=Value("not cool"),
    )
)
# SELECT ...
# CASE
#   WHEN "blog_entry"."rating" = 5 THEN 'super cool'
#   WHEN "blog_entry"."rating" = 4 THEN 'pretty cool'
#   ELSE 'not cool'
# END AS "coolness"
# FROM "blog_entry" LIMIT 5;

[f"Entry {e.pk} is {e.coolness}" for e in entries[:5]]
# ['Entry 4137 is super cool',
#  'Entry 4138 is not cool',
#  'Entry 4139 is not cool',
#  'Entry 4140 is pretty cool',
#  'Entry 4141 is not cool']
```

#### Subquery() 和 OuterRef()

用最新条目的标题标注每个博客

我发现 `Subquery()`的唯一用途是：查询一个一对多或者多对多关系，`OuterRef("pk")`来连接行，使用`values()`返回一个列，使用`[:1]`返回子查询中的一行，下面是django文档中的一个示例

```python
blogs = Blog.objects.annotate(
    most_recent_headline=Subquery(
        Entry.objects.filter(blog=OuterRef("pk"))
        .order_by("-pub_date")
        .values("headline")[:1]
    )
)
[(blog, str(blog.most_recent_headline)) for blog in blogs[:5]]
# SELECT
#   "blog_blog"."id",
#   "blog_blog"."name",
#   "blog_blog"."tagline",
#   (
#     SELECT
#       U0."headline"
#     FROM
#       "blog_entry" U0
#     WHERE
#       U0."blog_id" = ("blog_blog"."id")
#     ORDER BY
#       U0."pub_date" DESC
#     LIMIT
#       1
#   ) AS "most_recent_headline"
# FROM
#   "blog_blog"
# LIMIT
#   5;


# [(<Blog: Robinson-Wilson>, 'Three space maintain subject much.'),
#  (<Blog: Anderson PLC>, 'Rock authority enjoy hundred reduce behavior.'),
#  (<Blog: Mcneil PLC>, 'Visit beyond base.'),
#  (<Blog: Smith, Baker and Rodriguez>, 'Tree look culture minute affect.'),
#  (<Blog: George-Bray>, 'Then produce tree quality top similar.')]
```

### Select 更少的数据

#### values() --part 1

只选择指定的字段，如字典。您也可以像这样选择注释

```python
Entry.objects.annotate(num_authors=Count("authors")).values(
    "rating", "num_authors"
)[:5]
# SELECT
#   "blog_entry"."rating",
#   COUNT("blog_entry_authors"."author_id") AS "num_authors"
# FROM "blog_entry" LEFT OUTER JOIN "blog_entry_authors" ...;

# [{'num_authors': 3, 'rating': 1},
#  {'num_authors': 4, 'rating': 4},
#  {'num_authors': 4, 'rating': 2},
#  {'num_authors': 3, 'rating': 2},
#  {'num_authors': 3, 'rating': 5}]
```

#### exists()

如果您需要表中是否存在，那么`exists` 比获取整个行要快

```python
unrealistic_data_exists = Entry.objects.filter(
    mod_date__lt=F("pub_date")
).exists()
# SELECT
#  (1) AS "a"
#   FROM "blog_entry"
#   WHERE "blog_entry"."mod_date" < ("blog_entry"."pub_date")
#   LIMIT 1;

unrealistic_data_exists
# True
```

#### only()

类似于values()，但返回的是模型实例，而不是字典。但是要注意:如果你访问任何你最初没有获取的字段，你会做一个额外的DB查询。

### 数据分组

#### values() -- part2 和 GROUP BY

django文档

> 通常，注释是在每个对象的基础上生成的——一个带注释的QuerySet将为原始QuerySet中的每个对象返回一个结果。然而，当values()子句用于约束结果集中返回的列时……原始结果根据values()子句中指定的字段的唯一组合进行分组。然后为每个惟一的组提供注释;对组的所有成员计算注释。

例子：哪一天的博客点击率最高

```python
[
    (str(e["pub_date"]), e["avg_rating"])
    for e in Entry.objects.values("pub_date").annotate(avg_rating=Avg("rating"))
]
# SELECT
#   "blog_entry"."pub_date",
#   AVG("blog_entry"."rating") AS "avg_rating"
# FROM "blog_entry"
# GROUP BY "blog_entry"."pub_date";

# [
#  ('2022-04-07', 2.235294117647059),
#  ('2022-04-08', 2.8157894736842106),
#  ('2022-04-09', 2.0285714285714285),
#  ('2022-04-10', 2.96875),
#  ('2022-04-11', 2.3636363636363638),
#  ('2022-04-12', 2.725),
#  ('2022-04-13', 2.5),
#  ('2022-04-14', 2.761904761904762),
#  ...
# ]
```

