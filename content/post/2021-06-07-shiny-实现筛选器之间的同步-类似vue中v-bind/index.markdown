---
title: '[shiny]实现筛选器之间的同步 - 类似VUE中v-bind'
author: roy
date: '2021-06-07'
slug: []
categories:
  - R
tags:
  - shiny
---



## **实现筛选器选择结果一致，任何一方改动都会同步到另一个筛选器**



```r
library(shiny)


ui <- fluidPage(
  selectInput("inSelect1", "Select input",
              c("Item A", "Item B", "Item C")),
  selectInput("inSelect2", "Select input",
              c("Item A", "Item B", "Item C"))
)

server <- function(input, output, session) {
  observe({
    
    observeEvent(input$inSelect1, {
      if(input$inSelect1 != input$inSelect2){
        updateSelectInput(session, "inSelect2",
                          selected = input$inSelect1
        )
      }
    })
    
    observeEvent(input$inSelect2, {
      if(input$inSelect1 != input$inSelect2){
        updateSelectInput(session, "inSelect1",
                          selected = input$inSelect2
        )
      }
    })
    
    
  })
}

shinyApp(ui, server)
```

