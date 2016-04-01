---
layout: post
title: "Rasterizester Project"
date: 2016-01-28
backgrounds: 
    -../images/part1.png
    -../images/part3_4.png
thumb: ../images/part_5.png
category: Project
tags: 
---


<h2 align="middle">Part 1: Rasterizing Lines</h2>
<p>I used Bresenham's algorithm to rasterize lines. In order to rasterize the line, the algorithm needs to decide which pixels lie closest to it. The line will not directly intersect the center of every pixel, so it must be decided which pixels it intersects the most. To do this, it steps through the line, incrementing the x coordinate by one if the overall change in x is greater than the overall change in y, or incrementing the y coordinate if the opposite is true. For this description I will assume we are rasterizing a line dx > dy. </p> 
<p> Then the algorithm needs to make a decision about which pixel is closest to the next point on the line. It uses a decision parameter pk. If pk is less than zero, it plots the point (x, y) and increments pk by 2*dy. Otherwise, it plots the point (x, y + 1) or (x, y - 1) depending on if the slope is positive or negative, respectively. It that case it increments pk by 2*dy - 2*dx and continues stepping through the line. The multiplications by two make the algorithm only use integer calculations, which makes it more efficient. 

                    <img src="../images/part_1.png" width="800px" />
                    <figcaption align="middle"> Result Using Bresenham's Line Rasterizing Algorithm </figcaption><br>

<h2 align="middle">Part 2:Rasterizing Single-Color Triangles</h2>
<p>I ended up using two different methods to rasterize triangles. In the first method, I was breaking the triangles into "top flat" and "bottom flat" cases, and then stepping through the triangle and rendering it line by line. After part 5, I ended up switching to using barycentric coordinates to rasterize them. I needed to calculate barycentric coordinates to retrieve the correct color for each pixel, so it made sense to rasterize each pixel individually rather than rasterize a line at a time. Using barycentric coordinates to rasterize triangles also gave a cleaner result. 
					<img src="../images/part_2_1.png"/>
                    <figcaption align="middle">Original Method: Note the lines extending off the claw and tail.</figcaption>
                    <img src="../images/part_2_2.png"/>
                    <figcaption align="middle">Barycentric Method</figcaption>
            <p>In the first method I used to rasterize triangles, I broke them into three cases: 
            <ul>
            <li>Bottom Flat Triangles</li>
            <li>Top Flat Triangles</li>
            <li>Other</li></ul>
        For the bottom flat triangles, I started by using my rasterize_line function to rasterize the bottom (flat) edge of the triangle, which was just the line between the two vertices with the lowest y coordinate. From there I calculated the inverse slope of each non-flat side of the triangle, and added that to the respective x coordinate. Then I incremented the y coordinate and plotted a line between the current x values and the y value.</p>
        <p>Incrementing the x values and plotting the line: </p>
        <p align="middle">
        <pre align="middle">current_x1 = current_x1 + m1^-1</pre>
        <pre align="middle">current_x2 = current_x2 + m2^-1</pre>
        <pre align="middle">rasterize_line(current_x1, y, current_x2, y)</pre></p>
        <p> Where current_x1 is initialized to the x value of one of the bottom corners and current_x2 is the x value of the other bottom corner. Adding the inverse slope steps the x_values up along the edges of the triangle towards the top vertex. The subroutine to render top flat triangles was done similarly</p> 
        <p> For the "other" case, I would simply divide the triangle into two seperate triangles: a bottom flat triangle and a top flat triangle. I generated a "fourth" vertex that was directly across from the "middle" vertex, the one with the middle y coordinate. </p> 
        <p align="middle"><pre align="middle"> x4 = x0 + ((y1 - y0)/(y2 - y0) * (x2 - x0))</pre></p>
        <p align="middle"><pre align="middle">y4 = y1</pre></p>
        <p> Where coordinates are sorted according to ascending y coordinates (so (x0, y0) has the smallest y coordinate and (x2, y2) has the highest y coordinate). Using this new fourth coordinate, I could use my prewritten subroutines to render the original triangle as two seperate triangles, one that was flat on top and the other which was flat on bottom. </p> 
<img src="../images/part2.png"/>
<center>Final Result: Rendered Triangles</center>

