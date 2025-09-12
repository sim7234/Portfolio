# Sun Seed : [Back](https://github.com/sim7234/Portfolio/blob/main/README.md) <-- Click to go back.

* School group project, 2024 November - 2025 January
* Engine: Unity
* Genre: 2D, Asymmetric, Local Co-op, Pve
* 3 programmers and 4 artists

Download: https://yrgo-game-creator.itch.io/sun-seed

<table>
  <tr>
    <td ><img width="512" height="
" src="SunSeed\SunSeedHub.png"/></td>
  </tr>
</table>

## Game Description

Sun Seed Vegan Knights is a game played on controllers with a friend. The goal of the game is to go through a level and slay monsters by planting, and watering seeds that grow into powerful weapons.

## What i worked on

I mainly worked on the enemies, their pathfinding using unitys navmesh systems and their attack patterns, i made a total of 4 enemies.

* A melee enemy that walks up to the player, makes an indicator towards them then slams down.
* A ranged enemy that looks towards players and shoots a small fireball towards them.
* A dash enemy that slowly walks towards the player, then makes a long indicator and a few seconds later dashes and damaging players in the indicated area.
* A explosive enemy, just runs up to the player and explodes but has little health.


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

    private int randomTargetIndex;

    [SerializeField] bool targetPlayerPoints;
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

I also created a water display and gathering system that lets the player walk over water sources and slowly fill their water which gets displayed by UI.

I also helped in reworking scripts related to seed planting and growth