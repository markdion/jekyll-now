---
layout: post
title: Infinite Asteroid Belt Prototype
image: https://img.youtube.com/vi/Z4Bx42ldDkM/maxresdefault.jpg
---

Recently I discovered the 3D skybox. It's a bit of a misnomer because it's not really a skybox. It is a technique that uses 2 cameras rendered on top of each other, with each camera moving at different speeds to produce the illusion that the objects in the rear camera view are massive and extremely far away.

This can be really useful for games with objects that are much larger than the player. It's a pain to work with an object 50,000 times bigger than most of the other objects in your scene. Using the 3D skybox technique you can use a moderately large object in the editor to look huge in-game. It also comes in handy if you don't want your player to be able to reach a location, but you still want it to look like a distant part of the map.

I used this technique to create a planet with an asteroid belt that is so huge that the player can never seem to escape it. I put the planet and the asteroid belt in the rear camera view to appear nearly infinitely far away, and I made sure the player would be placed somewhere inside the belt. Then I added an infinite asteroid field to the player object that will infinitely spawn new spheres as she moves through the scene.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Z4Bx42ldDkM" frameborder="0" allowfullscreen></iframe>

It's not quite as effective as I'd like since the pop-in of the new asteroids in front of the player is pretty obvious. I'd like to add fog or sun shafts to hide the new asteroids a bit so they slowly fade into view. Stay tuned for that.