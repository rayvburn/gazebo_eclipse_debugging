#### This tutorial is based on [this](https://www.ceh-photo.de/blog/?p=899), but targets debugging `gzserver` application (and its libraries/plugins) in particular.

In the given URL there is great explanation how to setup Eclipse to debug and run ROS nodes. In this page there is a reference to [URL](https://answers.ros.org/question/10043/how-do-i-start-a-launch-file-from-eclipse-ide/).

***

### Requirements
* Imported ROS/Gazebo project into Eclipse workspace (see [IDEs](http://wiki.ros.org/IDEs) for details)
* Installed gdbserver (apt-get install gdbserver)

***

### 1) Change `catkin` workspace configuration (if using ROS) or compile plugin with `DEBUG` configuration.

`catkin clean`

`catkin config -G"Eclipse CDT4 - Unix Makefiles" -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_COMPILER_ARG1=-std=c++11 -D__cplusplus=201103L -D__GXX_EXPERIMENTAL_CXX0X__=1`

`catkin build`

After debugging, one can wish to bring back previous configuration which probably was like below:

`catkin config -GEclipse CDT4 - Unix Makefiles -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER_ARG1=-std=c++11 -D__cplusplus=201103L -D__GXX_EXPERIMENTAL_CXX0X__=1`


***

### 2) Clean and build again the workspace:

`catkin clean` (prompt will ask whether to delete files - OK)

`catkin build`

***

### 3) Start the Eclipse binary from a bash session that has sourced the relevant `setup.bash` file.

The easiest way is to add these lines at the end of `~/.bashrc` so everything will be done automatically during terminal startup.
NOTE: this is only an example, one must adjust the paths according to his workspace configuration.


`export CATKIN_WS_PATH=${HOME}/ros_workspace/ws_people_sim`

`source /usr/share/gazebo-8/setup.sh 		# Gazebo Env Variables Setup`

`source ${CATKIN_WS_PATH}/devel/setup.bash 	# Catkin Workspace Setup`

`cd ${CATKIN_WS_PATH}`

Run Eclipse IDE simply with:

`eclipse`

[Ref](https://answers.ros.org/question/248045/file-optroskineticbinroslaunch-line-34-in-module-importerror-no-module-named-roslaunch/)

***

### 4) *THIS POINT CAN BE OMMITED AND CHECKED ONLY IF SOMETHING FAILS LATER*
PROBLEM: lack of `PyDev` (Eclipse seems to overwrite some Python env variable?)

"Take a view on `Window > Preferences > PyDev > Interpreter Python ...` You can use `Auto Config` or declare the path manually."

[Ref](https://answers.ros.org/question/10043/how-do-i-start-a-launch-file-from-eclipse-ide/)

***

### 5) To run Gazebo plugin from Eclipse IDE debugger one must edit `GAZEBO_PLUGIN_PATH` environmental variable so plugin files (`.so`) could be found. This step can be ommited when `GAZEBO_PLUGIN_PATH` is properly modified in `~/.bashrc` file.

Example:

*Make sure Eclipse IDE is closed*

`export GAZEBO_PLUGIN_PATH=$GAZEBO_PLUGIN_PATH:${FULL_PATH_TO_YOUR_GAZEBO_PLUGIN(s)_.SO_FILE(s)}`

`eclipse`

*NOTE* What's very convenient is to paste `export ...` inside Eclipse's `Run->Debug Configurations->C/C++ Remote Application->[Select your project]->Main->Commands to execute before application` so the only thing one could remember is to start Eclipse from terminal with sourced workspace.

***

### 6) In Eclipse choose `Run->Debug Configurations`. Try to unfold `C/C++ Remote Application` (double click), currently edited project should appear there. Click it.

***

### 7) Choose `C/C++ Remote Application -> Main`. In the `C/C++ Application` textbox paste a path to gzserver (`/usr/bin/gzserver`)

***

### 8) Jump into the `Arguments` tab. `gzserver` must be ran separately with proper arguments (without `roslaunch`). So the aim is to simulate execution of `$(find gazebo_ros)/launch/empty_world.launch` without `roslaunch`.

Documentation of `gzserver` can be found [here](https://www.mankier.com/1/gzserver).

Example:

`gzserver [options] world_file`

So it can be expanded to:

`gzserver -v --verbose -u -e ode ${FULL_PATH_TO_YOUR_WORLD_FILE}`

NOTE: some error occurs when any options are added and `gzserver` is ran from Eclipse so it may not work with other than default physics of Gazebo.

***

### 9) If the application uses Gazebo-ROS interface then one additional step must be taken.

One of `gzserver` run options is loading a `server plugin` (see `gzserver --help` for details).
To successfuly run gzserver one need to load 2 plugins related to Gazebo-ROS interface. To do this additional launch arguments should be added in Eclipse (they don't produce some errors in contrary to those options from `(8)`):

`Run->Debug Configurations->C/C++ Remote Application->[Select your project]->Arguments`

and now add this BEFORE `.world` file path:

`-s /opt/ros/kinetic/lib/libgazebo_ros_paths_plugin.so -s /opt/ros/kinetic/lib/libgazebo_ros_api_plugin.so`

So finally `Arguments` tab is filled with:

`-s /opt/ros/kinetic/lib/libgazebo_ros_paths_plugin.so -s /opt/ros/kinetic/lib/libgazebo_ros_api_plugin.so ${FULL_PATH_TO_YOUR_WORLD_FILE}`

***

### 10) Debugging session could be ran successfully with Eclipse IDE now. 
One must remember that only `gzserver` will be started. The rest of the system (including ROS nodes) must be run manually in separate terminal (outside Eclipse). Usually it's safer to run `gzserver` after calling `roslaunch ...` with the rest of the system.

***

For an example how to prepare such startup files see my `gazebo_ros_people_sim` repo (oops, private now) (`actor_sim_utils/launch/example_full.launch`).
Crucial thing here is the `gdb` argument. So the system ready for debugging should be started like:

`Eclipse IDE  -> Launch in Debug mode` (cockroach icon)

`command line -> roslaunch actor_sim_utils example_full.launch world:=living_room_vid_friendly gdb:=true`
