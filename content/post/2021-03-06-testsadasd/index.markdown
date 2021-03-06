---
title: 特征工程 - 编码和分箱的几种方法
author: baoliu
date: '2021-03-06'
categories:
  - R
  - ML
  - binning
tags:
  - ML
---



```r
library(dplyr)
library(knitr)
library(DT)
library(echarts4r)
```

## **简介**

**特征工程是数据科学中模型拟合的重要一步，这篇文章讲述了如何进行从分类变量到连续变量以及从连续变量到分类变量的转换。**  

**分箱： 把连续变量转换为分类变量**    
**编码： 把分类变量转换为连续变量**  


## **分箱**

分箱(离散化)是指把连续型变量转换为分类特征的方法。常常能够提升模型表现，也能够帮助识别出离群值。
分箱的方式可以分为两种：
  - 无监督的分箱： 等距分箱、等频分箱
  - 监督分箱：基于熵的分箱


### **无监督分箱**

无监督的分箱无需考虑模型因变量。

#### **等距分箱**

顾名思义，假如有一列数据为价格，按照10为一组就是等距分箱；



```r
#
price <-  data.frame(
  price = floor(abs(rnorm(50, mean = 70 , sd = 20)))
  ) %>%
  arrange(price)

price$price
```

```
##  [1]  30  31  38  43  48  48  48  50  51  52  53  53  53  56  56  56  58  58  59
## [20]  61  62  64  64  64  65  67  68  68  70  72  72  73  75  77  79  81  83  86
## [39]  88  88  90  90  91  92  95  95  95  97  98 116
```




```r
new_data <- price %>%
  mutate(
    price_group = paste0("[", floor(price / 10), ", ",ceiling(price / 10), "]"),
    group_index = floor(price / 10)
  )

kable(new_data)
```



| price|price_group | group_index|
|-----:|:-----------|-----------:|
|    30|[3, 3]      |           3|
|    31|[3, 4]      |           3|
|    38|[3, 4]      |           3|
|    43|[4, 5]      |           4|
|    48|[4, 5]      |           4|
|    48|[4, 5]      |           4|
|    48|[4, 5]      |           4|
|    50|[5, 5]      |           5|
|    51|[5, 6]      |           5|
|    52|[5, 6]      |           5|
|    53|[5, 6]      |           5|
|    53|[5, 6]      |           5|
|    53|[5, 6]      |           5|
|    56|[5, 6]      |           5|
|    56|[5, 6]      |           5|
|    56|[5, 6]      |           5|
|    58|[5, 6]      |           5|
|    58|[5, 6]      |           5|
|    59|[5, 6]      |           5|
|    61|[6, 7]      |           6|
|    62|[6, 7]      |           6|
|    64|[6, 7]      |           6|
|    64|[6, 7]      |           6|
|    64|[6, 7]      |           6|
|    65|[6, 7]      |           6|
|    67|[6, 7]      |           6|
|    68|[6, 7]      |           6|
|    68|[6, 7]      |           6|
|    70|[7, 7]      |           7|
|    72|[7, 8]      |           7|
|    72|[7, 8]      |           7|
|    73|[7, 8]      |           7|
|    75|[7, 8]      |           7|
|    77|[7, 8]      |           7|
|    79|[7, 8]      |           7|
|    81|[8, 9]      |           8|
|    83|[8, 9]      |           8|
|    86|[8, 9]      |           8|
|    88|[8, 9]      |           8|
|    88|[8, 9]      |           8|
|    90|[9, 9]      |           9|
|    90|[9, 9]      |           9|
|    91|[9, 10]     |           9|
|    92|[9, 10]     |           9|
|    95|[9, 10]     |           9|
|    95|[9, 10]     |           9|
|    95|[9, 10]     |           9|
|    97|[9, 10]     |           9|
|    98|[9, 10]     |           9|
|   116|[11, 12]    |          11|

