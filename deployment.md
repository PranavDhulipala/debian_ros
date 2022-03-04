# Install dependencies

From the documentation of [bloom](https://bloom.readthedocs.io/en/latest/):

## What is bloom?

Bloom is a release automation tool, designed to make generating platform specific release artifacts from source projects easier. Bloom is designed to work best with catkin projects, but can also accommodate other types of projects.

## How does it work?

Bloom works by importing your upstream source tree into a git repository, where it is manipulated and used to generate build artifacts for different platforms like Debian or Fedora.

First bloom gathers information about your source repository and creates an archive for the version you want to release. Then the archive is imported into the release repository, and the source tree is run through a release track where it is tagged, can be patched, and has platform specific artifacts generated for it.

The individual stages of these release tracks are tagged with git and those tags are used by build infrastructure and deployment systems.

## How do I install bloom?

On Ubuntu the recommended method is to use apt:

```
$ sudo apt-get install python-bloom
```

On other systems you can install bloom via pypi:

```
$ sudo pip install -U bloom
```

Install fakeroot:

```
$ sudo apt-get install fakeroot
```

## Create a source debian

cd into destination of the ros node where the `package.xml` file is located, this is within the src the folder of the catkin workspace.

```
$ cd path/to/the/catkin/package
```

Run the below command to create a debian folder.

```
$ bloom-generate rosdebian --ros-distro noetic
```

Run the below command to build .deb file (https://wiki.debian.org/BuildingAPackage)

```
$ fakeroot debian/rules binary
```

By using the bloom-generate rosdebian ... ROS debian generator, will setup the package to install into /opt/ros/<rosdistro>. If you want it installed into /usr use the debian generator, i.e. replace rosdebian with debian.

## Note 1:

If trying to build a ROS node B that has a dependency on another ROS node A you need to create a .deb of the ROS node A first and create a file like  50-my-packages.list in `/etc/ros/rosdep/sources.list.d/` which needs to contain the link to the location of the .yaml file like `file:///path/to/the/catkin/package/rosdep.yaml`

The rosdep.yaml contains the following to resolve your package (which is already installed locally in this case ROS node A):

```
package_a:
  ubuntu: [ros-noetic-package-a]
```

Then you can build the debian for ROS node B with the same steps as above.

## Note 2:

If your ROS node has a custom message/srv file, then you need to seperate out the ROS node into two different packages wherein you first build the ROS node containing just the message. The following example helps to demonstrate this:

Say you have a ROS node named dynamic_tutorials which has a message of type acs.msg. The contents of the message is a string array something like this `string[6] acs`. You can create a new ros node in your catkin workspace with the name dynamic_tutorials_msgs. This dynamic_tutorials_msgs node just has the message definition of the dynamic_tutorials node which now doesn't have a msg folder, modified CMakeLists and package.xml to reflect the same. 

The new dynamic_tutorials_msgs node will have a CMakeLists like below:

```
cmake_minimum_required(VERSION 2.8.3)
project(dynamic_tutorials_msgs)

find_package(catkin REQUIRED COMPONENTS
  std_msgs
  message_generation
)

add_message_files(
  FILES
  acs.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)


catkin_package(
   CATKIN_DEPENDS message_runtime std_msgs
)
```

The new dynamic_tutorials_msgs node will have a package.xml as follows:

```
<?xml version="1.0"?>
<package format="2">
  <name>dynamic_tutorials_msgs</name>
  <version>1.0.0</version>
  <description>The dynamic_tutorials_msgs package</description>
  <maintainer email="prdhulip@microsoft.com">Pranav Dhulipala</maintainer>
  <license>MIT</license>
  <buildtool_depend>catkin</buildtool_depend>
  <build_depend>message_generation</build_depend>
  <depend>message_runtime</depend>
  <depend>std_msgs</depend>
</package>
```

The message file (acs.msg) in this case will be in a msg folder of the `dynamic_tutorials_msgs` node within the src folder of your catkin workspace. Then build the dynamic_tutorials_msgs node and the subsequent dynamic_tutorials node following the steps in [Note 1](#note-1).

## Next Steps

You can either Install the deb file using a package manager or you can create a repository to host the .deb file such that it can be installed using the apt-get install command. The latter is required if you want to control your deployment through [Device Update for Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub-device-update/device-update-ubuntu-agent#import-the-update).


## References 

https://answers.ros.org/question/173804/generate-deb-from-ros-package/

https://answers.ros.org/question/280213/generate-deb-from-dependent-res-package-locally/#280235

https://answers.ros.org/question/192419/how-to-setup-bloom-to-generate-a-local-deb-outside-of-rosorg-buildfarm/

https://help.launchpad.net/Packaging/PPA

https://wiki.debian.org/BuildingAPackage

https://docs.microsoft.com/en-us/azure/iot-hub-device-update/device-update-ubuntu-agent#import-the-update

https://bloom.readthedocs.io/en/latest/