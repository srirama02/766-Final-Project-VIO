# Estimating the Sim2Real Gap for Camera Simulation With Existing Visual Odometry Algorithms as a Judge
#### By Sriram Ashokumar and Huzaifa Unjhawala

## Demo
To see instructions on how to run the demo, click [here](DEMO.md).  
Link to [Codebase](https://github.com/srirama02/766-Final-Project-VIO/blob/main/README.md)  
Link to [Slides](https://uwmadison.box.com/s/3bmxfq3sb8hvkx6lm3n0k9awg59j58u1)  

## Goal
In our lab, we develop a simulator called Project Chrono. Chrono includes a high-fidelity camera model that accounts for field of view (FOV), radial distortion, and aperture vignetting. However, to assess the accuracy of the camera sensor models in Project Chrono for robotics applications, we need to test them with various algorithms. In this project, we tested various visual odometry (VO) and visual-inertial odometry (VIO) algorithms to validate the performance of our simulated sensor by comparing it to that of a real sensor.
![image](https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/3fc60f87-fccf-4480-9ddd-5c1d0be91758)



## Validating Camera Models

### Methods
1. **Pixel-wise differences**:
   - Ensures that pictures from the camera sensor look very similar to those from a real camera.
2. **In-the-loop performance differences**:
   - Evaluate how the quality of the image generated affects the performance of downstream tasks, which is more relevant in robotics.
  
<img width="342" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/af83e6fe-ebdb-4d10-9848-911ff729a054"> [1]
<img width="470" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/bdd3b572-72ad-48f6-a954-f10d77d159d5"> [2]


## Our Evaluation Approach

- Use a **VIO algorithm** as a "judge" to evaluate our camera models.
<img width="1163" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/d5261081-341d-469d-b71b-0a5aa1d31bb3">


## Major Components

1. **Simulation Environment**:
   - Project Chrono setup with vehicle dynamics, stereo camera and IMU virtual sensors, and a virtual world (3D mesh of ME3038).
2. **VIO/VO Algorithm**:
   - Chosen based on performance, computational and sensor requirements, including deep learning and optimization-based methods.
3. **Reality Environment**:
   - Setup in a motion capture room with real vehicle, stereo camera (ZED), and IMU (Wheeltec N-100).

<img width="718" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/a35ee377-78ff-4751-a0ba-9841f7470323">

## Simulation Environmnet
- Demo of the vehicle driving around in generated simulation environment
- The vehicle in the simulation has a stereo camera setup and the video also shows the depth map
- The 3D mesh shown in the simulation is generated in Blender

<iframe src="https://drive.google.com/file/d/1ZvL5P4deNLsipzVci7Zg_4PReqeX4EMU/preview" width="640" height="480" allow="autoplay"></iframe>


## Challenges in Simulation Environment Generation

### Simulation Room Mesh
- Due to NeRFs not outputting the actual mesh, it does not work with Chrono::Sensor.
- Tried to do NeRF to mesh conversion, but did not have luck with it.

### Camera Extrinsic and intrinsic parameter calibration
- Able to get camera relative pose to IMU/body frame available from simulator
- Camera intrinsic parameters and distortion parameters obtained through simulation experiment using the OpenCV camera calibration script
![image](https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/a45ab883-029a-4c10-97e0-714ae153cead)

### IMU accelerations are very noisy
- Even when 0 noise is set the accelerations are very noisy in simulation. This is due to the noisy DAE (Differential algebraic equation) solver that solves the dynamics equations
- To fix this we used a additional "IMU smoothing node" which reads the IMU readings at 1000 Hz from sim and publishes a rolling average at 100 Hz


## VO/VIO Algorithms
To decide which algorithm to test, a literature review was performed and 10 algorithms were picked based on interest and popularity.
Two algorithms were chosen from that group based on:
- Performance on standard datasets as we wanted test the algorithms that perform the best
- Whether it is deep learning based or optimization based as we wanted one of each kind
- Computation requirements as we want to run it in real time
- Sensor requirements - we wanted to try both VO and VIO

### Algorithms Chosen:
- VINS-Fusion (2020) - Optimization based VIO - Stereo Camera + IMU
- AirVIO (2023) - Optimization + DL based VO - Stereo Camera

### Challenges with VO/VIO algorithms
- Dependency and integration challenges with VINS-Fusion and AirVO.
- Due to our autonomy stack using ROS2, we need to convert the algorithms into ROS2 as AirVO is only available in ROS1.

### VINS-Fusion
- For each new image, existing features are tracked by the KLT sparse optical flow algorithm.
- Key frames selected based on parallax between current frame and latest keyframe.
- IMU pre-integration to save compute.
- DBoW2 place recognition approach used for loop detection.
- Connection between local sliding window and loop closure candidate through BRIEF descriptor matching with outlier rejection.
- Only 4 DOF Pose Graph optimization since we have IMU which makes roll and pitch observable.
![image](https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/6b309bf6-05e0-45ef-865e-a37027b1d42d) [3]

### AirVO
- Utilizes both a learning-based front end and a traditional optimization backend.
- Feature points are extracted and matched with previous key frame and line features are extracted. Used to detect the key frames.
- On the right images of the key frames the same point and line feature detection is performed.
- Local bundle adjustment is performed to optimize the points, lines, and keyframe poses.
- New line processing pipeline for VO that associates 2D lines with learning-based 2D points on the image.
![image](https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/0ab79f70-b594-40e9-a638-c2dca9f71613) [4]


### VINS-Fusion Results
Testing a circle trajectory path results in the estimated trajectory having a radius twice as big
- Video shows a screen recording of VINS-Fusion and the simulation running in the simulation docker container
- RViz visualization tool used ot visualize camera pose published by VINS-Fusion
- Circle estimated is much bigger than the one traversed.

<iframe src="https://drive.google.com/file/d/1w-qtHO_yQONQF1vptkJVyebjOz41BCLy/preview" width="640" height="480" allow="autoplay"></iframe>

The visualization shows nice feature tracking
- Can see the optical flow arrows
- We think the performance currently isn't that good due to:
   - Possible bug in the camera parameters, might need to repeat calibration process
   - The room doesn't have enough features to allow for feature tracking, we could improve the textures in the environment
   - Possible issues with the camera and IMU syncing due ot the use of simulation time

<iframe src="https://drive.google.com/file/d/1uhqunABXjBGA-kyIfh_kT4TuZ4uiJOFr/preview" width="640" height="480" allow="autoplay"></iframe>

## Future Steps
![image](https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/ce4a50c8-b223-4edd-8d99-22a3dc362c97)

- Due to trouble with reality setup to run the software, we were unable to test in reality but aim to continue working to debug and fix those issues
- Test in real Motion Capture room with newly calibrated cameras.
- Debug and improve current setup, focusing on sim calibration, room textures, and camera/IMU sync.
- Complete the AirVO ROS1 to ROS2 conversion to isolate issues specific to VINS.

## References
[1] https://www.youtube.com/watch?v=P1IcaBn3ej0  
[2] Asher et.al. - Modeling Cameras for Autonomous Vehicle and Robot Simulation: An Overview – IEEE Sensors  
[3] Qin et.al. - VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator- IEEE Transactions on Robotics  
[4] Xu et.al. - AirVO: An Illumination-Robust Point-Line Visual Odometry - IROS  

## Thank You
