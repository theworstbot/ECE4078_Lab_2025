# Milestone 1: Simultaneous Localization And Mapping (SLAM)
- [Introduction](#introduction)
    - [ArUco markers](#aruco-markers)
    - [SLAM helper scripts](#slam-helper-scripts)
    - [Map generator](#map-generator)
- [Activities](#activities)
    - [Calibration](#calibration-week-3)
    - [SLAM](#slam-week-4-5)
- [M1 Marking](#M1-marking)
    - [Evaluation scheme](#evaluation-scheme)
    - [Instructions](#marking-instructions)
- [Technical FAQs](#faqs-M1)

---
## Introduction
Imagine your robot arrives at a new supermarket which has an unknown layout. It can read aisle labels with its camera and remembers how it has moved since it started at location (0,0,0) (x,y,theta). How can it figure out where things are and where itself is positioned at a given time?

In M1, the robot will perform Simultaneous Localisation And Mapping (SLAM) in a small arena (supermarket), which contains 10 ArUco markers (aisle labels). The robot will always start at the centre of the map, i.e., (x,y,theta) = (0,0,0). 

You will use your C1 (teleoperation) code to drive your robot around this unknown supermarket strategically, so that it can estimate where things are (creating a map of the arena with marker coordinates) and where itself is at any given time using its camera and motion model.


### ArUco markers
[ArUco markers](http://www.uco.es/investiga/grupos/ava/node/26) are square fiducial markers introduced by Rafael Muñoz and Sergio Garrido. OpenCV contains a trained [function](https://docs.opencv.org/trunk/d5/dae/tutorial_aruco_detection.html) that detects the ArUco markers, which will be used in this project (the dictionary we used to generate the markers was ```cv2.aruco.DICT_4X4_100```). PenguinPi will be using these ArUco markers as aisle labels to help it map the environment and locate itself. 

You can make your own ArUco marker blocks for development outside of the labs using [the print file provided](diy_prints/LabPrintingMarker_A3_Compact.pdf) (or [the foldable version](diy_prints/LabPrintingMarker_A3.pdf)). The marker size used in this lab is **7cm** (black square area) as specified in [aruco_detector.py](slam/aruco_detector.py#L10). Please print the pdf using **A3-size papers** and beware of printing rescaling, as inaccurate size of the marker will lead to inaccurate estimations. You may also like to consider creating your own CAD model and 3D printing your own ArUco cubes.

### SLAM helper scripts
The following helper scripts are provided to support your development of SLAM
- [operate.py](operate.py) is the central control program that combines both keyboard teleoperation (C1) and SLAM. It's an extension from the operate.py script in C1. Running this file requires the [utility scripts](../Week00-01/util) and the [GUI pics](../Week00-01/pics) from C1, and the [calibration parameters](calibration/param/) and [SLAM scripts](slam/) of M1. After the GUI is launched, you can start or stop SLAM by pressing ENTER, and save a map generated by SLAM by pressing "s". It provides a visualisation of the robot's view with overlay of identified ArUco markers, the SLAM visualisation, and a 5-minute countdown clock. Feel free to change the countdown or remove it. It's provided as a reminder of the [overall marking slot time limit](M1_marking_instructions.md). Remember to replace the [keyboard control section](operate.py#L200) with your teleoperation codes from C1.
- [Calibration scripts](calibration/) and [default parameters](calibration/param/) for wheel and camera calibration: see [calibration section](#calibration-week-3) for details
- [Designs](diy_prints/) of the ArUco marker blocks and the camera calibration rig for setting up your own development arena outside of the labs
- [aruco_detector.py](slam/aruco_detector.py) uses OpenCV to detect ArUco markers and provides an estimation to their positions, which will be used for SLAM in addition to the drive signals
- [mapping_utils.py](slam/mapping_utils.py) saves the SLAM map for evaluation
- [SLAM_eval.py](SLAM_eval.py) evaluates a SLAM map against the ground truth
- [TrueMap.txt](TrueMap.txt) is an example map text file generated while setting up an arena and for evaluating your SLAM during development. This map will NOT be used for marking.

### Map generator
To help you generate ground-truth maps for evaluating your SLAM implementation in an arena outside of class times, we've provided a [map generator](../image_to_map_generator), which creates maps from an overview image of the arena. It's provided as-is and for reference only. Don't use it to cheat.

---
## Activities

### Calibration

#### Step 1) Preparing Working directory
Prior to working on this week's materials, please make sure you do the following:
- Copy over the [util folder](../Week00-01/util) and the [GUI pics](../Week00-01/pics) from the C1 lab into your working directory
- Replace the [keyboard control section](operate.py#L200) in operate.py with the code you developed for C1

**Tip:** For Steps 2 and 3, you are not enforced to implement the calibration scripts provided. In other words, you are encouraged to edit the scripts, or make new scripts from scratch, such that you save time calibrating, or calibrate more accurately!

#### Step 2) Wheel calibration
Complete [wheel_calibration.py](calibration/wheel_calibration.py) by completing the required functionality and filling in the lines of code (computing the [scale parameter](calibration/wheel_calibration.py#L46) and the [baseline parameter](calibration/wheel_calibration.py#L89)), run the [wheel calibration script](calibration/wheel_calibration.py) using the command ```python3 wheel_calibration.py```. This script will set the robot driving forward, and then spinning at various velocities, for durations that you specify, in order to compute the scale and baseline parameters for wheel calibration.

You can mark a 1m long straight line with masking tape on the floor, and use it as a guide to check if the robot has travelled exactly (as close as possible) 1m. Masking tape and measuring tape will be provided to you in the lab.

#### Step 3) Camera calibration
Complete [calib_pic.py](calibration/calib_pic.py) using your C1 teleoperation codes and run the completed script using ```python3 calib_pic.py```, then press ENTER to take a calibration photo (It will be saved as [calib_0.png](calibration/images/calib_0.png)) in your [Images folder](calibration/images/). You should place the robot fairly close to the calibration rig to get a good view of the 8 dots. The photo should look something like this:

![Real calibration photo](screenshots/RealCameraCalib.png?raw=true "Real calibration photo")

Once you have taken the `calib_0.png` photo with the robot, run ```python3 camera_calibration.py``` to perform camera calibration. This opens the calibration photo you just took. Selecting the 8 key points in the calibration photo following the ordering shown in [calibration-fixture.png](diy_prints/calibration-fixture.png) by left clicking on each point (right click to cancel a selected point). Once all 8 points are selected, close the figure window to compute the camera matrix. This will update the [intrinsic parameters](calibration/param/intrinsic.txt). Note: keep the [distortion coefficients](calibration/param/distCoeffs.txt) to all 0s.

**NOTE:** You may encounter an issue such as "Illegal Instruction" or some other issue, when running the `camera_calibration.py` script. If this happens, try running `camera_calibration_2.py` instead, which is an alternative script that uses OpenCV instead of machinevisiontoolbox.

![Calibration fixture](diy_prints/calibration-fixture.png?raw=true "Calibration fixture")

**Note:** You can make your own calibration rig for development outside of the labs using [the print file provided](diy_prints/LabPrintingRig_A3.pdf) (notice the small square in the top left corner in addition to the dots). The size of the rig is specified in [calibration-fixture.png](diy_prints/calibration-fixture.png). Please print the pdf using **A3-size papers** and beware of printing rescaling. Make sure that the panels are at 90deg to each other and the distance between the calibration dots are as specified when making the rig, as this will influence the accuracy of your estimations. We will bring a calibration rig to the labs for you to perform the calibration during the lab sessions too.

---
### SLAM
[operate.py](operate.py) makes use of the camera and wheels' [calibrated parameters](calibration/param) and the [SLAM](slam/) components to produce a map saved as ```slam.txt``` in the ```lab_output``` folder. This SLAM map contains a list of identified ArUco markers, their locations and the covariances of the estimation. Note: remember to replace the [keyboard control section](operate.py#L200) with your codes from C1.

[SLAM](slam/ekf.py) computes the locations of the ArUco markers using both [camera based estimation](slam/aruco_detector.py) and [motion model based estimation](slam/robot.py).

**Please complete [robot.py](slam/robot.py) by filling in the computation of the [derivatives](slam/robot.py#L79) and [covariance](slam/robot.py#L127) of the motion model. Please also complete [ekf.py](slam/ekf.py) by filling in the computation of the [predicted robot state](slam/ekf.py#L93) and the [updated robot state](slam/ekf.py#L117) to finish the extended Kalman filter function.**

Once robot.py and ekf.py are completed, you can test the performance of your SLAM by running ```python3 operate.py```

Below are examples of what the GUI running SLAM looks like on physical robot and in sim:

![Example SLAM visualisation on physical robot](screenshots/SLAMvis.png?raw=true "Example SLAM visualization in Gazebo")

#### Evaluating the SLAM performance
An evaluation script [SLAM_eval.py](SLAM_eval.py) is provided for evaluating the map generated by SLAM against the true map.

A [development map](TrueMap.txt) is provided which you can use for setting up the arena and for evaluation. 

Run ```python3 SLAM_eval.py TrueMap.txt lab_output/slam.txt``` and you should see a printout of the evaluation results:
 
![Example output of SLAM evaluation script](screenshots/SLAM_eval_output.png?raw=true "Example output of SLAM evaluation script")

---
## M1 Marking
### Evaluation Scheme and Marking Instructions

This will be updated during the semester. 

---
## FAQs: M1
- Refer to the comments in each of the [calibrateWheelRadius](calibration/wheel_calibration.py#L10) and [calibrateBaseline](calibration/wheel_calibration.py#L52) functions for that each of these values corresponds to on the physical robot
- Take a close look at the units of the expected output when formulating your calculations. Referring to these equations may be helpful:

![Useful equations for calculating baseline](screenshots/AngularVelocity.png?raw=true "Useful equations for calculating baseline")

- It is recommended that you keep the file structure for this lab material (and future weeks) unchanged to avoid path errors
- Beware of the sign error that can happen with calculating the difference between the measurement and estimate in the correction step
- The state vector x will be appended with the ArUco marker measurements. Take a note of the location of x that should be updated in the motion model.
- In the prediction step, we should update the mean belief by driving the robot.
- The example baseline, intrinsic and scale parameters on Github should be relatively close to your own parameters, though running your robot with those parameters may result in suboptimal SLAM performance. If you calibrate your robot and get vastly different values, consider double checking your calibration scripts to make sure no miscalculations are made.
