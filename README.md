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

## Running the trainning and validation:
Inside the data directory, there are planner models and bag pickles which are in a class called Bag with the necessary inputs.
Imitation planner loades the parsed data and perform the training and validaiton.
Run the following code in the root directory of the package. 
```bash
python -m MP.imitation_planner
```

## Path generation with imitation planner with SCAND dataset (both training data and validation):
Run the following code in the root directory of the package to load the planner.model and generate paths in pickle.
It generates the path_comparision directory inside the data directory. Inside the path comparison directory, there are control inputs generated by the imitation planner.
The files with the gen is the generated trajecotry.
```bash
python -m MP.path_gen
```

## Simulation tools for Boston Dynamics' Spot

The Spot package, model and basic control inputs, is based on [SpotMicro project](https://github.com/OpenQuadruped/spot_mini_mini) and [SpotControl](https://github.com/SoftServeSAG/spot_simulation/tree/spot_control)

All the simulation files are located under the spot_ws directory.

### Installation
This simulation uses ROS-Noetic and Gazebo. You will need to have a catkin_ws.
Run catkin_make and source devel/setup.bash before starting simulation.
Relocate the spot_ws inside the src folder inside your catkin workspace

### Start world in Gazebo
```bash
source devel/setup.bash
roslaunch spot_ws humanworld.launch
```
This launches a world with several humans walking or running around an empty world. The file humans.world can be edited to add/ remove more humans. These models are spawned animated humans based on [Gazebo Animated Actor Tutorial](https://classic.gazebosim.org/tutorials?tut=actor&cat=build_robot). There's three worlds that can be used: fullHumans.world (has both humans and static obstacles), humans.world (only has humans), and boxes.world (has 8 different sized boxes to imitate large groups of people and 2 balls). Edit the launch file to change world. 

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
1. main_spot.py: this one inputs the simulation humans positions and spot current location and orientation, as well as desired goal (manually set) and publishes the twist msg to the robot using trained model outputs
2. main_comp.py: this file compares the sideways movement of the spot robot agains the red ball. 
*The following files only need humanworld.launch to be launched
3. main_ball.py: this one performs RRT from custom initial to goal locations. Then it inputs the RRT waypoints as local goals, the location of the humans in world, the current position and orientation of the red ball, and finally publishes the poses to the ball.
4. main_ball2.py: this one works on the boxes.world. It inputs real obstacles position (of SCAND data) as well as the difference between the goal and the current position of the ball, and publishes poses of the ball. 
5. main_pathTesting.py: this file takes in the real and generated paths from the model and publishes to Path msg (these can be visualized in RVIZ). 



