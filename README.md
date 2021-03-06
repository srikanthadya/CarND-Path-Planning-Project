# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program

### Goals
The goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

#### The map of the highway is in data/highway_map.txt
Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

| XY Coordinate | Frenet Coordinate |
|:----------------:|:--------------:|
![](HighwayXY.png)|![](FrenetS.png)|

## Why use Frenet Coordinates?

Representing curved road profiles in a cartesian coordinate system is sometimes cumbersome. Tasks like determining the lane in which a car is or if two cars are in the same lane , in cartesian coordinates, adds unneccessary complications. Projecting the ***(x,y)*** s to the Frenet coordinate space helps us over come this problem. The `s` dimentsion (longitudinal axis) increases with the length of the lane . The `d` dimension (lateral axis) is centered on the middle yellow line. In this project the lanes are 4m wide and the highway has three lanes. That implies, 

| d range | lane |
|:-------:|:-----:|
` 0 < d < 4 ` |left lane|
`4 < d < 8 ` | center lane |
` 8 < d < 12 ` | right lane |

![](cartVsFre.png)

## Path planning approach

The tasks involved in this Highway driving challenge can be broken down into three parts

1. Localization and Sensor Fusion
2. Behavior Planning 
3. Trajectory planning

### Localization and Sensor Fusion 

The simulator feeds us the localization and sensor fusion data . Localization data gives us the precise location of the ego car. In our case we can get the ego car's (x,y) location, (s,d) location, its heading direction and the speed from the localization data. This can be found between lines 80-85 in `main.cpp`.

The sensor fusion data on the other hand gives us the information on the surrounding environment. If the onboard sensors (RADAR/LiDAR) senses vehicles around the ego car, their corresponding (s,d) and speed are extracted. With the current position and speed, we can estimate where the neighboring cars will be in the future as well. This can be found between lines 118-125 in `main.cpp`.

### Behavior Planning

A Finite State Machine approach has been followed in this project to define the behavior of the ego vehicle. The finite states that can exist are 

1. Keep Lane
2. Prepare Lane Change Left/Right
3. Lane change Left  
4. Lane change Right 

![](FSM.png)

### Keep Lane
While the ego car is driving within the speed limit, if there is no car ahead in the lane that could cause a deceleration, this state makes the car stay in the current lane. 

### Prepare Lane Change Left/Right
If we sense a car ahead in our current lane, we first look to the left lane (provided we are not in the left most lane already) and check if there is a car adjacent to us or within 30m ahead or behind the ego car. 
Else if there is a car detected in the left lane we perform the same check on the right lane.

### Lane Change Left/Right 
If there is no car detected in the left lane, we change left by setting the current lane to left lane. Else if there is no car to the right of the ego car, we prepare to change to the right by setting the current lane to the right lane. 

If there is no room to change either left or right, we reduce the speed to avoid collision with the car ahead. 

Finally if there is no car ahead and we are below the speed limit, we gradually increase the speed to reach the allowable speed such that the jerk condition is not violated. 


## Trajectory Planning

The trajectory that the ego car needs to take to drive around the simulator track should be such that there are no sudden lateral or longitudinal accelerations that could case the ego car to violate the jerk constraint. To meet this criteria, this implementation follows the spline approach suggested in the walkthrough video. 

A spline fits a smooth curve through the points we desire to pass through. In this case we use 5 control points through which we want the spline to be fit. Of these 5 points, we reuse two points from the previous path so that the transition is smooth between the two splines. The other three points are chosen such that they are 30, 60 and 90 meters ahead of the ego car. This can be found between lines 189-250 in `main.cpp`.

The spline function can now be used to find all the points required for a smooth path curve. Assuming we want to fit a curve that takes us 30m ahead, given a reference velocity and the controller update rate of the simulator, we can get the number of points required and find their interpolated x and y points that can be fed back to the simulator. 


## Result 

The car successfully went around the simulator track without any collision or jerk violation. The final state is shown below. The link to the video can be found [here](https://youtu.be/0Rb1QsdWksU) 

![](result.png)

## Improvements

There are several limitations due to the simplified Finite State Machine assumption used in this implementation. 

1. The ego car always choses the left lane first irrespective of the lane availability. The car doesnt have a way to identify the faster of the two lanes to make a lane change decision. 
2. The cost associated with lane change is only binary in this implementation ie. safe to change lane or not. A more descriptive cost function could help go around the track faster.
3. The sensor fusion data is used to check where the neighboring car is going to be in its own lane at a future time. Their heading directions are not used and hence if the tracked car does a sudden lane change we do not have a way to account for it. 


----------------------------------------------------------------------------------------------------------------------------------   
### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).  

To run the simulator on Mac/Linux, first make the binary file executable with the following command:
```shell
sudo chmod u+x {simulator_file_name}
```

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time. 

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values 

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates. 

## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.

## Tips

A really helpful resource for doing this project and creating smooth trajectories was using http://kluge.in-chemnitz.de/opensource/spline/, the spline function is in a single hearder file is really easy to use.

---

## Dependencies

* cmake >= 3.5
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!


## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).

