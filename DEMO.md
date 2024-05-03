# Running the Demo
Here we present the steps necessary to run the demo shown in the class. The steps have only been tested on a Linux based OS (Ubuntu 22.04). However, since we docker-ize the entire setup, it should work on any OS that supports Docker. Additionally, since the Sensor module of Project Chrono (the simulation environment used in this study) requires Nvidia GPU support, you would require a machine that has one. 

## Pre-requisites
- Docker
Please follow instructions from the [official Docker website](https://docs.docker.com/engine/install/ubuntu/) to install Docker on your machine.
- Nvidia Container Toolkit
Please follow instructions from the [official Nvidia website](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) to install the Nvidia Container Toolkit on your machine.

## Steps
1. First we recommend restarting the docker service to ensure that the Nvidia Container Toolkit is properly installed.
```bash
sudo systemctl restart docker
sudo systemctl enable docker
```
2. Next, we will pull the Docker image from Docker Hub. This image contains all the necessary dependencies to run the demo.
```bash
docker pull huzaifg/vio:submission
```

3. Once the image is downloaded, we can run the container. We will use the `--gpus all` flag to enable GPU support. Additionally, we will also use vnc to get a nice GUI to visualize the demo and the odometry published by VINS-Fusion
```bash
docker run -it --gpus=all -p 5901:5901 -p 6901:6901 huzaifg/vio:submission
```
You will now see a bunch of output on your terminal. Ideally, it should be okay to ignore this.

4. Next, open the VNC viewer on your machine and connect to `localhost:6901`. This can be done by opening a browser and typing `localhost:6901` in the address bar. The password is `sbel`. This should bring you to the below screen.
![VNC window](./images/vnc.png)

5. Next open a bunch of terminal windows on the host machine. We recommend using something like `Terminator` or `tmux` to manage multiple terminals. We will use these terminals to launch the simulation, the VINS-Fusion odometry, two nodes for post-processing the simulation sensor data, and the Rviz, a visualization tool to look at the odometry published by VINS-Fusion. This means a total of 6 terminals is recommended.

6. First, notice that we are still on the host terminal. We will need to move into the container terminal. For this, in all the terminals, run
```bash
docker exec -it <container_id> /bin/bash
```
where `<container_id>` can be found by running `docker ps` on the host terminal. This will move you to the container terminal.

7. Once this is done, in 5 of the 6 terminals, run the command to move into the ros2 workspace
```bash
cd ~/ros2_ws
```

8. Then, in any one of the terminal windows, build the ros2 packages using the following command
```bash
colcon build --symlink-install && source install/setup.bash
```

9. Next, in all the other ros2 terminal windows, source the workspace
```bash
source install/setup.bash
```
Now we are ready to run the ros2 setup. However, we are still not ready to run the simulation. 

10. In the one terminal that we have not moved into the ros2 workspace, run the following command to first compile the simulation demo
```bash
cd ~/Desktop/chrono/build
ninja
```
Ideally, you should see `ninja: no work to do.`. This means that we did not make any mistakes in pushing the right image to Docker Hub. 

11. Next, in the same terminal, run the following command to start the simulation
```bash
cd bin
./demo_ROS_ARTcar_VO
```
Now go back to your browser window with the VNC viewer. You should see a car moving in a simulated environment.
![Simulation](./images/chrono.png)
Ctrol+C to stop the simulation in the terminal as we need to do a few more things before we can visualize the odometry.

12. In 2 of the ros2 terminal windows, run the following command to start the post-processing nodes
```bash
ros2 run image_flip img_flip_node
```
This node flips the camera feed coming from the simulation to match the one needed by VINS-Fusion.

```bash
ros2 run imu_smooth imu_smoothing_node
```
This node smoothens the IMU data coming from the simulation (at 1000 Hz) and publishes it at 100 Hz. This is done because the solvers in simulation usually produce very noisy acceleration values that trip up the VINS-Fusion solver.

13. While these nodes are running, in another ros2 terminal window, run the following command to start Rviz
```bash
ros2 launch vins vins_rviz.launch.xml
```
This will open up Rviz. However, a slight detour here is needed to setup the Rviz configuration in order to efficiently visualize the odometry when everything is finally running. This detour is running the following command in the chrono terminal
```bash
./demo_ROS_ARTcar_VO 
```
This will start the simulation again. Now go back to the Rviz window select the `world` frame in the `Fixed Frame` dropdown.
![Rviz](./images/world.png)
Next `Add` -> `By topic` -> `/image_track/Image` to visualize the features tracked on the image. 
![Rviz2](./images/img_track.png)
Next `Add` -> `By topic` -> `/camera_pose/odometry` to visualize the odometry published by VINS-Fusion.
![Rviz3](./images/odo.png)

14. Now that the visualization is setup, go back to the terminal where the simulation is running and press `Ctrl+C` to stop the simulation. This will stop the simulation but the post-processing nodes and Rviz will still be running. Restart the simulation by running the following command in the terminal
```bash
./demo_ROS_ARTcar_VO
```

15. Very quickly, using another terminal, run the following command to start the VINS-Fusion odometry
```bash
ros2 run vins vins_node ./src/VINS-Fusion-ROS2/config/simulation/simulation_config.yaml 
```

16. Now you can see the car moving in the simulation and the odometry being published by VINS-Fusion in Rviz. You can also see the features being tracked in the image.


## Where is all the code?
To simplify the setup and running the demo, we have eliminated the users direct interaction with the code. However, if you are interested in looking at the code,

- The simulation code can be found in the `chrono` folder on the Desktop. The specific demo we are running is in the `chrono/src/demos/ros/demo_ROS_ARTcar_VO.cpp` file.
- The VINS-Fusion code can be found in the `VINS-Fusion-ROS2` folder in `~/ros2_ws/src`. The specific configuration file we are using is in the `VINS-Fusion-ROS2/config/simulation/simulation_config.yaml` file.
- The docker image we are using can be found on Docker Hub [here](https://hub.docker.com/r/huzaifg/vio). The Dockerfile used to build this image can be found in the `docker` folder in `~/ros2_ws/src/VINS-Fusion-ROS2/docker/withChrono`. However, this is not the exact Dockerfile used to build the image on Docker Hub since we made modifications to the container directly, committed it and pushed it to Docker Hub.