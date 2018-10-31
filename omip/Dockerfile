# Use an official Python runtime as a parent image
FROM ros:indigo-ros-base

# Install common packages (may be needed: ros-indigo-catkin g++)
RUN apt-get update
RUN apt-get -y install python-pip nano
RUN pip install catkin-tools

# Set the working directory to the workspace /ws_omip
WORKDIR /ws_omip

# Create workspace
RUN cd /ws_omip
RUN mkdir src
RUN /bin/bash -c "source /opt/ros/indigo/setup.bash && catkin build"
RUN /bin/bash -c "source devel/setup.bash"

# Clone the repos which are needed into the WS (specifically: in the src folder)
RUN git clone https://github.com/tu-rbo/omip.git src/omip
RUN git clone https://github.com/tu-rbo/omip_msgs.git src/omip_msgs
#RUN git clone https://github.com/roberto-martinmartin/rviz_plugin_camerarenderpublisher.git src/rviz_plugin_camerarenderpublisher
#RUN git clone https://github.com/laas/rviz_plugin_covariance.git src/rviz_plugin_covariance

#check out correct branches
#RUN /bin/bash -c "cd src/rviz_plugin_camerarenderpublisher/ && git checkout indigo"
#RUN /bin/bash -c "cd src/rviz_plugin_covariance/ && git checkout indigo"

# Install all packages needed by omip (a lot)
RUN apt-get -y install libboost-dev libeigen3-dev terminator ros-indigo-pcl-ros ros-indigo-openni-launch ros-indigo-openni-camera ros-indigo-openni2-launch ros-indigo-openni2-camera ros-indigo-cmake-modules ros-indigo-bfl ros-indigo-libpointmatcher ros-indigo-rviz ros-indigo-tf-conversions

# Copy modified config files from omip to global dirs
RUN mkdir -p /home/developer/.config/terminator/
RUN cp src/omip/omip_launch/cfg/terminator/config /home/developer/.config/terminator/
RUN cp src/omip/omip/thir d_party/bflConfig.cmake /opt/ros/indigo/share/bfl/
RUN cp src/omip/omip/third_party/libpointmatcherConfig.cmake /opt/ros/indigo/share/libpointmatcher/cmake

# change rostopics of feature tracker to pepper camera
RUN /bin/bash -c 'echo "rgb_img_topic: /pepper_robot/sink/front/image_raw" >> src/omip/feature_tracker/cfg/feature_tracker_cfg.yaml'
RUN /bin/bash -c 'echo "depth_img_topic: /pepper_robot/sink/depth_registered/image_rect" >> src/omip/feature_tracker/cfg/feature_tracker_cfg.yaml'
RUN /bin/bash -c 'echo "camera_info_topic: /pepper_robot/camera/front/camera_info" >> src/omip/feature_tracker/cfg/feature_tracker_cfg.yaml'

# change rosparams in omip_cfg (pepper cam has just 10 fps)
RUN /bin/bash -c 'echo "sensor_fps: 10" >> src/omip/omip_launch/cfg/omip_cfg.yaml'

# Build catkin workspace
RUN /bin/bash -c "source devel/setup.bash && catkin build omip && catkin build"

######### Make Docker export the display variable, used to get terminator to display, but usefull for any gui
# For it to work you need to start the container with like docker run -e DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v $HOME/.Xauthority:/home/developer/.Xauthority --net=host DOCKERFILENAME

# Replace 1000 with your user / group id
RUN export uid=1000 gid=1000 && \
    mkdir -p /home/developer && \
    echo "developer:x:${uid}:${gid}:Developer,,,:/home/developer:/bin/bash" >> /etc/passwd && \
    echo "developer:x:${uid}:" >> /etc/group && \
    echo "developer ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/developer && \
    chmod 0440 /etc/sudoers.d/developer && \
    chown ${uid}:${gid} -R /home/developer

USER developer
ENV HOME /home/developer
######### End export of dislay variable

# Build shell file to pass arguments to (necessary because of '/bin/bash -c' and 'source devel/setup.bash')
RUN sudo /bin/bash -c 'echo "#!/bin/bash" >> docker-entrypoint.sh'
RUN sudo /bin/bash -c 'echo "source devel/setup.bash && rosrun omip_launch omip.sh -p=30 \$RGBD \$RVIZ \$OMIP" >> docker-entrypoint.sh'
RUN sudo chmod 557 docker-entrypoint.sh

# Set default values for rgbd, rviz and omip:
ENV RGBD "--rgbd=0"
ENV RVIZ "--rviz=0" # should remain as such, launching a gui from docker is cancer
ENV OMIP "--omip=1"

# Run run.sh when the container launches (passing cmd params through)
ENTRYPOINT ["./docker-entrypoint.sh"]
CMD []