<h2 align="middle"> Part 3: Antialiasing triangles</h2>
<br>
<p> Part three was the most challenging for me! First I initialized the super sample buffer as a vector of unsigned chars, simiar to the frame rate buffer, except scaled up based on the sample rate. 
<p align="middle"><pre align="middle">total_sample_pts = width * height * sample_rate
<br> 
superframebuffer.resize(total_sample_pts * 4)</pre></p>

<center>
                    <img src="../images/part_3_off_by_4.png" width="800px" />
                    <figcaption align="middle">Sample_Rate = 4. Image colors are off and image is repeating. The colors are off due to the indexing in red shown above. It should be: 
                    <pre> int index = (k*dimension) + <font color="red"> <font color="blue">4 *</font>(i + j)</font></pre> Due to the supersamplebuffer storing rgba values for each pixel. </figcaption></center><br>

<p> The image is being repeated due to another indexing error when the pixels are drawn into the supersamplebuffer. Once the index was corrected, I got a new result:</p>


                    <img src="../images/part_3_indexbug.png" width="800px" />
                    <figcaption align="middle"><br>Sample_Rate = 4. Image is scaled up larger than it should be and is lighter due to super sample pixel being blended with blank pixels.  </figcaption>

<p> The super sample pixel is:
        <p><pre align="middle"> 1/sqrt(sample_rate) </pre>
            the size of a frame buffer pixel, so it's color blended with:
            <pre align="middle"> (sqrt(sample_rate) - 1)/sqrt(sample_rate) </pre> white pixels, giving the lighter, less opaque appearance.</p>
        <p> I went through countless other iterations of indexing errors, many of which I can't recreate now. I ended up changing my resolve method to keep track of the x and y coordinate of the current frame buffer pixel. However, just now when trying to recreate another bug, I got my original resolve code to work. Still, my supersamples were getting lighter due to the blending error I described above. </p> 
        <p> The biggest challenge I had was understanding how to actually anti-alias. At one point I was drawing to the super sample buffer correctly, my resolve method was working mostly correctly (the size of the images stayed consistent among different sample_rates), however there was no anti-aliasing. The jaggies looked the same whether the super_sample_rate was 1 or 16. </p> 
        <p> 
            Did I need to scale the points by sqrt(sample_rate) in my supersample_point method? Did I need to scale up the triangle according to the sample rate? Or was it both? Between this and the indexing stuff, I was really confusing myself. Thankfully, Ren patiently talked through the idea with me until I understood: I needed to scale one or the other. Either the points or the triangle, but not both.
        </p>  
        <p>  Scaling the size of the triangle felt more intuitive to me, so that is what I went with. In my <font color="blue"> void DrawRend::rasterize_triangle() </font> method, I scaled the input coordinates (x0,y0,...,x2,y2) up by a factor of sqrt(sample_rate). Then I repaired my <font color="blue"> DrawRend::supersample_point(x, y) </font> method so that it stored the true x and y coordinates, as opposed scaling them up as I did before. This allowed (sample_rate) more pixels to be rendered inside the triangle.
        </p>
        <p> Then the resolve method downsampled the pixels back to the resolution of the framebuffer.So colors from a sqrt(sample_rate) X sqrt(sample_rate) square of pixels in the superframebuffer were averaged, and the resulting color was assigned to one corresponding pixel in the frame buffer, giving the final result.

                    <td align="middle">
                    <img src="../images/part3_1.png" width="800px" />
                    <figcaption align="middle">Sample_Rate = 1</figcaption>
                    <td align="middle">
                    <img src="../images/part3_2.png" width="800px" />
                    <figcaption align="middle">Sample_Rate = 4</figcaption>
                    <td align="middle">
                    <img src="../images/part3_4.png" width="800px" />
                    <figcaption align="middle">Sample_Rate = 16</figcaption>

</p>

<h2 align="middle">Part 4: Transforms</h2>
<p>For part 4, I implemented the transform matrices as shown in the SVG spec. Then I created a new svg file, "first.svg". I grabbed one of the stars shown in another example file and pasted it in my file. Then I made four identical stars and alternated their colors. Next, I put them all in a group that would translate them closer to the center of the page. Finally, I put each star in it's own group with a rotation applied. I incremented each rotation by .25 and added enough stars to make a ring of stars!</p>

