# Estimating the Sim2Real Gap for Camera Simulation
### With Existing Visual Odometry Algorithms as a Judge
#### By Sriram Ashokumar and Huzaifa Unjhawala

## Goal

- We develop a simulator in our lab called Project Chrono.
- The camera model in Chrono:
  - High fidelity
  - Models FOV and Radial distortion
  - Aperture Vignetting
- Objective: To assess the efficacy of the Camera Sensor models in Project Chrono for robotics applications.

## Validating Camera Models

### Methods
1. **Pixel-wise differences**:
   - Ensures that pictures from the camera sensor look very similar to those from a real camera.
2. **In-the-loop performance differences**:
   - Evaluates how the quality of the image generated affects the performance of downstream tasks, which is more relevant in robotics.

## Our Evaluation Approach

- Use a **VIO algorithm** as a "judge" to evaluate our camera models.
- Comparison setup:
  - **Simulated Camera + IMU vs. Real Camera + IMU**
  - Metrics used:
    - Drift error in sim and real setups
    - Distribution of errors using Wasserstein Distance
    - Sim2real score
  - This process is repeated across various environments.

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
