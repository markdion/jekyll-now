---
layout: post
title: Release notes for v0.2
---

It's been a while since the last update. I've been waiting to release a new version and its finally ready! It has lots of new stuff, mostly little tweaks to make the game look more consistent and polished. No new features, I focused on refining a lot of the core elements of the game, but it looks like it has a coherent style now.
 
Here's a video, watch it!

# Detailed changes:
 
## Visual
 * Changed the project's Color Space from Gamma to Linear for more realistic lighting
 * Changed the font of all text in the game to one unified style
 * Adding speeding line particles to the camera to give boosting a sense of speed
 * Moved dialog so it's more visible and easier to read
 * Added camera shake that uses Perlin noise to determine its movement
   * Shake the camera when the player gets hit
   * Shake the camera when boosting
 * Added realistic atmosphere (i.e. outer glow) to planets
 * Prevented far away objects from sparkling (commonly referred to as Z fighting)
 * Increase/decrease thrusters based on the player's speed
 * Improved enemy target Indicator
   * Alpha gets lower as target gets closer
 * Improved objective indicator
 * Now displays the distance to the target under the objective icon
 * Improved menu buttons and quest menu look
 * Now displays the current objective in the top-right corner
 * Added an endless AI battle to the main menu screen
 * Restyled the player health bar
 
## Logic
 * Prevented the player from reaching a planet surface by starting a destruction countdown when they approach
 * Improved enemy AI (tutorial to see how!)
 * Improved the look of player movement
   * The player moves towards the edges of the screen when they turn, instead of always being in the              center
   * The player tilts properly on the z-axis when turning and strafing
 
A free download of Starship Siege v0.2 can be downloaded from the Downloads page for Windows. Hopefully I'll have a Mac version by v0.3. I appreciate any and all comments or questions you send me!