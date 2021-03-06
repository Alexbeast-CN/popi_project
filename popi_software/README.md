# popi_software

<img align="center" src="https://i.imgur.com/ua4r1Gw.png" width="100%"/>

*The Gazebo world used here isn't available yet in the repository, as we encountered some problems with it. It comes from this [GitHub project](https://github.com/yukkysaito/vehicle_sim) if you want to check it out.*
<br>
<br>

You will find here all the source code used on POPI. The code is based on [ROS]. You can run it without the robot, develop your own code and try it in a [Gazebo] simulation. Please read below to get all the information needed to install and run our code on a virtual POPI. If you build your own robot and need the instructions to configure the network and make it work, please refer to the user manual in the [popi_reports] folder.

#### What is available
:heavy_check_mark: Trajectory generation and optimization based on [Towr]  
:heavy_check_mark: Easy recording with [rqt_bag]  
:heavy_check_mark: Actuators' control based on [ROS control boilerplate]  
:heavy_check_mark: Available simulation on Gazebo  
<br>
<br>

<p align="center">
  <a href="#soft">POPI software</a> •
  <a href="#install-dependecies">Install dependencies</a> •
  <a href="#structure">Folder structure</a> •
  <a href="#run">Run</a> •
  <a href="#contribute">Contribute</a> •
  <a href="#team">Meet the team</a>
</p>

<p align="center">
	<img src="https://i.imgur.com/zi1gDp3.gif" />
</p>

## <a name="soft"></a> POPI software
As of now, POPI doesn't have any on-the-fly trajectory computation. Hence it is important to understand there is a two-step operating principle:
   * Generating a new trajectory and making sure it works fine on the virtual model
   * Sending the trajectory on POPI


The brains of POPI are split between 3 single-board computers (one Raspberry Pi 3B+ and two BeagleBone Black) and one (offboard) user computer. It can seem daunting at first, but it allows for a more distributed work, and endows POPI with a computational power that is sufficient even for its future developments. Moreover the source code is entirely based on [ROS], which makes it really easy to run code across multiple machines.

Each computer has a special use, as shown on the diagram below.
<p align="center">
  <img src="https://i.imgur.com/3t14bqz.png" /> 
</p>

If you build your own POPI, you will need to download the [flash images] for the 3 SD cards. The source code available here is actually the one running on the user computer when making POPI walk, and it is also the one needed to create and test new trajectories.
<br>
<br> 

## <a name="install-dependecies"></a> Install dependencies
  
The environment we used is Ubuntu 18.04 with [ROS Melodic].
<br>

