---
layout: post
title: "Configure Character Encoding for R under Linux and Windows"
date: 2013-11-15 09:27
comments: true
categories: R shiny
---

R is so powerful and elegant, that attacts more and more people switch to it. Those users come from different places, speack different languages and possiblely use other statistical softwares. So they may encounter one anoying problem: **garbled text or unrecognizable characters**! For example, some Chinese user wrote a script including variable names in Chinese, and his English colleage's computer can't show those character correctly, and can't run the script under R. Such kind of problem can be categorized as: **Character Encoding problem**. In the following context, I will share my experience of how to configure character encoding for R under Linux and Windows. Before we go further, let't take a look of the basic concept of character encoding. 



There are thousands of hundrads of characters in this world, like *LatinÆ characters, *Arabic* charaters, *Chinese*, *Korean*, etc. How to store and show them in computer? Yes, character encoding. The most widely known encoding called **ASCII**, which was invented at the beginning of computer techniqure. ASCII uses one byte to store the numbers 0-9, the letters a-z and A-Z, and some basic punctuation symbols, etc. ASCII encoding includes 128 (2^7) characters in total. We know that computer use *Byte* as the basic store unit, one byte has 8 bits (2^8 = 256), and ASCII used only half of them, why? I guess at that time the 128 characters are the most used characters, and the inventors may believe that's enought for daily tasks. With the developement of computer technology, more and more characters need to be stored and shown in computer, the ASCII encoding can't meet the demands. So some other character encodings were invented. For example, the **ISO-8859-1** encoding, which also is called **Latin-1**, is widlely used in Enroupe. *ISO-8859-1* can be taken as a superset of ASCII, which mean the characters encoded in ASCII can also be displayed correctly in *ISO-8859-1*, but the not the opposite version. Note, if the characters encoded in *ISO-8859-1* contains only the characters supported by ASCII, these characters can still be displayed correctly in ASCII. *ISO-8859-1* uses 8-bit byte to store characters, but the total characters *ISO-8859-1* supported is not 256 (2^8), that's because somes range of bits were not used. Perhaps, the inventors believe those characters are enough.  **Windows-1252** is another character encoding, similiar to *ISO-8859-1* but contains more characters, because it used total bits. So *Windows-1252* can be taken as the supperset of *ISO-8859-1*. But there are more characters than 256, e.g. Chinese contains thousands of characters, and many other kinds of lanuages also contain numerous characters. To solve this problem, **Unicode** was invented. The newest Unicode encoding uses 4 bytes, and have encoded more than 110000 characters, contains almost all writing characters. So, why not all computers use Unicode encoding, and people will not worry about garbled text? There are several reasons, but the obvious one is it waste too much storage space. For example, a European computer may use only *Latin* characters, why waste three times storage space to save all characters using *Unicode*? The usage of different character encoding lead to the prolbem of garbed text! Ok, now let's back to the configuration of character encoding for R. 

When encounter garbed text in R, we should first check the encoding type of R.  I will show how to check and setup encoding under Linux and Windows. 

###1. Under Linux 
 Under Linux, we start R programe by typing <code>R</code> in terminal (which can be called up by press <code>Ctrl+Alt+T</code>). To check the current encoding, we use the command **Sys.getlocale()**. This command will list the current locale setting, including *LC_CTYPE*, *LC_NUMERIC*, etc. 

``` ruby
> Sys.getlocale()
[1] "LC_CTYPE=nb_NO.ISO-8859-1;LC_NUMERIC=C;LC_TIME=nb_NO.ISO-8859-1;LC_COLLATE=nb_NO.ISO-8859-1;LC_MONETARY=nb_NO.ISO-8859-1;LC_MESSAGES=nb_NO.ISO-8859-1;LC_PAPER=nb_NO.ISO-8859-1;LC_NAME=C;LC_ADDRESS=C;LC_TELEPHONE=C;LC_MEASUREMENT=nb_NO.ISO-8859-1;LC_IDENTIFICATION=C"
```

For example, *LC_CTYPE=nb_NO.ISO-8859-1* means the current locale (<code>LC_</code>) for character type (<code>CTYPE</code>) is *nb_NO.ISO-8859-1*. *nb_NO.ISO-8859-1* contains the information for language (<code>nb</code> means Norwegian (Bokmål)), region (<code>NO</code> means Norway) and character encoding seperated with dot (<code>ISO-8859-1</code>). 

Ok, this is the current character encoding, and I encountered the garbed text problem, how to change the encoding to show text correctly, suppose I know the script was writen by a Chinese? There is function is R called **Sys.setlocale** can do the configuration. But wait! What's the correct string for the locale I want to setup? I know one encoding for Chinese character is CP18030, should I type "zh_CN.CP-18030" or "zh_CN.WINDOWS-18030", or something else? I mean where I can find a list contains the information of language and corresponding string? In R, there is no such a function, but we can get them in Linux. In Linux's terminal, type <code>locale</code> will list current locale setting:

``` ruby
tian@tian-VirtualBox:~$ locale
LANG=
LANGUAGE=
LC_CTYPE="nb_NO.ISO-8859-1"
LC_NUMERIC="nb_NO.ISO-8859-1"
LC_TIME="nb_NO.ISO-8859-1"
LC_COLLATE="nb_NO.ISO-8859-1"
LC_MONETARY="nb_NO.ISO-8859-1"
LC_MESSAGES="nb_NO.ISO-8859-1"
LC_PAPER="nb_NO.ISO-8859-1"
LC_NAME="nb_NO.ISO-8859-1"
LC_ADDRESS="nb_NO.ISO-8859-1"
LC_TELEPHONE="nb_NO.ISO-8859-1"
LC_MEASUREMENT="nb_NO.ISO-8859-1"
LC_IDENTIFICATION="nb_NO.ISO-8859-1"
LC_ALL=nb_NO.ISO-8859-1
``` 
Type <code>locale -a</code> will list all the encoding types Linux have installed. Go through them to see if the language you are looking for has already been there. 

