---
layout: post
title: "Git commit regularly to GitLab"
date: 2016-04-26 14:24
comments: true
categories: R Shiny
---






This post shows how to configure an autocommit environment for GitLab (an internal version of Github). 

Suppose you have a project under folder: **myproject**, you want to save each version of your project and save it on GitLab for security reason. The following steps will configure the system for committing your project automatically. 

### Step 1: create a repository on GitLab. 

This step is simple, and you will create a repository named: “myproject.git”. The repository name should be same as the name of your project folder. E.g:http://gitlab.kreftregisteret.no/huti/myproject.git


### Step 2: add SSH key of your server to GitLab

It’s not mandatory, but very common to add the public SSH key to GitLab, so you can transfer data between your server and GitLab safely. Your server may have already got its SSH key, you can check using command: 

```
cat ~/.ssh/id_rsa.pub
```

If there is no such file, then you can generate the key using command like: 

```
ssh-keygen -t rsa -C "myservername"
```

Using the command <code>cat ~/.ssh/id_rsa.pub</code> to show the public key, and copy it to GitLab -> Profile settings -> SSH keys.


### Step 3: configure git to ignore some kind of files

Some files, like dataset, you don’t want them be transfered to GitLab because, they are big and change regularly. So, we need to tell git to ignore those files. If your server has no git installed, you can use the following command to install it: 


```
sudo apt-get install git
```

Now, we tell git that all git repositories will ignore the file list in ~/.gitignore

```
git config --global core.excludesfile ~/.gitignore
sudo nano ~/.gitignore
```

For example, if I want to ignore all .RData files on my computer, I add ".RData" into **~/.gitignore**. More information can get [here](https://help.github.com/articles/ignoring-files/)

### Step 4: git commit & origin master

Link the project's repository of GitLab to git, so git knows where to push the commits. 


```
git remote add origin ssh://git@gitlab.kreftregisteret.no/huti/myproject.git
```


Now, let's push our first commit: 

```
cd /srv/shiny-server/myproject
git init 
git add -A
git commit -m "First commit"
git push -u origin master
```

### Step 5: commit automatically
We can create a bash file, and let it run periodically, if there are any changes, the project will be committed, and pushed to GitLab. 
The bash file (“~/myproject.sh”) will look like this: 

```
#!/bin/bash
cd /srv/shiny-server/myproject
git add -A
git commit -m "daily update"
git push -u origin master
```

We need to change its mode so it can be executed by bash: 

```
sudo chmod 765 ~/myproject.sh
```

To test if the bash script can be executed, just type the mand in terminal: 


```
 ~/myproject.sh
```

Then we add an event to run the bash script periodically by adding a line to */etc/crontab*: 

```
sudo nano /etc/crontab
01 18  *  *  * root  /home/huti/myproject.sh
```

The above command let the bash script run very day at 18:01. Note, due to if there is no changes, or modification, nothing will be pushed to GitLab, so don’t worry that the repository will be too messy. 
