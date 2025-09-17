# Sun Seed : [Back](https://github.com/sim7234/Portfolio/blob/main/README.md) <-- Click to go back.

* School group project, 2024 November - 2025 January
* Engine: Unity
* Genre: 2D, Asymmetric, Local Co-op, Pve
* 3 programmers and 4 artists

Itch: https://yrgo-game-creator.itch.io/sun-seed

Trailer: https://www.youtube.com/watch?v=URABgAn8mho


  <tr>
    <td ><img width="512" height="
" src="SunSeed\SunSeedHub.png"/></td>
  </tr>


## Game Description

Sun Seed Vegan Knights is a game played on controllers with a friend. The goal of the game is to go through a level and slay monsters by planting, and watering seeds that grow into powerful weapons.

## What i worked on

I mainly worked on the enemies, their pathfinding using unitys navmesh systems and their attack patterns, i made a total of 4 enemies.

* A melee enemy that walks up to the player, makes an indicator towards them then slams down.
* A ranged enemy that looks towards players and shoots a small fireball towards them.
* A dash enemy that slowly walks towards the player, then makes a long indicator and a few seconds later dashes and damaging players in the indicated area.
* A explosive enemy, just runs up to the player and explodes but has little health.


<tr>
    <td ><img width="512" height="
" src="SunSeed\Dash.png"/></td>
<tr>

<br>

>Sun seed is the first major project i worked on, and i had just started learning proper programming 4 months earlier, this code is representative of my skills at that time.

<details>

<summary>Pathfinding Code</summary>

```CSharp


using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class Pathfinding : MonoBehaviour
{
    [HideInInspector] public List<GameObject> target = new List<GameObject>();

    NavMeshAgent agent;

    [HideInInspector] public int totalTargets;

    [HideInInspector] public int finalTarget;
    [HideInInspector] public int randomTarget;

    [HideInInspector] public bool followTarget = true;

    [HideInInspector] public bool trackTarget = true;

    [HideInInspector] public Vector3 targetTransform;

    [SerializeField] private bool targetPlayerPoints;
    private int randomTargetIndex;

    private void Start()
    {
        randomTargetIndex = Random.Range(0, 2);
        followTarget = true;
        agent = GetComponent<NavMeshAgent>();
        agent.updateRotation = false;
        agent.updateUpAxis = false;
        transform.rotation = new Quaternion(0, 0, 0, 0);
    }

    // Update is called once per frame
    public void Update()
    {
        finalTarget = FindClosestTarget(totalTargets);

        if (target.Count > 0)
        {
            if (target[finalTarget].gameObject == null)
                return;
            targetTransform = (target[finalTarget].transform.position);

            if (followTarget)
            {
                if (trackTarget == true && targetPlayerPoints == true)
                {
                    agent.SetDestination(target[finalTarget].GetComponent<PlayerTargetPoints>().GetTargetPoint(randomTargetIndex).transform.position);
                }

                if (trackTarget == true && targetPlayerPoints == false)
                {
                    agent.SetDestination(target[finalTarget].transform.position);
                }
            }
        }
    }

    public int FindRandomTarget(int totalTargets)
    {
        int rnd = 0;
        int i = 0;

        do
        {
            rnd = Random.Range(0, totalTargets);
            i++;

        } while ((!target[rnd].CompareTag("Objective")) || i >= 5);

        return rnd;
    }
    public int FindClosestTarget(int totalTargets)
    {
        float closestTarget = Mathf.Infinity;

        for (int i = 0; i < totalTargets; i++)
        {
            if (target[i].gameObject != null)
            {
                Vector3 targetDistence = target[i].transform.position - transform.position;
                float targetDistenceSquared = targetDistence.sqrMagnitude;

                if (targetDistenceSquared < closestTarget && target[i].GetComponent<isTarget>().enabled == true)
                {
                    closestTarget = targetDistenceSquared;

                    finalTarget = i;
                }
            }
        }
        return finalTarget;
    }
}

 ```
 </details>

 <br>

I also created a water display and gathering system that lets the player walk over water sources and slowly fill their water which gets displayed by UI.

I also helped in reworking scripts related to seed planting and growth, here is a script that allows the player to water seeds.


<details>

 <summary> Water System script </summary>

