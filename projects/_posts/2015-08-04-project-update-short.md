---
layout: post
title: "Work in Progress"
date: 2015-08-01
backgrounds: 
    - https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/i-swear/background3.png  
thumb: https://github.com/kadie16/kadie16.github.io/raw/master/assets/images/posts/progress/angel8.png
category: "Project"
tags: C++ Work Project OpenGL Computer Graphics QT CGAL
---



<center><iframe width="604.8" height="453.6" src="https://www.youtube.com/embed/cjPib3gQU-k" frameborder="0" allowfullscreen></iframe></center> _<center>A Demonstration of my Project thus far</center>_

My project is a C++ program that reads in data describing 3D models, in .obj file format, and renders and lights the models with OpenGL. Since my last post, I have implemented some user interactions and use of the CGAL data structure. The models I use to test my program came from the [UC Berkeley Computer Graphics](http://graphics.berkeley.edu) website :)

## Latest Changes
- Automatically find and fit model.  
- Maintain aspect ratio on window resize. 
- Select Color.
- Shading based on vertex normal vectors. 
- Rotation by mouse drag. 
- Zoom by right click mouse drag. 
- Option to move model by mouse drag. 


### Find and Fit 
{% highlight c++ %}
cam.findModel(objModel);
cam.viewModel();
cam.moveToCenter();
zoomF = cam.fitModel(maxCoords.at(0), minCoords.at(0),
                     maxCoords.at(1), minCoords.at(1),
                     maxCoords.at(2), minCoords.at(2));
{% endhighlight %}

Now, whenever a new .obj file is selected, the "camera" automatically centers the model and makes it fit on the screen. I say "camera" because there is no actual camera in OpenGL. Instead, you manipulate the view by applying matrix transformations to the modelview and projection matrices. More on that later. I wrote a "camera" class, which handles these transformations. 

### Aspect Ratio 
I think this is probably the least noticeable new feature, but it was one of the bigger challenges. If you watch closely, you can see that when I resize the window in the video, the cube retains its shape. In otherwords, it doesn't turn into a rectangle if the viewer isn't perfectly square. You might also notice that the angel is more slender than before. 

![Angel Aspect Before](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/progress/angel7.png) _<center>Squished angel</center>_
![Angel Aspect After](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/progress/angel8.png) _<center>Angel with corrected aspect ratio</center>_

{% highlight c++ %}
void camera::adjustAspect(float window.width, float window.height) {
    /* Modify the Projection Matrix */
    glMatrixMode(GL_PROJECTION);
    /* Start from a 'clean slate' */ 
    glLoadIdentity();
    /* Calculate Aspect Ratio */
    float newAspect = window.width/window.height;
    /* Adjust Accordingly */
    leftAdjust = newAspect * left;
    rightAdjust = newAspect * right;
    glOrtho(zoomF * leftAdjust, zoomF * rightAdjust, zoomF * bottom, zoomF * top, near, far);
}
{% endhighlight %}


### Color Picking

![humanoid color pick](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/progress/humanoid1.png)

To me this feature seems more impressive than the constant aspect ratio, but it took about five minutes to implement. Thanks to QT, color picking boiled down to two lines of code. 

{% highlight c++ %}
void MainWindow::on_toolButton_clicked()
{
    QColor color = QColorDialog::getColor();
    ui->widget->grabColor(color.red(), color.green(), color.blue());
}
{% endhighlight %} _<center>ui is the User Interface, widget is the part of the ui that displays the model</center>_

Ok, I had to write grabColor too, so maybe 5 lines. Still. I actually threw this feature in because I needed a break from some other issue I was stuck on.  

###Vertex Shading

In my last post, I mentioned that every vertex needs a normal vector so openGL knows how to shade the model. At that time, I was just computing the normal for each face and lighting all of the vertices in that face uniformly. 

![Before and after cube picture](https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/posts/progress/shading1.png) _<center>Before and After per-vertex shading</center>_

Now, the normal for each vertex is assigned to be the _average_ of the normals of all of the faces it belongs to. So the vertices on the corners of the cube are assigned a normal equal to the average of the normal vectors of the three faces that meet at that corner. The vertices on the edge of two faces get the average of the two normal vectors of those faces. The vertices in the middle of the face just get the normal that belongs to that face. OpenGL then interpolates in between the vertices to produce the smooth gradient you see. 

### User Controls

#### Making it "feel" natural
I expected understanding the openGL calls to be challenging, but I didn't anticipate the challenge of making the controls intuitive. I guess this speaks to the idea that the best computer graphics go unnoticed.

The user controls are simulated by a series of repaints. Actually the model is being redrawn constantly, but without adjustments to the modelview or projection matrix, it is just drawn exactly the same. So my goal is to animate, or redraw, the model so the user feels like they are actually touching and manipulating the model with their mouse. 

#### Rotation  
Getting the model to rotate wasn't hard. I could just use glRotatef to do that. 
{% highlight c++ %}
    glRotatef(GLfloat angle, GLfloat x, GLfloat y, GLfloat z);
{% endhighlight %} _<center></center>_
Getting the model to rotate according to mouse drag wasn't a big deal either. All I had to do was set the parameters in glRotatef based on the change in position of the mouse. The hard part was getting the model to rotate in a way that felt _natural_. 

The first problem is glRotatef rotates the model around the origin, the point (0,0,0). This gives the appearance that the model was orbiting about an arbitrary point in space. Not what I want. I want the model to rotate about it's center. That way it stays where it is and just spins around. 

There is a clever trick to accomplish this. Since glRotatef rotates about the origin, you can just move whatever point you want to rotate about to the origin, apply the rotation, and then move the object back to where it was. Imagine I pick up the model, move it's center to the origin, rotate it, and move it back to it's original point in space. 

{% highlight c++ %}
/* Move center to origin */
glTranslatef(model.center().x, model.center().y, model.center().z);
/* Rotate */ 
glRotatef(GLfloat angle, GLfloat x, GLfloat y, GLfloat z);
/* Apply reverse translation to move center back to where it was */
glTranslatef(- model.center().x, - model.center().y, - model.center().z);
{% endhighlight %} _<center></center>_

Pretty neat trick! But still, I am left with a model that is rotating in crazy unpredictable patterns. 

<iframe width="604.8" height="453.6" src="https://www.youtube.com/embed/ebHibD78bCI" frameborder="0" allowfullscreen></iframe> _<center>?????????</center>_

The rotations seem accurate at the beginning of the demo, but soon deviate from what is expected. You can see that the effect of dragging the mouse in one direction (like left to right) is inconsistent as time goes on. 

This is because the rotations are accumulating. 

> **Desired Effect**: <br>
> 0.) Start Position. <br>
> 1.) Apply rotation A to position at 0. <br>
> 2.) Apply rotation **B** to position at 1. <br>

