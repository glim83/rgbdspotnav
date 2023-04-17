# rgbdspotnav

1. Process RGBD data to extract humans in the scene 
2. Setup a visualization of SCAND Dataset [1]
3. Train the spot robot imitation learning policy to navigate using only RGBD 
4. Process depth from the stereo

Paper:
https://www.cs.utexas.edu/~xiao/papers/scand.pdf

SCAND website:
https://www.cs.utexas.edu/~xiao/SCAND/SCAND.html

Dataset:
https://dataverse.tdl.org/dataset.xhtml?persistentId=doi:10.18738/T8/0PRYRH

## Data parsing:
Inside the data directory, make a bag directory and put the bag files there. run data_parser.py outside and the parsed data will be created in the bag directory.
Please check the requirement.txt in the data folder to meet the requirement.  
```bash
python data_parser.py
python depth_parser.py
```

## Simulation tools for Boston Dynamics' Spot

The Spot package, model and basic control inputs, is based on [SpotMicro project](https://github.com/OpenQuadruped/spot_mini_mini) and [SpotControl](https://github.com/SoftServeSAG/spot_simulation/tree/spot_control)

All the simulation files are located under the spot_ws directory.

### Installation
This simulation uses ROS-Noetic and Gazebo. You will need to have a catkin_ws
Run catkin_make and source devel/setup.bash before starting simulation
Relocate the spot_ws inside the src folder inside your catkin workspace

### Start world in Gazebo
```bash
source devel/setup.bash
roslaunch spot_ws humanworld.launch
```
This launches a world with several humans walking or running around an empty world. The file humans.world can be edited to add/ remove more humans. These models are spawned animated humans based on [Gazebo Animated Actor Tutorial](https://classic.gazebosim.org/tutorials?tut=actor&cat=build_robot)

### Spawn Spot robot
```bash
source devel/setup.bash
roslaunch spot_ws robot.launch
```
This spawns the spot robot into the world. If there is a "No module named 'rospkg'", make sure you are using python3 and run the following command: 
```bash
sudo apt install python-is-python3"
```

### To make robot move
This is a simple node that makes robot perform several actions such as standing up, sitting down or giving the left paw. The resources mentioned above will be used to write more complex algorithms to move the robot.
```bash
source devel/setup.bash
roslaunch spot_ws simple_moves.launch
```
 If you get this error: ROS-Gazebo Failed to load joint_state_controller when running robot.launch. Run the following commands:
```bash
sudo apt-get update
sudo apt-get install ros-noetic-ros-control ros-noetic-ros-controllers
```
To subscribe to all robot joints, run the following commands in a new terminal
```bash
source devel/setup.bash
roslaunch spot_ws inverse.launch
```
And to connect the controler for spot, run the next commands in a new terminal
```bash
source devel/setup.bash
roslaunch spot_ws quadruped_controller.launch
```

Finally to run the full simulation along with the NN, you will first need to run the following launch files: humanworld.launch, robot.launch, inverse.launch, quadruped_controller.launch. Then to move spot, theres three different files that can be used:
1. main.py: this one inputs the simulation humans positions and spot current location and orientation, as well as desired goal (manually set) and publishes the twist msg to the robot using trained model outputs
2. main2.py: this one inputs the bag inputs into the trained model and publishes twist msg to spot from trained model outputs
3. main3.py: reads real twist values from odom.csv file and publishes to spot

