---
title: '[数据工具]函数式编程与批处理-purrr'
author: baoliu
date: '2021-03-21'
slug: []
categories:
  - R
tags:
  - tidyverse
  - purrr
---



```r
library(purrr)
library(dplyr)

library(knitr)
```

# **purrr**

函数式编程能够提高代码的复用率, 通过把重复、通用的代码封装成函数，提高了可读性。  


```r
get_hello <- function(name, age){
  if(age >= 18){
    cat('hello shehuiren ', name, '\n')
  }else{
    cat('hello xiaoxianrou ', name, '\n')
  }
}

get_hello("roy", 17)
```

```
## hello xiaoxianrou  roy
```

purrr能够帮助用户实现快速循环遍历并执行函数，和baseR中apply家族类似。  


```r
name <- c("zhangsan", "lisi", "wangwu", "zhaoliu")
age <- c(17, 18, 16, 19)

walk2(name, age, get_hello)
```

```
## hello xiaoxianrou  zhangsan 
## hello shehuiren  lisi 
## hello xiaoxianrou  wangwu 
## hello shehuiren  zhaoliu
```

## map

### map_dfr

下面的数据计算了用户行为及其发生的日期，我写了一个函数通过传入某单个用户的行为数据，返回出该用户最长连续产生行为的天数。


```r
# 原始数据
sample_data <- data.frame(
  id = c('001', '002', '001', '003', '004', '002', '003', '003', '002', '001', '003', '002', '002', '001'),
  day = c("2021-03-22", "2021-03-23", "2021-03-26", "2021-03-27","2021-03-27", "2021-03-26","2021-03-26", "2021-03-27","2021-03-27", "2021-03-26","2021-03-27", "2021-03-27","2021-03-26", "2021-03-26")
)

# 数据整理，目标是整理成一个所有用户在所有日期的行为情况，有行为是1，没有行为是0
sample_data <- sample_data %>% 
  mutate(has_record = 1)

sample_data_complete <- sample_data %>% 
  # select(-X) %>% 
  tidyr::complete(id, day) %>% 
  tidyr::replace_na(list(has_record = 0))

kable(sample_data_complete)
```



|id  |day        | has_record|
|:---|:----------|----------:|
|001 |2021-03-22 |          1|
|001 |2021-03-23 |          0|
|001 |2021-03-26 |          1|
|001 |2021-03-26 |          1|
|001 |2021-03-26 |          1|
|001 |2021-03-27 |          0|
|002 |2021-03-22 |          0|
|002 |2021-03-23 |          1|
|002 |2021-03-26 |          1|
|002 |2021-03-26 |          1|
|002 |2021-03-27 |          1|
|002 |2021-03-27 |          1|
|003 |2021-03-22 |          0|
|003 |2021-03-23 |          0|
|003 |2021-03-26 |          1|
|003 |2021-03-27 |          1|
|003 |2021-03-27 |          1|
|003 |2021-03-27 |          1|
|004 |2021-03-22 |          0|
|004 |2021-03-23 |          0|
|004 |2021-03-26 |          0|
|004 |2021-03-27 |          1|



```r
# 封装函数get_max_num，对于传入的每一个用户，计算其行为最长连续天数
get_max_num <- function(df){
  has_record = df[,3]
  zero_index = c(0, which(has_record == 0), dim(has_record)[1]+1) %>% 
    tibble::as.tibble() %>% 
    mutate(
      num = value - lag(value, 1)
        )
  
  temp = data.frame(
    user_id = unique(df[,1]),
    max_num = max(zero_index$num, na.rm = T) - 1
    )
  
  return(temp)
}

# 把用户数据按照id分成多个，形成一个list
sample_data_complete_split <- split(sample_data_complete, sample_data_complete$id)

# 遍历list中每个元素，并作为参数传递给函数get_max_num, 并把最后得到的结果合并成dataframe
sample_data_complete_split %>% 
  map_dfr(get_max_num)
```

```
##    id max_num
## 1 001       3
## 2 002       5
## 3 003       4
## 4 004       1
```



## walk


