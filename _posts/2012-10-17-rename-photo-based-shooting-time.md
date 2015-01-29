---
layout: post
title: "Rename photos' names according to shooting time in a batch mode."
date: 2012-10-17 18:45
comments: true
categories: R
---

For some reason, you want to know the shooting time of your photos. Generally, shooting time can be found as *Date modified*, if you didn't made a lot of change to your photos. But, we often make change to our photos, such like copy them from a place to another, edit using *Photoshop*, *Picasa*, etc., and then, the *Date modified* of your photo is not the shooting time anymore. What we do?


Luckily, some photos taken by a camera also recorded the shooting time and save it (e.g. as **DateTimeOriginal** ) in the header of the photos, so we can find the shooting time by accessing the header of the photos. 

Now, the problem is how to access the header  information of a photo? I didn't find any R package can do that,  and one reason I believe is  that: there is wonderful open source software can do that well -- **[ImageMagick](http://www.imagemagick.org/script/binary-releases.php#windows)**. *ImageMagick is a software suite to create, edit, compose, or convert bitmap images*. After installing *ImageMagick*, We can using the <code>shell</code> function to execute *ImageMagick* from R.

The following is the R code for renaming photos according to the shooting time in a batch mode. 

``` ruby Rename photo using shooting time
# List all Jpeg photos in a directory,
# you can try some other image formats, like tiff, etc.
file.jpg <- list.files(path = "L:/PhotoDirectory/", 
  pattern = "\\.jpg$", ignore.case = TRUE, full.names = TRUE)
  
for (img in file.jpg) {
  name.1 <- basename(img)
  # Some photo names can not be used in shell mode, so we rename them.
  name.2 <- gsub("\\.?jpg$", ".jpg", ignore.case = TRUE, gsub(" |\\.|-", "", name.1))
  name.3 <- paste(dirname(img), name.2, sep = "/")
  file.rename(img, name.3)
  # Access the header inforamtion of a photo using ImageMagick. 
  header <- shell(paste("identify -verbose", name.3), intern = TRUE)
    # In case the photo didn't record the shooting time,
    if (any(grepl("DateTimeOriginal", header))) {
      date.m <- header[grepl("DateTimeOriginal", header, ignore.case = TRUE)]
      name.4 <- regmatches(date.m, regexec("[[:digit:]]{4}.*", date.m))[[1]]
      name.5 <- gsub(" ", "_", gsub(":", "", name.4))
      name.6 <- paste(dirname(img), paste(name.5, "jpg", sep = "."), sep = "/")
      file.rename(name.3, name.6)
   }
}
```
