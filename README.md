## implement nav2 as per their setip guide along with antonia's instructions
overall, look at these links

https://github.com/mvipin/perceptor/tree/main
  - implements ros2_controls, nav2, EKF, and a handheld controller. uses LIDAR, wheel encoders and an IMU for sensors.
    
https://github.com/tonitonitonitoni/asimov2026/tree/main
  - antonia's package for us
    
https://docs.nav2.org/setup_guides/index.html
  - nav2 set up guide, follow gazebo classic version unless that changes
    

1) Create the URDF files i.e. description/robot.urdf.xacro
    - create a file per each camera/sensor i.e. Orbbec Gemini 2 depth camera, motorized RGB turret camera, LIAR, IMU, (don't think we have wheel encoders).
        - some like the orbec have urdf.xacro files already made, so prob best to just google and see
            - https://github.com/westonrobot-dev/orbbec_sdk_ros2/blob/v2-main/orbbec_description/urdf/gemini2.urdf.xarco
    - for gazebou simulation, each file/thing needs its own gazebou reference
        - also, eachs sensor need its plugin set up to ensure it will publish its sensor_msgs/X
        -  https://docs.nav2.org/setup_guides/sensors/setup_sensors_gz_classic.html
    - https://docs.nav2.org/setup_guides/urdf/setup_urdf.html
    - https://github.com/mvipin/perceptor/blob/main/description/robot.urdf.xacro
    - https://github.com/ros-navigation/navigation2_tutorials/blob/rolling/sam_bot_description/src/description/sam_bot_description.urdf
    - https://github.com/tonitonitonitoni/asimov2026/blob/main/src/description/sam_bot_description.urdf

2) Create basic launch script i.e. launch/display.launch.py **technically done in asimov2026**
    - will be edited as other features like ros2_controls is implemented
    - Nav2's guide shows an in-progress display.launch.py file, their complete version (next link) shows more as they added more sensors and features.
    - https://docs.nav2.org/setup_guides/urdf/setup_urdf.html#build-and-launch
        
      
3) Create a basic RVIZ config file i.e. rviz/config.rviz **technically done in asimov2026**
    - scroll down more to the rviz/config.rviz file example https://docs.nav2.org/setup_guides/urdf/setup_urdf.html#build-and-launch
      
4) set up odometry via ros2_control and VIO **needs more research bc implementing ros2_controls is its own whole thing**
    - nav2 notes that ultimately we need both  **the nav_msgs/Odometry message** and **odom => base_link transforms to be published** so we need to keep this in mind when we set up odometry from the various sensors we have
    - **sensor_msgs/LaserScan should also be set up**
        - required for SLAM, see 6) 
    - https://docs.nav2.org/tutorials/docs/integrating_vio.html
        - tutorial to integrate VIO, which is going to be of interesting due to Orbbec Gemini 2, sadly this one uses ZED as an example
        - will prob need to refer to the ROS2 orbbec wrapper doc to set up the camera
            - https://orbbec.github.io/OrbbecSDK_ROS2/en/source/4_application_guide/services.html
            - https://github.com/westonrobot-dev/orbbec_sdk_ros2?tab=readme-ov-file#supported-devices
    - https://docs.nav2.org/setup_guides/odom/setup_odom_gz_classic.html
    - https://control.ros.org/kilted/doc/getting_started/getting_started.html#running-the-framework-for-your-robot
        - Antonia outlined using the Diff dRive plugin https://github.com/ros-controls/ros2_controllers/blob/master/diff_drive_controller/doc/userdoc.rst      
    - 1) create an YAML file in config folder
    - 2) edit the URDF to include the <ros2_control> required tags
        - https://control.ros.org/rolling/doc/getting_started/getting_started.html#hardware-description-in-urdf
        - https://github.com/ros-controls/roadmap/blob/master/design_drafts/components_architecture_and_urdf_examples.md
        - https://github.com/mvipin/perceptor/blob/main/description/ros2_control.xacro
    - 3) edit the launch file to add a node with the ros2_controls Controller Manager 
         
