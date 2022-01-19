# 苹果CMS资源站采集API接口参数

### 列表页

ac=list

t=类别ID

pg=页码

wd=搜索关键字

h=几小时内的数据

例如： api.php?ac=list&t=1&pg=5  分类ID为1的列表数据第5页



### 详情页

ac=videolist 采集数据

ids=数据ID，多个ID都好分割

t=类型ID

pg=页码

h=几小时内的数据

例如:  api.php?ac=videolist&ids=123,567   **ID为123和567的数据信息

​    api.php?ac=videolist&h=24   **24小时内更新数据信息