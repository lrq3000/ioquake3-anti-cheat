Anti-Cheat Module for Ioquake3
=======================

### Copyright (C) 2012 Laszlo Menczel

(mirror by Stephen Larroque, 2016)
(see anticheat.html for original information)

### 1\. Intro

This document describes the server side anti-cheat module I have implemented for ioquake3\. The goals of the module are the following:

*   to prevent the use of wallhacks
*   to detect the use of aimbots

It should be quite easy to port to any game that uses the Q3A engine.

This is an alpha release for testing, only wallhack prevention is implemented at this time. Also it is not yet restricted to dedicated servers, so you can test it with bots on a single machine w/o starting a separate server. I have included a simple client-side wallhack (see 2.1) to make testing possible.

### 2\. Wallhack prevention

Wallhacks are prevented by restricting the information sent by the server. The positions of opponents that the player cannot actually see are changed so that the opponent is directly below the player. The distance of the player and its opponent is maintained so that sound scaling works correctly. A little bit of X/Y offset may also be added to help in estimating the direction of the sound (this is an option controlled by a Cvar, see below).

#### 2.1\. Cvars controlling wallhack prevention

**wh_active**  
If you set this Cvar to "1", the wallhack prevention code is activated. Default is zero (inactive).

**wh_add_xy**  
If you set this Cvar to "1", then the modified position of invisible opponents will contain a bit of X/Y (direction) information. Default is zero (do not include X/Y info).

**_Note:_** This information can be used to determine the exact position of hidden players. I am convinced that this option must be omitted from the final version. I just left it in so that testers can estimate the subjective difference between the two cases.

**wh_bbox_horz**  
**wh_bbox_vert**  
These are the dimensions (in Quake units) of the players' bounding boxes used for performing line-of-sight traces. All eight corners of the bounding box are checked for visibility. Default values are 30 and 60, respectively.

**_Note on bounding box dimensions:_**  
The above default values are considerably larger than the normal values (16 and 32) used by Q3A. The reason for this is that it is better to predict a player as visible when he/she is not than the contrary. It may give a slight advantage to cheaters using wallhacks, but IMO it is not significant. You can change these Cvars, but if you set them to smaller values then it may happen that players do not become immediately visible when you go around corners.

**cg_wallhack**  
This is provided for testing the functions of the wallhack detector. If you set this to "1" then a simple wallhack (based on changing an OpenGL parameter) is activated in the client. Default value is zero (inactive).

### 3\. Aimbot detection

[NOTE: NOT IMPLEMENTED YET!]  

Aimbots will be detected by using Dynamic Bayesian Networks trained to distinguish the playing style of cheating and honest players.  

### 4\. Implementation notes

### 4.1\. Wallhack prevention

The bulk of wallhack prevetion code is in the source file 'sv_wallhack.c'. There are other small changes in some of the server modules. The client side wallhack code is in 'cg_local.h', 'cg_main.c' and 'cg_players.c'.

All additions and changes are flagged by the marker '_ML_' and are enclosed in '#if defined ANTICHEAT ... #endif'.

The details of the algorithm of wallhack prevention are the following:

1\. When potentially visible entities are added to the snapshot sent to a client, we first check if the entity is another player. If yes, we check if the client can actually see the opponent or not (in this frame or in the next predicted one). Visibility checking is done as follows:

*   we check if the opponent is within the FOV of the player
*   we check if there is a direct line-of-sight from the viewpoint of the player to any of the corners of the opponent's bounding box
*   if the above tests are negative, we predict the position of the player and its opponent for a time point 100 msec later, and repeat the FOV and line-of-sight tests

2\. If the opponent is visible (or is predicted to be visible in the next frame) then its entity is added normally.

3\. If the opponent is not visible, we save its position then change it so that it is directly below the player at the same distance as before. The modified opponent entity is then added to the client snapshot.

4\. When the snapshot is built, each client entity is checked if its position has been changed by the wallhack code. If yes, it is reset to the original values after the entity is added to the snapshot.

The result is that a client using a wallhack cheat will always see the hidden players at a position directly below his/her feet (it will contain no X/Y info). Therefore, it is quite impossible to determine the real position from the data available to the client.

**_Note on sound position:_**  
As a consequence of changing opponent positions, the location of generated sounds are not correct. You may ask if this could interfere with the client's ability to estimate the location and distance of the opponent based on the sounds he/she hears.  

IMO it works because the distance of the sound source remains the same so the attenuation of sound is correct. The X/Y information is obviously lost, but I believe that it is less important. This loss may be an acceptable price to pay for the elimination of wallhack cheats.

### 5\. Compiling

For compiling under Windows you will need the MinGW/Msys system (a port of the GCC compiler suite to Windows). If you are unfamiliar with this system, see [this](http://linradiant.intron-trans.hu/mingw.html) document.

First you should adjust the variable COPYDIR in the file 'Makefile.local'. Set its value to the path of your Q3A game install directory.

To build the game + AC module switch to the directory 'ioquake3' and run the command 'make copyfiles'. This will build the game and install it to the directory you specified in 'Makefile.local'.

### 6\. Usage

The build system is set up to create dynamic libraries (.dll or .so) instead of VM modules. The supplied small 'autoexec.cfg' file contains the settings necessary to force ioquake3 to use these libraries. It also has settings for activating the client side wallhack and the server side anticheat code. Copy the content of this file to the end of your 'autoexec.cfg'.

### 7\. Feedback

If you have comments, questions or suggestions, or if you have found a bug please, send me an e-mail to this address: laszlo dot menczel at gmail.
