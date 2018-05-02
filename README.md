# Rough Terrain Exploration With a Legged Humanoid Robot

## Introduction

Legged robots have the potential to become a widespread in the society, since all of the infrastracture and

Legged robots are capable of traversing terrains the are simply inaccessible to the wheeled type of robots, however the accessibility to more difficult terrains come with the problem of traversing unstructured rough terrain. One way to approach the traversal of the unstructured terrain is to ensure the stability of it footsteps that the robot will take throughout the terrain.

The stability of a footstep can be described by using the characteristics of the terrain itself. As Chestnutt [1] describes in their paper the terrain traversability can be described via the slope, step height, roughness, and curvature of the area. Another problem in traversing unstructured terrain is finding the right sequence of footstep, a number of different methods were proposed that generate the sequence of footstep the most common, amongst the literature I have encountered is to use graph search techniques like A* search and its family after analyzing the terrain in question as a traversability grid where each cell contains the information about the local terrain data and giving each cell a cost based on the traverisiblity estimation, thus converting the terrain into a traversability map and letting the graph search algorithm find a path through out the terrain.

For this project I have chosen a different method that relies on variations of the RRT algorithms, t-RRT[2] and a hierarchical-RRT[3] algorithm and a hierarchical structure for their implementation[4]. The Unity engine served as a platform for visualization of my application of these algorithms.

# Methods

## Unity
The simulation is built with the Unity engine, mostly used for game development unity offers a number of features that are helpful for its application in the field of robotics. Unity engine offers powerful scripting framework that allows for extensive control and manipulations over the objects and models used in this simulation. The main reason for this choice for this problem was the research on the which framework to use, unity seemed to offer many of the features I was looking for especially in term of visualization.

Since Unity is primarily built for making video games, aside from the ability of moving the camera during the simulation and conversion of the heightmap into the terrain mesh all of the code was written from scratch.

##Traversability Map

## Transition Based RRT
Transition based RRT or t-RRT is a variant of the RRT that I am using in order to find a global path through which our footstep will be planned. t-RRT combines the exploratory strength of the RRT algorithm together with the stochastic optimization methods in order to generate the least cost path. Since we are working with an unstructured terrain which we convert into a traversability map that the t-RRT is using to find the global path described by a series of node that are supposed to guarantee a traversable path through the terrain.

## Hierarchical RRT

Hierarchical RRT is used to generate a sequence of footsteps between each of the nodes generated by the t-RRT. It works in a similar fashion as the multi-RRT algorithm where each step a number of nodes generated. In this case however the nodes are footstep that the robot can take to ensure a stable ZMP-type of gait.These footsteps must be within the reachable area of the non-swing foot and adhere to the stable transition of the robot model in use. While there is no set number of different nodes that can be generated each turn, there is a trade off between using too few and too many of them. While giving the Hierarchical RRT too few nodes decreases the time between each step it also limits the movement capabilities of the robot. Too many specified footstep grows the number of expanded nodes significantly thus increasing the time between each step, however it give room for a smoother movement.

Another core feature of Hierarchical-RRT algorithm is the use of AlignExtend function. This function is used for adjustment of each generated node that is either in collision or who’s cost is too high. This function works by using a specified range of for the configuration of parameters for each node. Generating a specified amount of random configurations from that range the nodes are tested for cost and collision, the first node to be accepted is the node that added to the tree and the rest of the nodes from this function are discarded.
Hierarchical-RRT is also capable of adjusting the gait of the robot by defining which node is the “closest”. Instead of using euclidean distance the algorithm considered the orientation of each potential and chooses the best combination based on the weights given. A number of different approaches were described in the paper but for this project I chose quickly expanding orientation for the purpose of higher rate of local terrain exploration.

## Overall Structure

The overall structure was inspired by the a similar hierarchical implementation of footstep checking as described in [5]. The hierarchical structures checks the potential region for a footstep starting from a broad region and reducing the size of a region until only the foot itself fits into the specified region. This structure was implemented in attempt to increase the computational efficiency when traversing terrain with large areas of low cost surface. As soon as the traversability cost function identifies the area of 0 cost, we can immediately add our potential nodes to the tree and move on to the next step. If, however our error checking does identify a non zero cost for our area we proceed a layer deeper to check a smaller area, and if
needed again we check the exact area of the foot for fitting to validate this potential step.

# Results
First we evaluate the ability of the t-RRT algorithm to find a path through rough terrain. I approached testing this algorithm by looking at three variations of the terrain, stair case, a smooth path through rough but passable terrain, and a doorway.

## Traversability Map Visualization
Travesibility map visualization takes in 4 parameters to determine high and low cost areas, 3 weights that determine how we treat the characteristics of the local region and the 4th parameter determines the size of the region. The weights have a significant impact on the way the terrain is treated and what counts for passable and impassable. At the same time the size of a local region acts as a padding designating certain regions as high cost or not based on all the point in the area. We can see the effect of varying these input in the following images.
Balanced: Slope Weight - 0.5, Step Height Weight - 0.25, Roughness Weight - 0.25

