# Speed Streak : [Back](https://github.com/sim7234/Portfolio/blob/main/README.md) <-- Click to go back.

* School project, March  2025 - April 2025
* Engine: Unreal Engine 5
* Genre: Parkour, Speed run, Movement
* 1 Programmer


<td ><img width="512" height="
" src="Unreal\level1.png"/></td>

## Goal:
To make a game with smooth responsive movement with wall running capability inspired by Titanfall 2.

How i started: I began by figuring out how custom movement would work in unreal, i made a simple ground slide by pushing the player forward, then a downwards slide by altering gravity and friction.





<td ><img width="512" height="
" src="Unreal\UnrealGroundSlide.png"/></td>

### Problems and solutions <br>
The first major problem for this project had to do with sliding down slopes, or more specifically going down then up a slope, as i used the normal of the ground underneath the player to calculate how much gravity to apply, this made going up a slope impossible and instantly stopped any sliding as if the player hit a wall.

Problem solving: So i thought about how to solve this problem, and decided on trying to predict when a player is about to go up a slope with the help of raycast that points towards the players velocity. <br>

 Then i preemptively set the players gravity to default so they could slide up the slope without issues and still slow down.

 <td ><img width="512" height="
" src="Unreal\UnrealNormals.png"/></td>