We want each mouse drag to produce a new, independent rotation. So when I drag the mouse from left to right, the model spins right, regardless of the rotations I performed previously. 

> **Actual Effect**: <br>
> 0.) Start Position. <br>
> 1.) Apply rotation A to position at 0. <br>
> 2.) Apply rotation **(A + B)** to position at 1. <br>

As is, the rotations are adding on top of eachother. So when I drag the mouse left to right, the model spins according to the sum of the prior rotation and the desired left to right rotation. Confusing right?!?!?!?!

#### OpenGL Matrix Stack 
Here, the openGL matrix stack comes in. Here is the deal: In order to rotate my model, I am manipulating the Model View Matrix. But openGL actually maintains a "stack" of Model View Matrices for me to work with. If I want to save my current matrix, I can "push" it to the stack. Then, I can change the matrix however I want. When I decide that I want to go back to that matrix I pushed earlier, I can "pop" the matrix and I will get the next one on the stack. More confusion, no? 

Let's pretend that coloring is a matrix operation. So I start out with a regular white model view matrix. I call glPushMatrix(). Then I paint the matrix yellow. 

{% highlight c++ %}
/* My matrix is white */
glPushMatrix(); // Saving white matrix for later. 
applyPaint(Yellow);
drawMyMatrix();
{% endhighlight %} _<center> displays a yellow matrix </center>_

