---
layout: post
title: Tutorial - Simple Spaceship AI
---

I've spent a while trying to think of the right way to create AI that looks fluid, makes gameplay fun, and is also challenging. Here I've figured at least some of that out. I've posted a video of 2 factions of ships battling to show how the AI looks in action. Some of the movement is a bit disjointed, and it obviously isn't perfect, but I think the core ideas are sound.

<iframe width="560" height="315" src="https://www.youtube.com/embed/_EiyNRBwuN0" frameborder="0" allowfullscreen></iframe>

With this AI I was really just trying to figure out a way to mimic how players act in dogfights. Obviously doing this accurately would be extremely complex and time-consuming, but I found a way to make it LOOK kinda close.
 
I'm not going to post any code in this tutorial, but I will go over all the concepts I used to design this AI, so implementing them yourselves should be pretty straightforward.
 
First, the AI has 2 modes it can be in: Evade and Attack, similar to what a real player would do. The purpose of Evade is to look like the ship is either trying to dodge an attack, or searching for a new target to fire at. Attack just makes the AI face a target, and start flying towards it while shooting at it.




### Evade
The ship is given a constant forward force to its Rigidbody so it is always flying forward. Then every 1-3 seconds it will choose a random new direction within 90 degrees in any direction of its current rotation, and then rotate towards that direction (using Slerp to make that transition appear smooth). This gives it the appearance of not having a particular target yet but just looking around and trying not to get attacked.

### Attack
The ship is given a random target (of the opposing faction) and then it is rotated towards that target (again using Slerp for a smooth rotation). Then it starts shooting and flying towards that target, constantly adjusting to face it. As soon as it gets close to the target it switches back to Evade mode. By doing this it gives the appearance that it is just doing an attack run, and then veering off before hitting its own hull against its target to prepare for its next attack.
 
Choosing between these 2 modes is also simple. The default and starting mode is Evade, and then every 5-10 seconds it'll switch into Attack mode, then switch back if it gets too close or destroys its target.
 
And that's it!
Incredibly simple with decent results. Now, of course I will be adding complexity to this logic for special cases like new weapons, abilities, special targets like larger ships, etc. but this is the core.
I'd also like to add reacting to getting hit by immediately dodging (or if in Attack mode switching to Evade), and also predictive aiming because as of now as long as I'm moving they rarely hit me.
I'll also probably have to add some sort of pathfinding since these ships could easily bang into each other or other obstacles like asteroids. But so far, in empty space, I haven't seen it happen, and the probability is quite low (I think).
 
Most importantly, this AI is fun to play against. Currently, its a bit on the easy side, but I'm confident with some adjustments it'll be better. It's fun chasing after these guys as they erratically change directions and it seems like they were designed to be difficult targets to hit as they fly around you, sometimes making runs at you forcing you to dodge and fly away. It reminds me of an old game called Freelancer that actually partly inspired this game, and it makes me happy that I could recreate some semblance of that.
 
This AI will be in v0.2 which will hopefully be released in the next few weeks.
 
Hopefully this provided a starting point for you to create your own AI!