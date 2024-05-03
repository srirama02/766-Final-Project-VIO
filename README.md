# Estimating the Sim2Real Gap for Camera Simulation With Existing Visual Odometry Algorithms as a Judge
#### By Sriram Ashokumar and Huzaifa Unjhawala

## Goal

- We develop a simulator in our lab called Project Chrono.
- The camera model in Chrono:
  - High fidelity
  - Models FOV and Radial distortion
  - Aperture Vignetting
- Objective: To assess how good the camera sensor models are in Project Chrono for robotics applications.
<img width="282" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/0610bf9a-226c-46ef-b324-b132e2a61416">

<img width="249" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/61d1e842-fa87-4b2e-ab4f-a0205039bbff">

<img width="249" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/937fa03c-ed7e-45a2-949f-844ac6d0a5db">

<img width="178" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/e97d4322-3a4a-48d4-9774-76cf6ce3a580">



## Validating Camera Models

### Methods
1. **Pixel-wise differences**:
   - Ensures that pictures from the camera sensor look very similar to those from a real camera.
2. **In-the-loop performance differences**:
   - Evaluate how the quality of the image generated affects the performance of downstream tasks, which is more relevant in robotics.
<img width="342" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/af83e6fe-ebdb-4d10-9848-911ff729a054">
<img width="332" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/bdd3b572-72ad-48f6-a954-f10d77d159d5">


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
<img width="690" alt="image" src="https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/a7453723-8a02-48f0-9412-67b79d1800b6">


## Challenges in Simulation Environment Generation

### Simulation Room Mesh
- Due to NeRFs not outputing the actual mesh, it does not work with Chrono::Sensor.
- Tried to do NeRF to mesh conversion, but did not have luck with it.

### Camera Extrinisic and intrinsic parameter calibration
- Able to get camera relative pose to IMU/body frame available from simulator
- Camera intrinsic parameters and distortion parameters obtaiend through simulation experiment using the OpenCV camera calibration script
![image](https://github.com/srirama02/766-Final-Project-VIO/assets/71669451/a45ab883-029a-4c10-97e0-714ae153cead)

### IMU accelerations are very noisy
- Even when 0 noise is set the acclerations are very noisy in simulation. This is due to the noisy DAE (Differential algebraic equation) solver that solves the dynamics equations
- To fix this we used a additional "IMU smoothing node" which reads the IMU readings at 1000 Hz from sim and publishes a rolling average at 100 Hz


## VO/VIO Algorithms
To decide which algorithm to test, a literature review was performed and 10 algorihtms were picked based on interest and popularity.
Two algorithms were chosen from that group based on:
- Performace on standard datasets as we wanted test the algorithms that perform the best
- Whether it is deep learning based or optimization based as we wanted one of each kind
- Computation requirements as we want to run it in real time
- Sensor requirements - we wanted to try both VO and VIO

### Algorithms Chosen:
- VINS-Fusion (2020) - Optimization based VIO - Stereo Camera + IMU
- AirVIO (2023) - Optimization + DL based VO - Stereo Camera

### Challenges with VO/VIO algorithms
- Dependency and integration challenges with VINS-Fusion and AirVO.
- Due to our autonomy stack using ROS2, we need to convert the algorithms into ROS2 as AirVO is only available in ROS1.

## Future Steps (Before May 3)

- Test in real Motion Capture room with newly calibrated cameras.
- Debug and improve current setup, focusing on sim calibration, room textures, and camera/IMU sync.
- Complete the AirVO ROS1 to ROS2 conversion to isolate issues specific to VINS.

## Conclusion

Thank you for viewing our presentation. Feel free to ask any questions you may have!
