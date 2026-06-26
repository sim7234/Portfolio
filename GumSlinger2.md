# Gumslinger 2 : [Back](https://github.com/sim7234/Portfolio/blob/main/README.md)

* Full time internship 2025 December - 2026 June
* Engine: Unity
* Genre: Mobile, 3D, Physics based duel game against bots with player stats.
* Released on January 20 2026 for Android and IOS.
* Worked with 10 other people.

## Link to [Itatake and Gumslinger2](https://itatake.com/portfolio/gumslinger-2-ducks-nukes/)

## Concepts i have worked with


* ### Systems that support remote controlled balancing and rebalancing once live

* ### Refactored existing code with data that was already live and could not be modified

* ### Followed code standards and practices of an existing project

* ### Dynamic UI for any aspect ratio with localization support

### Features i have worked on:

* Animation triggers in gameplay with corresponding sound cues

* A way to gain points from leaderboard and showing it as medals in profile



<details>

<summary>Animation trigger code snippet</summary>

```CSharp
private const float CloseHitSqrDistance = 15;
public bool CloseHit => closeHit && !HitSoftbody;

protected bool closeHit;


private void FixedUpdate()
{
    if (IsLive && ammo.TargetTransform != null && !CloseHit)
    {
        var sqrDist = Vector3.SqrMagnitude(transform.position - ammo.TargetTransform.position);
        if (sqrDist < CloseHitSqrDistance)
        {
            closeHit = true;
        }
    }
}
 ```
Then we check before a player starts their turn if the opponents last bullet was a close hit and check if
we should play a taunt animation to avoid annoying the player we reduce the chance for taunts to be played each time they trigger.

```CSharp
var rnd = Random.Range(0, 10 - TauntsPlayed);
var lastBullet = opponent.LastBullet;
if (rnd >= 5 && lastBullet != null && lastBullet.Projectile.CloseHit)
    {
         TauntsPlayed++;
         await PlayAvatarAnimationAsync(AvatarAnimation.TauntMiss, token);
     }
 ```

 Close miss logic was quite simple but playing a hit animation was more difficult, as we want to replace the normal stand up animation with a "Taunt" animation and as softbodies animations uses forces applying them to soon or late would give internal errors for the softbody, effectivly crashing it temporarily which looked of.
 </details>

 <br>
