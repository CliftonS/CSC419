# CSC419 - Fast and Robust QEF Minimization using Probabilistic Quadrics

https://graphics.rwth-aachen.de/media/papers/308/probabilistic-quadrics.pdf

http://www.cs.cmu.edu/~garland/Papers/quadrics.pdf

By Clifton Sahota - 1005179860

Core Files: main.cpp, qef.h, qef_probabalistic.h, double_min_heap.h, qef.cpp, qef_probabalistic.cpp, double_min_heap.cpp

Overview:

This paper expands upon classical quadric functionality for removing the "best" vertices in meshes through decimation via pair contraction. 
In both this and past papers a 4X4 matrix Q is assigned to each vertex through which cost and optimal new vertex locations are. These Q matrices are
determined by adding up matrices K_n for each face touching a given vertex.

The 1998 paper which proposed classical quadrics uses the homogenous variant of vectors to allow for 4X4 instead of just 3X3, and has the K_n matrices 
constructed via the normal in such a way that cost will increase based on distance to the plane. When two Q's are added together to determine the cost of
contracting a given edge, this then will highly prioritze flat faces, keeping as many geometric features as possible.

This new paper uses the same base costs, however adds in sigma_n, the a chosen 3X3 matrix for each face which corresponds to a chosen variance of the normal
for that face. This sigma_n is further multiplied to be some percent of edge length for a given face/region, if chosen as in the paper. This sigma_n by 
construction is guaranteed to have a rank of 3, so combined with the K_n's the optimal point can always be solved for with x_min = A^-1 * b, allowing for
faster running speed. Additionally, based on the selection of sigma_n and the associated percent (s_percent in my implementation), this paper's implementation
will also smooth surfaces somewhat, and may have a more even distribution of edge contractions across the shape, since the percent is effected by 
edge length in their choice of it in the paper for the cubic examples.

The paper also has a non-planar triangular probablistics under guassian noise variant which I have not implemented. Similar to in the other methods it uses
a 4X4 Q matrix as the base measure of cost and to determine the optimal location for contraction. It however utilizes the guassian's far more greatly,
having 3 new distributions. It uses a method more similar to the original quadric, in using p normal values instead of normal values as in this paper's
planar probablistic quadric. It does not use the same base fully, using only parts and variables from the original, instead opting to utilize Ci;
the cross interference matrix matrix, a symmetric 3X3 matrix. Like in the planar portion, the triangular portion also derives the distribution result for 
each of A, b, c to contstruct the Q matrices, with each being modified seperately and the most complicated being c.

All methods have shared general structure:
1. Determine initial costs/optimal locations for contractions
2. Insert into a min_heap to retrieve lowest costs as needed
3. Remove lowest cost; re-insert with significantly increased cost if performing contraction would cause faces to flip
4. Update vector positions to new optimal positions; 
5. Move all links from second vector to the first 
6. Remove any now duplicate edges
7. Update effected face's normals
8. Update edges with costs effected by the contraction, removing then finding new optimal locations/costs and re-inserting into the min_heap
9. Repeat until enough vertices have been removed
10. Remove now missing/invalid faces from the output (may also remove unused vertices if desired, though this will not effect igl display)

The key differences are the cost functions, with the remaining core functionality being very similar. 60% of qef and qef_probabalistic are the same for example,
with both parts sharing the same structure.

Completed Paper Parts (description):

I have fully completed the 1998 classical quadrics paper in qef.cpp, with the exception of simplifiying the decision process when choosing an optimal point
for a given selected pair of vertices to contract to. Some edge cases were not covered, such as making sure that edge vertices are not given seperate
treatment, so in cases where the mesh has one or more boundaries, it may change the shape of the boundary. It can be seen in the outputs that it focused 
on flat areas to contract vertices in.

Examples:

Input pelvis (./prob 0 ../data/pelvis.obj):

Output pelvis (./prob 0.20 ../data/pelvis.obj):

Input cactus (./prob 0 ../data/cactus.obj):

Output cactus (./prob 0.40 ../data/cactus.obj):

Input cube (./prob 0):

Output cube (./prob):

The planar probablistic quadric is implemented in qef_probabalistic.cpp. It is similar to the classical planar quadric, with the core difference being
in that is has an added sigma_n value. I selected this value to be of the principal axis of the normal on the planes. It appears to have a smaller effect
then in the examples provided in the paper, however nonetheless has a notable impact. The change in calculation of the Q matrix, cost and cost minimizing
position for the contraction are the main differences from the original paper.

Examples:
Input Sphere:

Output Sphere:

Input Pelvis:

Output Pelvis:

Instructions (setup):
1. Extract folder somewhere (which you have already done due to reading this)
2. Navigate to the top level of the folder using terminal/command prompt (with this file and main.cpp in it)
3. mkdir build
4. cd build
5. cmake ..
6. make

For step 5 may optionally use [cmake -DCMAKE_BUILD_TYPE=Release ../] for substantially faster run times, helpful especially for large numbers or removed vertices or for large data files.

Optimizations that could be used if expanded upon in qef and qef_probabalistic would consist of adding a map to the double min heap so that references can be found without iterating through the array for a particular one. Additionally keeping track of which faces a given vertex are a corner of would also increase the running speed. In general based on the optimization sections of the original papers, these implementation become slow very quickly with larger objects.

Use pelvis.obj or torus.obj for faster run times as they are much smaller objects.

Instructions (Running options):
1. Perform qef.cpp on a 8 vertex cube: ./prob
2. Perform qef.cpp on an obj file removing 20% of vertices: ./prob 0.20 ../data/cactus.obj
3. Perform qef_probabalistic on an obj file with given s_percent value (5%) removing 20% vertices: ./prob 0.20 ../data/cactus.obj 0.05

I reccomend limiting to less than 0.40 (40%) removed vertices for larger files, as it take a lot longer to run, due to needing to discard changes which would invert faces otherwise.

s_percent value is read as a double, so inputing 5 will have very different results from inputting 0.05.

Video?
