---
title: '[数据工具]数据清理神器 - dplyr'
author: baoliu
date: '2021-02-28'
slug: []
categories:
  - R
tags:
  - dplyr
df_print: paged
output: 
  df_print: paged
  html_document:
    toc: true
    toc_depth: 4
    toc_float: 
      collapsed: false
      smooth_scroll: false
      
    df_print: paged
    theme: cerulean
---



  *“There are definitely some academic statisticians who just don’t understand why what I do is statistics, but basically I think they are all wrong . What I do is fundamentally statistics. The fact that data science exists as a field is a colossal failure of statistics. To me, that is what statistics is all about. It is gaining insight from data using modelling and visualization. Data munging and manipulation is hard and statistics has just said that’s not our domain.” _ Hadley Wickham*

  数据的清理是数据统计分析的第一步，通常也是最耗时费力的一步，这通常给多数没有接触过编程的学者带来很多麻烦。
  对于没有编程基础的人而言，为了解决一个简单的问题去学习base R的数据处理或者pandas等会耗费很多时间，投入产出不划算。  
  **dplyr**的设计就是为了解决这个问题，它通过select、filter、 arrange、mutate、summarise、group_by等容易理解和记忆的函数，极大的简化了数据清理的过程。
  
<!-- | 函数名称            | 主要作用              | 备注   | -->
<!-- | :----:              | :----:                | :----: | -->
<!-- |   select            |  选择指定的列         |        | -->
<!-- |   filter            |  选择指定的行         |        | -->
<!-- |   arrange           |  排序                 |        | -->
<!-- |   mutate            |  修改、新增列         |        | -->
<!-- |   summarise         |  汇总计算             |        | -->
<!-- |   group_by          |  分组                 |        | -->

  
  下面为大家一一介绍。
  为了展示方便我引入了knitr包，主要用kable() 方便数据展示。

**安装和加载**  
运行以下程序加载包，如果没有安装会执行安装程序。


```r
if(!"dplyr" %in% installed.packages()){
    install.packages("dplyr")
  }

library(dplyr)

# 同时加载knitr， 方便数据展示
library(knitr)
```


**构建一个简单数据集**


```r
# 一个简单的学生成绩表
scores <-  data.frame(
  name = c("zhangsan", "lisi", "wangwu"),
  gender = c("F", "M", "F"),
  math = c(90, 87, 92)
)

kable(scores)
```



|name     |gender | math|
|:--------|:------|----:|
|zhangsan |F      |   90|
|lisi     |M      |   87|
|wangwu   |F      |   92|

## 基本用法

**新建一列 mutate**  

```r
# 新建一列英文成绩
scores <- mutate(scores, eng = c(77, 93, 81))
# 新建一列均分
scores <- mutate(scores, mean_score = (math + eng) / 2)


# 使用管道符的写法
scores <- scores %>% 
  mutate(eng = c(77, 93, 81)) %>% 
  mutate(mean_score = (math + eng) / 2)

# 同时生成多列
scores <- scores %>% 
  mutate(eng = c(77, 93, 81),
         mean_score = (math + eng) / 2)

kable(scores)
```



|name     |gender | math| eng| mean_score|
|:--------|:------|----:|---:|----------:|
|zhangsan |F      |   90|  77|       83.5|
|lisi     |M      |   87|  93|       90.0|
|wangwu   |F      |   92|  81|       86.5|

**管道符**  
dplyr提供了非常方便的管道符, 最常用的是 %>% ，它的作用是把管道符前的内容，当作变量传递给后面的方法。这个管道符可以通过快捷键 “Ctrl + Shift + M”输入(赋值符号 <- 可以通过快捷键 “Alt + -”输入)。  
在上面的例子中，管道符把scores传递给后面的mutate()并当作第一个参数。  
这种方法在需要进行快速复杂处理的时候非常有用，后面的例子我会尽量使用两种方式展示。


**排序 arrange**  
通过arrange()实现按照某一列或几列排序，默认升序。可以通过desc()实现降序排列


```r
# 把成绩数据按照数学成绩升序排序
scores <- arrange(scores, math)
kable(scores)
```



|name     |gender | math| eng| mean_score|
|:--------|:------|----:|---:|----------:|
|lisi     |M      |   87|  93|       90.0|
|zhangsan |F      |   90|  77|       83.5|
|wangwu   |F      |   92|  81|       86.5|

