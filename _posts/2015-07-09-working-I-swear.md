---
layout: post
title: "I'm working, I swear!"
date: 2015-07-09
backgrounds:
    - https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/background3.png   
thumb: https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/6angel.png
category: Project
tags: C++ Work Project OpenGL Computer Graphics QT 
---

So far all I've mentioned are the adventures I've been having! If you're wondering, as my Grandpa Joe put it: "when the hell are you going to work", then this post is for you :) 

#About My Project
I am building a program that will allow users to load 3D models (.obj file format) and interact with them via transformations such as rotation, translation, deformation etc. My ultimate goal is to implement the ability to select parts of a volume mesh, so the user can select and interact with seperate parts of their model. I am using C++, OpenGL, and QT for my project. 

I was first inspired to learn about computer graphics when my (amazing) materials science professor told me about his son who works on lighting movies at Pixar. I have always loved developing my technical background, but I also love painting and drawing and creating visual things. Until I learned about the opportunity to work in computer graphics, it never occurred to me that I could excercise both of my interests at the same time. 

I have virtually no experience with C++, definitely none with OpenGL and I hadn't even heard of QT until my supervisor told me about it. It is also my first time building a GUI (Graphical User Interface ... the window and buttons). Initially, I was feeling a little lost and intimidated while trying to familiarize myself with all three new things simultaneously. So I completeley understand if this is too much computer science jargon. I won't be offended if you just look at the pictures. But hopefully some of you will find this interesting :)

After a lot confusion (days), my supervisor helped me organize my learning strategy and I finally got a program running. 

![Triangle](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/0triangle.PNG) _<center>Nothing special, just a rotating triangle with a quit button, but this was exciting haha :)</center>_

Next I worked on loading actual objects. To do this, I needed to read information from a user selected .obj file. These files are lists of the vertices (lines beginning with v), and faces (lines beginning with f). Each face is a triangle, made up of three of the vertices. 

{% highlight c++ %}

v  0.0  0.0  0.0  
v  0.0  0.0  1.0 
v  0.0  1.0  0.0  
v  0.0  1.0  1.0 

... etc ...  

f  1/2  7/2  5/2
f  1/2  3/2  7/2 
f  1/6  4/6  3/6 

... etc ... 

{% endhighlight %}
_<center>Part of the .obj file for a cube. The first face is using vertices 1, 7, and 5 (which I deleted to make this example smaller).</center>_

Once I have all of the information from the file stored, I have to iterate through each face and ask OpenGL to draw the respective triangle. I've been using some models from the [Berkeley Garment Library](http://graphics.berkeley.edu/resources/GarmentLibrary/) as test files.


{% highlight c++ %}
/* So OpenGL knows I want it to draw triangles */
glBegin(GL_TRIANGLES);
	/* For every Face : */
        for (unsigned i = 0 ; i < faces.size() ; i++)
            {
                /* Get vertices from the Face */
                f = faces.at(i);
                v1 = f.getVertex(1);
                v2 = f.getVertex(2);
                v3 = f.getVertex(3);

                /* Rendering, aka Drawing, the Face (OpenGL Stuff) */
                glColor4f(0,1,1,1); // Color it Cyan for Fun  
                glVertex3f(v1.getX(), v1.getY(), v1.getZ());
                glVertex3f(v2.getX(), v2.getY(), v2.getZ());
                glVertex3f(v3.getX(), v3.getY(), v3.getZ());
            }
    }
glEnd();
{% endhighlight %}

![Robe1](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/1robe.png) _<center>Yeah, pink and cyan. I'll change to neutral colors later, OK?</center>_

Yay!! I've rendered a model! But it isn't very exciting looking. It's supposed to look 3D, but it looks flat because it hasn't been lit yet. If you draw a sphere on a piece of paper, and paint it all one color, it looks like a circle. You need to add lighting and shading to make it look like a sphere, right? This is the same idea. Cool, I'll enable lighting. 

{% highlight c++ %}
    glEnable(GL_LIGHTING);
{% endhighlight %}

![Robe2](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/2robe.png) 

Ok so enabling lighting isn't enough. Now all of the models I load are just black. At this point, OpenGL has no idea how this model is supposed to be lit. I can't just expect OpenGL to know that my vertices are supposed to look like a robe. 

In order to really light the model, OpenGL needs the normals for each vertex. A normal is a vector that is perpendicular to the face, that points outward. Knowing this, OpenGL can determine where each vertex is actually facing at any given position of the model, and depict the light accordingly. For the most accurate, smooth shading, the normal for each vertex should be an average of the normals of all of the faces that the vertex belongs to. But right now, for simplicity I am just using one.

![Dress3](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/3dress.png) 

Cool! Now the models are starting to take some shape :) But obviously something is still wrong. After a couple hours, I realized there was a [1] that was supposed to be a [0] in my normal calculations.

![Angel](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/4angel.png) 

Getting the angel to load was really exciting. This is something I was looking forward to from the beginning, when I just had a triangle. But, the black part is there because the angel is crossing the "clipping plane". The clipping plane is where OpenGL determines that the face is either too close or too far along the z axis to be drawn. Have you ever played a video game and once you get close to an object it dissapears? So my next task is to make sure that no matter where the coordinates of the model are, my program finds them and displays the model on the screen. 

![AngelAgain](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/5angelSide.png)

_<center>Isn't it neat that this is just a bunch of triangles and normals?</center>_

![Angel](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/6angel.png)

_<center>Angel data set courtesy of the U.C. Berkeley Computer Animation and Modeling Group.</center>_