```r
# new_data %>%
#   group_by(price_group, group_index) %>%
#   summarise(freq = n()) %>%
#   ungroup() %>%
#   arrange(group_index) %>%
#   e_charts(price_group) %>%
#   e_bar(freq) %>%
#   e_grid(left = '7%', right = '2%') %>%
#   e_x_axis(axisLabel = list(interval = 0, rotate = 315))
```



#### **等频分箱**

顾名思义，假如有一列数据为价格，把它分成等量的几组就是等距分箱；


```r
# x指要分成的组数
x <- 10
cat("一共要分成", x, "组", "每组", dim(price)[1] / 10, "个")
```

```
## 一共要分成 10 组 每组 5 个
```

```r
price_group_index <- c(rep(1:10, each = dim(price)[1] / 10))

new_data <- price %>%
  mutate(
    group_index = price_group_index
  ) %>%
  group_by(group_index) %>%
  mutate(price_group = paste0("[", min(price), ", ", max(price), "]"))


kable(new_data)
```



| price| group_index|price_group |
|-----:|-----------:|:-----------|
|    30|           1|[30, 48]    |
|    31|           1|[30, 48]    |
|    38|           1|[30, 48]    |
|    43|           1|[30, 48]    |
|    48|           1|[30, 48]    |
|    48|           2|[48, 52]    |
|    48|           2|[48, 52]    |
|    50|           2|[48, 52]    |
|    51|           2|[48, 52]    |
|    52|           2|[48, 52]    |
|    53|           3|[53, 56]    |
|    53|           3|[53, 56]    |
|    53|           3|[53, 56]    |
|    56|           3|[53, 56]    |
|    56|           3|[53, 56]    |
|    56|           4|[56, 61]    |
|    58|           4|[56, 61]    |
|    58|           4|[56, 61]    |
|    59|           4|[56, 61]    |
|    61|           4|[56, 61]    |
|    62|           5|[62, 65]    |
|    64|           5|[62, 65]    |
|    64|           5|[62, 65]    |
|    64|           5|[62, 65]    |
|    65|           5|[62, 65]    |
|    67|           6|[67, 72]    |
|    68|           6|[67, 72]    |
|    68|           6|[67, 72]    |
|    70|           6|[67, 72]    |
|    72|           6|[67, 72]    |
|    72|           7|[72, 79]    |
|    73|           7|[72, 79]    |
|    75|           7|[72, 79]    |
|    77|           7|[72, 79]    |
|    79|           7|[72, 79]    |
|    81|           8|[81, 88]    |
|    83|           8|[81, 88]    |
|    86|           8|[81, 88]    |
|    88|           8|[81, 88]    |
|    88|           8|[81, 88]    |
|    90|           9|[90, 95]    |
|    90|           9|[90, 95]    |
|    91|           9|[90, 95]    |
|    92|           9|[90, 95]    |
|    95|           9|[90, 95]    |
|    95|          10|[95, 116]   |
|    95|          10|[95, 116]   |
|    97|          10|[95, 116]   |
|    98|          10|[95, 116]   |
|   116|          10|[95, 116]   |

```r
# new_data %>%
#   group_by(price_group, group_index) %>%
#   summarise(freq = n()) %>%
#   ungroup() %>%
#   arrange(group_index) %>%
#   e_charts(price_group) %>%
#   e_bar(freq) %>%
#   e_grid(left = '7%', right = '2%') %>%
#   e_x_axis(axisLabel = list(interval = 0, rotate = 315))
```



### 监督分箱

监督分箱是指在分箱的时候考虑目标分类结果，即根据已有目标分类结果来衡量怎么对一个连续型自变量分箱最好。

#### 基于熵的分箱

基于MDLP的熵分组是Fayyad和Irani于1993年提出的，它是一种有指导的数据分箱方法。其基本思想是，如果分组后的输入变量对输出变量取值的解释能力显著低于分组之前，那么这样的分组是没有意义的。所以，待分组变量(视为输入变量)应在输出变量的“指导”下进行分组。  
简单的理解为，假设有一列商品价格，有一列是是否畅销商品，那么如果要把商品价格转换成分类变量，在转换的时候可以考虑怎么分箱可以使得分箱的结果最接近于是否畅销商品这一已有结果(使分箱结果与已有结果相吻合)。

