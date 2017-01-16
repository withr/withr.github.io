---
layout: post
title: "Authentication of Shiny-server Application Using a Simple Method"
date: 2014-01-03 12:41
comments: true
categories: R Shiny
---

The *commercial* version of Shiny-server has an authentication feature for shiny application. Before  the release of the commercial (and still for open source version) version Shiny-server, it's [not easy](https://groups.google.com/forum/#!searchin/shiny-discuss/authenticate/shiny-discuss/3HbwHXBTzNo/bPMwqIs-WosJ) to add authentication feature to shiny application. I want to share my experience of adding authentication feature to shiny application. It's simple, but I can’t guarantee its security, if you can hack the feature, I am very pleased to hear. 

The idea is: add a conditional command to encapsulate the output code, so if the condition is false, the output code will not be run. For example: 

``` ruby
output$distPlot <- renderPlot({
  if (USER$Logged == TRUE) {
    dist <- NULL
    dist <- rnorm(input$obs)
    hist(dist, breaks = 100)
  }
})
```

Where *USER* is an **activeValue**: 

``` ruby
USER <- reactiveValues(Logged = FALSE)
```

Then we can create a **login** input widget to let user type user name and password. 

``` ruby
output$uiLogin <- renderUI({
  if (USER$Logged == FALSE) {
    wellPanel(
      textInput("Username", "User Name:"),
      textInput("Password", "Pass word:"),
      br(),
      actionButton("Login", "Log in")
    )
  }
})

output$pass <- renderText({  
  if (USER$Logged == FALSE) {
    if (input$Login > 0) {
      Username <- isolate(input$Username)
      Password <- isolate(input$Password)
      Id.username <- which(PASSWORD$Brukernavn == Username)
      Id.password <- which(PASSWORD$Passord    == Password)
      if (length(Id.username) > 0 & length(Id.password) > 0) {
        if (Id.username == Id.password) {
          USER$Logged <- TRUE
        } 
      } else  {
        "User name or password failed!"
      }
    } 
  }
})

```

The usernames and passwords can be saved in a data.frame in global environment. Also, we need to assign an initial value to **Logged** to control whether we want users to input their user names and passwords. 

``` ruby
Logged = FALSE;
PASSWORD <- data.frame(Brukernavn = "withr", Passord = "123456")
```

Here is an [example](http://spark.rstudio.com/withr/authentication/). Note: the username is *withr* and password is *123456*.






















