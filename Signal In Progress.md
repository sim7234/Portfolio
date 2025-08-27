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

### How it started
>Originally i started making a base script for all minigames, it was supposed to handle all common functionality, like interaction, Sucess and failure detection, and at the time camera positioning. After many reworks of the script due to game design changes it was eventually scrapped as each minigame became to distinct to justify keeping a base script.


<td ><img src="Signal_In_Progress\SkillCheckGif.gif"/></td>


#### This minigame's job is to reset the monster upon completion and it has gone through many itterations.


### Thought process first minigame
>I started with figuring out how to detect when a rotating arrow hits a zone inside a circle, at the time we had no need to allow the skill check to appear anywhere so i settled on 4 locations and allowing its size to randomly go clockwise or counterclockwise which allows for 8 different configurations, togheter with a size variance this felt good enough.

>All though if i remade it today i would allow for the zone to appear anywhere on the circle except the first and last ~30 degrees.

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

<details>
<summary>Zone selection Code</summary>
        
```csharp

void RandomizeFillOrigin()
{
    do
    {
        currentOrigin = Random.Range(0, 4);
    } while (currentOrigin == lastOrigin);

    rndClockwise = Random.Range(0, 2);

    if (rndClockwise == 0)
    {
        skillCheckZone.fillClockwise = true;
    }
    else
    {
        skillCheckZone.fillClockwise = false;
    }

    skillCheckZone.fillOrigin = currentOrigin;
    lastOrigin = currentOrigin;
}

```
</details>

* Hammer nails minigame

<td ><img src="Signal_In_Progress\HammerGif.gif"/></td>

#### This minigames function is to keep the monsters out, the nails reprisent health of the hull and fall off when they have been hit enough.

### Though process second minigame

> At the start of the project this was supposed to be a base script for a "press button" type minigame, where other minigames could inherit the functionality mouse press detection, click amount and if a button could decrease its clicked amount over time.

>This worked well in the beginning but over time with game design changes the script was changed from a base class to a single minigame "Hammer nails", which was also made into a hull for the monster to attack.

With time running out and mutliple scripts being reworked that was connected to this minigame we mostly made the decision to make the code *work* instead of being the most readable, something we all came to regret, but we made it work with no noticable bugs or performance problems.

<details>

<summary>Nail hit functionality</summary>
        
```csharp

void ClickOnObject(GameObject target)
{
    if (target == null) return;

    if (!target.TryGetComponent<CanBeClicked>(out CanBeClicked clickable)) return;

    if (clickable.TryGetComponent<Rigidbody>(out var rb)) return;

    float rndPitch = UnityEngine.Random.Range(0.8f, 1.2f);

    if (clickable.canClick && clickable.activated &&
        (toolsRequirement == miniGameToolRequirements.none || usedTool.toolType == toolsRequirement))
    {
        if (clickable.nailHealth < nailMaxHP)
        {
            clickable.gameObject.transform.localPosition += new Vector3(moveNailAmount / nailMaxHP, 0, 0);

            clickable.nailHealth += 1;

            if (clickable.shouldFall == true)
            {
                clickable.shouldFall = false;
            }
            hullHP += 1;

            if (clickable.nailHealth > nailMaxHP)
            {
                clickable.nailHealth = nailMaxHP;
            }

            if (hullHP > dangerZoneHP && playInDangerSound && source != null)
            {
                playInDangerSound = false;
                StartCoroutine(nameof(StopDangerSound));
            }

            if (clickable.nailHealth == nailMaxHP)
            {
                nailCompletedParticle.gameObject.SetActive(true);
                nailCompletedParticle.transform.position = clickable.transform.position;

                nailCompletedParticle.gameObject.transform.parent = gameObject.transform;
                nailCompletedParticle.transform.rotation = clickable.transform.rotation;
                nailCompletedParticle.gameObject.transform.localPosition -= new Vector3(0.15f, 0, 0);
                nailCompletedParticle.Emit(10);
                nailCompletedParticle.Stop();
            }
        }


        if (AudioManager.Instance is not null && clickable.nailHealth != nailMaxHP)
            AudioManager.Instance.PlayAudioClip(transform.position, "Hammer", "2D", true, 0.08f, rndPitch);
        else if (AudioManager.Instance is not null && clickable.nailHealth == nailMaxHP)
            AudioManager.Instance.PlayAudioClip(transform.position, "Hammer", "2D", true, 0.13f, rndPitch);
    }

  ```
  
  </details>