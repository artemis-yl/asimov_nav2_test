## how to implement nav2 along with antonia's instructions
1) Add these cameras to the simulation of Asimov (the Orbbec is RGBD, the turret is RGB) as well as a GPS and IMU
2) Create a GUI in Foxglove or improve the pre-existing one to display the simulated camera feeds

3) Use the RGBD camera along with the YOLO network to estimate the size of detected objects (hint: use the depth data to find the mean distance of the detected bounding box)
4) Take the mgonz simulations and use them to create a proper Gazebo world for our rover which has the same GPS coordinates as Drumheller, AB.  Add more stuff for us to look at.
5) Use the IMU to estimate the rover heading. Display the direction in an image (eg. N10E)
6) (Work with comms if necessary) Set up the Diff Drive plugin so that we can control the simulated Asimov with the keyboard and get simulated odometry data.   Eventually, add the teleop plugin to Foxglove.

7) Add a lidar to Asimov and display its data in RVIZ.
8) Use the lidar data to create an obstacle avoidance node which either moves the robot to avoid the obstacle (publish twist velocity) or subscribes to a camera feed and displays a warning on it

9) Implement the EKF and Navsat transform to combine the gps/fix, imu/data and odometry topics (no Nav2 yet) .
10) Set up the route logging using the GPS data.  Display the map overlay in Foxglove. 

11) Combine the overlays of heading and obstacle avoidance, publish that image to Foxglove

12) 
## on this repo
> this should be good if it doesnt build just remove the build install log and just build again should work
> this is basically just to launch in rviz and like control the angle of the wheels
> ur gonna have to change the fixed frame to base link probably and add a Robotmodel and set the description topic of robot_description
> other than that its basically gonna be like starting from scratch so try looking at other implementations and ill look at ur progress next week

## from antonia
>  decided on two main cameras - a forward facing Orbbec Gemini 2 depth camera for navigation and a motorized turret camera which will be able to rotate 360 degrees.  The cameras will arrive very early in the new year. I have a lot of tasks that I've asked Muntasir and Mina to disburse: 
> 1) Add these cameras to the simulation of Asimov (the Orbbec is RGBD, the turret is RGB) as well as a GPS and IMU
> 2) Create a GUI in Foxglove or improve the pre-existing one to display the simulated camera feeds
> 3) Use the RGBD camera along with the YOLO network to estimate the size of detected objects (hint: use the depth data to find the mean distance of the detected bounding box)
> 4) Take the mgonz simulations and use them to create a proper Gazebo world for our rover which has the same GPS coordinates as Drumheller, AB.  Add more stuff for us to look at.
> 5) Use the IMU to estimate the rover heading. Display the direction in an image (eg. N10E)
> 6) (Work with comms if necessary) Set up the Diff Drive plugin so that we can control the simulated Asimov with the keyboard and get simulated odometry data.   Eventually, add the teleop plugin to Foxglove.
> 7) Add a lidar to Asimov and display its data in RVIZ.
> 8) Use the lidar data to create an obstacle avoidance node which either moves the robot to avoid the obstacle (publish twist velocity) or subscribes to a camera feed and displays a warning on it
> 9) Implement the EKF and Navsat transform to combine the gps/fix, imu/data and odometry topics (no Nav2 yet) .
> 10) Set up the route logging using the GPS data.  Display the map overlay in Foxglove. 
> 11) Combine the overlays of heading and obstacle avoidance, publish that image to Foxglove
> This is a lot, but it is most of the pre-autonomous stack.
> I will post a package soonish which has most of what you need, but it uses the "Sam_Bot" tutorial bot, not our Asimov. 