High Slope Weight

High Step Height Weight

High Roughness Weight

High Local Area

Small Local Area

## t-RRT

![](https://i.imgur.com/BqJeobI.gif)

![](https://i.imgur.com/Jby7gE0.gif)

![](https://i.imgur.com/1IUawSV.gif)


My variation of the t-RRT algorithm includes interpolation for additional checking, which is done by subdividing the distance of between the new node and the closest node into ten point and checking each points. While this does help to avoid setting points that cannot be reached when the maximum distance to a new node (epsilon) is small relative to the terrain and the obstacles. Problems bebegin to occur we increase the size of epsilon relative to the size of the terrain and the obstacles

![](https://i.imgur.com/tc1QJKo.gif)

As we can see in the example points are being considered that are otherwise unreachable by the robot. We can see the beginning of this problem in the second gif with rough but passable terrain and it becomes worse as we increase the epsilon value. Nodes of the global path are generated in places where while it is possible for the robot to transverse are not optimal when considering stability.

## Generating Steps
Hierarchical-RRT (HRRT) is able to generate the sequence of steps to reach each midpoints that are by the t-RRT however parameters for this algorithm must be carefully. Specifically the distance between each points of the path affects the performance of the HRRT. The longer distance between the points of the path the longer RRT will take to solve and the more deviation from the specified path will occur, which is not so much of a problem on its own, but when there is a sequence of sharp turn that the robot must take time it takes to find the necessary footstep increase significantly. Similar problem occurs epsilon is small compared to the size of the footsteps. This can be seen the two following examples.

Low epsilon value and large footstep

High epsilon value and small footstep

The best options is to tune the size of the footsteps and the epsilon used to the t-RRT, these tuned parameters may produce a much better sequence of steps as we can see in the following example.




# Conclusion
	While it is possible for the described setup to discover a sequence of stable footprints that the our robot can use to traverse the unstructured rough terrain, when applying this algorithm the parameters for both layers of the search must be carefully chose which may be acceptable when working with know terrain however unknown terrain will be detrimental for this approach. However the general structure of this approach account for the ability to swap the two part as only minimal data is passed between them.
	As we have seen the quality of the path and the efficiency in which it is found depends on both algorithms you can replace the t-RRT with a completely different approach as long as the input into the HRRT consists out a set of points that described the global path the our robot can take. HRRT is much more rooted in this model but with some modification it can also be swapped with a different algorithm as well. As mentioned above the two input points actually describe an area of traversability in the algorithm can find the stable footsteps. The implementation of a different algorithm for this layer is possible in this regard. Algorithms from the A* search family, designed for generating footsteps for legged robots, need to only convert the area described by the two nodes into a grid and search through it.


## Using and Installing

To download this repository you can use the following string in the command line
`git clone git@gitlab.com:Scathach/height_map.git`
Or just download the zip file with the link on the top of the page.
To test some of the simulation either launch the the following files directly
* Item
* Item

Or if you are familiar with the unity setup you can import the project into you Unity workspace and change the different variables in the comm script on the right to see how the simulation reacts.

## Additional Comments


# References
1.   Chestnutt, Joel E.. “Navigation Planning for Legged Robots.” (2006).
    URL: https://pdfs.semanticscholar.org/8762/98fac00a165ecc90bb4f965f77e79ff74732.pdf

2.   L. Jaillet, J. Cortes and T. Simeon, "Transition-based RRT for path planning in continuous cost spaces," 2008 IEEE/RSJ International Conference on Intelligent Robots and Systems, Nice, 2008, pp. 2145-2150.
    doi: 10.1109/IROS.2008.4650993
    URL: http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=4650993&isnumber=4650570

3.   H. Liu, Q. Sun and T. Zhang, "Hierarchical RRT for humanoid robot footstep planning with multiple constraints in complex environments," 2012 IEEE/RSJ International Conference on Intelligent Robots and Systems, Vilamoura, 2012, pp. 3187-3194.
    doi: 10.1109/IROS.2012.6385836
    URL: http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=6385836&isnumber=6385431

4.   A. Chilian and H. Hirschmüller, "Stereo camera based navigation of mobile robots on rough terrain," 2009 IEEE/RSJ International Conference on Intelligent Robots and Systems, St. Louis, MO, 2009, pp. 4571-4576.
    doi: 10.1109/IROS.2009.5354535
    URL: http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5354535&isnumber=5353884

5.  M. Wermelinger, P. Fankhauser, R. Diethelm, P. Krüsi, R. Siegwart and M. Hutter, "Navigation planning for legged robots in challenging terrain," 2016 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS), Daejeon, 2016, pp. 1184-1189.
    doi: 10.1109/IROS.2016.7759199
    URL: http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7759199&isnumber=7758082
