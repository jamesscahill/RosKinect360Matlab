# Getting the Kinect 360 Camera to connect to Ros-Melodic and Interface with Matlab

Unfortunately, getting the Kinect 360 camera to work with ROS melodic was not so straightforward as I'd hoped, much of the relevant software/drivers were not compatible with the latest Ros release. Hence I am writing this as a guide if I ever need to do it again.

## Before you start
You will need the following:
### Hardware
1. Kinect 360 Camera
2. Kinect 360 USB & Power adaptor (as the kinect uses a proprietary plug & requires 12v to operate)
### Software
1. Ubuntu 18.04 LTS - Installation tutorial found [Here](https://ubuntu.com/tutorials/tutorial-create-a-usb-stick-on-ubuntu#1-overview)
2. ROS-Melodic - Installation steps found [Here](http://wiki.ros.org/melodic/Installation/Ubuntu)
3. Python & Doxygen

To install the required python packages use the following code in your terminal:

    sudo apt-get install git build-essential python libusb-1.0-0-dev freeglut3-dev openjdk-8-jdk
    
Doxygen can be installed using the following:

    sudo apt-get install doxygen graphviz mono-complete
    
### Alternative Software/method
You can also use the following:
1. libfreenect
2. rgbd_launch
3. freenect_stack

Also documented below, I ended up using these packages as they proved more reliable than the first method described.

*Note: I have moved on the the alternative method as I found this was too unreliable*
## Alternative Method
This method uses the freenect package, rather than the openni package
### Install libfreenect
Use the following:

```
cd Documents

sudo apt-get install git-core cmake freeglut3-dev pkg-config build-essential libxmu-dev libxi-dev libusb-1.0-0-dev

git clone git://github.com/OpenKinect/libfreenect.git
cd libfreenect
mkdir build
cd build

cmake .. -DCMAKE_INSTALL_RPATH:STRING="/usr/local/bin;/usr/local/lib" -DBUILD_REDIST_PACKAGE=OFF

make
sudo make install
sudo ldconfig /usr/local/lib64/
```
Now we need to make sure you can access the feed without root access, use the following:
(use your username instead of $user)
```
sudo adduser $user video

sudo gedit /etc/udev/rules.d/51-kinect.rules

# ATTR{product}=="Xbox NUI Motor"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02b0", MODE="0666"
# ATTR{product}=="Xbox NUI Audio"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02ad", MODE="0666"
# ATTR{product}=="Xbox NUI Camera"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02ae", MODE="0666"
# ATTR{product}=="Xbox NUI Motor"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02c2", MODE="0666"
# ATTR{product}=="Xbox NUI Motor"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02be", MODE="0666"
# ATTR{product}=="Xbox NUI Motor"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02bf", MODE="0666"

```

Now we can see the video feed using the following:

    freenect-glview

### Install rgbd_launch
rgbd_launch contains the generic launch files needed by either the openni or freenect packages. We can install it using the following:

```
cd ~/catkin_ws
git clone  https://github.com/ros-drivers/rgbd_launch.git
cd ..
catkin_make
```

### Install freenect_stack

```
cd ~/catkin_ws
git clone  https://github.com/BlueWhaleRobot/freenect_stack.git
cd ..
catkin_make
```
Now that this is done, we can test the connection using:

    roslaunch freenect_launch freenect-xyz.launch

we can now use rviz to visualise the incoming data stream
    rviz




## Installing required Packages (not needed if you followed the above)

For the kinect to work properly you will need OpenNI, Kinetic OpenNI and NITE.

I installed these seperately from the catkin workspace, Making a new directory called kinect

    cd ~/
    mkdir kinect
    
### Installing OpenNI

```bash
cd ~/kinect
git clone https://github.com/OpenNI/OpenNI.git
cd OpenNI
git checkout Unstable-1.5.4.0
cd Platform/Linux/CreateRedist
chmod +x RedistMaker
./RedistMaker
cd ../Redist/OpenNI-Bin-Dev-Linux-[xxx]  
```
Where `[xxx]` is your architecture (x86/x64) and particular OpenNI release.

    sudo ./install.sh

Now that OpenNI is installed, we require an additional module to get OpenNI to talk to the Kinect, use the following code:

```bash
cd ~/kinect
git clone https://github.com/avin2/SensorKinect
cd SensorKinect
cd Platform/Linux/CreateRedist
chmod +x RedistMaker
./RedistMaker
cd ../Redist/Sensor-Bin-Linux-[xxx] 
```
Again, where `[xxx]` is your architecture (x86/x64) and particular OpenNI release.

    chmod +x install.sh
    sudo ./install.sh

**DO NOT RUN THIS MORE THAN ONCE** - it will cause problems. If you do encounter problems navigate to your `/Sensor-Bin-Linux-[xxx]` folder and run the following:
    
    sudo ./install.sh -u
    
This will uninstall it, then try again.

### Installing Kinetic OpenNI
Run the following lines:

    sudo apt-get install ros-melodic-openni*

### Install NITE

NITE is now owned by apple, so it was a bit of a pain finding a version to use, I found mine [Here](https://github.com/arnaud-ramey/NITE-Bin-Dev-Linux-v1.5.2.23)
```bash
cd ~/kinect
git clone https://github.com/arnaud-ramey/NITE-Bin-Dev-Linux-v1.5.2.23
cd NITE-Bin-Dev-Linux-v1.5.2.23
./install.sh
```

Now that this is done we can now run the code using the following in subsequent terminals:
```
roscore
roslaunch openni_launch openni.launch
rosrun openni_tracker openni_tracker
```

*Note: I have moved on the the alternative method as I found this was too unreliable*

## Interfacing With Matlab

Make sure that you have the following addons installed:
1. ROS Toolbox
2. Image Processing Toolbox

If not already running, start the freenect process in a terminal with

    roslaunch freenect_launch freenect-xyz.launch

Open Matlab, and use the following test code to test the connection with ros, and that data could be pulled from the ros node

```Matlab
close all;
clear all;
clc;


rosinit;

rosservice list;

subDepth = rossubscriber("/camera/depth/points");

subImage = rossubscriber("/camera/rgb/image_raw");

depthCloud=subDepth.LatestMessage;

 DlgH = figure;
 H= uicontrol('Style','PushButton','String','Break','Callback','delete(gcbf)');
  while(ishandle(H))
      %hold on
      figure(1)
     subDepth= rossubscriber("/camera/depth/points");
     pause(0.5);
      scatter3(subDepth.LatestMessage);
      figure(2)
      image = readImage(subImage.LatestMessage);
      imshow(image);
    drawnow()
 end

rosshutdown;
```
You should see two figures, one of the point cloud scatter plot, and the other of the rgb image captured of the scene.
