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

## Challenges and Solutions

### Simulation Environment Generation
- Issues with room mesh generation in Blender and NeRFs.
- Camera and IMU parameter calibration and noisy IMU readings.

### VO/VIO Algorithms
- Dependency and integration challenges with VINS-Fusion and AirVO.
- Conversion of algorithms for ROS2 compatibility.

## Future Steps (Before May 3)

- Test in real Motion Capture room with newly calibrated cameras.
- Debug and improve current setup, focusing on sim calibration, room textures, and camera/IMU sync.
- Complete the AirVO ROS1 to ROS2 conversion to isolate issues specific to VINS.

## Conclusion

Thank you for viewing our presentation. Feel free to ask any questions you may have!
