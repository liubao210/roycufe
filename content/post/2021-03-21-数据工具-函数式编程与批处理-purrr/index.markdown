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
purrr能够帮助用户实现快速循环遍历并执行函数，和baseR中apply家族类似。  


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

一个简单但毫无实用价值的例子:  
map函数把x_里的每一个元素，当作参数传递给get_square，并将返回的结果合并成一个list
map函数把x_里的每一个元素，当作参数传递给get_square，并将返回的结果合并成一个向量


```r
get_square <- function(x){
  x^2
}

x_ <- c(1, 2, 3, 4, 5, 6)

x_ %>% map(get_square)
```

```
## [[1]]
## [1] 1
## 
## [[2]]
## [1] 4
## 
## [[3]]
## [1] 9
## 
## [[4]]
## [1] 16
## 
## [[5]]
## [1] 25
## 
## [[6]]
## [1] 36
```

```r
x_ %>% map_dbl(get_square)
```

```
## [1]  1  4  9 16 25 36
```


## map

### map_dfr
map_dfr把每一个产生的data.frame按照行合并，合并成一个dataframe.  

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
walk适用于没有返回值的函数，比如函数的作用是修改某个固定的变量。  


```r
x <- 0

add_score = function(a){
  x <<- x + a
}

x_ <- c(1, 2, 3, 4, 5, 6)

x_ %>% walk(add_score)

x
```

```
## [1] 21
```

一个实用的案例是我自己的plumber接口，在挂载各个分开编写的文件到root的时候，使用walk遍历文件夹下的所有接口文件，并将其挂载到root对象上。  


```r
mount_fn = function(children, dev_or_not = F){
  file.name = children
  path.name = paste0("/", gsub("\\.[rR]$", "", file.name))
  
  tryCatch({
    if(gsub("\\.[rR]$", "", file.name) %in% ignore_list){
      cat("Ignore apis file:", file.name, "\n")
    }else{
      if(dev_or_not){
        api.functions = pr(file.path("main/apis_dev", file.name))
      }else{
        api.functions = pr(file.path("main/apis", file.name))
      }
      root %>% pr_mount(path.name, api.functions)
      cat("Add apis file:", file.name, " successfully ", "\npath: ", path.name, "\n")
    }
    
  },
  error = function(e){
    print(e)
    cat("\nAdd apis file:", file.name, " error\n")
  })
}

## mount all====
root = pr()
ignore_list = c()

file.list = list.files("main/apis/")

file.list %>% 
  walk(mount_fn, dev_or_not = F)
```