```r
# 把成绩数据按照数学、英语成绩升序排序
scores <- arrange(scores, math, eng)
kable(scores)
```



|name     |gender | math| eng| mean_score|
|:--------|:------|----:|---:|----------:|
|lisi     |M      |   87|  93|       90.0|
|zhangsan |F      |   90|  77|       83.5|
|wangwu   |F      |   92|  81|       86.5|

```r
# 把成绩数据按照数学成绩降序排序
scores <- arrange(scores, desc(math))
kable(scores)
```



|name     |gender | math| eng| mean_score|
|:--------|:------|----:|---:|----------:|
|wangwu   |F      |   92|  81|       86.5|
|zhangsan |F      |   90|  77|       83.5|
|lisi     |M      |   87|  93|       90.0|

```r
# 使用管道符书写
scores <- scores %>% arrange(math)
scores <- scores %>% arrange(desc(math))
```


**选择列 select**  
可以通过减号，实反向选择。  
选择的时候列会按照指定顺序重新排序


```r
# 删除均分mean_score列
## 减号待变从原数据删除某列
new_data <- select(scores, -mean_score)
## 若不使用减号，代表仅取出指定的列
new_data <- select(scores, name, gender, math)
kable(new_data)
```



|name     |gender | math|
|:--------|:------|----:|
|wangwu   |F      |   92|
|zhangsan |F      |   90|
|lisi     |M      |   87|

```r
# 管道符
new_data <- scores %>% select(-mean_score)
new_data <- scores %>% select( gender, math, name)
```


**选择符合条件的行**  


```r
# 仅筛选出女生
new_data <- filter(scores, gender == 'F')
kable(new_data)
```



|name     |gender | math| eng| mean_score|
|:--------|:------|----:|---:|----------:|
|wangwu   |F      |   92|  81|       86.5|
|zhangsan |F      |   90|  77|       83.5|

```r
# 筛选出女生及英语在80分以上的
new_data <- filter(scores, gender == 'F', eng > 80)
kable(new_data)
```



|name   |gender | math| eng| mean_score|
|:------|:------|----:|---:|----------:|
|wangwu |F      |   92|  81|       86.5|

```r
## 管道符写法
new_data <- scores %>% filter(gender == 'F', eng > 90)
```


**汇总计算**  


```r
# 计算各科均分
new_data <- scores %>% 
  summarise(
    math_mean = mean(math),
    eng_mean = mean(eng)
  ) %>% 
  ungroup()

kable(new_data)
```



| math_mean| eng_mean|
|---------:|--------:|
|  89.66667| 83.66667|


**分组 group_by**  
记得group_by()之后最好跟上一个ungroup()否则如果直接拿去做别的处理，有可能出现很多意想不到的问题。


```r
# 计算不同性别各科均分
new_data <- scores %>% 
  group_by(gender) %>% 
  summarise(
    math_mean = mean(math),
    eng_mean = mean(eng)
  ) %>% 
  ungroup()
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
kable(new_data)
```



|gender | math_mean| eng_mean|
|:------|---------:|--------:|
|F      |        91|       79|
|M      |        87|       93|



## 其他常用方法

**表连接**


```r
# 新建一个表储存学生身高和历史课成绩表，注意这里的学生名字和之前的scores有些差异，是为了对比后面不同的表连接方式
scores_2 = data.frame(
  name = c("zhangsan", "lisi", "zhengliu"),
  height = c(178, 162, 173),
  history = c(79, 73, 67)
)

kable(scores_2)
```



|name     | height| history|
|:--------|------:|-------:|
|zhangsan |    178|      79|
|lisi     |    162|      73|
|zhengliu |    173|      67|


```r
# 把两个表合并到一起
## 左连接
new_data <- scores %>% 
  left_join(scores_2, by = "name")

kable(new_data)
```



|name     |gender | math| eng| mean_score| height| history|
|:--------|:------|----:|---:|----------:|------:|-------:|
|wangwu   |F      |   92|  81|       86.5|     NA|      NA|
|zhangsan |F      |   90|  77|       83.5|    178|      79|
|lisi     |M      |   87|  93|       90.0|    162|      73|

