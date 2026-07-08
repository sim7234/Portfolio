# Gumslinger 2 example of feature that works with remote : [Back](https://github.com/sim7234/Portfolio/blob/main/Gumslinger2.md)

Some of the features for GS2 required remote controll access, meaning the code i wrote should be able to handle variables that changes from a remote website and then update the feature as needed.

Here is an example of a feature i made that could be remote controlled.

## A medal score system

### Gumslinger 2 has a leaderboard system with weekly leaderboards, i was tasked with adding medals that you gain by competing in weekly leaderboards. 

<td ><img width="512" height="" src="Gumslinger2\gs2GainMedal.gif"/></td>

<td ><img width="512" height="" src="Gumslinger2\gs2ShowMedal.gif"/></td>

Parts of this system is dynamically controlled by a remote data base, here are what can be controlled:

* Score required for each medal
* Score gained by placing top X in weekly
* If medals are re-balanced the system will update to reflect any new medals gained and any requirements to get a new medal.

Any code comments are added for this portfolio and are not in the project.


```CS
private async Task LoadLeaderboardsAsync()
{
    //Here we get the current time and compare it to the last time we checked leaderboards, we dont want to send updates to often as it is a website call.
    var timeNow = WorldTime.GetCurrentTime();
    var progressManager = StorageLocator.Service.Progress.LeaderboardProgressManager;

    if ((timeNow - lastLeaderboardCheck).TotalSeconds < 150 && progressManager.GetBestGlobalPlacement() >= 0)
    {
        return;
    }

    lastLeaderboardCheck = timeNow;

    var localPlayerData = new LeaderboardData();
    var globalPlayerData = new LeaderboardData();

    localPlayerData = await UgsLeaderboards.GetLocalSplashShotPlayer();
    globalPlayerData = await UgsLeaderboards.GetSplashShotPlayer();

    progressManager.SetPlacements(globalPlayerData.rank + 1, false);
    progressManager.SetPlacements(localPlayerData.rank + 1, true);
    progressManager.CheckForNewMedals();
}

//then we check if the players current points is greater then any required points, this should only happen if medals has been updated by remote as its usually handled elsewhere in a simular way.

 public void CheckForNewMedals()
 {
     var points = MyProgress.MedalPoints;
     for (int i = MyProgress.CurrentMedal; i < PointsToMedalIndex.Length; i++)
     {
         if (points >= PointsToMedalIndex[Mathf.Clamp(i + 1, 0, PointsToMedalIndex.Length - 1)])
         {
             if (MyProgress.CurrentMedal >= PointsToMedalIndex.Length - 1)
             {
                 continue;
             }
             MyProgress.CurrentMedal++;
         }
     }
 }

 //where PointsToMedalIndex has all the required data already loaded from remote, and if remote could not be accessed default variables are used instead.
 private int[] PointsToMedalIndex
{
    get
    {
        if (_pointsToMedalIndex == null)
        {
            _pointsToMedalIndex = DynamicContentLocator.Service.LeaderboardProfileData.MedalRequirements;
        }
        return _pointsToMedalIndex;
    }
}
```