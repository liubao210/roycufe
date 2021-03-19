---
title: '[数据工具]数据整形神器 - tidyr'
author: baoliu
date: '2021-03-19'
slug: []
categories:
  - R
tags:
  - tidyverse
  - tidyr
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


# **tidyr**

**Tidyr**(https://tidyr.tidyverse.org/) 是数据处理经常使用的package,主要用数据形状的改变，处理缺失值等，一般情况作为dplyr的上游处理步骤。  

**主要功能**：  
- **长宽数据转换** (pivot_longer pivot_wider)  
- **数据分割与合并** (separate unite)  
- **缺失数据处理** (drop_na  fill)  
- **嵌套数据处理** (nest unnest unnest_wider hoist)  


## 长宽数据转换


```r
library(dplyr)
library(tibble)
library(tidyr)
library(knitr)

mtcars = mtcars %>% rownames_to_column('name')

print_df = function(df){
  kable(df)
}

print_df(head(mtcars))
```



|name              |  mpg| cyl| disp|  hp| drat|    wt|  qsec| vs| am| gear| carb|
|:-----------------|----:|---:|----:|---:|----:|-----:|-----:|--:|--:|----:|----:|
|Mazda RX4         | 21.0|   6|  160| 110| 3.90| 2.620| 16.46|  0|  1|    4|    4|
|Mazda RX4 Wag     | 21.0|   6|  160| 110| 3.90| 2.875| 17.02|  0|  1|    4|    4|
|Datsun 710        | 22.8|   4|  108|  93| 3.85| 2.320| 18.61|  1|  1|    4|    1|
|Hornet 4 Drive    | 21.4|   6|  258| 110| 3.08| 3.215| 19.44|  1|  0|    3|    1|
|Hornet Sportabout | 18.7|   8|  360| 175| 3.15| 3.440| 17.02|  0|  0|    3|    2|
|Valiant           | 18.1|   6|  225| 105| 2.76| 3.460| 20.22|  1|  0|    3|    1|



### 宽变长 pivot_longer

这里相当于是把数据变窄变长了


```r
long_data = mtcars %>% 
  pivot_longer(
    -name,
    names_to = 'feature',
    values_to = 'value'
  )

print_df(head(long_data))
```



|name      |feature |  value|
|:---------|:-------|------:|
|Mazda RX4 |mpg     |  21.00|
|Mazda RX4 |cyl     |   6.00|
|Mazda RX4 |disp    | 160.00|
|Mazda RX4 |hp      | 110.00|
|Mazda RX4 |drat    |   3.90|
|Mazda RX4 |wt      |   2.62|

可以看到原来的属性名mpg、cyl等都被合并到了feature列，对应的值都被合并到了value列。而且信息是一样的，没有损失。  
所以pivot_longer可以理解为把数据变长，那变长会吧原来的列整合成新的列，所以需要指定新的列名和值的列名。  


### 长变宽 pivot_wider

进行与上相反的操作，把刚才变长的数据恢复原来的样子。


```r
origin_data = long_data %>% 
  pivot_wider(
    names_from = feature,
    values_from = value
  )

print_df(head(origin_data))
```



|name              |  mpg| cyl| disp|  hp| drat|    wt|  qsec| vs| am| gear| carb|
|:-----------------|----:|---:|----:|---:|----:|-----:|-----:|--:|--:|----:|----:|
|Mazda RX4         | 21.0|   6|  160| 110| 3.90| 2.620| 16.46|  0|  1|    4|    4|
|Mazda RX4 Wag     | 21.0|   6|  160| 110| 3.90| 2.875| 17.02|  0|  1|    4|    4|
|Datsun 710        | 22.8|   4|  108|  93| 3.85| 2.320| 18.61|  1|  1|    4|    1|
|Hornet 4 Drive    | 21.4|   6|  258| 110| 3.08| 3.215| 19.44|  1|  0|    3|    1|
|Hornet Sportabout | 18.7|   8|  360| 175| 3.15| 3.440| 17.02|  0|  0|    3|    2|
|Valiant           | 18.1|   6|  225| 105| 2.76| 3.460| 20.22|  1|  0|    3|    1|

这里变宽可以理解为把一列拆分成多列，因此需要传入参数新列名称的来源，新列值的来源。  
细心的同学应该还会看到，pivot_wider里传递变量的名字和pivot_longer不同，后者变量名加了引号，可以简单记忆成如果该列在原有数据中已经存在就不需引号，否则加引号。


### 更多长宽转换例子


```r
scores = data.frame(
  school = c("A", "B", "A" ,"B" ,"A", "B"),
  grade = c("grade_1","grade_2","grade_2","grade_2","grade_1","grade_1"),
  gender = c("F", "F", "M", "M", "M", "F"),
  eng = c(90, 98, 78, 73, 82, 94),
  math = c(99, 100, 87, 89, 93, 99),
  physical = c(76, 78, 82, 83, 68, 74)
)

print_df(scores)
```



|school |grade   |gender | eng| math| physical|
|:------|:-------|:------|---:|----:|--------:|
|A      |grade_1 |F      |  90|   99|       76|
|B      |grade_2 |F      |  98|  100|       78|
|A      |grade_2 |M      |  78|   87|       82|
|B      |grade_2 |M      |  73|   89|       83|
|A      |grade_1 |M      |  82|   93|       68|
|B      |grade_1 |F      |  94|   99|       74|

把学生成绩合并成一列


```r
long_data =  scores %>% 
    pivot_longer(
      -c(school, grade, gender),
      names_to = "subject",
      values_to = "score"
    )

print_df(long_data)
```



|school |grade   |gender |subject  | score|
|:------|:-------|:------|:--------|-----:|
|A      |grade_1 |F      |eng      |    90|
|A      |grade_1 |F      |math     |    99|
|A      |grade_1 |F      |physical |    76|
|B      |grade_2 |F      |eng      |    98|
|B      |grade_2 |F      |math     |   100|
|B      |grade_2 |F      |physical |    78|
|A      |grade_2 |M      |eng      |    78|
|A      |grade_2 |M      |math     |    87|
|A      |grade_2 |M      |physical |    82|
|B      |grade_2 |M      |eng      |    73|
|B      |grade_2 |M      |math     |    89|
|B      |grade_2 |M      |physical |    83|
|A      |grade_1 |M      |eng      |    82|
|A      |grade_1 |M      |math     |    93|
|A      |grade_1 |M      |physical |    68|
|B      |grade_1 |F      |eng      |    94|
|B      |grade_1 |F      |math     |    99|
|B      |grade_1 |F      |physical |    74|

```r
# 或者通过使用下面的方式实现同样的效果，和上面的区别在于前者是指定保留哪些列，这里是指定保留哪些列
# 指定的保持不变的列不会改变，只会在原有基础上进行(xing)行(hang)的复制扩充
# scores %>% 
#     pivot_longer(
#       c(eng, math, physical),
#       names_to = "subject",
#       values_to = "score"
#     )
```

把学校信息拆分成两列


```r
print_df(
  
  long_data %>% 
  pivot_wider(
    names_from = school,
    values_from = score
  )
  
)
```



|grade   |gender |subject  |  A|   B|
|:-------|:------|:--------|--:|---:|
|grade_1 |F      |eng      | 90|  94|
|grade_1 |F      |math     | 99|  99|
|grade_1 |F      |physical | 76|  74|
|grade_2 |F      |eng      | NA|  98|
|grade_2 |F      |math     | NA| 100|
|grade_2 |F      |physical | NA|  78|
|grade_2 |M      |eng      | 78|  73|
|grade_2 |M      |math     | 87|  89|
|grade_2 |M      |physical | 82|  83|
|grade_1 |M      |eng      | 82|  NA|
|grade_1 |M      |math     | 93|  NA|
|grade_1 |M      |physical | 68|  NA|


把年级信息拆分成两列


```r
print_df(
  
  long_data %>% 
  pivot_wider(
    names_from = grade,
    values_from = score
  )
  
)
```



|school |gender |subject  | grade_1| grade_2|
|:------|:------|:--------|-------:|-------:|
|A      |F      |eng      |      90|      NA|
|A      |F      |math     |      99|      NA|
|A      |F      |physical |      76|      NA|
|B      |F      |eng      |      94|      98|
|B      |F      |math     |      99|     100|
|B      |F      |physical |      74|      78|
|A      |M      |eng      |      82|      78|
|A      |M      |math     |      93|      87|
|A      |M      |physical |      68|      82|
|B      |M      |eng      |      NA|      73|
|B      |M      |math     |      NA|      89|
|B      |M      |physical |      NA|      83|

把学校和年级信息拆分成两列


```r
print_df(
  
  long_data %>% 
  pivot_wider(
    names_from = c(school, grade),
    values_from = score
  )
  
)
```



|gender |subject  | A_grade_1| B_grade_2| A_grade_2| B_grade_1|
|:------|:--------|---------:|---------:|---------:|---------:|
|F      |eng      |        90|        98|        NA|        94|
|F      |math     |        99|       100|        NA|        99|
|F      |physical |        76|        78|        NA|        74|
|M      |eng      |        82|        73|        78|        NA|
|M      |math     |        93|        89|        87|        NA|
|M      |physical |        68|        83|        82|        NA|


把学校和年级信息拆分成两列，使用names_glue参数自定义列名规则


```r
print_df(
  
  long_data %>% 
  pivot_wider(
    names_from = c(school, grade),
    values_from = score,
    names_glue = "{grade}@{school}"
  )
  
)
```



|gender |subject  | grade_1@A| grade_2@B| grade_2@A| grade_1@B|
|:------|:--------|---------:|---------:|---------:|---------:|
|F      |eng      |        90|        98|        NA|        94|
|F      |math     |        99|       100|        NA|        99|
|F      |physical |        76|        78|        NA|        74|
|M      |eng      |        82|        73|        78|        NA|
|M      |math     |        93|        89|        87|        NA|
|M      |physical |        68|        83|        82|        NA|




## 数据分割与合并


### 数据分割

把原来的grade分割成两列， 新的grade只保留数字


```r
print_df(
  
  scores %>% 
  separate(grade, into = c("trash", "grade"))
  
)
```



|school |trash |grade |gender | eng| math| physical|
|:------|:-----|:-----|:------|---:|----:|--------:|
|A      |grade |1     |F      |  90|   99|       76|
|B      |grade |2     |F      |  98|  100|       78|
|A      |grade |2     |M      |  78|   87|       82|
|B      |grade |2     |M      |  73|   89|       83|
|A      |grade |1     |M      |  82|   93|       68|
|B      |grade |1     |F      |  94|   99|       74|

### 数据合并


```r
new_data = scores %>% 
    unite("score", eng, math, physical) %>% 
    mutate(subject = "eng_math_physical")

print_df(
  new_data
)
```



|school |grade   |gender |score     |subject           |
|:------|:-------|:------|:---------|:-----------------|
|A      |grade_1 |F      |90_99_76  |eng_math_physical |
|B      |grade_2 |F      |98_100_78 |eng_math_physical |
|A      |grade_2 |M      |78_87_82  |eng_math_physical |
|B      |grade_2 |M      |73_89_83  |eng_math_physical |
|A      |grade_1 |M      |82_93_68  |eng_math_physical |
|B      |grade_1 |F      |94_99_74  |eng_math_physical |

### 数据分割 

针对上面的数据，如果不知道score是几个科目的拼接时，由于separate需要指定分割之后的名称，这里就会很麻烦。尤其是当有的学生有4科成绩，有的是3科时更麻烦，不过separate_rows()能够解决这个问题。  
这个函数能够把两列按照同样规则拼接起来的数据分割成多行。
仔细观察这个数据和上面数据的对应关系。


```r
print_df(
  
  new_data %>% 
  separate_rows(subject, score)
  
)
```



|school |grade   |gender |score |subject  |
|:------|:-------|:------|:-----|:--------|
|A      |grade_1 |F      |90    |eng      |
|A      |grade_1 |F      |99    |math     |
|A      |grade_1 |F      |76    |physical |
|B      |grade_2 |F      |98    |eng      |
|B      |grade_2 |F      |100   |math     |
|B      |grade_2 |F      |78    |physical |
|A      |grade_2 |M      |78    |eng      |
|A      |grade_2 |M      |87    |math     |
|A      |grade_2 |M      |82    |physical |
|B      |grade_2 |M      |73    |eng      |
|B      |grade_2 |M      |89    |math     |
|B      |grade_2 |M      |83    |physical |
|A      |grade_1 |M      |82    |eng      |
|A      |grade_1 |M      |93    |math     |
|A      |grade_1 |M      |68    |physical |
|B      |grade_1 |F      |94    |eng      |
|B      |grade_1 |F      |99    |math     |
|B      |grade_1 |F      |74    |physical |


## 缺失数据处理


```r
new_data =   long_data %>% 
  pivot_wider(
    names_from = grade,
    values_from = score
  )

print_df(
  new_data
)
```



|school |gender |subject  | grade_1| grade_2|
|:------|:------|:--------|-------:|-------:|
|A      |F      |eng      |      90|      NA|
|A      |F      |math     |      99|      NA|
|A      |F      |physical |      76|      NA|
|B      |F      |eng      |      94|      98|
|B      |F      |math     |      99|     100|
|B      |F      |physical |      74|      78|
|A      |M      |eng      |      82|      78|
|A      |M      |math     |      93|      87|
|A      |M      |physical |      68|      82|
|B      |M      |eng      |      NA|      73|
|B      |M      |math     |      NA|      89|
|B      |M      |physical |      NA|      83|

可以看到这里有些数据是缺失的


### 删除缺失的行

默认只要一行出现缺失值就删除


```r
print_df(
  new_data %>% 
  drop_na()
)
```



|school |gender |subject  | grade_1| grade_2|
|:------|:------|:--------|-------:|-------:|
|B      |F      |eng      |      94|      98|
|B      |F      |math     |      99|     100|
|B      |F      |physical |      74|      78|
|A      |M      |eng      |      82|      78|
|A      |M      |math     |      93|      87|
|A      |M      |physical |      68|      82|

指定按照某些列是否缺失进行删除


```r
print_df(
  new_data %>% 
  drop_na(grade_1)
)
```



|school |gender |subject  | grade_1| grade_2|
|:------|:------|:--------|-------:|-------:|
|A      |F      |eng      |      90|      NA|
|A      |F      |math     |      99|      NA|
|A      |F      |physical |      76|      NA|
|B      |F      |eng      |      94|      98|
|B      |F      |math     |      99|     100|
|B      |F      |physical |      74|      78|
|A      |M      |eng      |      82|      78|
|A      |M      |math     |      93|      87|
|A      |M      |physical |      68|      82|


### 填充缺失值
使用fill填充缺失值，可以规定填充的方向。
“down”代表使用上面的填充，“up”代表使用下面的填充  


```r
print_df(
new_data %>% 
  fill(grade_1, .direction = "down")
)
```



|school |gender |subject  | grade_1| grade_2|
|:------|:------|:--------|-------:|-------:|
|A      |F      |eng      |      90|      NA|
|A      |F      |math     |      99|      NA|
|A      |F      |physical |      76|      NA|
|B      |F      |eng      |      94|      98|
|B      |F      |math     |      99|     100|
|B      |F      |physical |      74|      78|
|A      |M      |eng      |      82|      78|
|A      |M      |math     |      93|      87|
|A      |M      |physical |      68|      82|
|B      |M      |eng      |      68|      73|
|B      |M      |math     |      68|      89|
|B      |M      |physical |      68|      83|


### 替换缺失值

分别使用各自平均分替换缺失值


```r
mean_grade_1 = mean(new_data$grade_1, na.rm = T)
mean_grade_2 = mean(new_data$grade_2, na.rm = T)


print_df(
  
  new_data %>% 
  replace_na(list(
    grade_1 = mean_grade_1, 
    grade_2 = mean_grade_2
  ))
  
)
```



|school |gender |subject  |  grade_1|   grade_2|
|:------|:------|:--------|--------:|---------:|
|A      |F      |eng      | 90.00000|  85.33333|
|A      |F      |math     | 99.00000|  85.33333|
|A      |F      |physical | 76.00000|  85.33333|
|B      |F      |eng      | 94.00000|  98.00000|
|B      |F      |math     | 99.00000| 100.00000|
|B      |F      |physical | 74.00000|  78.00000|
|A      |M      |eng      | 82.00000|  78.00000|
|A      |M      |math     | 93.00000|  87.00000|
|A      |M      |physical | 68.00000|  82.00000|
|B      |M      |eng      | 86.11111|  73.00000|
|B      |M      |math     | 86.11111|  89.00000|
|B      |M      |physical | 86.11111|  83.00000|