<img src="../images/p4_2.png" width="800px" />
                    <figcaption align="middle"> Ring of Stars </figcaption><br>
                    <img src="../images/p_4_3.png" width="800px" />
                    <figcaption align="middle"> Ring of Stars Showing Zoom In, Translate GUI Features<br> </figcaption><br>
                    <img src="../images/p_4_5.png" width="800px" />
                    <figcaption align="middle"> Ring of Stars Showing Zoom Out, Translate GUI Features</figcaption><br>


<h2 align="middle">Part 5: Barycentric coordinates</h2>
<p> Barycentric coordinates give us a way to assign each vertex in a triangle an attribute, and then linearly interpolate those attributes to assign the appropriate value for the other pixels within the triangle. </p>

<img src="../images/part_5_tri.png" width="800px" />
                    <figcaption align="middle"> Triangle with a red vertex, blue vertex, and green vertex. The pixels between the vertices are assigned a color based on their barycentric coordinates.</figcaption><br>


<p> In the example above, I will refer to the upper left vertex as <font color="red">R</font>, the upper right vertex as <font color="green">G</font>, and the lowest vertex as <font color="blue">B</font>. All of the pixels between these vertices are assigned a color depending on where they are relative to <font color="red">R</font>, <font color="green">G</font>, and <font color="blue">B</font>.
<p> Observe the pixels that lie on the edge between <font color="red">R</font> and <font color="blue">B</font>. The pixels closest to <font color="red">R</font> are all red, and the pixels closest to <font color="blue">B</font> are all blue. However, the pixels right in the middle of the edge are a purple shade, since they lie directly between the red and blue vertices. The pixel that lies exactly in the center of the two vertices gets:
	<pre align="center">.5*color(<font color="blue">B</font>) + .5*color(<font color="red">R</font>) + 0*color(<font color="green">G</font>)</pre>
The number that is multiplied by the color of each vertex is either alpha, beta, or gamma. Each one represents the relative distance from the pixel to one of the three triangle vertices. In the above example, alpha and beta are 0.5, since the pixel is halfway between the red and blue pixels. Gamma is zero since it is relatively far away. 
To implement this in my program, I first calculate the alpha, beta, and gamma barycentric coordinates using the coordinates passed into <font color="blue"><DrawRend::render_barycentric_triangle()</font>. Then, these are passed to <font color="blue"> ColorTri::color()</font>. This method then multiplies alpha by the "a" vertex color, beta by the "b" vertex color, and gamma = (1 - alpha - beta) by the "c" vertex co
                    <img src="../images/part_5.png" width="800px" />
                    <figcaption align="middle"> Color Wheel Result.</figcaption><br>


<h2 align="middle">Part 6: Pixel sampling for texture mapping</h2>
<h4> Finding the UV Texture Coordinates </h4>
<p> Implementing part 6 was very similar to part 5. The same way I used barycentric coordinates to interpolate the colors of three vertices of a triangle for the pixels in the triangle, I could map a coordinate inside a triangle to its corresponding uv texture coordinate. In <font color="blue"> Color TexTri::color(Vector2D xy, Vector2D dx, Vector2D dy, SampleParams sp) </font>, I multiply alpha by the "a" vertex's uv coordinate vector, beta by the "b" coordinate uv vector, and gamma by the "c" coordinate uv vector. </p>
<h4> Retrieving the Color from the MipMap </h4>
Once I had adjusted the uv vector, it is passed into <font color="blue">Texture::sample(const SampleParams &sp)</font>, which then calls the appropriate sampling method according to the sp.psm parameter. The sampling method retreives the level 0 mip map (for this portion). The uv coordinates are between 0 and 1, so the sampling method then scales them up to match the proportions of the mipmap. 
<pre align="center"> x = uv.x * mipmap.width <br> y = uv.y * mipmap*height </pre> Then, the color of the pixel is retrieved from the mipmap. 
<pre align="center">return MipMapPixelColor(x, y, level)</pre>
<h4> Nearest vs Bilinear Sampling </h4>
The nearest level sampling method simply returns the color of the pixel as above. The Bilinear method returns an interpolation of the four nearest uv coordinates. I used the algorithm in the book to implement bilinear sampling. The difference between nearest and bilinear sampling is the most apparent in the map sample images, which have distinct thin vertical lines through them. </p>