Next, I decide I want a red matrix. If I just start using red paint to my yellow matrix, I am going to end up with an orange matrix. So, first I call glPopMatrix(). Now I get the white matrix back, paint it red, and I have a true red matrix. 

{% highlight c++ %}
/* My matrix is yellow */
glPopMatrix(); // Getting the white matrix back. 
applyPaint(red);
drawMyMatrix();
glPushMatrix(); // Saving red matrix for later.
{% endhighlight %} _<center> displays a red matrix </center>_

Now, I call glPushMatrix() again. I want to turn the matrix purple this time. So I paint this matrix blue, and get a purple matrix. Now I want a green matrix! But I don't want to slather green paint on a purple thing.  

{% highlight c++ %}
applyPaint(blue);
/* Now my matrix is purple! */
drawMyMatrix();
{% endhighlight %} _<center> displays a purple matrix </center>_

{% highlight c++ %}
glPopMatrix(); // Got my red matrix back. 
glPopMatrix(); // Got my white matrix back. 
applyPaint(green);
drawMyMatrix();
{% endhighlight %} _<center> displays a green matrix </center>_

#### Back to Rotation

{% highlight c++ %}

QQuaternion GLWidget::drag2Rotate(float dx, float dy)
{
    /* Define Axis of Rotation */
    axisOfRotation.setX(-dy);
    axisOfRotation.setY(-dx);
    axisOfRotation.setZ(0);
    magnitude = sqrt(dx*dx + dy*dy);
    /* Update Rotation Quaternion */
    QQuaternion newQ = QQuaternion::fromAxisAndAngle(axisOfRotation, magnitude);
    currQ = newQ * currQ;
    return currQ;
}
{% endhighlight %} _<center> I keep track of the current Quaternion (represents the current rotation) by multiplying the new, applied rotation by the former rotation </center>_

{% highlight c++ %}
        glPushMatrix(); /* Saves state before rotation */
        QMatrix4x4 rotationMatrix; / Begins as Identity Matrix, Like a "white" matrix from my example
        /* Here I rotate by the Quaternion, which holds the current rotation */
        rotationMatrix.rotate(currQ);
        glMatrixMode(GL_MODELVIEW);
        /* Translate so rotation occurs about model center */
        glTranslatef(m.center().at(0), m.center().at(1), m.center().at(2));
        glMultMatrixf(mat.constData());
        glTranslatef(-m.center().at(0), -m.center().at(1), -m.center().at(2));
        m.drawMe();
        glPopMatrix(); /* Reverts to state before rotation */ 
{% endhighlight %} _<center> This way, the next time the model is drawn, the rotation is applied to the original, "white", matrix. </center>_


#### Zoom
My first idea was to use glScalef on the Model View Matrix to zoom.
{% highlight c++ %}
void MainWindow::on_toolButton_clicked() {
    glMatrixMode(GL_MODELVIEW); // Specifies which matrix will be scaled
    glScalef(xScale, yScale, zScale);
}
{% endhighlight %} _<center> Calling glScalef(0.5,0.5,0.5) would uniformly scale the Model View Matrix down 50%</center>_

That isn't quite right. Here I am actually changing the size of the model, rather than getting closer or further. This is kind of like the desired effect, but if I get too close holes start appearing in my models. By scaling the Model View Matrix, the model can get so big that parts of it would lie outside the clipping planes. 

<iframe width="604.8" height="453.6" src="https://www.youtube.com/embed/crWdEMgDwNE" frameborder="0" allowfullscreen></iframe>

Instead of adjusting the Model View Matrix, I need to adjust the Projection Matrix. Doing so changes the way I view the "world", as opposed to changing the model itself. 

{% highlight c++ %}
void camera::setZoom(float factor) {
    if (factor > 0.01)
        zoomF = factor;
    else
        zoomF = 0.01;
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity(); // Make sure zoom isn't applied to the previous state
    glOrtho(leftAdjust*zoomF,rightAdjust*zoomF,bottom*zoomF,top*zoomF,-near,-far);
}
{% endhighlight %}

---

That's it for now :) My next post will show my final project! 



