openni2_tracker
===============

This work is developed from [here](https://github.com/futureneer/openni2-tracker), which is based on out-dated rosbuild system. I made it support catkin system, and write a more detailed README to make user can build their development environment step by step.

`openni2_tracker` is a ROS Wrapper for the OpenNI2 and NiTE2 Skeleton Tracker. This is designed as a companion package to the `openni2_camera` package (found [here](https://github.com/ros-drivers/openni2_camera)).  Currently, all this node does is publish TF frames of the current tracked user's joint locations.

** Note:  These instructions only been tested with ASUS Xtion Pro Live in ROS-kinetic, Ubuntu 16.04.

### Installation
1. Install OpenNI2:

    ```bash
    sudo apt install git libusb-1.0-0-dev libudev-dev
    sudo apt install openjdk-8-jdk  # for xenial; openjdk-6-jdk for trusty; if not using other java version.
    sudo apt install freeglut3-dev
    cd  # go home
    mkdir -p src; cd src  # create $HOME/src if it doesn't exist; then, enter it
    git clone https://github.com/occipital/OpenNI2.git  # We used to have a fork off 6857677beee08e264fc5aeecb1adf647a7d616ab with working copy of Xtion Pro Live OpenNI2 driver.
    cd OpenNI2
    make -j$(nproc)  # compile
    sudo ln -s $PWD/Bin/x64-Release/libOpenNI2.so /usr/local/lib/  # $PWD should be /yourPathTo/OpenNI2
    sudo ln -s $PWD/Bin/x64-Release/OpenNI2/ /usr/local/lib/  # $PWD should be /yourPathTo/OpenNI2
    sudo ln -s $PWD/Include /usr/local/include/OpenNI2  # $PWD should be /yourPathTo/OpenNI2
    sudo ldconfig
    ```
    
2. Install ASUS Xtion Pro Live OpenNI driver

    ```bash
    sudo apt install libopenni-sensor-primesense0
    ```

2. Install Nite2 from the OpenNI Website [here](http://www.openni.org/files/nite/?count=1&download=http://www.openni.org/wp-content/uploads/2013/10/NiTE-Linux-x64-2.2.tar1.zip).  Be sure to match the version (x86 or x64) with the version of OpenNI2 you installed above.
You will probably need to create a free account.

    Change directories to the NiTE location and install with 
    
    ```bash
    cd NiTE-Linux-x64-2.2
    sudo ./install.sh
    ```
    Try and run one of the examples in `.../NiTE-Linux-x64-2.2/Samples/Bin/NiTE2`.  If these don't work, then something went wrong with your installation.

3. Clone `openni2_tracker` to your ROS workspace.

    ```bash
    git clone git@github.com:futureneer/openni2-tracker.git
    ```

4. Configure CMake
    In the CMakeLists.txt file inside the `openni2_tracker` package, you will need to change the path where CMake will look for OpenNI2 and NiTE2.  These two lines:
    
    ```makefile
    set(OPENNI2_DIR ~/dev/OpenNI2)
    set(NITE2_DIR ~/dev/NiTE-Linux-x64-2.2/)
    ```
    need to point to the root directories of where you extracted or cloned OpenNI2 and NiTE2.

    
5. Make openni2_tracker

    ```bash
    roscd openni2_tracker
    rosmake
    ```
    
6. Set up NiTE2: Right now, NiTE requires that any executables point to a training sample directory at `.../NiTE-Linux-x64-2.2/Samples/Bin/NiTE2`.  If you run the NiTE sample code, this works fine because those examples are in that same directory.  However, to be able to roslaunch or rosrun openni2_tracker from any current directory, I have created a workaround script `setup_nite.bash`.  This script creates a symbolic link of the NiTE2 directory in your .ros directory (the default working directory for roslaunch / rosrun).  You will need to modify this file so that it points to YOUR NiTE2 and .ros locations.  I would be pleased if anyone has a better solution to this.
7. Run openni2_tracker
    
    ```bash
    roslaunch openni2_tracker tracker.launch
    ```

    In the lauch file, you can rename both the tracker name and the tracker's relative frame.  I have included a static publisher that aligns the tracker frame to the world frame, approximately 1.25m off the floor.
    
    ```xml
    <!-- openni2_tracker Launch File -->
    <launch>
    
      <arg name="tracker_name" default="tracker" />
      
      <node name="tracker" output="screen" pkg="openni2_tracker" type="tracker" >
        <param name="tf_prefix" value="$(arg tracker_name)" />
        <param name="relative_frame" value="/$(arg tracker_name)_depth_frame" />
      </node>
    
      <!-- TF Static Transforms to World -->
      <node pkg="tf" type="static_transform_publisher" name="world_to_tracker" args=" 0 0 1.25 1.5707 0 1.7707  /world /$(arg tracker_name)_depth_frame 100"/> 
    
    </launch>
    ```
    
    Currently, this node will broadcast TF frames of the joints of any user being tracked by the tracker.  The frame names are based on the tracker name, currently `/tracker/user_x/joint_name`
