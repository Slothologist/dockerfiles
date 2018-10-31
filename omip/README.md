#OMIP

#### Overview

This is a Dockerfile for TU Berlins robotics divisions framework for Online Interactive Perception (omip).
Rostopics and -params have been adjusted for a pepper robot.
You can find their project [here](https://github.com/tu-rbo/omip).

#### How to build

```
docker build -t omip_node .
```

#### How to run

```
docker run -it -e DISPLAY -e ROS_MASTER_URI="url" (-e RGBD="" -e RVIZ="" -e OMIP="")-v /tmp/.X11-unix:/tmp/.X11-unix -v $HOME/.Xauthority:/home/developer/.Xauthority --net=host omip_node
```

#### Aditional information

There is a interesting hack in this dockerfile: Omip uses terminator to show all its nodes,
 but terminator runs on X-Server. Thus we export the display variable and are able to get 
 the GUI of a program which runs inside the docker container. 
 (This is the reason for the ugly run command)