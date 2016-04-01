---
layout: post
title: "Raytracer"
date: 2016-03-31
backgrounds: 
thumb: ../images/p3/fun/dragon_hd.png
category: Project
tags: 
---



<div>
        <p> In this project, I wrote a ray tracer! Ray tracing involves tracing the path of light rays to generate images. By tracing the path of many many rays, I can determine how objects in a scene interact with eachother. For example, I can trace a bunch of rays from a light source, to an object, to the camera, determine how the object appears in the scene, and render or "draw" it to the screen accordingly.</p>
        <img src="../images/p3/part4/CBbunny_hd.png">
        <figcaption align="middle">You have to read the whole thing to appreciate how much effort went into rendering this bunny :)!</figcaption>

    <h2 align="middle">Part 1: Ray Generation and Scene Intersection </h2>
        <p>The first thing I did was implement a raytrace_pixel() method. The method integrates the irradiance of the pixel.  </p>
        </div>
> **Irradiance:** <br> 
The flux of radiant energy per unit area (normal to the direction of flow of radiant energy through a medium) 

<p>In other words, I implemented raytrace_pixel() to find the amount of light energy in the given pixel, which tells me how bright the pixel should be. I did this by generating a given number of "rays" through the pixel. </p>

> **Ray:** <br> 
A line with a start point, but no end point. <br>