``` ruby
tian@tian-VirtualBox:~$ locale -a
C
C.UTF-8
nb_NO
nb_NO.iso88591
nb_NO.utf8
nn_NO
nn_NO.iso88591
nn_NO.utf8
POSIX
zh_CN.utf8
zh_SG.utf8
``` 

If it's not there, then we need to install it, and before we start to install we should first know the name of the locale, which should be in the list of supported locales of Linux. Type <code>less /usr/share/i18n/SUPPORTED</code> will list all the supported locales, choose the one you want and generate it using the command: <code>sudo locale-gen</code>. E.g <code>sudolocale-gen nb_NO</code> will install the Norwegian (Bokmål) for Norway. It didn't show the character encoding type, because by default Norwegian (Bokmål) using *ISO-8859-1*. Update the generated locale to system using command: <code>sudo update-locale</code>. Now the locale has been installed to computer, but we still didn't make it as the default system locale. We can change the default locale by editing the file <code>/etc/default/locale</code> to the locale we wanted.  

``` ruby
sudo nano /etc/default/locale
``` 
Save the change and restart the computer, you can check the current locale setting using <code>locale</code> command. 

By default, R will use the locale setting of the operation system. So if we type command **Sys.getlocale()** in R console, the installed locale setting will be displayed. R also has a function for temporary setting of locale: **Sys.setlocale**. If the current locale setting can not show some script correctly, we need to guess which locale is suitable, so we use *Sys.setlocale*. For example, <code>Sys.setlocale(category = "LC_ALL", locale = "zh_CN.utf-8")</code> will change current locale to Chinese (Simplified)_People's Republic of China using character encoding "UTF-8". Note, this changes the locale temporary, when you quit R and restart it again, the locale is still the system's locale. 

###2. Under Windows
The configuration of locale under Windows is a bit less concise than under Linux.

We can get the current locale in R using **Sys.getlocale()**.  

``` ruby
> Sys.getlocale()
[1] "LC_COLLATE=Norwegian (Bokmål)_Norway.1252;LC_CTYPE=Norwegian (Bokmål)_Norway.1252;LC_MONETARY=Norwegian (Bokmål)_Norway.1252;LC_NUMERIC=C;LC_TIME=Norwegian (Bokmål)_Norway.1252"

``` 
Note, the displayed locale details are a little different with the ones under Linux. For example, **LC_CTYPE=Norwegian (Bokmål)_Norway.1252**  means Norwegian Bokmål for Norway using **codepage** 1252. The new term *codepage* is actually the character encoding used for Windows. For example codepage '936' means the character encoding for *GB18030* which is a Chinese character encoding. Unlike Linux, Windows supplies only one encoding for each language. 

If I want to change current locale to Chinese, how should I proceed? Well, we can change the locale in Windows' **Control Panel -> Region and Language -> Format**, choose the format you want, e.g. "Chinese (Simplified, PRC)", and apply change. Now, restart R and check the locale information using command <code>Sys.getlocale()</code>. We can see the default locale for R is "Chinese (Simplified)_People's Republic of China.936", and the code page '936' is the character encoding "GB18030". 

``` ruby
> Sys.getlocale()
[1] "LC_COLLATE=Chinese (Simplified)_People's Republic of China.936;LC_CTYPE=Chinese (Simplified)_People's Republic of China.936;LC_MONETARY=Chinese (Simplified)_People's Republic of China.936;LC_NUMERIC=C;LC_TIME=Chinese (Simplified)_People's Republic of China.936"
``` 

What's if we want to change the locale temporary in R? For example, we need to set the encoding for traditional Chinese instead of Simplified Chinese, Can we use the command: <code>Sys.setlocale(category = "LC_ALL", locale = "Chinese (Traditional)_People's Republic of China.936")</code>? Yes, you can. But it doesn't work! Why? That's because '936' is the encoding for simplified chinese characters not for traditional chinese characters. Then you want a list of available locales to determine the legal locale options. Unlike Linux, we can't (or have no easy method to) find such list. But you can find the list by searching **Windows Language strings** on internet. The currently language strings list can be found [here](http://msdn.microsoft.com/en-us/library/39cwe7zf(v=vs.90).aspx). The "Language string" column contains the legal input for setting locale in R. For example, if I want to change current locale to Traditional Chinese, I use command: <code>Sys.setlocale(category = "LC_ALL", locale = "cht")</code>. 

``` ruby
> Sys.setlocale(category = "LC_ALL", locale = "cht")
[1] "LC_COLLATE=Chinese (Traditional)_Taiwan.950;LC_CTYPE=Chinese (Traditional)_Taiwan.950;LC_MONETARY=Chinese (Traditional)_Taiwan.950;LC_NUMERIC=C;LC_TIME=Chinese (Traditional)_Taiwan.950"
``` 
Check the current locale setting, we can see it changed to "LC_CTYPE=Chinese (Traditional)_Taiwan.950;LC_MONETARY=Chinese (Traditional)_Taiwan.950", where codepage '950' is encoding **Big5**. Windows uses only one character encoding for each language, which make the recognization of encoding type easy. 

If I received a script contains Simplified Chinese characters and generated under Windows OS, the encoding must be **GB18030**. To show it correctly, I can set or change the locale of R to "chinese", and import the source code using <code>readLines</code>, or there is a even easy way: using Office Word! If using RStudio as the editor for R, we just need to change the default encoding to "GB18030" in "Tools -> Options -> General -> Default text encoding". Apply the setting, then open the script, it should be displayed correctly. 