下面的例子是对鸢尾花数据集中的Sepal.Length进行分组使之与鸢尾花品种吻合。


建立用于计算熵的函数

```r
## 用于计算熵的函数
get_ent <-  function(x){
  ## 计算不同种类的频次
  # x = ifelse(is.factor(x), x, is.factor(x))
  x_count <- as.data.frame(table(x))$Freq
  
  #计算频率
  p <- x_count / sum(x_count)
  
  #计算熵
  if(is.na(p) | cumprod(p)[length(p)] == 0){
    ent <-  0
  }else{
    ent <- -sum(p * log2(p))
  }
  
  
  
  ent
}
```

建立用于找到最佳分割点的函数，原理是取熵增最大的分割点

```r
## 用于找到最佳分割点的函数，原理是取熵增最大的分割点
get_cut = function(x, y){
  df <- data.frame(x, y) %>% 
    arrange(x) %>% 
    mutate(id = 1:n())
  
  index = 0
  entropy <- 9999
  for (i in 1:length(x)) {
    df <- df %>% 
      mutate(temp_group = ifelse(id >= i, "T", "F"))
    
    ent <- nrow(filter(df, temp_group == "T"))*get_ent(filter(df, temp_group == "T")$y) + nrow(filter(df, temp_group == "F"))*get_ent(filter(df, temp_group == "F")$y)
    
    if(ent < entropy){
      entropy = ent
      index = i
    }
    
    # cat(round(i/length(x)*100, 2), "%")
  }
  
  return(list(
    "index" = index,
    "entropy" = entropy
  ))
  
}
```



```r
temp = iris %>% filter(Species != "Species")

x = temp$Sepal.Length
y = temp$Species
y
```

```
##   [1] setosa     setosa     setosa     setosa     setosa     setosa    
##   [7] setosa     setosa     setosa     setosa     setosa     setosa    
##  [13] setosa     setosa     setosa     setosa     setosa     setosa    
##  [19] setosa     setosa     setosa     setosa     setosa     setosa    
##  [25] setosa     setosa     setosa     setosa     setosa     setosa    
##  [31] setosa     setosa     setosa     setosa     setosa     setosa    
##  [37] setosa     setosa     setosa     setosa     setosa     setosa    
##  [43] setosa     setosa     setosa     setosa     setosa     setosa    
##  [49] setosa     setosa     versicolor versicolor versicolor versicolor
##  [55] versicolor versicolor versicolor versicolor versicolor versicolor
##  [61] versicolor versicolor versicolor versicolor versicolor versicolor
##  [67] versicolor versicolor versicolor versicolor versicolor versicolor
##  [73] versicolor versicolor versicolor versicolor versicolor versicolor
##  [79] versicolor versicolor versicolor versicolor versicolor versicolor
##  [85] versicolor versicolor versicolor versicolor versicolor versicolor
##  [91] versicolor versicolor versicolor versicolor versicolor versicolor
##  [97] versicolor versicolor versicolor versicolor virginica  virginica 
## [103] virginica  virginica  virginica  virginica  virginica  virginica 
## [109] virginica  virginica  virginica  virginica  virginica  virginica 
## [115] virginica  virginica  virginica  virginica  virginica  virginica 
## [121] virginica  virginica  virginica  virginica  virginica  virginica 
## [127] virginica  virginica  virginica  virginica  virginica  virginica 
## [133] virginica  virginica  virginica  virginica  virginica  virginica 
## [139] virginica  virginica  virginica  virginica  virginica  virginica 
## [145] virginica  virginica  virginica  virginica  virginica  virginica 
## Levels: setosa versicolor virginica
```

```r
get_cut(x, y)
```

```
## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素

## Warning in if (is.na(p) | cumprod(p)[length(p)] == 0) {: 条件的长度大于一，因此
## 只能用其第一元素
```

```
## $index
## [1] 75
## 
## $entropy
## [1] 80.31319
```



## 编码


### 标签编码