* First, install [ROS]:
   ```bash
   sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
   sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
   sudo apt-get update
   sudo apt-get install ros-melodic-desktop-full
   ```
	Do the following to get the latest version of Gazebo 9 (we used the 9.12.0):
   ```bash
   sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
   wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
   ```
  
	Then update your packages:
   ```bash
   sudo apt-get update
   sudo apt-get upgrade
   ```
   Finally, install the other dependencies:
   ```bash
   sudo apt-get install ros-melodic-ros-control ros-melodic-ros-controllers ros-melodic-robot-controllers ros-melodic-robot-state-publisher ros-melodic-gazebo-ros-pkgs ros-melodic-gazebo-ros-control ros-melodic-rosparam-shortcuts ros-melodic-joy git cmake libeigen3-dev coinor-libipopt-dev libncurses5-dev libgflags-dev libboost-all-dev xterm
   ```
   If you encounter any problem or would like any details about these dependencies, you will find [here](https://github.com/popi-mkx3/popi_project/blob/master/popi_software/CONFIGURATION.md) a list of the versions of the packages we had installed, as well as links to their respective websites.
<br>

* Create the workspace:
   ```bash
   mkdir -p ~/catkin_ws/src
   cd ~/catkin_ws/src
   ```
* Either download the whole repository:
   ```bash
   git clone https://github.com/popi-mkx3/popi_project.git
   ```
* Or only the popi_software folder:
   ```bash
   git init
   git remode add -f origin https://github.com/popi-mkx3/popi_project.git
   git config core.sparseCheckout true
   echo "popi_software" >> .git/info/sparse-checkout
   git pull origin master
   ```
<br>

* Build the workspace:
   ```bash
   cd ~/catkin_ws
   source /opt/ros/melodic/setup.bash
   catkin_make_isolated
   ```
<br>

* Configuration:  
   Copy the following command to add these four lines to your `~/.bashrc`: 
   ```bash
   echo "source /opt/ros/melodic/setup.bash
   source ~/catkin_ws/devel_isolated/setup.bash
   #export ROS_IP=192.168.7.4
   #export ROS_MASTER_URI=http://192.168.7.1:11311" >> ~/.bashrc

   ```
	Leave the two last lines commented if you're not building your own robot.
<br>
<br>

## <a name="structure"></a> Folder structure
Let's break the folder structure down so that you better understand how it works.
<p align="center">
  <img src="https://i.imgur.com/Zt2IFjE.png" /> 
</p>

Here is a short explanation of what lies in each folder:
* [popi_code] includes source code automatically generated from our URDF model with [RobCoGen], using first the [urdf2robcogen] tool to get a .kindsl format. We actually don't use this generated code for now but thought this might be of use at some point.
* [popi_control] is based on [ROS control boilerplate] and includes ROS-type controllers. This is what allows us to control our twelve actuators, in the simulation as well as with the real robot. In the latter case, a hardware interface is needed. It is also included in this package, but the node actually runs on the Raspberry Pi when making POPI walk.
* [popi_description] includes the [URDF] model of POPI and the STL meshes. In fact we first generated the URDF from our mechanical design thanks to the [SolidWorks to URDF exporter], and then made modifications, additions and simplifications to get the current [xacro] model.
* [popi_gazebo] includes the files needed to define the physics of the world where we spawn our virtual POPI. It also includes a [SDF] model of POPI. Although, please note that we created this SDF model early in the project before switching to URDF, which is needed to work with ROS.
* [popi_robot] includes the definition of messages used and the main launch files.
* [ifopt] is a C++ Interface to Nonlinear Programming Solvers used by Towr.
* [towr] is the package used to generate trajectories. It was developed by [Alexander W. Winkler], we only added there our robot model.
* [xpp] is a package used by Towr to allow the visualization of trajectories on RViz. This is where is included the inverse kinematic model of our robot to compute the required joint angles to get the foot in the desired position.
* actionneurs includes ROS nodes sending commands to the actuators (only in the flash images).
* capteurs includes ROS nodes reading the sensors' values (only in the flash images).
<br>
<br>

## Run
* Run Towr to generate a new trajectory:
   ```bash
   roslaunch towr_ros towr_ros_popi.launch
   ```
   The user interface is pretty easy to understand. The constraints used by Towr can make a big difference in the generated trajectories, and to change them you will need to get your hands in the code. The maximum deviation of the feet from the nominal position, represented by the blue boxes, can be changed in *popi_software/towr/towr/towr/include/towr/models/examples/popi_model.h*. We would still have to work with Towr to use the constraints efficiently. Moreover our inverse kinematic model is unfortunately restricted with quite a lot of singularities. To get rid of them, we lowered the maximum deviation parameters, but this probably isn't the best way around.
   We still generated some good trajectories we were able to try out on our real robot and which worked fine !
<br>

* Save the trajectory as a rosbag file:
   ```bash
   cd ~/catkin_ws/src/popi_project/popi_software/popi/popi_robot/trajectories/bags
   rosrun rqt_bag rqt_bag
   ```
   Click the red recording button, select the 12 topics called */popi/.../command*, then click *Record*, give a name to your bag file (like *test.bag*), then go to the towr user interface and click *v* to replay the last calculated trajectory.
   The rqt_bag window should look like this:
   <p align="center">
      <img src="https://i.imgur.com/ny4yXxI.png" /> 
   </p>
   Note the X number, as it may be useful to understand the timestamps of your bag file if you want to work on your commands as a CSV file. When this is done, close the rqt_bag window.
<br>

* To convert your data fril rosbag to CSV, you can do the following:   
   ```bash
   cd ~/catkin_ws/src/popi_project/popi_software/popi/popi_robot/trajectories/bags
   rostopic echo -b test.bag -p /popi/aileAVD_eff_position_controller/command > ../csv/aileAVD.csv
   rostopic echo -b test.bag -p /popi/aileAVG_eff_position_controller/command > ../csv/aileAVG.csv
   rostopic echo -b test.bag -p /popi/aileARD_eff_position_controller/command > ../csv/aileARD.csv
   rostopic echo -b test.bag -p /popi/aileARG_eff_position_controller/command > ../csv/aileARG.csv
   rostopic echo -b test.bag -p /popi/epauleAVD_eff_position_controller/command > ../csv/epauleAVD.csv
   rostopic echo -b test.bag -p /popi/epauleAVG_eff_position_controller/command > ../csv/epauleAVG.csv
   rostopic echo -b test.bag -p /popi/epauleARD_eff_position_controller/command > ../csv/epauleARD.csv
   rostopic echo -b test.bag -p /popi/epauleARG_eff_position_controller/command > ../csv/epauleARG.csv
   rostopic echo -b test.bag -p /popi/coudeAVD_eff_position_controller/command > ../csv/coudeAVD.csv
   rostopic echo -b test.bag -p /popi/coudeAVG_eff_position_controller/command > ../csv/coudeAVG.csv
   rostopic echo -b test.bag -p /popi/coudeARD_eff_position_controller/command > ../csv/coudeARD.csv
   rostopic echo -b test.bag -p /popi/coudeARG_eff_position_controller/command > ../csv/coudeARG.csv
   ```
   This isn't optimal as it creates 12 differents CSV files you then have to concatenate to work on the whole data in one file. We will release a short script to make it easier. To get your timestamps into actual seconds past after your hit the *Record* button, use the following formula : t = (%time / 10^9) − X.
<br>

* If you changed your data on Excel and want to get the modified .csv back into rosbag, make sure first the columns headings are the following : %time, aileAVD, aileAVG, aileARD, aileARG, epauleAVD,
epauleAVG, epauleARD, epauleARG, coudeAVD, coudeAVG, coudeARD, coudeARG. This means the columns need to be in that order, so be careful when you concatenate your 12 CSV files.
You can then save your file, for example as *modified_test.csv*. Make sure the CSV format is the same as the one it was in the previous files. Finally, you can run the following commands:
   ```bash
   cd ~/catkin_ws/src/popi_project/popi_software/popi/popi_robot/trajectories/csv
   ./csv_to_rosbag.py modified_test
   mv modified_test.bag ../bags/
   ```
<br>

* To open the Gazebo environment, you just have to run:
   ```bash
   roslaunch popi_robot popi_simu.launch
   ```
<br>

* If you have a Xbox controller, this launch file allows you to use it to run some simple functions:
<p align="center">
   <img src="https://i.imgur.com/jfdKSFc.png" /> 
</p>

To build your own controls, you can have a look at *popi_software/popi/popi_robot/src/manette.cpp* .
<br>

* Finally, to use the trajectory you just created:
   ```bash
   cd ~/catkin_ws/src/popi_project/popi_software/popi/popi_robot/trajectories/bags
   rosbag play modified_test.bag
   ```
   You can also try out the ones we saved:
   ```bash
   rosbag play short_static_walking.bag
   rosbag play short_dynamic_walking.bag
   rosbag play long_dynamic_walking.bag
   rosbag play pit_30cm.bag
   rosbag play step_20cm.bag
   ```
<br>
<br>

## Contribute
The whole point of making this project fully open-source is to have anyone who is interested contribute to POPI ! Whether it includes documentation translations, new functionalities, bug fixes or code improvements, we'll be glad to receive your pull request.

We're fully aware we still have a long road to go before POPI becomes a more autonomous robot, and we'll be happy to take anyone with us onboard. You can see here the list of [contributors](https://github.com/popi-mkx3/popi_project/graphs/contributors) who participated in this project.
<br>
<br>

## <a name="team"></a> Meet the team !
If you want to contact us you can send us an email at popi.mkx3@gmail.com.

<p align="center"><b> Project Leader </b></p>
<p align="center">
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/clément-thomaso-6b9ab910b/"><img src="https://i.imgur.com/Q7O4i6e.png" width="10%" alt="Clément Thomaso"/></a>
</p>

<p align="center"><b> Mechanics </b> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <b> Electronics </b> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <b> Programming </b></p>

<p align="center">
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/rémi-combacal-16032214a/"><img src="https://i.imgur.com/Dx2GRi4.png" width="10%" alt="Rémi Combacal"/></a>
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/anaïs-gutton-655383a6/"><img src="https://i.imgur.com/yyJLZzI.png" width="10%" alt="Anaïs Aharram-Gutton"/></a>
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/olivier-peres/"><img src="https://i.imgur.com/6CifAs4.png" width="10%" alt="Olivier Peres"/></a>
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/guillaume-rougé-913a10108/"><img src="https://i.imgur.com/d3qdMGd.png" width="10%" alt="Guillaume Rougé"/></a>
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/clémence-graton-a02200174/"><img src="https://i.imgur.com/XqYmJ73.png" width="10%" alt="Clémence Graton"/></a>
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/jean-pelloux-prayer-b38a32143/"><img src="https://i.imgur.com/WhTJOjs.png" width="10%" alt="Jean Pelloux-Prayer"/></a>
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/yannis-oddon-442717128/"><img src="https://i.imgur.com/5yZow0U.png" width="10%" alt="Yannis Oddon"/></a>
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/karla-brottet-69440/"><img src="https://i.imgur.com/ORwYYx3.png" width="10%" alt="Karla Brottet"/></a>
  <a target="_blank" rel="noopener noreferrer" href="https://www.linkedin.com/in/lucas-labarussiat/"><img src="https://i.imgur.com/pJAIIhz.png" width="10%" alt="Lucas Labarussiat"/></a>
</p>
<br>
<br>

<img src="https://i.imgur.com/h6RkNK1.jpg" height="151"/> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <img src="https://i.imgur.com/MZJbr31.png" height="151"/> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <img src="https://i.imgur.com/P2nOGKx.jpg" height="151"/>

[popi_reports]: https://github.com/popi-mkx3/popi_project/tree/master/popi_reports
[popi_mechanics]: https://github.com/popi-mkx3/popi_project/tree/master/popi_mechanics
[popi_electronics]: https://github.com/popi-mkx3/popi_project/tree/master/popi_electronics
[popi_software]: https://github.com/popi-mkx3/popi_project/tree/master/popi_software
[popi_code]: https://github.com/popi-mkx3/popi_project/tree/master/popi_software/popi/popi_code
[popi_control]: https://github.com/popi-mkx3/popi_project/tree/master/popi_software/popi/popi_control
[popi_description]: https://github.com/popi-mkx3/popi_project/tree/master/popi_software/popi/popi_description
[popi_gazebo]: https://github.com/popi-mkx3/popi_project/tree/master/popi_software/popi/popi_gazebo
[popi_robot]: https://github.com/popi-mkx3/popi_project/tree/master/popi_software/popi/popi_robot
[towr]: https://github.com/popi-mkx3/popi_project/tree/master/popi_software/towr/towr
[ifopt]: https://github.com/popi-mkx3/popi_project/tree/master/popi_software/towr/ifopt
[xpp]: https://github.com/popi-mkx3/popi_project/tree/master/popi_software/towr/xpp
[flash images]: https://drive.google.com/drive/folders/1-L8BeBzDw7JF44kGpImWbDnX2oDaef4I?usp=sharing

[SolidWorks to URDF exporter]: https://wiki.ros.org/sw_urdf_exporter
[urdf2robcogen]: https://github.com/leggedrobotics/urdf2robcogen
[RobCoGen]: https://robcogenteam.bitbucket.io/whatisit.html
[SDF]: http://sdformat.org/spec
[ROS]: https://www.ros.org/
[Towr]: https://github.com/ethz-adrl/towr
[rqt_bag]: http://wiki.ros.org/rqt_bag
[RViz]: http://wiki.ros.org/rviz
[URDF]: http://wiki.ros.org/urdf
[xacro]: http://wiki.ros.org/xacro
[Gazebo]: http://gazebosim.org/
[rqt_gui]: http://wiki.ros.org/rqt_gui
[ROS Melodic]: http://wiki.ros.org/melodic/Installation/Ubuntu

[BalenaEtcher]: https://www.minimachines.net/tutos/etcher-flasher-image-usb-50144
[ROS control boilerplate]: https://github.com/PickNikRobotics/ros_control_boilerplate

[Alexander W. Winkler]: https://www.alex-winkler.com
[Dave Coleman]: http://dav.ee/
[TiMOTION]: https://www.timotion.com/fr
[IMT Mines d'Alès]: https://www.mines-ales.fr/

[Clément]: https://www.linkedin.com/in/clément-thomaso-6b9ab910b/
[Rémi]: https://www.linkedin.com/in/rémi-combacal-16032214a/
[Anaïs]: https://www.linkedin.com/in/anaïs-gutton-655383a6/
[Olivier]: https://www.linkedin.com/in/olivier-peres/
[Guillaume]: https://www.linkedin.com/in/guillaume-rougé-913a10108/
[Clémence]: https://www.linkedin.com/in/clémence-graton-a02200174/
[Jean]: https://www.linkedin.com/in/jean-pelloux-prayer-b38a32143/
[Yannis]: https://www.linkedin.com/in/yannis-oddon-442717128/
[Karla]: https://www.linkedin.com/in/karla-brottet-69440/
[Lucas]: https://www.linkedin.com/in/lucas-labarussiat/