<p> In my program, a ray is given an origin (it's start point), a vector which dictates it's direction, a depth, and a "min_t" and "max_t". The "min_t" and "max_t" allow me to treat the ray as a segment, if I wish. So I can cut it off at the minimum time and maximum time I set. This becomes important later! </p> 
<p> In this case, all of the rays I generate have the "camera"'s position as their origin and a vector towards the pixel as their direction. You can think of the camera as where your eye is in the scene. So basically I am drawing a bunch of lines from my eyeball to a point in space and summing up all of the radiant energy, or irradiance, that my eyeball is detecting at that point in space.</p> 
<h3> Primitive Intersections </h3>
> **Primitive:** <br> 
These are the building blocks of the scenes. Triangles are used most commonly, but all kinds of shapes can be used! <br>

<p> Next, I used the Moller Trumbore algorithm to implement a method, Triangle::intersect(), that lets me know if a given Ray intersects that particular triangle. The Moller Trumbore algorithm gives a way to move the origin of the ray and change the base of it to get a vector: [t, u, v] where t represents the distance from the ray's original origin to the plane the triangle is on, and u and v are the Barycentric coordinates within the triangle that the ray intersects. (See part 5 of my <a href="/rasterizester"> rasterizester </a> project for more on Barycentric coordinates). 

<p> Since most of the scenes I will be rendering are made up of triangles, it is super important to be able to tell whether the Rays I trace are intersecting them. If I trace a ray from a light's position out in some direction and it intersects a triangle, T1, I can tell exactly which point on that triangle T1 is lit up by that particular ray... almost. What if there is another, larger triangle T2 in between the light and T1?? Then the point I found on T1 wouldn't be lit up at all, since it would be obstructed by T2.</p>

<p> Here is where "min_t" and "max_t" come in. When I trace a ray from a light and use the Moller Trumbore algorithm to test whether it is intersecting the primitives triangles, T1 and T2, in my scene, I will find out it intersects both of them and the "t's" (distances) the triangles were from the origin of the ray. However, I am only interested in the closest intersection since that is the one which is lit! So, once the ray intersects T2, I set the ray's "max_t" to the distance T2 is from the origin of the ray. That way, when I test T1, I will be able to tell it isn't the closest intersection, since it's t value will be greater than the ray's "max_t". </p>

<p> I implemented a similar method for intersecting spheres, and then I was able to render my first images! </p>

<div>
            <img src="../images/p3/part_1/spheres.png"/>
            <figcaption align="middle">Cornell Box Spheres</figcaption>

            <img src="../images/p3/part_1/coil.png"/>
            <figcaption align="middle">Cornell Box Coil</figcaption>

<h2 align="middle">Part 2: Bounding Volume Hierarchy (BVH)</h2>
<p> So, remember that all of the scenes I am rendering are made up of a bunch of "primitives", usually triangles. As you can imagine, really complicated scenes require A LOT of triangles. We could be talking on the scale of hundreds of thousands of triangles, or even more. </p>
            <img src="../images/p3/part_2/wall-e_tri.png" />
            <figcaption align="middle">Look at all of those triangles!</figcaption>
<p> With that in mind, it isn't efficient to loop through every single triangle of the mesh for every ray I need to trace. I am tracing potentially thousands of rays per primative per scene. For simple meshes like those in part one, that will suffice, but for more complicated meshes like Wall-E I need a better solution. This is where bounding volume heirarchys come in. </p>
<p> By subdividing my mesh into bounding volumes, I can first test if a ray intersects a large bounding volume containg part of the mesh. If it doesn't, I just saved a bunch of time. Before, I would have had to test all of the primitives in that volume! If it does, I can descend into the smallest bounding volume the ray intersects, and test only the primitives in that box. </p>
            <img src="../images/p3/part_2/walle_bvh0.png" />
            <figcaption align="middle">The boxes are the bounding volumes that divide the different pieces of Wall-E </figcaption>
<h3> Constructing the BVH </h3>
<p> To construct the BVH, I find a bounding box that contains all of the primitives in the scene. Then, I create an empty BVH node. If the amount of primitives in the box are under a specified max_leaf_size, then I am done. Otherwise, I need to divide that bounding box into smaller ones. </p> 
<p> To accomplish this, I split the current bounding box in half. It is a 3D bounding volume, so I split it along it's longest dimension. Then, I put each of the primitives into either the left or right box. If their centroid in the chosen axis is less than the split point, they go in the left box, and vice versa. This continues recursively until all of the primitives are contained in boxes with <= max_leaf_size total primitives. </p>
            <img src="../images/p3/part_2/walle_bvh2.png" />
            <img src="../images/p3/part_2/walle_bvh1.png" />
            <figcaption align="middle">Showing some smaller bounding volumes in Wall-E's BVH tree</figcaption>
<p> Now if I am testing a ray which intersects a primitive Wall-E's hand, I save myself a bunch of work, since I don't have to loop through any of the primitives outside of the little bounding volume that contains his hand. Again, this is super important, because sometimes I am tracing thousands of rays per pixel!</p>
<h3> Intersecting a Bounding Volume </h3>
<p> To find intersections in my bounding volume, I start with the ray and the root node of my bvh tree (the giant box that holds all of the primitives). </p>
    <li> If the ray doesn't intersect the root node, I'm done. I know it can't intersect any of the triangles inside the box if it doesn't even intersect the box itself! 
    <li> If the node is a leaf (it contains less than max_leaf_size primitives), then I test if the ray intersects all of the primitives inside the leaf.
    <li> Otherwise, I recursively check the node's children left and right boxes. 
    </li>
<p> This continues until I reach a bounding volume the ray doesn't intersect, or until I reach the leaf node and test the primitives inside. </p>
        <img src="../images/p3/part_2/walle.png" />
        <figcaption align="middle">Now I can render super big meshes like Wall-E!</figcaption>
        <img src="../images/p3/part_2/max.png" />
        <figcaption align="middle">And Max Planck!</figcaption>


 <h2 align="middle">Part 3: Direct Illumination</h2>
<p>Up until this part, I was only tracing camera rays. The lighting in the scenes was just based on the normal vectors from the primitive to the camera. Now I will start tracing rays from lights to the scene to determine the radiance of each pixel!</p> <p> To determine the direct lighting of the scene, I sum over all of the light sources in the scene. From each light, I take sample rays and compute the incoming radiance from those directions. Then I convert the incoming radiance to outgoing radiance using the Bidirectional Scattering Distribution Function (BSDF) of the surface. </p>

        <img src="../images/p3/part3/dragon_l1.png" />
        <figcaption align="middle">Dragon with 1 sample per Light Source <br> The image is very noisy!</figcaption>
        <img src="../images/p3/part3/dragon_l16.png" />
        <figcaption align="middle">Dragon with 16 samples per Light Source <br> Notice the shadows on the dragon's neck and tail are much more distinct! </figcaption> </p>
<br>

 <h2 align="middle">Part 4: Indirect Illumination</h2>
<p>Now, instead of just considering light rays directly from the light to the surface, I let the rays "bounce". This lets me detect radiance that is coming not only directly from the light, but from light bouncing off the other objects! </p>
            <img src="../images/p3/part4/bunny_indirect.png" />
            <figcaption align="middle">Cornell Box Bunny with only Indirect Lighting ... </figcaption>
            <img src="../images/p3/part4/bunny_direct.png" />
            <figcaption align="middle">Cornell Box Bunny with only Direct Lighting </figcaption>
            <img src="../images/p3/part4/CBbunny_hd.png" />
            <figcaption align="middle">Cornell Box Bunny with Global Lighting (Indirect and Direct) <br> Notice the colors visible in the shadows!</figcaption> 
            <br> 
<p> To accomplish this, first I take a sample from the surface BSDF at the point where the ray hits. Using the illumination from that sample, I determine whether to stop there or let the ray bounce again. I can't just let the rays bounce forever (too expensive!), but fortunately I can achieve a pretty realistic result using "Russian Roulette" to decide how whether or not to terminate the ray. I flip a biased coin with probability proportional to illumination of returning true. If the illumination is already very low, I am more likely to terminate the ray. If it is very high, I am more likely to let it keep bouncing. This way even though I am sampling "randomly", I am making sure the brightest rays are being represented. </p>
<p> If the coinflip returns true, I recursively trace the ray again, offsetting it's origin slightly and sending it off in the direction of the incoming radiance converted to world coordinates. The light from the ray is accumulated in the scene until the russian roulette terminates it. </p> 
<h3> Global Illumination at varying Sample Rates </h3>
            <img src="../images/p3/part4/spheres_s1.png" />
            <figcaption align="middle">Lambertian Spheres with 1 Sample per Pixel </figcaption>
            <img src="../images/p3/part4/spheres_s4.png" />
            <figcaption align="middle">Lambertian Spheres with 4 Samples per Pixel </figcaption>
            <img src="../images/p3/part4/spheres_s16.png" />
            <figcaption align="middle">Lambertian Spheres with 16 Samples per Pixel </figcaption>
            <img src="../images/p3/part4/spheres_s64.png" />
            <figcaption align="middle">Lambertian Spheres with 64 Samples per Pixel </figcaption>
            <img src="../images/p3/part4/spheres_s1024.png" />
            <figcaption align="middle">Lambertian Spheres with 1024 Samples per Pixel </figcaption>
            <br>
<p> As you can see, it takes a lot of samples per pixel to completely eliminate the noise in the image! Images with only one sample per pixel take seconds to render, while images with 1024 samples take hours! </p>
<h3> Varying Camera Ray Depths </h3>
<p> This is skipping ahead a little, but once I add the glass and metallic materials, you can clearly see the effects of increasing the max ray depth. </p>
            <img src="../images/p3/part4/spheres2_m1.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 1. <br> Rays enter the glass sphere but they never leave! </figcaption>
            <img src="../images/p3/part4/spheres2_m2.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 2. <br> Now the rays bounce inside the glass! <br> You can tell they are still trapped inside because the mirror ball still shows a dark glass sphere. </figcaption>
            <img src="../images/p3/part4/spheres2_m3.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 3. <br> Finally the rays are making it outside of the glass. </figcaption>
            <img src="../images/p3/part4/spheres2_m4.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 4. <br> Now the rays are showing up on the right wall too! </figcaption>
            <img src="../images/p3/part4/spheres2_m100.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 100. </figcaption> <br> 


<h2 align="middle">Part 5: Materials</h2>
<img src="../images/p3/part5/spheres2_100_1024.png" />
            <figcaption align="middle"> Mirror and Glass Ball </figcaption>
<p> Remember the Bidirectional Scattering Distribution Functions I mentioned earlier? They take convert the incoming radiance of a ray to outgoing radiance. Different materials have different BSDFs, since light reacts differently to them. I implemented glass and mirror BSDFs to generate the image you see above.</p>  
<h3> Mirror and Glass BSDF </h3>
<p> The mirror ball (left) requires a mirror BSDF. This was relatively simple to implement, since the ray just reflects off of the mirror. The glass ball (right) is much more complicated. Sometimes light reflects off glass, and sometimes it refracts inside! To simulate this, I used another biased coin flip based off of the Schlick's Approximation. If the coin returned true, I reflected the incoming radiance as I did with the mirror BSDF, except with the Schlick coefficient as the probability density function (for mirror the probability density function was 1). Otherwise, I refracted the incoming radiance. </p>

            <img src="../images/p3/part5/spheres2_1_1024.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 1. <br> Rays enter the glass sphere but they never leave! </figcaption>
            <img src="../images/p3/part5/spheres2_2_1024.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 2. <br> Now the rays bounce inside the glass! <br> You can tell they are still trapped inside because the mirror ball still shows a dark glass sphere. </figcaption>
            <img src="../images/p3/part5/spheres2_3_1024.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 3. <br> Finally the rays are making it outside of the glass. </figcaption>
            <img src="../images/p3/part5/spheres2_4_1024.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 4. <br> Now the rays are showing up on the right wall, and bouncing back onto the glass! </figcaption>
            <img src="../images/p3/part5/spheres2_100_1024.png" />
            <figcaption align="middle"> Spheres with max_ray_depth = 100. </figcaption> <br> 

<h3> Increasing Samples per Pixel </h3>
<figcaption align="middle"> One Sample per Light and max_ray_depth = 100 </figcaption>
            <img src="../images/p3/part5/spheres2_s1_m100.png" />
            <figcaption align="middle"> 1 sample per pixel </figcaption>
            <img src="../images/p3/part5/spheres2_s4_m100.png" />
            <figcaption align="middle"> 4 samples per pixel </figcaption>
            <img src="../images/p3/part5/spheres2_s16_m100.png" />
            <figcaption align="middle"> 16 samples per pixel </figcaption>
            <img src="../images/p3/part5/spheres2_s64_m100.png" />
            <figcaption align="middle"> 64 samples per pixel </figcaption>
            <img src="../images/p3/part5/spheres2_s1024_m100.png" />
            <figcaption align="middle"> 1024 samples per pixel </figcaption> <br> 



<h2 align="middle"> Sick Renders! </h2>
<figcaption align="middle"> Here are my favorite renders! All with 1024 Samples/Pixel.</figcaption>
            <img src="../images/p3/fun/dragon_hd.png" />
            <figcaption align="middle"> Mirror Dragon </figcaption>
            <img src="../images/p3/fun/CBgems_hd.png" />
            <figcaption align="middle"> Gems </figcaption>

</div>
</body>
</html>