<img src="../images/part_6_nearest_1.png" width="800px" />
                    <figcaption align="middle"> Nearest Level Pixel Sampling, sample_rate = 1. <br>Jaggies! Blegh!</figcaption><br>
                    <img src="../images/part_6_bilinear_1.png" width="800px" />
                    <figcaption align="middle"> Bilinear Pixel Sampling, sample_rate = 1 <br>
               		SoooOossosoSo smooth <3 </figcaption><br>
Even with a high sample rate, the difference between the two methods is apparent on these images. <br><br>
                    <img src="../images/part_6_nearest_16.png" width="800px" />
                    <figcaption align="middle"> Nearest Level Pixel Sampling, sample_rate = 16 <br>
                    	Gross! Blurry Jaggies!</figcaption><br>
                    <img src="../images/part_6_bilinear_16.png" width="800px" />
                    <figcaption align="middle"> Bilinear Pixel Sampling, sample_rate = 16 <br> 
                   Ridiculously good looking.</figcaption><br>

<br><br>
<p> 
	However, without these distinct lines, the effect is much less noticeable. In the campanille image, I actually prefer the nearest sampling method. The features of the image seem slightly better with the nearest sampling.  </p>

<img src="../images/part_6_camp_nearest.png" width="800px" />
                    <figcaption align="middle"> Nearest Level Pixel Sampling, sample_rate = 16 <br>
                    	Looks pretty nice.</figcaption><br>
                    <img src="../images/part6_camp_bi.png" width="800px" />
                    <figcaption align="middle"> Bilinear Pixel Sampling, sample_rate = 16 <br> 
                    </figcaption><br>
<p> 
	Initially when retrieving the mip map colors, I had a bug because I wasn't dividing my colors by 255. Thanks to piazza, I knew I had to divide them by 255 since the color was expecting a float between 0 and 1. However at first that was just giving me black images. Then I realized I literally had to divide it by "255." to prevent a precision error. 
                    <img src="../images/part_6_1.png" width="800px" />
                    <figcaption align="middle"> Weird Colors</figcaption><br>
                    <img src="../images/part_6_5.png" width="800px" />
                    <figcaption align="middle"> Looks Like Sprinkles!</figcaption><br>
                    <img src="../images/part_6_4.png" width="800px" />
                    <figcaption align="middle"> Corner Sprinkles</figcaption><br>


<h2 align="middle">Part 7: Level sampling with mipmaps for texture mapping</h2>
<p>In the final part, I implemented the "get level" method using the math described in the textbook. This allows the texture sampling to grab different mip maps for each pixel, depending on which is most appropriate. Sometimes the effects are desirable, and other times it ends up blurring the image. </p>
<p> In trilinear sampling, the color from the getLevel() mipMap and the color from the adjacent level are blended to give the resulting color. 
                    <img src="../images/part_7_lzero_pnear.png" width="800px" />
                    <figcaption align="middle"> Level Zero, Nearest Sampling</figcaption><br>
                    <img src="../images/part_7_lzero_plin.png" width="800px" />
                    <figcaption align="middle"> Level Zero, Linear Sampling <br> </figcaption><br>
                    <p> I thought the above combination, Level Zero with Linear Sampling, gave the best result. It is the only one that eliminates the Jaggies in the "Maleficent" chrome text. </p>
                    <img src="../images/part_7_lnear_plin.png" width="800px" />
                    <figcaption align="middle"> Nearest Level, Linear Sampling</figcaption><br>
                    <img src="../images/part_7_lnear_pnear.png" width="800px" />
                    <figcaption align="middle"> Nearest Level, Nearest Sampling</figcaption><br>
<p> For closer images, there was visable blurring when using the trilinear sampling met
                    <img src="../images/part_7_llin_maleficent.png" width="800px" />
                    <figcaption align="middle"> Trilinear Sampling <br> Observe the blurring in Maleficent's face. </figcaption><br>
                    
                    <img src="../images/part_7_llin_maleficent2.png" width="800px" />
                    <figcaption align="middle"> Trilinear Sampling</figcaption><br>
                    
                    <img src="../images/part7_lnear_pnear_2.png" width="800px" />
                    <figcaption align="middle"> Nearest Level, Nearest Sampling<br>I felt this was the best result for this image. </figcaption><br>
                    
                    <img src="../images/part_7_plin_lzero.png" width="800px" />
                    <figcaption align="middle"> Level Zero, Linear Sampling <br> The jaggies are less apparent in the writing, but Maleficent is a little more blurry.</figcaption><br>
</div>
</body>
</html>
            


