# Model Predictive Control Project

Here I will consider the [rubric](https://review.udacity.com/#!/rubrics/896/view) points individually and describe how I addressed each point in my implementation.

## Compilation
### 1. Your code should compile.
_Meets Specifications: Code must compile without errors with cmake and make._

Yes, it does compile on my MacBookAir with MacOS Sierra (10.12.6).

## Implementation
### 1. The Model
_Meets Specifications: Student describes their model in detail. This includes the state, actuators and update equations._

The model is a kinematic bicycle model as described below.
```
x_[t+1] = x[t] + v[t] * cos(psi[t]) * dt
y_[t+1] = y[t] + v[t] * sin(psi[t]) * dt
psi_[t+1] = psi[t] + v[t] / Lf * delta[t] * dt
v_[t+1] = v[t] + a[t] * dt
cte[t+1] = f(x[t]) - y[t] + v[t] * sin(epsi[t]) * dt
epsi[t+1] = psi[t] - psides[t] + v[t] * delta[t] / Lf * dt
```
_x_ and _y_ denote the position vector of the car, _psi_ is the heading direction, _v_ is the velocity, _cte_ is the cross-track error and _epsi_ is the orientation error. _Lf_ is the distance between the center of mass of the vehicle and the front wheels and affects the maneuverability. The vehicle model can be found in the class _FG_eval_. The limitations of the kinematic bicycle model as opposed to a 9 degrees of freedom model are described [here](http://ieeexplore.ieee.org/document/7995816/). 

### 2. Timestep Length and Elapsed Duration (N & dt).
_Meets Specifications: Student discusses the reasoning behind the chosen N (timestep length) and dt (elapsed duration between timesteps) values. Additionally the student details the previous values tried._

The time _N dt_ defines the prediction horizon. Short prediction horizons lead to more responsive control, but are less accurate and can suffer from fluctuations. Long prediction horizons lead to smoother controls. For a given prediction horizon, a shorter time step _dt_ leads to more accurate control but also require a larger MPC problem to be solved, thus increasing the latency. I chose the values of _N_ and _dt_ such that drives the car smoothly around the track for velocity between approximately 25 mph and 70 mph (_N=12_, _dt=0.05_).

### 3. Polynomial Fitting and MPC Preprocessing.
_Meets Specifications: A polynomial is fitted to waypoints. If the student preprocesses waypoints, the vehicle state, and/or actuators prior to the MPC procedure it is described._

The coordinates of waypoints in vehicle coordinates are obtained by first shifting the origin to the current poistion of the vehicle and a subsequet 2-D rotation to align the x-axis with the heading direction. A cubic polynomial is then fitted to the waypoints. The transformation between coordinate systems is implemented in _transformGlobalToLocal_. The transformation used is 
```
waypoints(0,i) =   cos(psi) * (ptsx[i] - x) + sin(psi) * (ptsy[i] - y);
waypoints(1,i) =  -sin(psi) * (ptsx[i] - x) + cos(psi) * (ptsy[i] - y);
```
Ihe initial position of the car and heading direction are zero in this frame. Therefore, the initial state of the car in the vehicle cordinate system is 
```
state << 0, 0, 0, v, cte, epsi;
```

### 4. Model Predictive Control with Latency.
_Meets Specifications: The student implements Model Predictive Control that handles a 100 millisecond latency. Student provides details on how they deal with latency._

I deal with the latency problem from the current position and time onwards. I constrain the control to the values of the previous iteration for the latency duration. Computation of optimal trajectory begins after the latency period. The values fed to the simulator are considered as the first control parameters of the optimal trajectory:
```        
Solution sol = mpc.Solve(state, coeffs);
double steer_value = sol.Delta.at(latency_ind);
double throttle_value= sol.A.at(latency_ind);
```

## Simulation

### 1. The vehicle must successfully drive a lap around the track.
_Meets Specifications: No tire may leave the drivable portion of the track surface. The car may not pop up onto ledges or roll over any surfaces that would otherwise be considered unsafe (if humans were in the vehicle)._

Yes, it does successfully drive a lap around the track without leaving the drivable portion of the track surface.


## References
1. https://jeremyshannon.com/2017/06/30/udacity-sdcnd-mpc.html
2. https://github.com/NikolasEnt/Model-Predictive-Control
3. https://github.com/jessicayung/self-driving-car-nd/tree/master/term-2/p5-model-predictive-control
4. https://medium.com/@mithi/a-review-of-udacitys-self-driving-car-engineer-nanodegree-second-term-56147f1d01ef
5. https://medium.com/@NickHortovanyi/carnd-controls-mpc-2f456ce658f
6. https://medium.com/self-driving-cars/five-different-udacity-student-controllers-4b13cc8f2d0f
7. http://stamp3.com/download/model-predictive-control-for-self-driving-car-109-mph-run-unstable
8. http://hectorratia.com/index.php/2017/07/25/model-predictive-controller-udacitys-autonomous-car-engineer-nanodegree-term-2-project-5/

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
Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.
* Fortran Compiler
* Mac: `brew install gcc` (might not be required)
* Linux: `sudo apt-get install gfortran`. Additionall you have also have to install gcc and g++, `sudo apt-get install gcc g++`. Look in [this Dockerfile](https://github.com/udacity/CarND-MPC-Quizzes/blob/master/Dockerfile) for more info.
* [Ipopt](https://projects.coin-or.org/Ipopt)
* Mac: `brew install ipopt`
+  Some Mac users have experienced the following error:
```
Listening to port 4567
Connected!!!
mpc(4561,0x7ffff1eed3c0) malloc: *** error for object 0x7f911e007600: incorrect checksum for freed object
- object was probably modified after being freed.
*** set a breakpoint in malloc_error_break to debug
```
This error has been resolved by updrading ipopt with
```brew upgrade ipopt --with-openblas```
per this [forum post](https://discussions.udacity.com/t/incorrect-checksum-for-freed-object/313433/19).
* Linux
* You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/).
* Then call `install_ipopt.sh` with the source directory as the first argument, ex: `sudo bash install_ipopt.sh Ipopt-3.12.1`. 
* Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
* Mac: `brew install cppad`
* Linux `sudo apt-get install cppad` or equivalent.
* Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.

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

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
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
