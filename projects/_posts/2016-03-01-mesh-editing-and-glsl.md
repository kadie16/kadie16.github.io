---
layout: post
title: "C++ Mesh Editing + GLSL Shaders"
date: 2016-03-01
backgrounds: 
    - ../images/part5/up_teapot.png
    - ../images/part6/reflectbug.png
thumb: ../images/part6/reflectbug.png
category: Project
tags: 
---



<div>
        <p>In this project, I first learned to tesselate bezier patches to generate triangular meshes from bezier curves. Then, I implemented the ability to modify those meshes by flipping and splitting edges, or "upsampling" to increase the amount of triangle faces that represent the mesh. Finally, I wrote some cool GLSL shaders. My favorite was the reflection shader, which maps each pixel on the surface to a place on a reflection texture. </p>

    <h2 align="middle">Part 1: Fun with Bezier Patches</h2>
        <p>The first thing I needed to do was generate a triangle mesh from an input Bezier Surface. A Bezier surface is specified by control points (16, in this case). The control points specify curves, and the convex hull of the control points contains a surface. That surface is the mesh I am trying to represent. </p>
        <p>In order to represent the surface as a triangular mesh, I need to tesselate the Bezier surface specified by the control points into triangles. I accomplished this using the Bernstein Polynomials.I wrote some helper functions to help me evaluate the Bernstein Polynomials, used them to evaluate each control point. Once I determined which triangles I needed, I added them to a halfedge data structure to represent the mesh. </p>
                    <img src="../images/part1.png" />
                    <figcaption align="middle">My Rendering of the teapot.bez Mesh</figcaption>
        <h2 align="middle">Part 2: Average normals for half-edge meshes</h2>
        <p>Next, I computed per vertex normals that were an average of the normals of their faces. This results in a nicer, smoother shading effect than when normal vectors are just computer per face. To accomplish this, I traversed the faces that the vertex belonged to in the half edge data structure, computing the normal for each face along the way. First I got the half edge that the vertex was attached to. Then, I saved it as h_original so I could keep track of where I started. Then I got the halfedge's twin and next half edge, and computed the vectors those half edges represented. I computed the normal of that face by crossing those two vectors before advancing to the next face. Once I reached h_original again, indicating I had tranversed all of the faces of that vertex, I returned the unit vector of the sum of all of the normal vectors as the normal vector for that particular vertex. </p>
                    <img src="../images/part2/facenormals.png" />
                    <figcaption align="middle">Each vertex is shaded uniformly according to the normal vector of one face.</figcaption>

                    <img src="../images/part2/avgnormals.png" />
                    <figcaption align="middle">This shows the teapot shaded according to per vertex normal vectors.</figcaption>

                    <img src="../images/part2/bug.png" />
                    <figcaption align="middle">Initially I was calculating the vectors for the first face incorectly, which gave this cool swirly effect. </figcaption>

         <h2 align="middle">Part 3: Edge Flip</h2>
        <p>Next, I implemented the abillity to flip edges in the mesh. As I mentioned before, the mesh is represented as a half edge data structure. A half edge data structure is mostly a few key components: vertices, halfedges, edges, and faces. Vertices, edges, and faces each keep track of a single half edge. Halfedges each keep track of a twin halfedge (the one on the other side of the edge), a next halfedge (the next one on the face), an edge (made up of the half edge and its' twin), and a face. </p>
        <p>In order to accomplish the edge flip, I just needed to reassign the pointers of all of the elements involving the edge before and after the flip.</p>
                    <img src="../images/part3/bug.png" />
                    <figcaption align="middle">Bug: Every edge I tried to flip poked a hole in my mesh!</figcaption>
        <p> Initially I was only reassigning pointers for the elements on the <i>inside</i> of the two faces between the edge I was trying to flip. Can you imagine why this would poke holes in my mesh? I was neglecting to reassign pointers for the halfedges on the outside of the borders of the faces. So while the edge may have been flipped correctly, the halfedges in the rest of the mesh weren't informed of the changes, and lost track of (could no longer traverse) that part of the mesh!</p> 
        <p> Once I assigned the pointers to the outside half edges too, I got the desired result.  </p>
                    <img src="../images/part3/preflip.png" />
                    <figcaption align="middle">A Torus mesh before any edges are flipped.</figcaption>
                    <img src="../images/part3/postflip.png" />
                    <figcaption align="middle">Torus mesh after flipping some edges.</figcaption>
         <h2 align="middle">Part 4: Edge Split</h2>
        <p>Next, I implemented the ability to split edges of the mesh. This was somewhat similar to flipping edges in that it involved a lot of reassigning of pointers. However, splitting became a little more complicated because it involved adding some new edges, faces, and a new vertex to the mesh. </p>
                    <img src="../images/part4/bug2.png" />
                    <figcaption align="middle">Bug: Whenever I attempted an edge split, I punched a huge dent in my teapot! This shows a bunch of split-attempts near eachother, resulting in one giant heart dent. </figcaption>
        <p> This initial error was caused by an error calculating the position of my new vertex. In fact, I wasn't calculating the position of the vertex at all. So everytime I split an edge, I was adding a new vertex at the origin. That's why when I split, all of these dents were getting punched toward the same spot! </p> 
        <p> Once I updated the position of the new vertex to be the midpoint between the other corner vertices, I achieved the desired result. </p>
                    <td align="left">
                    <img src="../images/part4/pre_split.png" />
                    <figcaption align="middle">Bean mesh before edges are split.</figcaption>

                    <td align="right">
                    <img src="../images/part4/post_split.png" />
                    <figcaption align="middle">Bean mesh with some split edges in the middle.</figcaption>

        <h2 align="middle">Part 5: Upsampling via Loop Subdivision</h2>
        <p>Upsampling was the most challenging part of the project for me. There were quite a few issues to be debugged, but at a high level the approach is pretty straight forward.</p>
        <li> 
            First I marked all of the existing vertices as "old", indicating they were part of the mesh before it was upsampled. 
            <li> 
            Next I calculated new positions for all of the old vertices according to the vertex subdivision rule.
        </li> 
            <p align="middle"><pre align="middle">n = vertex degree</pre></p> 
            <p align="middle"><pre align="middle">u = 3/(8*n)</pre></p>
            <p align="middle"><pre align="middle">(1 - 3/8) * original_position + u * neighbor_position_sum</pre></p>
        <li> 
            Then I calculated positions for all of the new (to be added) vertices according to: 
        </li> 
            <p align="middle"><pre align="middle">3/8 * (A + B) + 1/8 * (C + D)</pre></p>
            <p> where A and B are the vertex positions on the edge that the new vertex will lie, and C and D are the vertex positons on the adjacent edges of the faces on either side of the edge that the new vertex will lie. </p>
        <li> Next, I split every "old" or prexisting edge in the mesh.
        <li>Finally, I flipped any edge that connected a new vertex and an existing vertex, and then updated all of the vertex positions. 
        </li> 
                    <td align="left">
                    <img src="../images/part1.png" />
                    <figcaption align="middle">Teapot mesh before upsample</figcaption>

                    <td align="right">
                    <img src="../images/part5/up_teapot.png" />
                    <figcaption align="middle">Teapot mesh after upsample</figcaption>

        <p> Some times a little pre processing before upsampling a mesh can help significantly. A good example of this is the simple cube mesh. The way the edges are oriented can cause some irregularity when then mesh is upsampled. The upsampling behavior can be improved significantly if the edges that cross the faces of the cube are split prior to upsampling. This gives a more symmetrical mesh, which helps the upsampling give a symmetrical result. </p>
        <p>Here you can see a side by side comparison of the upsampling of two cubes. The <b> left cube </b> is the original mesh being upsampled. The <b> right cube </b> had each of the face crossing edges split before it is upsampled.  

                    <td align="left">
                    <img src="../images/part5/cube.png" />
                    <figcaption align="middle">Regular cube mesh before upsample</figcaption>

                    <td align="right">
                    <img src="../images/part5/split_cube.png" />
                    <figcaption align="middle">Cube mesh with each face of cube split before upsample</figcaption>
                    <td align="left">
                    <img src="../images/part5/up_cube.png" />
                    <figcaption align="middle">Regular cube after upsample (1X). Note the irregularity</figcaption>

                    <td align="right">
                    <img src="../images/part5/up_split_cube.png" />
                    <figcaption align="middle">Pre-split cube mesh after upsample (1X).</figcaption>
                    <td align="left">
                    <img src="../images/part5/up_cube_2.png" />
                    <figcaption align="middle">Regular cube after multiple upsamples. The irregularity has propagated.</figcaption>

                    <td align="right">
                    <img src="../images/part5/up_split_cube_3.png" />
                    <figcaption align="middle">Pre-split cube mesh after multiple upsamples. Looks like a nice smooth cube!</figcaption>
         <h2 align="middle">Part 6: Fun with Shaders</h2>
        <p>Part 6 was my favorite! I wrote GLSL shaders which specify how OpenGL should light the scene. I implemented a simple Blinn Phong shader, and a really cool reflection environment map shader. </p>
                    <img src="../images/part6/phongcow.png" />
                    <figcaption align="middle">Cow With Pink Phong Shader</figcaption>
        <p>For the Phong shader, I followed the following procedue:
            <li> Calculate the light vector, l 
            <li> Calculate the vector to my "eye position", v
            <li> Normalize the sum of the vectors l and v
            <li> Return a linear combination of the ambient, diffuse, and specular lights: 
            <p align="middle"><pre align="middle">LightVec = ambient_light + diffuse_light*max(dot(normal_vector, l), 0) + specular_light*(max(dot(n,normalize(l+v)), 0)^shininess_factor</pre></p>
            </p>
                    <td align="left">
                    <img src="../images/part6/defaultcow.png" />
                    <figcaption align="middle">Cow With Default Shader</figcaption>
             
                    <td align="right">
                    <img src="../images/part6/greyphongcow.png" />
                    <figcaption align="middle">Cow With Grey Phong Shader</figcaption>
                <br>
                    <td align="left">
                    <img src="../images/part6/defaultbug.png" />
                    <figcaption align="middle">Beetle With Default Shader</figcaption>
                    <td align="right">
                    <img src="../images/part6/phongbug.png" />
                    <figcaption align="middle">Beetle With Phong Shader</figcaption>

        <p> I liked the environment map reflection shader the best. This is what I did to accomplish this effect: 
            <li> Calculate the vector to my "eye position", v
            <li> Calculate the reflection vector using v and the normal vector of the vertex 
            <li> Convert the reflection vector into polar coordinates
            <li> Convert the polar coordinates into uv coordinates 
            <li> Retrieve the colors for the vertex from the environment map using the uv coordinates. 
        </p> 
                    <img src="../images/part6/reflectcow.png" />
                    <figcaption align="middle">Cow With Reflection Map Shader</figcaption>
                    <img src="../images/part6/reflectbug.png" />
                    <figcaption align="middle">Beetle With Reflection Map Shader</figcaption>

</div>
</body>
</html>
