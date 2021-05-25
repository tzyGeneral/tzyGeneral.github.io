---
layout:     post   				    # 使用的布局（不需要改）
title:      Selenium单击Element，出现点击无法生效问题 				# 标题
subtitle:   ElementClickInterceptedException #副标题
date:       2021-05-25 				# 时间
author:     Tzy 						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
- 爬虫
---

## 使用selenium进行click点击事件时经常出现无法点击问题（ElementClickInterceptedException）



### 方法一

```python
element = driver.find_element_by_css(``'div[class*="loadingWhiteBox"]'``)
driver.execute_script("arguments[0].click();", element)

```



## 方法二



```python
element = driver.find_element_by_css('div[class*="loadingWhiteBox"]')

webdriver.ActionChains(driver).move_to_element(element ).click(element ).perform()
```

