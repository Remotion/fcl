## FCL -- The Flexible Collision Library


FCL is a library for performing three types of proximity queries on a pair of geometric models composed of triangles. 
 - Collision detection: detecting whether the two models overlap, and optionally, all of the triangles that overlap.
 - Distance computation: computing the minimum distance between a pair of models, i.e., the distance between the closest pair of points.
 - Tolerance verification: determining whether two models are closer or farther than a tolerance distance.
 - Continuous collision detection: detecting whether the two moving models overlap during the movement, and optionally, the time of contact.
 - Global penetration depth: detecting the minimum motion required to separate two in-collision objects, and return the configuration where the collision has been resolved.
 - Contact information: for collision detection, continuous collision detection and global penetration depth, the contact information (including contact normals and contact points) can be returned optionally.

FCL has the following features
 - C++ interface, heavily use the boost
 - Compilable for either linux or win32 (both makefiles and Microsoft Visual projects can be generated using cmake)
 - No special topological constraints or adjacency information required for input models – all that is necessary is a list of the model's triangles
 - Supported different object shapes:
  + sphere
  + box
  + cone
  + cylinder
  + mesh
  + octree (optional, octrees are represented using the octomap library http://octomap.github.com)


## Installation

CMakeLists.txt is used to generate makefiles in Linux or Visual studio projects in windows. In command line, run
``` cmake
mkdir build
cd build
cmake ..
```
Next, in linux, use make to compile the code. 

In windows, there will generate a visual studio project and then you can compile the code.

### Interfaces
Before starting the proximity computation, we need first to set the geometry and transform for the objects involving in computation. The geometry of an object is represented as a mesh soup, which can be set as follows:

```cpp
// set mesh triangles and vertice indices
std::vector<Vec3f> vertices;
std::vector<Triangle> triangles;
// code to set the vertices and triangles
...
// BVHModel is a template class for mesh geometry, for default OBBRSS template is used
typedef BVHModel<OBBRSS>* Model;
Model* model = new Model();
// add the mesh data into the BVHModel structure
model->beginModel();
model->addSubModel(vertices, triangles);
model->endModel();
```

The transform of an object includes the rotation and translation:
```cpp
// R and T are the rotation matrix and translation vector
Matrix3f R;
Vec3f T;
// code for setting R and T
...
// transform is configured according to R and T
Transform3f pose(R, T);
```


Given the geometry and the transform, we can also combine them together to obtain a collision object instance and here is an example:
```cpp
//geom and tf are the geometry and the transform of the object
BVHModel<OBBRSS>* geom = ...
Transform3f tf = ...
//Combine them together
CollisionObject* obj = new CollisionObject(geom, tf);
```

Once the objects are set, we can perform the proximity computation between them. All the proximity queries in FCL follow a common pipeline: first, set the query request data structure and then run the query function by using request as the input. The result is returned in a query result data structure. For example, for collision checking, we first set the CollisionRequest data structure, and then run the collision function:
```cpp
// Given two objects o1 and o2
CollisionObject* o1 = ...
CollisionObject* o2 = ...
// set the collision request structure, here we just use the default setting
CollisionRequest request;
// result will be returned via the collision result structure
CollisionResult result;
// perform collision test
collide(o1, o2, request, result);
```

By setting the collision request, the user can easily choose whether to return contact information (which is slower) or just return binary collision results (which is faster). 


For distance computation, the pipeline is almost the same:

```cpp
// Given two objects o1 and o2
CollisionObject* o1 = ...
CollisionObject* o2 = ...
// set the distance request structure, here we just use the default setting
DistanceRequest request;
// result will be returned via the collision result structure
DistanceResult result;
// perform distance test
distance(o1, o2, request, result);
```

For continuous collision, FCL requires the goal transform to be provided (the initial transform is included in the collision object data structure). Beside that, the pipeline is almost the same as distance/collision:

```cpp
// Given two objects o1 and o2
CollisionObject* o1 = ...
CollisionObject* o2 = ...
// The goal transforms for o1 and o2
Transform3f tf_goal_o1 = ...
Transform3f tf_goal_o2 = ...
// set the continuous collision request structure, here we just use the default setting
ContinuousCollisionRequest request;
// result will be returned via the continuous collision result structure
ContinuousCollisionResult result;
// perform continuous collision test
continuousCollide(o1, tf_goal_o1, o2, tf_goal_o2, request, result);
```
For global penetration depth computation, we need a pre-computation step. Beside that, the online penetration depth function uses exactly the same pipeline as shown above:
```cpp
// Given two objects o1 and o2
CollisionObject* o1 = ...
CollisionObject* o2 = ...
// the precomputation step for global penetration computation (see test_fcl_xmldata.cpp for an example)
...
// set the penetration depth request structure, here we just use the default setting
PenetrationDepthRequest request;
// result will be returned via the penetration depth result structure
PenetrationDepthResult result;
// perform global penetration depth test
penetrationDepth(o1, o2,request, result);
```