```r
## 内连接
new_data <- scores %>% 
  inner_join(scores_2, by = "name")

kable(new_data)
```



|name     |gender | math| eng| mean_score| height| history|
|:--------|:------|----:|---:|----------:|------:|-------:|
|zhangsan |F      |   90|  77|       83.5|    178|      79|
|lisi     |M      |   87|  93|       90.0|    162|      73|

```r
## 全连接
new_data <- scores %>% 
  full_join(scores_2, by = "name")

kable(new_data)
```



|name     |gender | math| eng| mean_score| height| history|
|:--------|:------|----:|---:|----------:|------:|-------:|
|wangwu   |F      |   92|  81|       86.5|     NA|      NA|
|zhangsan |F      |   90|  77|       83.5|    178|      79|
|lisi     |M      |   87|  93|       90.0|    162|      73|
|zhengliu |NA     |   NA|  NA|         NA|    173|      67|


**纵向拼接**  a


```r
# 一个简单的学生成绩表
scores_3 <-  data.frame(
  name = c("zhaoliu"),
  gender = c("M"),
  math = c(90),
  eng = c(87)
)

kable(scores_3)
```



|name    |gender | math| eng|
|:-------|:------|----:|---:|
|zhaoliu |M      |   90|  87|



```r
# 拼接到scores

new_data <- scores %>% 
  bind_rows(
    scores_3
  )

kable(new_data)
```



|name     |gender | math| eng| mean_score|
|:--------|:------|----:|---:|----------:|
|wangwu   |F      |   92|  81|       86.5|
|zhangsan |F      |   90|  77|       83.5|
|lisi     |M      |   87|  93|       90.0|
|zhaoliu  |M      |   90|  87|         NA|



**列重命名**


```r
# 使用select重命名
new_data <- scores %>% 
  select(姓名 = name, 性别 = gender)
kable(new_data)
```



|姓名     |性别 |
|:--------|:----|
|wangwu   |F    |
|zhangsan |F    |
|lisi     |M    |

```r
# 使用mutate命名
new_data <- scores %>% 
  mutate(姓名 = name, 性别 = gender) %>% 
  select(-name, -gender)
kable(new_data)
```



| math| eng| mean_score|姓名     |性别 |
|----:|---:|----------:|:--------|:----|
|   92|  81|       86.5|wangwu   |F    |
|   90|  77|       83.5|zhangsan |F    |
|   87|  93|       90.0|lisi     |M    |

```r
# 使用rename命名
name_list = c(
  "姓名" = "name",
  "性别" = "gender",
  "数学" = "math",
  "英语" = "eng",
  "平均分" = "mean_score"
)

new_data <- scores %>% 
  rename(name_list)
```

```
## Note: Using an external vector in selections is ambiguous.
## i Use `all_of(name_list)` instead of `name_list` to silence this message.
## i See <https://tidyselect.r-lib.org/reference/faq-external-vector.html>.
## This message is displayed once per session.
```

```r
kable(new_data)
```



|姓名     |性别 | 数学| 英语| 平均分|
|:--------|:----|----:|----:|------:|
|wangwu   |F    |   92|   81|   86.5|
|zhangsan |F    |   90|   77|   83.5|
|lisi     |M    |   87|   93|   90.0|

```r
# 构建一个常用名字对照，使用rename重命名
# 区别在于这里的history在scores中并不存在,但是通过筛选去除，不会报错；如果要对另外一个表进行操作可以直接用，也不会报错。好处在于只用维护一个名字对照表
name_list = c(
  "姓名" = "name",
  "性别" = "gender",
  "身高" = "height",
  "数学" = "math",
  "英语" = "eng",
  "历史" = "history",
  "平均分" = "mean_score"
)

new_data <- scores %>% 
  rename(name_list[name_list %in% colnames(scores)])
kable(new_data)
```



|姓名     |性别 | 数学| 英语| 平均分|
|:--------|:----|----:|----:|------:|
|wangwu   |F    |   92|   81|   86.5|
|zhangsan |F    |   90|   77|   83.5|
|lisi     |M    |   87|   93|   90.0|





#### —————————————————— 
**作者：刘保**  
**邮箱：roycufe@gmail.com**  
**手机/微信：13051897793**  


