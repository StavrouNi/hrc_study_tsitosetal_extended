# Human-Agent Co-Learning on a shared task

## Citation 

If you want to cite this work, please use the following bibtex:

```python
@misc{https://doi.org/10.48550/arxiv.2211.13070,
  doi = {10.48550/ARXIV.2211.13070},
  url = {https://arxiv.org/abs/2211.13070},
  author = {Tsitos, Athanasios C. and Dagioglou, Maria},
  keywords = {Robotics (cs.RO), Artificial Intelligence (cs.AI), Human-Computer Interaction (cs.HC), FOS: Computer and information sciences, FOS: Computer and information sciences},
  title = {Enhancing team performance with transfer-learning during real-world human-robot collaboration},
  publisher = {arXiv},
  year = {2022},
  copyright = {Creative Commons Attribution 4.0 International}
}
```

## Description
A human-agent collaborative game in which the team needs to co-operate in order to control the position of the end-effector(EE) of a robotic manipulator in a 2D plane, as shown in the following figure. Collaborative learning is achieved through Deep Reinforcement Learning (DRL). 

![setup](https://github.com/ThanasisTs/human_robot_collaborative_learning/blob/main/pictures/setup.png)

The human controls the position of the EE in one axis (x-axis) while the DRL agent controls the positiion in a perpendicular axis (y-axis). Therefore, the EE is able to move in a 2D plane (xy-plane, plane of the table). The EE is confined to move inside a rectangle (xy-plane) with predifined boundaries and at a specified height (z-axis). 

![court](https://github.com/ThanasisTs/human_robot_collaborative_learning/blob/main/pictures/court.png)

In the beginning of a game, the EE is automatically placed in one of the four starting positions denoted with the symbol "S". The task is solved if the team manages to bring the EE inside the goal space with a maximum allowed velocity during a specific time duration. The goal position is denoted with the symbol "X" and the goal space is the denoted with the circle around it. Once the EE is placed at the initial position, a certain sound is produced indicating the start of the game. Furthermore, two different sounds are produced at the end of the game depending on the outcome. Lastly, the outcome of the game, the score of the team and the number of the game are visualized in a window.

<img src="https://github.com/ThanasisTs/human_robot_collaborative_learning/blob/main/pictures/visualization.png" height="500" width="450">

The entire game has been implemented in ROS using both roscpp and rospy APIs and has been tested on Ubuntu 18.04 and ROS Melodic distribution.

### Control
Both partners control the position of the EE (in their respective axis) by commanding the sign of the acceleration. Specifically, they can apply a positive acceleration, a negative acceleration and zero acceleration. The human achieves it using a keyboard.

### Reinforcement Learning
The Soft Actor-Critic (SAC) algorithm is used with modifications for discrete action space[1]. The MDP is summarized as follows:

* Observation space: The observation space is the position and velocity of the EE in the xy plane (4D vector).
* Action space: The action space is the positive, negative and zero acceleration (3D vector).
* Reward function: At each timestep, the agent receives a reward of -1 if the EE transitioned to a non-goal state and 10 if it reached the goal. 

The implementation of the SAC algorithm is based on [here](https://github.com/kengz/SLM-Lab) and [here](https://github.com/EveLIn3/Discrete_SAC_LunarLander/blob/master/sac_discrete.py).

### Transfer Learning
The game can be played using two different conditions.
* No transfer learning: The agent selects either a random action or an action derived by his policy.
* Transfer learning. Probabilistic Policy Reuse is implemented. The agent can select a random action, an action derived by his policy or an action derived by an expert, pre-trained agent.

### Baseline
A version of the game exists where the DRL agent is not used and the EE can move only to the human-controlled axis (x-axis). This way, the human can gain expertise on how to control the motion of the EE in his axis.

## Installation
* Run `sudo apt-get install ros-<ROS-DISTRO>-teleop-twist-keyboard python-sklearn ffmpeg`. This will install the ROS package for sending commands from the keyboard.
* Run `source install_dependencies/install.sh`. A python virtual environemnt will be created and the necessary libraries will be installed. Furthermore, the directory of the repo will be added to the `PYTHONPATH` environmental variable. In case you do not wish to work in a separate environment, run `pip install -r install_dependencies/requirements.txt`
* Check the `*.yaml` files and change the paths based on your setup

## Run
Before running anything go to the `game_control_sign.py` file and change the python interpreter path to the absolute path of the virtual environment in your machine.

The game has been tested only on the real robot and not on a simulated environment. The following instructions assume that the [Universal Robot UR3 Cobot](https://github.com/UniversalRobots/Universal_Robots_ROS_Driver) is used as a robotic manipulator along with a [Cartesian Velocity Controller Interface(CVCI)](https://github.com/Roboskel-Manipulation/manos/tree/updated_driver/manos_cartesian_control). Clone the [repo](https://github.com/Roboskel-Manipulation/manos), which contains important utilities for the robot, and install the necessary repos listed in its README.md.

* Run `roslaunch manos_bringup manos_bringup.launch robot_ip:=<robot_ip> kinematics_config:=<path_to_catkin_ws>/src/manos/manos_bringup/config/manos_calibration.yaml`. This will launch the UR3 drivers and the CVCI.
* Run `roslaunch human_robot_collaborative_learning game.launch`. This will launch the entire game. This includes the node of game loop, the node for the robot motion generation, the visualization node and the SAC implementation.
* Run `rosrun teleop_twist_keyboard teleop_twist_keyboard.py`. This launches the keyboard node.

<b>Note</b>: The first time you run the commands, a folder named `games_info` will be created. Each time you play a new set of games (run the commands), a folder will be created in the `games_info` folder which will contain data about the episodes. The name of the folder has the following structure:

* If no transder learning is applied: `<total_update_cycles>K_<learn_every_n_episodes>_<action_duration>ms_<participant_name>_<no_TL>_<increasing_number>`
* if transfer learning is applied: `<total_update_cycles>K_<learn_every_n_episodes>_<action_duration>ms_<participant_name>_<increasing_number>`

The `<total_update_cycles>`, `<learn_every_n_episodes>`, `<action_duration>` and `<participant_name>` are parameters set [here](https://github.com/Roboskel-Manipulation/human_robot_collaborative_learning/blob/main/config/rl_params.yaml) while the `<increasing_number>` is set automatically and depends on how many times the same game configuration has been used before.

<b>Note</b>: Be aware that the mouse cursor needs to be in the terminal window from which the `teleop_twist_keyboard` was launched so that keyboard commands can be sent to the robot.

If you want to run the baseline version, replace the `roslaunch human_robot_collaborative_learning game.launch` command with `roslaunch human_robot_collaborative_learning baseline.launch`.
## Folders
* `audio_files`: Contains the sound files.
* `games_info`: Files with data regarding the games.
* `install_dependencies`: Files for seting up the virtual environment.
* `rl_models`: The files of the trained RL models are stored here.

The rest are default folders of a ROS package.

[1] Christodoulou, Petros. "Soft actor-critic for discrete action settings." arXiv preprint arXiv:1910.07207 (2019).

## Contributions

In this fork, Deep Q-Learning from Demonstrations (DQfD) Transfer Learning method is integrated, which is designed to enhance the learning capabilities of the Reinforcement Learning (RL) agent within a collaborative task environment. Firthermore an initialized agent is integrated during the first interaction games, in order to reduce the performance variance of a random one.


## Usage

The configuration for the previously implemented probabilistic policy reuse (PPR) method remains unchanged. To enable the Learning from Demonstrations (LfD) Transfer Learning method, adjust the settings in the `config/rl_params.json` file as follows:

- To record expert gameplay for later use with LfD, set `lfd_expert_gameplay` to `true`.
- To have the agent interact using the LfD method, set `lfd_participant_gameplay` to `true`.
- Expert gameplay sessions are saved automatically. To select specific sessions for LfD, use the `scripts/ExperttoDemoBufferTestFunc.py` script.
- Define the percentages of expert demonstrations used during LfD in the `scripts/sac_discrete_agent.py` file.
- To initialize the learning agent with prior knowledge instead of starting from scratch, set `initialized_agent` to `true` in the first game block.


The branch compatible with the hardware setup is `init-pos+various-fix`. 