``` CSharp
using UnityEngine;
using UnityEngine.InputSystem;
using UnityEngine.UI;

public class WaterSystem : MonoBehaviour
{
    //this script needs a "CanWater" script on any potential object you wish to be able to water
    //this script interacts with NaturalWater, PlantSeedSystem.

    public float maxWater = 10.0f;

    public float currentWater;

    private float wateringCooldown = 0.012f;
    //this value is changed from PlantSeedSystem to make sure you cant water at the same time you plant.
    [HideInInspector] public float wateringTimer;

    private float baseWaterRefillCooldown = 1f;
    //these values can be changed by other scripts. Mainly "NaturalWater" script.
    [HideInInspector] public float waterRefillCooldown;
    [HideInInspector] public float waterRefillTimer;

    private float waterPercentage;

    bool waterButtonHeld;

    private InputAction waterAction;

    [SerializeField] Image waterOutLineImage;
    [SerializeField] Image playersWaterImage;

    bool canDisableWater = true;

    float displayWaterTimer;
    void Start()
    {
        DisplayWater(false);
        waterRefillCooldown = baseWaterRefillCooldown;
        currentWater = maxWater;
        waterRefillTimer = waterRefillCooldown;
    }
    void Update()
    {

        if (waterAction.ReadValue<float>() > 0)
        {
            waterButtonHeld = true;
        }
        else
        {
            waterButtonHeld = false;
        }

        changeImageFill();
        refillWater();

        if (displayWaterTimer > 0)
        {
            DisplayWater(true);
            displayWaterTimer -= Time.deltaTime;
        }
        else
        {
            if (canDisableWater == true)
            {
                DisplayWater(false);
            }
        }
    }
    private void Awake()
    {
        var playerInput = GetComponent<PlayerInput>();
        if (playerInput != null)
        {
            waterAction = playerInput.actions["Water"];
        }
    }

    void changeImageFill()
    {
        waterPercentage = (currentWater / maxWater);

        if (playersWaterImage.fillAmount > waterPercentage)
        {
            playersWaterImage.fillAmount -= Time.deltaTime * 0.8f;
        }

        if (playersWaterImage.fillAmount < waterPercentage)
        {
            playersWaterImage.fillAmount += Time.deltaTime * 0.4f;
        }
    }
    void refillWater()
    {
        if (wateringTimer > 0)
        {
            wateringTimer -= Time.deltaTime;
        }

        if (waterRefillTimer > 0)
        {
            waterRefillTimer -= Time.deltaTime;
        }

        if (waterRefillTimer <= 0 && currentWater < maxWater)
        {
            currentWater++;
            waterRefillTimer = waterRefillCooldown;
        }
    }

    private void OnTriggerStay2D(Collider2D other)
    {
        var canWater = other.GetComponent<CanWater>(); //Comment from future me, get component each frame... why.
        if (canWater != null)
        {
            DisplayWater(true);
            canDisableWater = false;

            if (waterButtonHeld && wateringTimer <= 0 && canWater.canBeWatered)
            {
                wateringTimer = wateringCooldown;
                if (TakeWater(canWater.waterCostPerAction))
                {
                    canWater.currentWater += canWater.waterCostPerAction;
                }
            }
        }
        else if (other.GetComponent<WaterObjective>() != null)
        {
            DisplayWater(true);
            canDisableWater = false;
        }

        var waterObjective = other.GetComponent<WaterObjective>();
        if (waterObjective != null && waterButtonHeld && wateringTimer <= 0)
        {
            wateringTimer = wateringCooldown;
            if (TakeWater(1))
            {
                waterObjective.AddWater(1);
            }
        }

        if (other.GetComponent<NaturalWater>() != null)
        {
            DisplayWater(true);
            canDisableWater = false;
        }
    }

    private void OnTriggerExit2D(Collider2D other)
    {
        if (other.GetComponent<NaturalWater>() != null)
        {
            DisplayWater(false);
            canDisableWater = true;
        }

        if (other.GetComponent<CanWater>() != null)
        {
            DisplayWater(false);
            canDisableWater = true;
        }
        else if (other.GetComponent<WaterObjective>() != null)
        {
            DisplayWater(false);
            canDisableWater = true;
        }
    }


    public void DisplayDropForTime()
    {
        displayWaterTimer = 0.5f;
        DisplayWater(true);
        
    }

    /// <summary>
    /// Takes a water cost, returns true if player had enough water and had the cooldown to water, returns false otherwise.
    /// </summary>
    /// <param name="waterCost"></param>
    /// <returns></returns>
    public bool TakeWater(int waterCost)
    {
        if (currentWater >= waterCost)
        {
            currentWater -= waterCost;
            return true;
        }
        else
        {
            return false;
        }
    }

    private void DisplayWater(bool displayWater)
    {
        waterOutLineImage.enabled = displayWater;
        playersWaterImage.enabled = displayWater;
    }

    public void ChangeWaterRefillRate(float rate)
    {
        waterRefillCooldown = rate;

        if (rate == 0)
        {
            waterRefillCooldown = baseWaterRefillCooldown;
        }
    }

    public float GetCurrentWater()
    {
        return currentWater;
    }
}

```

</details>