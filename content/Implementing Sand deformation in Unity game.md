---
title: Implementing Sand deformation in Unity game
description: 
date: 2020-06-20
permalink: 
aliases: 
tags: 
draft: false
---
This article teaches you how to implement the sand balls game mechanics in unity.
![](https://i.imgur.com/8S0alQE.png)

I wanted to implement the **deforming** of the mesh as in the sand balls game the sand plane gets deformed on the player touch input.

After a vigorous search in the internet I have completely implemented the mechanics of deforming though I don’t know this is the right way to do it. But it **works** right!!😉

Before we continue I want you to know that the source code of my game code in in the **[github](https://github.com/Hemanth759/sand-balls-game)**.

----
## **Some basics on 3d models:**
- All models are made up of vertices and triangles.
- More the vertices more the complexity of the 3d model.
- The triangles form the shape of the model.
- The triangles are made up of any three vertices from the vertices list of the 3d model.
![](https://i.imgur.com/IfvrV3h.png)
# Implementation
## Sand plane
- The plane is simple primitive game object from the inbuilt unity editor.
- I used the pro-builder tool from the package manager to sub-divide the sand plane into many sub parts. This adds many vertices to the sand plane. (Default plane contains only four vertices)
- Don’t sub-divide the plane many times as that kills the performance of the game and takes more time to compute the mesh (more on this later).
- Also don’t sub-divide the plane less times even though that increases the performance the sand plane won’t be smooth.
- Just find the best number of times you want to sub-divide the plane after testing the with the performance of the game on different number of vertices.

## Physics
The source code of the project is hosted on my github [repo](https://github.com/Hemanth759/sand-balls-game).
<script src="https://gist.github.com/Hemanth759/6b63260c9aa6f2a93b8351da9328b3d5.js"></script>
- When ever the touch inputs gets detected we get the position of hit with the raycast and deform the plane area with some radius around the hit point forming a circle
- To deform the area we just loop through the vertices of the plane and calculate the distance of the vertices from the hit point. If the distance if less then the specified radius then push the vertex back by adding vector.forward to the vertex position.
- Don’t forget to update the meshcollider mesh with the new modified mesh as the balls need to fall through the deformed area.
![](https://i.imgur.com/da1bHdv.png)

----
# Game Play:

![](https://youtu.be/639eq9GLMhU?si=H_wUWqVeETThWfJW)

----
Now that we have created the physics behind the sand balls game implementation add some gameplay feature to it the way you want like some cool particle effects on success and some nice UI whichever you want. Always remember to have fun doing projects on unity. Thanks for reading and if you find this helpful consider liking the post. If anybody has any optimization to the method I have discussed I am always keen to learn new ways so don’t mind to ping me in Unity connect. Thanks for reading till the end😁. Play the game here: https://hemanth759.github.io/sand-balls-game/.
