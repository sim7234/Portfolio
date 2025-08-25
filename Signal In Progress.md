# Signal In Progress : [Back](https://github.com/sim7234/Portfolio/blob/main/README.md) <-- Click to go back.
<img width="1080" height="678" alt="Signal in progress main menu" src="https://github.com/user-attachments/assets/49aa1f66-fc5f-4cfc-a390-1fbefac2e23e" />

# What i worked on
* Minigames: A core part of the gameplay is minigames and i worked on two of them,
* Connecting different scripts to work together in a cohesive way.
* Patching problems and smoothing out gameplay where needed.
* Helped with getting art properly integrated, including animations.

* Skillcheck minigame
![Unity_qHTh1048mZ-ezgif com-optimize](https://github.com/user-attachments/assets/0c35b6dc-7667-4b03-ac3a-aedb9acad41e)

This minigame's job is to reset the monster upon completion and it has gone through many itterations,

<details>
<summary>SkillCheck detection code</summary>
```CS
        
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


* Hammer nails minigame