5) implement **robot_localization** package to combine all sensor's odometry and finally publish the  **odom => base_link** transform via EKF
    - https://docs.nav2.org/setup_guides/odom/setup_robot_localization.html
    - 1) create config/ekf.yaml file **file existsin asimov2026**
        - need to edit for the other sensors??? not sure, please someone look into it more
           - https://docs.ros.org/en/noetic/api/robot_localization/html/configuring_robot_localization.html   
          - look here for more parameters https://github.com/cra-ros-pkg/robot_localization/blob/rolling-devel/params/ekf.yaml 
    - 2) edit launch file to include the ekf_node, robot_state_publisher_node, and launch args **done in asimov2026**
    - 3) edit CMakeLists.txt to include the config  **done in asimov2026**
    - 4) Also do **navsat** somehow
        - https://docs.ros.org/en/noetic/api/robot_localization/html/navsat_transform_node.html
          
6) set up SLAM via slam_toolbox (or nav2_acml)
    - **note that both slam_toolbox and nav2_amcl use sensor_msgs/LaserScan inside /scan by default**
       - generally, both packages subscribe to /scan, where sensor_msgs/LaserScan is published
    - https://docs.nav2.org/setup_guides/sensors/mapping_localization.html
    - https://docs.nav2.org/tutorials/docs/navigation2_with_slam.html
    - 1) copy nav2_params.yaml from navigation2's sambot tutorial, and set up the global_costmap and local_costmap as needed (will also need to be worked on again in 7) )
        - the example uses both "LaserScan and PointCloud2 as the sensor sources"
        - https://github.com/ros-navigation/navigation2_tutorials/blob/rolling/sam_bot_description/config/nav2_params.yaml
        -  > example configuration that uses static layer, obstacle layer, voxel layer, and inflation layer. We set both the obstacle and voxel layer to use the LaserScan messages published to the /scan topic by the lidar sensor. We also set some of the basic parameters to define how the detected obstacles are reflected in the costmap.*
        -  https://github.com/mvipin/perceptor/blob/main/config/nav2_params.yaml
    - 2) seems to just need proper launch arguments and order after that
        - terminal1: ros2 launch sam_bot_description display.launch.py
        - terminal2: ros2 launch slam_toolbox online_async_launch.py use_sim_time:=true
        - terminal3: ros2 launch nav2_bringup navigation_launch.py use_sim_time:=true
          
7) set up the footprint
    - **required for collison avoidance**
    - need the footprint from the global_costanp and local_costmaps of 6). footprint cause rover, vs robot_radius for robots that can turn on itself like roombas
    - https://docs.nav2.org/setup_guides/footprint/setup_footprint.html
    - edit the footprint parameter inside nav2_params.yaml to match ASIMOV or new robot
    - should publish /local_costmap/published_footprint
      
8) implement the navigation plugins, which will include the **obstable avoidnace algs**
    - https://docs.nav2.org/configuration/index.html
    - https://docs.nav2.org/setup_guides/algorithm/select_algorithm.html
    - should be all done inside /nav2_params.yaml
    -  1) implement Planner. probably Smac Hybrid-A* planner 
        - > It is a highly optimized and fully reconfigurable Hybrid-A* implementation supporting Dubin and Reeds-Shepp motion models. This algorithm expands the robot’s candidate paths while considering the robot’s minimum turning radius constraint and the robot’s full footprint for collision avoidance. Thus, this plugin is suitable for arbitrary shaped robots that require full footprint collision checking. It may also be used for high-speed robots that must be navigated carefully to not flip over, skid, or dump load at high speeds.
        - https://docs.nav2.org/configuration/packages/smac/configuring-smac-hybrid.html
           - example set up code at bottom 
    - 2) set up controller server. 
        - we can set up different controllers for different tasks
        - example, DWB and TEB are for dynamic obstacle avoidance; RPP for path following; VP for low computation resources; etc
        - https://docs.nav2.org/plugins/index.html#controllers
        - https://github.com/ros-navigation/navigation2_tutorials/blob/rolling/sam_bot_description/config/nav2_params.yaml @line 78
    - 3) implement Smoother 
        - need to select one based on tasks
        - https://docs.nav2.org/plugins/index.html#smoothers
    - 4) implement behavor plugins?
        - not sure if we want, but we can
        - https://docs.nav2.org/configuration/packages/configuring-behavior-server.html 
    - 5) we can implement many more navigation plugins, like CollisionMoniter
        - https://docs.nav2.org/plugins/index.html
        - https://github.com/mvipin/perceptor/blob/main/config/nav2_params.yaml @ line 358 for collison monitor example
    - since we should be copying from github.com/ros-navigation/navigation2_tutorials/blob/rolling/sam_bot_description/config/nav2_params.yaml most will be already set up, but needs to be changed for specific plugins 
      
   




