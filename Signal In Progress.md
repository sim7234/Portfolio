# Signal In Progress : [Back](https://github.com/sim7234/Portfolio/blob/main/README.md) <-- Click to go back.
<img width="512" height="
" alt="Signal in progress main menu" src="Signal_In_Progress\MainMenu.png" />

* School group project, April 2025 - June 2025
* Engine: Unity
* Genre: 3D, Horror, Minigames
* 4 Programmers and 2 artists

# What i worked on
* Minigames: A core part of the gameplay is minigames and i worked on two of them.
* Connecting different scripts to work together in a cohesive way.
* Patching problems and smoothing out gameplay where needed.
* Helped with getting art properly integrated, including animations.

* Skillcheck minigame

<td ><img src="Signal_In_Progress\SkillCheckGif.gif"/></td>



This minigame's job is to reset the monster upon completion and it has gone through many itterations.

<details>
<summary>SkillCheck detection code</summary>
        
```csharp
        
bool CheckIfHitZone()
{
    float skillZonePositionMin = 0;
    float skillZonePositionMax = 0;

    //the 4 different origins change in 90 degrees top is 0, right is 90 bottom 180, left 270, top 360/0

    //the skillcheck arrow is based on rotationValue which goes from 0 to -360.

    switch (currentOrigin)
    {
        case 0:
            //bottom
            skillZonePositionMin = 180;
            skillZonePositionMax = 180;
            break;
        case 1:
            //right
            skillZonePositionMin = 90;
            skillZonePositionMax = 90;
            break;
        case 2:
            //top
            skillZonePositionMin = 0;
            skillZonePositionMax = 0;
            break;
        case 3:
            //left
            skillZonePositionMin = 270;
            skillZonePositionMax = 270;
            break;
        default:
            //top
            skillZonePositionMin = 0;
            skillZonePositionMax = 0;
            break;
    }

    if (rndClockwise == 0)
    {
        skillZonePositionMax += (skillCheckZone.fillAmount * 360);
        //skillzone max is bigger nummber then min
    }
    else
    {
        if (skillZonePositionMin == 0)
            skillZonePositionMin = 360;

        float temp = skillZonePositionMin;

        skillZonePositionMin -= (skillCheckZone.fillAmount * 360);
        skillZonePositionMax = temp;

        //skillzone max is smaller then min (because skillcheck goes opposite direction)
    }

    float arrowPosition = rotationValue * -1;

    if (arrowPosition >= skillZonePositionMin && arrowPosition <= skillZonePositionMax)
    {
        return true;
    }
    else
    {
        return false;
    }
}
```
</details>

* Hammer nails minigame
