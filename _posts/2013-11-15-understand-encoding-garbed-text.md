---
layout: post
title: "Understand Encoding Garbed Text of R"
date: 2013-11-15 12:54
comments: true
categories: R
---
I have shown how to set character encoding through configuring system locale for R in previous post. But to find the correct encoding could be an not easy task if you don't know some basic knowledge of encoding. I would like to share my experience of choosing the correct encoding type under Windows.



The encoding problem can be encounted in data files and script files. In data files, we may need to import some data from different source, for example, a data tale in *.txt* or *.csv* format, a data output from *STATA*, *SAS*, *SPSS*, etc. The encoding problem is we don't know which encoding these file used. If we open "Notepad", type some special character (e.g. åø) and save it. We will be asked to determine which format to save, like 'ANSI', 'Unicode', 'UTF-8', etc. 

If the data was saved using 'UTF-8' encoding, while the computer's encoding type is 'ISO-8859-1', some characters like **ø** can not be dispalyed correctly, it will be displayed as <code>Ã¸</code>.  Because **ø** is encoded as "c3 b8" (two bytes: 11000011 10111000, [test](http://easycalculation.com/hex-converter.php)) in [UTF-8](http://www.utf8-chartable.de/unicode-utf8-table.pl), while in [ISO-8859-1](http://www.ic.unicamp.br/~stolfi/EXPORT/www/ISO-8859-1-Encoding.html), "c3 (11000011)" is the code for <code>Ã</code>, and "b8 (10111000)" is the code for <code>¸</code> . So, if a character was displayed as two in a one-byte encoding type, its encoding type must be a multi-byte encoding. 

[UFT-8](http://en.wikipedia.org/wiki/UTF-8) uses vary-length bytes to store characters. If a byte is used for UTF-8 encoding, computer can know the length in byte for that character. E.g. the one byte encoding for **M** is **01001101**, the first "0" tells computer this character stored in one byte; the binary bits for **ø** is **11000011 10111000**, the **11** in the beginning tells computer that character stored in two bytes, and the **10** in the beginning of second byte tells computer that byte is a continuation of the first byte. So, it's not strange that it will be shown as **Ã¸** in "ISO-8859-1", because, "ISO-8859-1" encoding doesn't need a tag to tell computer the length of the storage. 

What if the data was saved in "ISO-8859-1", and my computer encoding is "UTF-8", how the "ø" will be displayed? In "ISO-8859-1", "ø" was encoded as "F8 (11111000)", and "11111000" encoded in "UTF-8" will tell computer that it is the first byte of a 5-bytes characters. If the following four characters' binary code starts not from "10" all, then it can not be displayed correctly, and shown as �. 

For script files. R's default script editor is simple, you can type special characters, like chinese, and send them to console, but the script can not be saved correctly. If you use RStudio as the editor, the encoding format of script can be set in "Tools -> Options -> General", if the script contains no special character, it will be saved directly as the default encoding format, by default its "ISO-8859-1".  So, we can try to figure out the correct encoding by set the default encoding format, and reload the script to see if it can be displayed correctly. 

Note, the correctly showing of script doesn't mean it can work in R console, if they have different encoding format. The encoding configured in RStudio works only for script. There two functions are very useful: <code>iconv</code> and <code>Encoding</code>. *Encoding* tells the encoding of a string, if the encoding information missed, then R takes it as same as the encoding of R's locale. *iconv* can convert the encoding of a string from one to another. Note, even *iconv* can convert the encoding correctly, the strings still need correct locale encoding to show them correctly. 






