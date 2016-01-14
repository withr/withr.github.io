---
layout: post
title: "Setup website of your Github account with Jekyll"
date: 2016-01-14 13:32
comments: true
categories: Git
---



I have to write this blog because it always needs half day to get everything work. **Note**: always download 64-bit version softwares, if you use 64-bit computer.




## Install git. 
Download [Git](https://git-scm.com/download/win) and install it. 
Run *Git Bash*, and set up the user name and email address, like: 

```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```


Generate **SSK key** (*id_rsa.pub*) and add its content to your Github repository. You can use command **pwd** to find where the *id_rsa.pub* generated. 

```
cd ~/.ssh
ssh-keygen
```

## Fork Jekyll to your Github account. 

Log onto your Github account, and find the Jekyll [repository]()https://github.com/jekyll/jekyll). Fork it to your account, and rename the repository to: **yourgithubusername.github.io**


## Install Ruby
Download 64-bit version of Ruby, e.g. [Ruby 2.2.3(x64)](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.2.3-x64.exe), and install it. Remember toggle the option: *Add Ruby executables to your PATH* 

## Install DevKit
Download 64-bit version of [DevKit](http://dl.bintray.com/oneclick/rubyinstaller/DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe) and unzip it to somewhere. Note: there must have no space in its directory. 
Go to the unzipped directory, then bind DevKit to Ruby using the following commands:

```
ruby dk.rb init
ruby dk.rb install
```


 
## Install Python 2.7
Install [Python 2.7](https://www.python.org/ftp/python/2.7.11/python-2.7.11.msi) and add its **bin** directory to system environment variable **PATH**.



## Clone Jekyll forked to local machine.

```
git clone git@github.com:yourusername/yourusername.github.io.git
```

## Install Jekyll

```
gem install github-pages
```

Navigate to the cloned folder, and run command: 

```
jekyll serve
```


If everything is OK, you can view your website at: http://0.0.0.0:4000


## Normal operation
You can always **git pull** from the repository on Github if you change the repository on Github. If you prefer edit your website locally, remember the following commands for pushing change to website:

```
git add -A
git commit -m “some message”
git push
```

