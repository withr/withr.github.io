---
layout: post
title: "Build a Compact Raspberry Pi 3 cluster (part 1): assemble"
date: 2016-04-20 14:24
comments: true
---


TensorFlow's official website presented different ways to install [TensorFlow](https://www.tensorflow.org/versions/r0.7/get_started/os_setup.html#configure-tensorflows-canonical-view-of-cuda-libraries). I tried the **pip install**, and after several times fail, finially I can pass the *test your installation*. However, when I try to run the tutorial of [Image Recognition](https://www.tensorflow.org/versions/r0.7/tutorials/image_recognition/index.html), I can't find the **imagenet** directory under *models/image*. After searching for a while, I got the [words](https://github.com/tensorflow/tensorflow/issues/463): **Most of the example sources are no longer in the pip package ...**. So, I decided to install it from source (github). 

### Step 1: install pip.

When installing from source you will build a pip wheel that you then install using pip. 

```
sudo apt-get install python-pip python-dev
```

### Step 2: Clone TensorFlow repository

```
git clone --recurse-submodules https://github.com/tensorflow/tensorflow
```

### Step 3: Install [Bazel](http://bazel.io/)

+ Install JDK 8

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

If encount error message: *GPG error: http://cran.uib.no trusty/ Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 51716619E084DAB9* when running <code>sudo apt-get update</code>, run the following command to replace the key with the one that is displayed in the error message ([more](http://askubuntu.com/questions/20725/gpg-error-the-following-signatures-couldnt-be-verified-because-the-public-key
)). 

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 51716619E084DAB9
```

+ Install required packages

```
sudo apt-get install pkg-config zip g++ zlib1g-dev unzip
```

+ Install Bazel

```
wget https://github.com/bazelbuild/bazel/releases/download/0.2.0/bazel-0.2.0-installer-linux-x86_64.sh
chmod +x bazel-0.2.0-installer-linux-x86_64.sh
./bazel-0.2.0-installer-linux-x86_64.sh --user
export PATH="$PATH:$HOME/bin"
```

### Step 4: Install other dependencies and configure the installation

```
sudo apt-get install python-numpy swig python-dev

cd tensorflow
./configure
```

### Test the installation

Navigate to directory *imagenet* under tensorflow.

```
cd tensorflow/tensorflow/models/image/imagenet/
```

There is a Python script called "classify_image.py", which is used to recoganize image. Run the following command:

```
sudo python classify_image.py
```

If you can see message, like: 

>giant panda, panda, panda bear, coon bear, Ailuropoda melanoleuca (score = 0.89233)
>indri, indris, Indri indri, Indri brevicaudatus (score = 0.00859)
>lesser panda, red panda, panda, bear cat, cat bear, Ailurus fulgens (score = 0.00264)
>custard apple (score = 0.00141)
>earthstar (score = 0.00107)

You have completed the installation!

A [shiny-app](http://188.166.116.72:3838/tf_ImageClassify/) was created according to the codes of [Yuki Katoh](http://opiateforthemass.es/articles/mini-ai-app-using-tensorflow-and-shiny/). 


