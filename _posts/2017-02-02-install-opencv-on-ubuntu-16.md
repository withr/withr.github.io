---
layout: post
title: "Install opencv 3.2.0 on Ubuntu 16.04 for python 2.7"
date: 2017-02-02 09:47
comments: true
categories: opencv
---


If you want to play computer vision, [opencv](http://opencv.org/) is the most popular tool. However, installing opencv is not like install a normal software, it's more complete, and you may need hours to finish the installation. [Adrian Rosebrock ](http://www.pyimagesearch.com/2016/10/24/ubuntu-16-04-how-to-install-opencv/) wrote a excellent tutorial on how to install opencv 3.1.0 on Ubuntu 16.04 with explaination for each step. I didn't success by following the steps, however, I successed with installing a newer version: 3.2.0. This post presents three code blocks to help you install opencv with a couple of copy&paste.


## Step 1: update & upgrade Linux system, install libraries/tools, download opencv/opencv_contrib 

~~~~
echo -e "\nUpdate and Upgrade system..." && sleep 3
sudo apt update -y && sudo apt upgrade -y

echo -e "\nInstall developer tools..." && sleep 3
sudo apt install -y build-essential cmake pkg-config

echo -e "\nInstall tools for image process..." && sleep 3
sudo apt install -y libjpeg8-dev libtiff5-dev libjasper-dev libpng12-dev

echo -e "\nInstall tools for video process..." && sleep 3
sudo apt install -y libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev

echo -e "\nInstall tools for GUI & optimization..." && sleep 3
sudo apt install -y libgtk-3-dev libatlas-base-dev gfortran

echo -e "\nInstall python development library & pip..." && sleep 3
sudo apt install -y python2.7-dev python-pip

echo "\nDownloading opencv & opencv_contrib version:..." && sleep 3
cd && wget -O opencv.zip https://github.com/opencv/opencv/archive/3.2.0.zip && unzip opencv.zip 
cd && wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/3.2.0.zip && unzip opencv_contrib.zip 

~~~~

## Step 2: In stall & setup python virtual environment

~~~~
echo -e "\nSetup virtual environment..." && sleep 3
if [[ $(grep 'virtualenvwrapper' ~/.bashrc) ]]; then
  echo ".bashrc has been updated!"
else  
    sudo pip install virtualenv virtualenvwrapper
    echo -e "\n# virtualenv and virtualenvwrapper" >> ~/.bashrc
    echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.bashrc
    echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc
    source ~/.bashrc
fi
test -d "~/.virtualenvs/cv" && mkvirtualenv cv -p python2 || workon cv
~~~~

## Step 3: Install numpy, configure, compile & install opencv

~~~~
echo -e "\nInstall opencv ..." && sleep 3
pip install numpy
cd ~/opencv-3.2.0/
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D INSTALL_C_EXAMPLES=OFF \
    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-3.2.0/modules \
    -D PYTHON_EXECUTABLE=~/.virtualenvs/cv/bin/python \
    -D BUILD_EXAMPLES=ON ..
   
sudo make -j4
sudo make install

cd ~/.virtualenvs/cv/lib/python2.7/site-packages/
sudo ln -s /usr/local/lib/python2.7/site-packages/cv2.so cv2.so
~~~~

