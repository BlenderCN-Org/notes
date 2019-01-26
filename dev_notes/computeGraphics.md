
# Introduction to Compute Graphics

[website](http://math.hws.edu/graphicsbook/)

[course pdf](http://math.hws.edu/eck/cs424/downloads/graphicsbook-linked.pdf)

[labs used to teach course 2017](http://math.hws.edu/eck/cs424/index_f17.html)


In this book , will use a subset of OpenGL 1.1 to introduce the fundamental concepts of three-dimensional graphics. 


## Chapter 1 Introduction

## 1.2 Elements of 3D Graphics

 - geometric primitives
    - such as line, segments and triangles 
 - material
    - the properties that determine the intrinsic visual appearance of a surface.
    - Essentially, this means how the surface interacts with light that hits the surface.
    - can include a basic color as well as other properties such as shininess, roughness, and transparency.
 - texture
    - One of the most useful kinds of material property
    - Textures allow us to add detail to a scene without using a huge number of geometric primitives;


## 1.3 Hardware and Software
 
 - OpenGl
    - designed as a "client/server" system
    - communicating from the client (the CPU) to the server (the GPU)  are very slow
        - One approach is to store information in the GPU's memory.  
            - If some data is going to be used several times, it can be transmitted to the GPU once and stored in memory there.
        - Another approach is to try to decrease the number of OpenGL commands that must be transmitted to the GPU to draw a given image.
            - in OpenGL 1.1, all the data for the object would be loaded into arrays , which could then be sent in a single step to the GPU. 
            - Unfortunately, the data would have to be retransmitted each time the object was drawn.
            - This was fixed in OpenGL 1.5 with *Vertex Buffer Objects* .  A VBO is a block of memory in the GPU that can store the coordinates or attribute values for a set of vertices.

    - OpenGL 1.1 introduced *texture objects to* make it possible to store several images on the GPU for use as textures. 
    - OpenGL was a giant machine, but still not pleasing everyone. The real solution was to make the machine **programmable**.
        - With OpenGL 2.0, it became possible to write programs to be executed as part of the graphical computation in the GPU. 
        - The programs are called *shaders*.
    - In OpenGL 3.0, the usual per-vertex and per-fragment processing was deprecated. 
    - And in OpenGL 3.1, it was removed from the OpenGL standard, although it is still present as an optional extension.
    - In practice, all the original features of OpenGL are still supported in desktop versions of OpenGL and will probably continue to be available in the future. 
        - On the embedded system side, however, with OpenGL ES 2.0 and later, the use of shaders is mandatory, and a large part of the OpenGL 1.1 API has been completely removed. 
        - WebGL, the version of OpenGL for use in web browsers, is based on OpenGL ES 2.0, and it also requires shaders to get anything at all done.
    - Nevertheless, we will begin our study of OpenGL with version 1.1. Most of the concepts and many of the details from that version are still relevant, and it offers an easier entry point for someone new to 3D graphics programming.

 - OpenGL shaders are written in GLSL (OpenGL Shading Language).
    - We will spend some time later in the course studying GLSL ES 1.0, the version used with WebGL 1.0 and OpenGL ES 2.0. 
    - GLSL uses a syntax similar to the C programming language.

---

# Chapter 2 Two-Dimensional Graphics

## 2.1 Pixels, Coordinates, and Colors

 - *Antialiasing* is a term for techniques that are designed to mitigate the effects of aliasing. 
    - The idea is that when a pixel is only partially covered by a shape, the color of the pixel should be a mixture of the color of the shape and the color of the background. 
    - In practice, calculating this area exactly for each pixel would be too difficult, so some approximate method is used.


## Section 2.2 Shapes

 - Line
    - A simple one-pixel-wide line segment, without antialiasing, is the most basic shape.
        - One of the first computer graphics algorithms,  **Bresenham's** algorithm for line drawing, implements a very efficient procedure for doing so. 
    - In any case, lines are typically more complicated. Antialiasing is one complication. Line width is another. A wide line might actually be drawn as a rectangle.
    
## Section 2.3 Transforms

### 2.3.8  Matrices and Vectors

 - If we want to apply a scaling followed by a rotation to the point v = (x,y), we can compute either R(Sv) or (RS)v. 
 - Translation is not a linear transformation. 
    - To bring translation into this framework (rotate and scale) , we do something that looks a little strange at first: Instead of representing a point in 2D as a pair of numbers (x,y), we represent it as the triple of numbers (x,y,1). 
    - It then turns out that we can then represent rotation, scaling, and translation—and hence any affine transformation—on 2D space as multiplication by a 3-by-3 matrix.
    - The matrices that we need have a bottom row containing (0,0,1). Multiplying (x,y,1) by such a matrix gives a new vector (x1,y1,1).

![](../imgs/cg_transform.png)


## 2.4 Hierarchical Modeling 

In this section, we look at how complex scenes can be built from very simple shapes. The key is hierarchical structure.

### 2.4.1  Building Complex Objects
 
 - When drawing an object, use the coordinate system that is most natural for the object.
 - Usually, we want an object in its natural coordinates to be centered at the origin, (0,0), or at least to use the origin as a convenient reference point. 
    - Then, to place it in the scene, we can use a scaling transform, followed by a rotation, followed by a translation to set its size, orientation, and position in the scene. 
    - Recall that transformations used in this way are called  *modeling transformations*.
    - The transforms are often applied in the order scale, then rotate, then translate, because scaling and rotation leave the reference point, (0,0), fixed.
        - in the code, the translation would come first, followed by the rotation and then the scaling:  TRS
        - Modeling transforms are not always composed in this order, but it is the most common usage.
 - The modeling transformations that are used to place an object in the scene should not affect other objects in the scene. 
    - we can save the current transformation before starting work on the object and restore it afterwards. 
    - but let's suppose here that there are subroutines saveTransform() and restoreTransform() for performing those tasks.

```
saveTransform()
translate(dx,dy) // move object into position
rotate(r)        // set the orientation of the object
scale(sx,sy)     // set the size of the object
     .
     .  // draw the object, using its natural coordinates
     .
restoreTransform()
```

 - Note that we don't know and don't need to know what the saved transform does. 
    - The modeling transform moves the object from its natural coordinates into its proper place in the scene. 
    - Then on top of that, a coordinate transform that is applied to the scene as a whole would carry the object along with it.
 - Let's look at a little example. Suppose that we want to draw a simple 2D image of a cart with two wheels.
    - ![](../imgs/cg_draw_2d_car.png)
    - The wheel has radius 1.
    - The rectangular body of the cart has width 6 and height 2.
        - it is convenient use the midpoint of the base of the large rectangle as the reference point.
        - assume that the positive direction of the y-axis points upward, which is the common convention in mathematics.
        - so the coordinates of the lower left corner of the rectangle are (-3,0)
    - In wheel's coordinate system, the wheel is centered at (0,0) and has radius 1.
 - Here is pseudocode for a subroutine that draws the cart in its own coordinate system:

```
subroutine drawCart() :
    saveTransform()       // save the current transform
    translate(-1.65,-0.1) // center of first wheel will be at (-1.65,-0.1)
    scale(0.8,0.8)        // scale to reduce radius from 1 to 0.8
    drawWheel()           // draw the first wheel
    restoreTransform()    // restore the saved transform
    saveTransform()       // save it again
    translate(1.5,-0.1)   // center of second wheel will be at (1.5,-0.1)
    scale(0.8,0.8)        // scale to reduce radius from 1 to 0.8
    drawWheel(g2)         // draw the second wheel
    restoreTransform()    // restore the transform
    setDrawingColor(RED)  // use red color to draw the rectangles
    fillRectangle(-3, 0, 6, 2)      // draw the body of the cart
    fillRectangle(-2.3, 1, 2.6, 1)  // draw the top of the cart
```
     
 - Once we have this cart-drawing subroutine, we can use it to add a cart to a scene. When we do this, we apply another modeling transformation to the cart as a whole. Indeed, we could add several carts to the scene, if we wanted, by calling the drawCart subroutine several times with different modeling transformations.
 - Building up a complex scene out of objects is similar to building up a complex program out of subroutines.
 
### 2.4.2  Scene Graphs
 
 - Logically, the components of a complex scene form a structure. 
    - In this structure, each object is associated with the sub-objects that it contains. 
    - If the scene is hierarchical, then the structure is hierarchical.
    - This structure is known as a **scene graph**.   
    - A scene graph is a tree-like structure, with the root representing the entire scene, the children of the root representing the top-level objects in the scene, and so on.


