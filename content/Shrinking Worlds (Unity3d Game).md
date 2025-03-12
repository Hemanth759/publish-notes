---
title: Shrinking World a Unity3d Game
description: 
date: 2019-12-10
permalink: 
aliases: 
tags: 
draft: false
---
![](https://i.imgur.com/IoiKXUc.jpeg)

# Introduction
Our idea is to make a unity 3d game where the user controls a car on a small planet while the planet is showered with meteors in all directions with small delay in between each meteor. Sounds fun!!. Lets make the planet size shrink over time to increase the difficulty of the game. This project is inspired by the famous Youtuber [**Brackeys**](https://www.youtube.com/channel/UCYbK_tjZ2OrIZFBvU6CCMiA) in one of his video tutorials. So without any further ado lets get to the development part.

For Source code of our project follow below **[github](https://github.com/Hemanth759/Shrinking-World)** link.

# Prerequisites:

- Basic programming in C# language
- Basics of the Unity3d engine
- Vector algebra (for movement player on the planet)
- Particle System in Unity3d
- Basics of Blender for 3d modelling

# Mechanism:

## Planet:

Planet is just a 3d model of the sphere created in blender and adding some texture to look like a grass and a ring which acts like a road on the planet.

<p align="center">
<img width="33%" src="https://i.imgur.com/DaO5kFo.png">
</p>

## Car:

Its a simple car model created again with blender. (BTW blender is super cool for modelling and its open source). Adding some color to each part of the car model gave us pretty nice model.

![](https://i.imgur.com/7ULEV7V.png)

## Working:

Now that we got the models we need to add these to the unity project folder and have to do some coding for implementing gravity, player controller.

- First of all disable the unity’s inbuilt gravity as we will be making the gravity ourselves. To disable gravity head to edit-> project setting-> physics and change the ‘y’ value of gravity to zero.

![](https://i.imgur.com/gcvOFvR.png)
- Now that gravity is disabled just create a instance of the planet model in scene window a origin. Change the transform position properties of the planet model in the scene to orgin by changing like below.

![](https://i.imgur.com/pLCJdPx.png)

Adding Lighting and placing player(car) on planet:

![](https://i.imgur.com/QqI2dXW.png)

- Place the Planet a the center and add some directional lights to light up the world.

![](https://i.imgur.com/zLWG5mW.png)

- Place the car model on the surface of the planet model don’t bother about perfection of that the car should touch the surface of the planet. Our gravity code will take care of that particulars.
- As the planet is the attractor which attracts all other object lets add some components to that gameobject(models in unity are called gameobjects).

![](https://i.imgur.com/1gnzTOq.png)

- Add sphere collider to detect if any other gameobject collides with the planet object.
- Add a script called FauxGravityAttractor.cs to the planet model and the code for that script follow as —

<script src="https://gist.github.com/Hemanth759/3146586691b3e10168f60fb84a531ff1.js"></script>

- Here awake function is called when the class is initialized for the first time so there will be only one instance of the FauxGravityAttractor this type of class are called singleton class as there will only be on instance of the class. For more information on singleton classes I suggest you to learn C# OOPS.
- Place on surface function is used when the objects which are static on the planet need to be fixed on the surface of the planet as the planet size is shrinking continuously.
- Attract function is what we are most interested in. This function is called by every game object which is to be attracted by the planet and is a moving object. Since the rotation change in the attract function.
- The first line in the attract function finds the gravity normalized direction and second line adds the force to the gameobject towards the planet direction (gravityup). The next lines change the rotation of the gameobject so that the ‘y’ of the game object is always to the gravityup direction. This ensures the car not to rotate to different weird orientations.

## **CAR:**

![](https://i.imgur.com/mhQYcIR.png)

- Add the rigidbody for the player or car game object, box collider to detect the collision, and add a script called FauxGravityBody which on each frame of the game updates the gravity force direction and rotation of the game object according to the planet (FauxGravityAttractor) gameobject transform values(location).

<script src="https://gist.github.com/Hemanth759/60a7bb236b839ce69d3599cde001e2ce.js"></script>

- In the start function the fauxGravityAttractor gets the only instance of that class which is the only planet and on every fixed update(Physic update) this gameobjects rigidbody adds force towards the planet and changes the rotation if it is a moving object else places the gameobject on the planet surface (Remember the planet shrinks every frame with some speed, which we will implement soon).

<script src="https://gist.github.com/Hemanth759/9a12cebbceba36486ae0861a96282b70.js"></script>

- Add some public variables movespeed for speed of the player and rotation speed of the car.
- In the start function make the isDead of the player to false and on each frame update increase the score by delta time (time taken to update the frame) if the player is not dead else don’t increase the score of the player and also take the input information of horizontal axis (mapped to keys left, right arrows and keys a,d) in each frame.
- In each physics update(fixed update) move the rigidbody of the player game object with speed towards the ‘z’ direction(forward direction of the gameobject) with speed multiplied by fixed delta time (time taken to update the physic frame).
- To turn the player car rotate the ‘y’ direction of the player game object with the value we got from the horizontal axis. But Unity engine uses Quaternion orientation so we need to convert the euler rotation value to quaternion rotation and apply rotation in quaternion system. That is what the last four lines of code are for.

<script src="https://gist.github.com/Hemanth759/ad5593b6fda6b1eba5cd0579bc5f219a.js"></script>

- Also add another script called PlayerCollision.cs on the player game object to detect the collision of the player object and see if the collision is due to the meteor or a crater by the tag on that gameobject.

## Meteor Spawner:
- Now add a empty game object to the scene with name MeteorSpawner and add a MeteorSpawner.cs script to that game object.

![](https://i.imgur.com/ToA30Ys.png)

<script src="https://gist.github.com/Hemanth759/f659731954b45315affce00841dab01d.js"></script>

- This script takes the meteor prefab(a saved gameobject in the project folder) and instantiates that prefab in this case meteor prefab with a random rotation of normalized vector multiplied by a distanceFromPlanet float value.
## Meteor Prefab:

- Meteor prefab is made of a particle system which gives off smoke and flashes fire at the center and gives smoke trails on movement.

![](https://i.imgur.com/UmLeHu9.png)

- You can get the free fire particle system from the above link or can make it by yourself with some practice.

![](https://i.imgur.com/uL410uH.png)

- Add sphere collider, rigidbody, and fauxGravityBody script to the meteor object which make the object fall towards the planet.

<script src="https://gist.github.com/Hemanth759/5a13d0d3d6842697c9666b9515bd0600.js"></script>

- Meteror script gives the onCollisionEnter method which instantiates the creater prefab on the point of collision.
- GameManager is a SingleTon class like FauxGravityAttractor class which has one creater gameobject prefab called ‘createrPrefab’.

![](https://i.imgur.com/ebsKwVH.png)

- Since Creater game object is a non-moving object it has a FauxGravityBody script attached to it with placeOnSurface bool to true.

![](https://i.imgur.com/IhCqhVh.png)

- Here crater script is a class in c# with extending FauxGravityBody class instead of MonoBehaviour like in all other class which helps us with only one script being on the creater prefab instead of two scripts. Just a method of OOPS to optimize the code. If you didn’t understand this step I suggest you to learn OOPS in C# which is quite useful in making Unity games.
- As you might have observed meteor and crater has tags of Meteor on them which is used to detect the collision with the player.

----
# Being creative from here onwards:
With this all done you must have completed the basic mechanism of the game physics. All thats left to do now is to create the UI for game, Audio, Animation effects and so on. Just being creative and experimenting with different tools will give you so much fun in the Unity.

For the source code of this project follow this [**github**](https://github.com/Hemanth759/Shrinking-World) link

Below is a game where we added some animation, UI, to the above mechanism and we hosted on github Pages for people to play for free and Enjoy.
https://hemanth759.github.io/Shrinking-World/

# References:

- [https://www.youtube.com/watch?v=gHeQ8Hr92P4](https://www.youtube.com/watch?v=gHeQ8Hr92P4) — for creating worlds and gravity
- [https://www.youtube.com/watch?v=jBqYTgaFDxU&list=WL&index=5](https://www.youtube.com/watch?v=jBqYTgaFDxU&list=WL&index=5) — for blender tutorial
- [https://www.youtube.com/watch?v=fZeZ75gM9p4](https://www.youtube.com/watch?v=fZeZ75gM9p4) — for music for games
- Music by **Jordan** Winslow on [https://jordanwinslow.me/royaltyfreemusic](https://jordanwinslow.me/royaltyfreemusic)
