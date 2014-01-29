rim-bmw
=======

Rosinstall checkout for the RIM-BMW project.

# Installation Procedure

You can run this procedure once you've installed ROS Hydro and before you check
this repo out.

Install these dependencies if you haven't:
```
sudo apt-get update
sudo apt-get install ros-hydro-urdfdom-py ros-hydro-cmake-modules ros-hydro-moveit-full
```

Checkout the rosinstalls from the `rim-bmw` repo and then checkout the branches into
their respective workspaces.
```
cd
wstool init
wstool set dev --git https://github.com/gt-ros-pkg/rim-bmw.git -v hydro
wstool update dev
cd dev/fuerte_ws/
wstool up
cd ../hydro_ws/src
wstool up
```

Add the following lines to your ~/.bashrc 
```
export ROS_PACKAGE_PATH=$HOME/dev/hydro_ws:$ROS_PACKAGE_PATH
source $HOME/dev/hydro_ws/devel/setup.bash
```

Then open a new workspace in the hydro branch and make everything.
```
source ~/.bashrc
cd ~/dev/hydro_ws/src
catkin_init_workspace 
cd ..
catkin_make
```

Note that although we have checked out the Fuerte branch, we will not compile it.
This is because Fuerte code is incompatable with Hydro code until properly catkinized.

If you want to make the hydro_ws, you have to go to the directory and run `catkin_make`.
If you want to make this more convenient, you can add this command to your bashrc:
```
alias hmake='(export LASTWD=`pwd`; cd $HOME/dev/hydro_ws; catkin_make; cd $LASTWD)'
```

# Package creation
If we want to make a package, you need to put it in `dev/hydro_ws/src`

```
cd ~/dev/hydro_ws/src
catkin_create_pkg test_pkg roscpp
cd test_pkg
vim src/test_pkg_node.cpp
```

Here's some example code:
```
#include <ros/ros.h>
#include <std_msgs/Int32.h>

class Loop
{
public:
  Loop(ros::NodeHandle& nh, int32_t in_local);
  void loopInt(const std_msgs::Int32::ConstPtr& msg);
private:
  ros::Subscriber sub;
  int32_t local_num;
};

Loop::Loop(ros::NodeHandle& nh, int32_t in_local) : 
  local_num(in_local)
{
  sub = nh.subscribe("loop_int", 1, &Loop::loopInt, this);
}

void Loop::loopInt(const std_msgs::Int32::ConstPtr& msg)
{
  ros::Rate r(1.0);
  int32_t i = 1;
  while(ros::ok()) {

    ROS_INFO("Sum: %d + %d = %d", i, local_num, i+local_num);
    ROS_INFO("Printing msg loop: %d/%d", i, msg->data);

    if(i == msg->data)
      break;

    r.sleep();
    i++;
  }
  ROS_INFO("Done.");
}

int main(int argc, char** argv)
{
  ros::init(argc, argv, "test");
  ros::NodeHandle nh;
  Loop loop(nh, atoi(argv[1]));
  ros::spin();
  return 0;
}
```

You can compile by editing the `CMakeLists.txt` file.
Find these lines and uncomment them:

```
add_executable(test_pkg_node src/test_pkg_node.cpp)

target_link_libraries(test_pkg_node
  ${catkin_LIBRARIES}
)
```

Then call `hmake` or `catkin_make` in the Hydro root dir.
Run a roscore in one window, then run the node using `rosrun`:

```
roscore # window 1
rosrun test_pkg test_pkg_node 5 # window 2
```

In another tab, communicate with the node by sending the number 6 in a message:
```
rostopic pub /loop_int std_msgs/Int32 "data: 6" # window 3
```

Look at the terminal in the previous window.  Try changing the number.
Do Ctrl-C on the previous window when you're done.


