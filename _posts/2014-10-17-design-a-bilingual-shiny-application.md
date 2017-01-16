---
layout: post
title: "Design A Bilingual Shiny Application"
date: 2014-10-17 13:25
comments: true
categories: R Shiny
---

It's a common requirement that your *Shiny* application should be **bilingual**. I have tried several methods and finally I got one which is not so difficult to maintain. 

My idea is: 

- Create a bilingual dictionary or a lookup table which contains three columns: *value*, *English name* and *Norwegian name*, like this one: 

{% highlight R %}
value EN    NO
----------------
time  Time  Tid
day   Day   Dag
week  Week  Uke
month Month Måned
{% endhighlight %}

- Create a radio input with *English* and *Norsk* two options in file *ui.R*. 

``` ruby
radioButtons(inputId = "lang", label = "", 
             choices = c("English" = "en", "Norsk" = "no"), 
             selected = "English"),
```               
               
- Create a function that converts *value*'s name to *Norsk* if user choose the *Norsk* radio.

``` ruby
## Bilingual lookup table;
LANG <- unique(read.csv("www/lang.csv",  sep = ";", stringsAsFactors = FALSE))
E2N <- function(en, D = LANG) {
  for (i in 1:length(en)) {
    id <- which(D$value %in% en[i])
    if (length(id) == 0) {
      id <- which(D$EN %in% en[i])
      if (length(id) == 1) {
        en[i] <- ifelse(input$lang == "no", D$NO[id], D$EN[id]) 
      } else if (length(id) >  1) {
        print("Mulit-variable matched!")
      }
    } else if (length(id) == 1) {
      en[i] <- ifelse(input$lang == "no", D$NO[id], D$EN[id]) 
    } else if (length(id) >  1) {
      print("Mulit-variable matched!")
    }
  } 
  return(en)
}

``` 

The *ui.R* file can be written as follow: 

``` ruby
shinyUI(pageWithSidebar(
  ## English or Norwegain?
  radioButtons(inputId = "lang", label = "", 
               choices = c("English" = "en", "Norsk" = "no"), 
               selected = "English"),
  # Select unit;
  sidebarPanel(
    uiOutput("uiUnit")
  ),
  mainPanel(
    h1(textOutput("text1"))
  )
))

``` 

In *server.R* file, we need load the **E2N** function first: 

``` ruby
shinyServer(function(input, output, session) {
  LANG <- data.frame(value = c("time", "day", "week", "month"), 
                     EN    = c("Time", "Day", "Week", "Month"),
                     NO    = c("Tid",  "Dag", "Uke",  "Maaned"), 
                     stringsAsFactors = FALSE)
  observe({
    E2N <- function(en, D = LANG) {
      for (i in 1:length(en)) {
        id <- which(D$value %in% en[i])
        if (length(id) == 0) {
          id <- which(D$EN %in% en[i])
          if (length(id) == 1) {
            en[i] <- ifelse(input$lang == "no", D$NO[id], D$EN[id]) 
          } else if (length(id) >  1) {
            print("Mulit-variable matched!")
          }
        } else if (length(id) == 1) {
          en[i] <- ifelse(input$lang == "no", D$NO[id], D$EN[id]) 
        } else if (length(id) >  1) {
          print("Mulit-variable matched!")
        }
      } 
      return(en)
    }
    ## Input;
    output$uiUnit <- renderUI({
     Items <- c("day", "week", "month")
     selectInput(inputId   = "unit", 
                  label     = E2N("time"), 
                  choices   = E2N(Items), 
                  multiple  = TRUE)
    })
    ## Output;
    output$text1 <- renderText({ 
          E2N(input$unit)
     })
  })
}) 

``` 

One shortcoming of this method is: when switching language, the application goes back to its original state.
You can run above demo in R console: 

``` ruby
library(shiny)
runGist("https://gist.github.com/withr/3c01413b70b8a82706ab")

```
