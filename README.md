# desktop_setup
This repository documents the setup process of a brand new desktop with (GTX 1080 Ti, Ubuntu 16.04, Python 2, Python 3, CUDA 8.0, cuDNN 6.0, Tensorflow 1.4, Kera) for training deep neural network. The process referred to many other websites, blogs, forums, and repositories; and the they may NOT be all cited. Contact me if you feel your source should be cited. 

## 1. Install GTX 1080 Ti driver
1. Go to Nvidia website to find out which version is needed. 

2. Add repository and install the correct version of driver
```
sudo add-apt-repository ppa:graphics-drivers
sudo apt-get update
sudo apt-get install nvidia-384
```

3. Restart

4. Verify installation
```
lsmod | grep nvidia
```
and you should see the nvidia driver in the output. Otherwise the installation may have failed. Also checking Nvidia settings
```
nvidia-settings
```

## 2. Partition disk
Partition the disk such that the system takes 240 GB, home folder takes 760 GB, and data folder takes 1 TB. This partition process can be easily done during installation process. The following documents how to do the partition after installation. Please refer to this [link](https://www.howtogeek.com/116742/how-to-create-a-separate-home-partition-after-installing-ubuntu/) for more detailed instructions. 

1. Create new partitions using live USB

2. Copy home files to the new partition
```
sudo cp -Rp /home/* /mount/location
```
Double check all files moved to the new partition

3. Locate the new partitions UUID
```
sudo blkid
```

4. Modify the fstab file
```
sudo cp /etc/fstab /etc/fstab.backup
gksu gedit /etc/fstab
```
Add the following line
```
UUID=<uuid> /home ext4  nodev,nosuid 0 2
```
Save and exit

5. To avoid confusion, move the current home directory to old and create a new empty home directory for mounting the new partition
```
cd /&& sudo mv /home /home_old && sudo mkdir /home
```

6. Restart and check all mounts are good
```
sudo shutdown -r now
```

## 3. (Incorrect, jump to step 5) Install CUDA 9.1
As date of Dec 25th, 2017, TensorFlow does not yet support CUDA 9.1 (up to CUDA 8.0 and cuDNN 6.0). Did not find this earlier. So installed CUDA 9.1 and then remove it.

1. Update the system 
```
sudo apt-get update && sudo apt-get upgrade
```

2. Update kernel
```
sudo apt-get install linux-headers-$(uname -r)
```

3. Download CUDA Toolkit 9.1, deb version

4. Install CUDA Toolkit
```
sudo dpkg -i file.deb
```

5. Install key
```
sudo apt-key add /var/cuda-repo-9-1-local/7fa2af80.pub
```

6. Update and install
```
sudo apt-get update
sudo apt-get install cuda
```

## 4. (If performed step 3) Remove CUDA 9.1
The first line should removed both cuda and cuda9.1 foder in /usr/local.
```
sudo apt-get --purge remove cuda	
sudo apt autoremove
sudo apt-get clean
```

## 5. Install CUDA 8.0
1. Go to Nvidia website to find out which version and download the deb file. 

2. Install CUDA 8.0 GA2
```
sudo dpkg -i cuda-repo-ubuntu1604-8-0-local-ga2-8.0.61-1_amd.deb
sudo apt-get update
sudo apt-get install cuda-8-0
```

3. Install cuda patch 2.
```
sudo dpkg -i cuda-repo-ubuntu1604-8-0-local-cublas-performance-update_8.0.61-1_amd64.deb
sudo apt-get update
sudo apt-get install cuda-8-0
```
4. Add CUDA to the environment variables
```
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' 
source ~/.bashrc
```

5. Verify the installed version
```
nvcc -V
```

6. Test CUDA installation
Install samples in the CUDA directory. Then compile and check if passes.
```
/usr/local/cuda/bin/cuda-install-samples-8.0.sh ~/cuda-samples
cd ~/cuda-samples/NVIDIA*Samples
make -j $(($(nproc) + 1))
bin/x86_64/linux/release/deviceQuery
```

## 6. Install cuDNN 6.0
1. Download cuDNN 6.0 from Nvidia website. 

2. Install 
```
tar xvf cudnn-8.0-linux-x64-v6.0.tgz
sudo cp -P cuda/lib64/* /usr/local/cuda/lib64/
sudo cp cuda/include/* /usr/local/cuda/include/
```

3. Update the paths for CUDA library and executables
```
echo 'export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"' >> ~/.bashrc
echo 'export CUDA_HOME=/usr/local/cuda' >> ~/.bashrc
echo 'export PATH="/usr/local/cuda/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 7. Install dependencies for deeplearning frameworks
```
sudo apt-get update
sudo apt-get install -y libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev protobuf-compiler libopencv-dev
- install python 2 and python 3
sudo apt-get install -y --no-install-recommends libboost-all-dev doxygen
sudo apt-get install -y libgflags-dev libgoogle-glog-dev liblmdb-dev libblas-dev 
sudo apt-get install -y libatlas-base-dev libopenblas-dev libgphoto2-dev libeigen3-dev libhdf5-dev 
 
sudo apt-get install -y python-dev python-pip python-nose python-numpy python-scipy
sudo apt-get install -y python3-dev python3-pip python3-nose python3-numpy python3-scipy
sudo apt-get install -y python-matplotlib python3-matplotlib
```

## 8. Install tensorflow and Keras
1. Setup python 2 environment

Create a cirtual environment for python 2
```
mkvirtualenv vir-py2 -p python2
```

Activate the virtual environment
```
workon vir-py2
```

Install packages
```
pip install numpy scipy matplotlib scikit-image scikit-learn ipython protobuf jupyter
```

With CUDA installed, install tensorflow and keras
```
pip install tensorflow-gpu
pip install keras
pip install dlib
deactivate
```

2. Setup python 3 environment

Create a cirtual environment for python 3
```
mkvirtualenv vir-py3 -p python3
```

Activate the virtual environment
```
workon vir-py3
```

Install packages
```
pip install numpy scipy matplotlib scikit-image scikit-learn ipython protobuf jupyter
```

With CUDA installed, install tensorflow and keras
```
pip install tensorflow-gpu
pip install keras
pip install dlib
deactivate
```

## 9. Install OpenCV 3.3
1. Install dependencies
```
sudo apt-get remove x264 libx264-dev
sudo apt-get install -y checkinstall yasm
sudo apt-get install -y libjpeg8-dev libjasper-dev libpng12-dev
```

For ubuntu 16.04
```
sudo apt-get install -y libtiff5-dev
sudo apt-get install -y libavcodec-dev libavformat-dev libswscale-dev libdc1394-22-dev
 
sudo apt-get install -y libxine2-dev libv4l-dev
sudo apt-get install -y libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev
sudo apt-get install -y libqt4-dev libgtk2.0-dev libtbb-dev
sudo apt-get install -y libfaac-dev libmp3lame-dev libtheora-dev
sudo apt-get install -y libvorbis-dev libxvidcore-dev
sudo apt-get install -y libopencore-amrnb-dev libopencore-amrwb-dev
sudo apt-get install -y x264 v4l-utils
```

2. Download OpenCV and OpenCV-contrib
```
- Download OpenCV and OpenCV-contrib
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.3.0
cd ..

git clone https://github.com/opencv/opencv_contrib.git
cd opencv_contrib
git checkout 3.3.0
cd ..
```

3. Configure and generate the MakeFile
```
cd opencv
mkdir build
cd build
```

Remove the line WITH_CUDA=ON if you dont have CUDA in your system
```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D INSTALL_C_EXAMPLES=ON \
      -D INSTALL_PYTHON_EXAMPLES=ON \
      -D WITH_TBB=ON \
      -D WITH_V4L=ON \
      -D WITH_QT=ON \
      -D WITH_OPENGL=ON \
      -D WITH_CUDA=ON \
      -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules \
      -D BUILD_EXAMPLES=ON ..
```

4. Compile and install (takes about one hour on i3-8350k 4 core 4.0GHz)
```
make -j4 
sudo make install
sudo sh -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
```

5. Link OpenCV to the virtual environment
```
find /usr/local/lib/ -type f -name "cv2*.so"
```

Should get 
```
/usr/local/lib/python3.5/dist-packages/cv2.cpython-35m-x86_64-linux-gnu.so
/usr/local/lib/python2.7/dist-packages/cv2.so
```

Then create symlinks. For python 2
```
cd ~/.virtualenvs/vir-py2/lib/python2.7/site-packages
ln -s /usr/local/lib/python2.7/dist-packages/cv2.so cv2.so
```

For Python 3
```
cd ~/.virtualenvs/vir-py3/lib/python3.5/site-packages
ln -s /usr/local/lib/python3.5/dist-packages/cv2.cpython-35m-x86_64-linux-gnu.so cv2.so
```

6. Check OpenCV installation
```
workon virtual-py2
python
import cv2
cv2.__version__
```

## 11. Ubuntu with two monitors with different resolutions.
I have one 4k monitor (3840x2160) and one 1080p monitor (1920x1080). Using the built in Ubuntu display settings, I can scale two monitors for the labels to appear at the right sizes. However, when moving a window from 4k to 1080p, the window goes giant. I still do not have a good solution yet after searching a while. The following documents the trials, and possible commands that may help. The trials are primarily based on two links, [two_dpi](https://askubuntu.com/questions/393400/is-it-possible-to-have-two-different-dpi-configurations-for-two-different-screen), [external-monitor](https://unix.stackexchange.com/questions/47749/cannot-add-new-mode-in-xrandr-for-external-monitor).

1. A few prerequisites.

- Use the following commend to find out the current display settings
```
xrandr
```
The number with ```+``` denotes the preferred setting while with ```*``` denotes the current setting. Use 

```
xrandr --help
```
to learn more about ```xrandr```, which will be the primary tool used for setting monitors.

- Use the following command to find teh current dpi settings
```
xrdb -query
```

2. Try with ```xrandr``` scale.
I want to scale up the 1080p display by a factor 2x2, hence 1920x1080 to 3840x2160, and place it on the left of the 4k display. The 1080p monitor is HDMI-0, and 4k monitor is HDMI-1.
Use the following command
```
xrandr --output HDMI-1 --scale 1x1 --fb 7680x2160 --pos 3840x0 && xrandr --output HDMI-0 --scale 2x2 --mode 1920x1080 --fb 7680x2160 --pos 0x0
```
The canvas size is ```--fb 7680x2160```, where the first number is the total width after scalling: 1920x2 + 3840, and the second number is the total height: max(1080x2, 2160).
The position ```--pos <x>x<y>``` denotes the position of the screen on the canvas where the left top corner is 0x0. The positions are all left top corner of the display.

Ideally, the above command should solve all problems. However, the 1080p display looks good for one moment, but then turns black and is just unstable. The cause might be it might still be trying to refress at 60 Hz, which the HDMI cable can not handle. The next step is try to reduce the frame rate to 30 to check if 1080p monitor can be stable

3. Try to set frame rate at 30 Hz. 

First use ```xrandr``` to check if the frame rate 30 is supported by 1080p at 1920x1080. Unfortunately, it is not. So need to add a new mode. Use ```cvt 1920 1080 30``` or ```gtf 1920 1080 30``` to get new mode parameters. 
Then create and add new mode
```
xrandr --newmode "1920x1080_30.00" <the parameters>
xrandr --verbose --addmode HDMI-0 "1920x1080_30.00"
xrandr --output HDMI-0 --mode "1920x1080_30.00"
```

Unfortunately, the ```--addmode``` gives a ```BadMatch``` error. To remove a mode
```
xrandr --verbose --rmmode "1920x1080_30.00"
```

So setting 1080p framerate at 30 Hz failed. The lowest it can go is 50 Hz, which can be done here
```
xrandr --output HDMI-0 --mode 1920x1080 --rate 50.00
```
You may want to switch to a different mode and framerate first and switch back to the above resolution and framerate, since originally directly using the above command did not take any effect. Check ```*``` location in the output of ```xrandr``` to identify the current mode and framerate. 

4. Summary of ```xrandr```.
Also tried to scale 1080p by 1.5x1.5 instead of 2x2, and the 1080p monitor was stable. However, regardless of all scaling factors (tried 1.1x1.1, up to 2x2), the scaling all result in a blurry display, which is harder to read. 

5. (Temperary solution) Use Genome Tweak tools and Settings/display.

Download the genome tweak tools. The current configuration is scale 4k by 1.25 and 1080p by 0.75 in System All settings/Display/Scale for menu and title bars. And then in tweak tools, change the Fonts/Scalling Factor to 1.25. Buy another 4k monitor to replace the 1080p monitor should solve the probelm. 

## 12. Install other software
notepadqq, pycharm, VS code.












  
  
  
  
  
