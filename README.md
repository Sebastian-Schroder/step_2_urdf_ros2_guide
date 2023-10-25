
## Creating a URDF file

A urdf can either be created from scratch, by editing a pre-existing urdf or built using specific software packages. We've found from experience that making a working and reliable URDF is a time consuming an difficult process, therefore we use a [add-on pack for solidworks](http://wiki.ros.org/sw_urdf_exporter) that allows us to directly convert a model from a step file to a urdf model without needing to spend time moving models into place and ensuring that all the relationships are correct. 

this section should run you through the steps needed to make your own URDF file from a prexisting model

### 1.  Convert the model
1.  download the above add-on pack for solidworks
2. convert your assembly into individual step files and reconstruct the arm via those step files in solidworks
3. Ensure that the arm is able to move in solidworks i.e. not fixed.
---
### 2. Using the add-on pack
1. download the correct version and install the add-on pack by putting the extracted folder inside your solidworks. once done there should be an `export to URDF` option in the `tools` menu bar.
2. start `export to URDF`
3. in the bottom left, right click and build out a tree of the joints as demonstrated ![URDF_Exporter](/media/Solidworks_Exporter.png)
4. for each joint, you need to give a `name`, a `joint name`, `joint type`, and select the body for each joint. to select a subassembly its best to select it via the design tree, rather than selecting the components individually.
5. Leave `Reference Coordinate System` `Reference axis` to automatically generate.
6. once this is done click preview and export to automatically generate all the reference angles and coordinate systems
7. A secondary windows will open up. just close this windows for now. ![Close this window](/media/Assembly_Exporter.png)
---
### 3. Fixing the frames / axes
1. you should see x frames and <= x axes added to the Design tree, with names like `Origin_<name of joint>` and `Axis_<name of joint>`. the position of these are important. first add axes via reference geometry for each missing one, **however the model will not rotate around this axes, instead it will rotate around the Origin of the model in the direction of the axis** by default the Coordinate frames for each model are placed at the origin of the model and thus the intersection of all three main construction planes. if for instance these planes do not coincide with the axis of rotation of the model. then add a cooridnate frame at a point in which it does.

![Overridden Origin Point](/media/Overridden%20Origin.png)

for example in this case, the URDF exporter would put the components origin point at the red dot. this however doesn't line up with the axis of rotation I want, therefore I create a new coordinate frame in which it does.

2. once you have all the axes and origin points, it's time to redo the process. `tools-> export to URDF` and then under reference axes and coordinate frames you put all the ones that worked. **do not leave any as automatically generate**. once done click preview and export
3. once the exporter tab is opened, go through each of the joints and fix any that have had their joint_type changed (idk why this happens).
4. ensure that the Origin and axis values are correct. the origin position values should be the distance from the origin of the previous joint to the origin of this joint, in the reference frame of the previous joint. the axis should be in **this** joints reference frame. 
5. ensure that youy've set joint_limits. the calibration, dynamics and safety controls are optional, then click `next`.
6. on this page you are able to update inertial values such as the moment of inertia, this should be generated from the original model as the step files used to create the assembly in solidworks doesn't contain any material properties. (colour doesn't do anything).
7. once done, always export URDF **and** meshes, this will create a ros1 package.

---
## 4. converting to ros2
converting it to ros2 is very simple. 

1. create a package using `ros2 pkg create`. make sure the name of the package is the same as the ros1 package, if not, just go through the ros1 package files and replace all instances of the first name with the new name.
2. copy the meshes, textures& urdf folder into the new ros2 package
3. build and source the package `colcon build`

## Adding the URDF file to MoveIt
this process will use the moveit setup assistant to improve the urdf and allow it to be visualised using moveit rviz

1. run the moveit setup assistant via 
```bash
 ros2 launch moveit_setup_assistant setup_assistant.launch.py
```
2. from here click create new moveit configuration package and then select the `.urdf` file in your ros2 package. 
    - if it states that it cannot find the package you made earlier, then ensure you've sourced it.
    - if its stating that it cannot find files in the  `install/<package_name>/share/<package_name folder>` folder, you can get colcon to put it there by adding the following line to the `cmakelist.txt` file
```cmake
    #this will copy the directory urdf in the same folder as the cmakelists.txt file into the 
    # share/${PROJECT_NAME} folder when you run colcon build
    install(DIRECTORY urdf DESTINATION share/${PROJECT_NAME})
```
3. you can now check if everything you've done before is working fine go through each of the tabs on the left in order, roughly following [this guide](http://docs.ros.org/en/kinetic/api/moveit_tutorials/html/doc/setup_assistant/setup_assistant_tutorial.html). once you reach the poses tab, you can move the arm around and check if the axes/ coordinate frames are working. if it looks fine, then continue on with this process.
4. once you've gone thorugh, give the package a distinct name and store it in the same src folder with your previous package. now build the new package with `colcon build --packages-select <package_name>` and source them. 
5. there should be a demo launch file which can be run to test if the model is working fine. run to open rviz.
```bash
ros2 launch <package name> demo.launch.py
```
6. move the model and observe to see if the model has any issues. if it does and you know how to fix it, then you can either edit to urdf for simple fixes, or go back to the solidworks model for larger problems. 



7. To get the model working in Gazebo, the URDF file will need to adjusted further, check the simulation readmes on how to do this.
