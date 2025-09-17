# 2D Shooter : [Back](https://github.com/sim7234/Portfolio/blob/main/README.md)

* Side project 2025 Feb - Ongoing
* Engine: Unity
* Genre: 2D, Top down shooter
* 1 programmer
<td ><img width="512" height="
" src="ChaosShooter\Fighting.gif"/></td>

# Goal

For this game my goal was simply to experiment with different concepts in programming, especially scriptable objects and class inheritance that i didn't fully understand at the start of this project. <br> But for the game itself i
just had the idea you win when you crash and i wanted you to win late with chaos everywhere.

The first major system i worked on is the games perk creation system, i wanted a way to quickly and easily create perks using scriptable object with lots of settings on how they are triggered and what the perk does once triggered.

  <tr>
    <td ><img width="512" height="
" src="ChaosShooter\PerkCreation.gif"/></td>
  </tr>

  I can't display the entire perk creation and selection system but i can explain and show parts of it, here is code for how a perk is selected or leveled up after the user as given input on what perk they want.

  <details>

  <summary> Selection of perks </summary>


  ``` CSharp

  public void UserSelectedPerk(int index)
{
    if (selectMajorPerk == false)
    {
        Stats.AddStatsFromMinorPerks(statList[index].perkName, statList[index].amount);
        statList.Clear();
    }
    else
    {
        MajorPerks[] activePerks =
            (from majorPerks in MajorPerkManager.instance.AllPerks
             where MajorPerkManager.instance.ActivePerks.Contains(selectablePerks[index])
             select majorPerks).ToArray();

        if (activePerks.Length > 0)
        {
            selectablePerks[index].LevelUp();
        }
        else
        {
            MajorPerkManager.instance.ActivePerks.Add(selectablePerks[index]);
            selectablePerks[index].OnGetPerk();
        }

        selectablePerks.Clear();
        selectMajorPerk = false;

    }

    for (int i = 0; i < buttonArray.Length; i++)
    {
        majorPerkIndex[i] = 0;
    }

    foreach (Button b in buttonArray)
    {
        b.gameObject.SetActive(false);

    }

    perkAmountLeft--;
    ChangePerkText();
    perkSelected = true;
    if (perkAmountLeft <= 0)
    {
        PauseManager.isPaused = false;
    }
    buttonIndex = 0;
    perkModulo = (perkModulo + 1) % howOftenGreatPerk;
    KeyWordDisplay.instance.ClearText();

}
```

</details>

### Some explanations of how i setup the perk system

Each perk has a single or multiple trigger(s), a rarity a description/name keywords that might be associated and requirements.

Below i have picked out part of the script as the full script repeats with different trigger conditions, all perks inherit from this script.

I have also sent an example of a perk "Arc perk" which just activates the logic of an arc, which i picked out a function for.

<details>

<summary> Base class for perks </summary>

``` CSharp
using System;
using UnityEngine;
[CreateAssetMenu(fileName = "MajorPerks", menuName = "Scriptable Objects/MajorPerks")]
public class MajorPerks : ScriptableObject
{
    [SerializeField] protected MajorPerkTrigger trigger;
    [SerializeField] protected EffectTriggers EffectTriggers;

    public string perkDescription;
    public KeyWords KeyWords;

    public Rarity rarity;

    private int perkLevel = 0;

    [Header("Decide what perks are requied to get this perk")]
    public bool requireAll;
    public MajorPerks[] perkRequirement;


    public int GetPerklevel()
    {
        return perkLevel;
    }

    /// <summary>
    /// Each perk effect has to impliment this individually.
    /// Decides what happens when a perk is picked more then once.
    /// </summary>
    public virtual string LevelUp()
    {
        perkLevel++;
        return null; // Description of what the level up does.
    }

    public virtual void OnGetPerk()
    {

    }

    public virtual void OnHit(GameObject arrow, GameObject hitTarget)
    {
        if ((trigger & MajorPerkTrigger.OnHit) == MajorPerkTrigger.OnHit)
        {
            Activate(arrow);
            Activate(arrow, hitTarget);
        }
    }

    public virtual void OnKill(GameObject arrow)
    {
        if ((trigger & MajorPerkTrigger.OnKill) == MajorPerkTrigger.OnKill)
        {
            Activate(arrow);
        }
    }


```

</details>

<br>

<details>

<summary> Arc Perk </summary>

``` CSharp

using UnityEngine;

[CreateAssetMenu(fileName = "MajorPerks", menuName = "Scriptable Objects/Perk/ElectricArc")]
public class ElectricArc : MajorPerks
{
    public int arcAmount;
    public float arcJumpDistance;
    public int damage;

    public int lvUpAmount;
    public int lvUpJumpDistance;
    public int lvUpDamage;

    public LayerMask targetsHitByArc;

    public GameObject lightningArcObj;

    public int chanceOutOf100;
    public Gradient colorOfLine;

    bool addToPool = true;

    //TODO: Object pool ArcLogic gameObjects
    public override string LevelUp()
    {
        base.LevelUp();
        damage += lvUpDamage;
        arcJumpDistance += lvUpJumpDistance;
        arcAmount += lvUpAmount;

        string description = "";
        if (lvUpAmount > 0)
        {
            description += "Arc Amount + " + lvUpAmount + " ";
        }
        if (lvUpJumpDistance > 0) description += "Arc max jump distance +" + lvUpJumpDistance + " ";

        if (lvUpDamage > 0) description += "Damage + " + lvUpDamage + " ";

        return description;
    }

    public override void Activate(GameObject projectile)
    {
        ArcLogic temp = ArcPool.instance.GetArcLogic();
        temp.gameObject.SetActive(true);
        temp.SetArcSettings(projectile.transform.position, this);

        if (addToPool == true)
        {
            GeneralObjectPool.instance.AddObjectToPool(lightningArcObj);
        }
    }
}

```

</details>

<br>

<details>

<summary> Arc perk logic </summary>

``` CSharp
 private void SpawnArc()
 {
     AudioClip clip = AudioManager.instance.GetClip(AudioClips.Zap);
     if (clip != null)
         AudioManager.instance.PlaySound(clip, 0.3f, 0.8f, 0.9f);

     Vector2 lineStart = targetsInRangeList[currentArcs - 1].transform.position;
     Vector2 lineEnd;
     if (currentArcs < arcAmount && currentArcs < targetsInRangeList.Count)
     {
         lineEnd = targetsInRangeList[currentArcs].transform.position;
     }
     else
     {
         lineEnd = targetsInRangeList[currentArcs].transform.position;
     }

     MajorPerkManager.instance.OnArcHit(targetsInRangeList[currentArcs]);
     

     lrendererObj.SetActive(true);
     LineRenderer lineRenderer = lrendererObj.GetComponent<LineRenderer>();
     lineRenderer.colorGradient = colorOfLine;
     lineRenderer.SetPosition(0, lineStart);
     lineRenderer.SetPosition(1, lineEnd);

     if (targetsInRangeList[currentArcs].TryGetComponent<Health>(out Health hp))
     {
         hp.TakeDamage(damage, damageType.Lightning);
     }
 }

```

</details>