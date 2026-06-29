# Gumslinger 2 features that work with remote data : [Back](https://github.com/sim7234/Portfolio/blob/main/Gumslinger2.md)

Here is an example of a feature i made that could be remote controlled.

## A medal gain system

### Gumslinger 2 has a leaderboard system with weekly leaderboards, i was tasked with adding medals that you gain by competing in weekly leaderboards. 

<td ><img width="512" height="" src="Gumslinger2\gs2GainMedal.gif"/></td>

<td ><img width="512" height="" src="Gumslinger2\gs2ShowMedal.gif"/></td>

Parts of this system is dynamically controlled by a remote data base, here are what can be controlled:

* Score required for each medal
* Score gained by placing top X in weekly
* If medals are re-balanced the system will update to reflect any new medals gained and any requirements to get a new medal.

Any code comments are added for this portfolio and are not in the project.

```CS
public override void OnParentInteractionChanged(bool isEnabled)
{
    //this function is called when entering or exiting the weekly leaderboards tab.
    GameObjectUtils.SafeActivate(xpBar, isEnabled);
    GameObjectUtils.SafeActivate(scrollRect, isEnabled);
    GameObjectUtils.SafeActivate(xpText, isEnabled);
    GameObjectUtils.SafeActivate(medalPointsInfoButton, isEnabled);
    ToggleInteractions(isEnabled);

    if (isEnabled)
    {
        //Here we get existing data for medals and update UI
        MedalAssetsIndexes.Clear();
        var medalArray = manager.GetMedalArray();
        if (firstInteraction)
        {
            firstInteraction = false;

            //here we get what medal index you are at from all medals
            //we also spawn in prefabs of all the medals and one separate prefab for the highlighted medal (the one you are currently at index wise).
            var currentMedalIndex = Mathf.Clamp(manager.GetCurrentMedalIndex(), 0, manager.GetTotalMedalAmount());
            var contentTransform = Bindings.Get<Transform>(UiB.Content);
            var highlightRect = new RectTransform();

            var hasSpawnedHighlight = false;
            var highlightIndex = 0;

            var totalMedals = manager.GetTotalMedalAmount();

            for (int i = 1; i < totalMedals; i++)
            {
                //medalObjects is a list we use to cleanup any gameObjects when we unload later, this is a safety check so we don't spawn objects twice.
                if (medalObjects.Count > i) break;

                GameObject medal;
                //check if we are on the last medal or highlighted one and spawn the highlighted prefab, else we spawn normal prefab. 
                if (i == currentMedalIndex || (i == totalMedals - 1 && !hasSpawnedHighlight && currentMedalIndex != 0))
                {
                    hasSpawnedHighlight = true;
                    highlightIndex = i;
                    medal = Instantiate(highlightedMedalPrefab, contentTransform);
                    highlightRect = medal.GetComponent<RectTransform>();
                }
                else
                {
                    medal = Instantiate(medalPrefab, contentTransform);
                }


                //Update text for each medal to reflect their point requirement
                var outlinedText = medal.GetComponentInChildren<OutlinedTextField>();
                outlinedText.text = medalArray[i].ToString();

                //Set a lower alpha and darker color to make a medal show its not unlocked yet.
                if ((hasSpawnedHighlight && i != highlightIndex && highlightIndex != totalMedals - 1) || currentMedalIndex == 0)
                {
                    var image = medal.GetComponent<Image>();
                    image.color = lockedMedalColor;
                    outlinedText.SetColor(lockedTextColorBackground, 0);
                    outlinedText.SetColor(lockedTextColor, 1);
                }
                //add gameObject for cleanup later.
                medalObjects.Add(medal);
            }

                //load images for each medal with addressables plugin
                //and then scroll the horizontal scroll bar to the highlighted medal's position.
                //Code for this function further down.
            _ = LoadMedalsAndScrollToTarget(highlightRect, highlightIndex, currentMedalIndex > 0);
        }

        var target = StorageLocator.Service.Progress.LeaderboardProgressManager.GetPercentageTillNextMedal();
        var medalPoints = manager.GetMedalPoints();
        var medalPercentage = (float)medalPoints / (float)(manager.GetPointsTillNextMedal() + medalPoints);


        var medalTarget = Mathf.Clamp(manager.GetCurrentMedalIndex() + 1, 0, medalArray.Length);
        if (manager.GetCurrentMedalIndex() >= medalArray.Length - 1)
        {
            medalPercentage = 1;
        }

        //fill the XP bar to t he percentage of the next medal
        DOVirtual.Float(0, medalPercentage, 3, value =>
        {
            currentXpImage.fillAmount = value;
        });

        var pointsTillNextMedal = medalArray[Mathf.Clamp(manager.GetCurrentMedalIndex() + 1, 0, medalArray.Length - 1)];

        var currentPointsToMedal = manager.GetMedalPoints();

        //Display your points and required point for the next medal
        DOVirtual.Int(0, currentPointsToMedal, 3, value =>
        {
            xpText.text = $"{value} / {pointsTillNextMedal}";
        });
    }
}
```

```CS
private async Task LoadMedalsAndScrollToTarget(RectTransform target, int highlightedIndex, bool scroll)
{
    //Load all the images with a addressable library and wait for all of them to load
    var content = scrollRect.content;

    var medalArray = manager.GetMedalArray();
    var loadTasks = new Task[medalArray.Length - 1];
    for (int i = 0; i < medalArray.Length - 1; i++)
    {
        loadTasks[i] = LoadMedalSpriteAsync(medalObjects[i].GetComponent<Image>(), i + 1);
        MedalAssetsIndexes.Add(i + 1);
    }

    await Task.WhenAll(loadTasks);

    if (!scroll)
    {
        return;
    }

    LayoutRebuilder.ForceRebuildLayoutImmediate(content);

    Canvas.ForceUpdateCanvases();

    var viewport = scrollRect.viewport;

    //Here we get the world position center of the highlighted medal,
    //we then get the center of content which is where all the medals are stored
    //and then we move the position of contents anchored position to the medals center over time.

    var targetWorldCenter = target.TransformPoint(target.rect.center);
    var targetCenterInContent = content.InverseTransformPoint(targetWorldCenter);
    var viewportWorldCenter = viewport.TransformPoint(viewport.rect.center);
    var viewportCenterInContent = content.InverseTransformPoint(viewportWorldCenter);
    var deltaX = targetCenterInContent.x - viewportCenterInContent.x;
    var targetX = content.anchoredPosition.x - deltaX;
    var minX = -(content.rect.width - viewport.rect.width);
    var maxX = 0;

    targetX = Mathf.Clamp(targetX, minX, maxX);

    DOVirtual.Float(
        content.anchoredPosition.x,
        targetX,
        1f,
        value =>
        {
            content.anchoredPosition = new Vector2(
                value,
                content.anchoredPosition.y
            );
        }
    );
}
```

```CS
private async Task AwaitOverlayOutomeAsync()
{
    while (!token.IsCancellationRequested)
    {
        var source = outcomeSource = new TaskCompletionSource<LeaderboardProfileOutcome>();
        var outcome = await source.Task;
        switch (outcome)
        {
            case LeaderboardProfileOutcome.None:
            case LeaderboardProfileOutcome.Canceled:
                weeklyWidget.ReleaseAssets();
                break;
            case LeaderboardProfileOutcome.Back:
                ViewLocator.Service.Hide<ILeaderboardProfileView>();
                ViewLocator.Service?.Show<IProfileView>();
                weeklyWidget.ReleaseAssets();
                break;
            case LeaderboardProfileOutcome.Global:
                SetActiveWidget(globalWidget);
                SetHightlightedButton(outcome);
                continue;
            case LeaderboardProfileOutcome.Local:
                SetActiveWidget(localWidget);
                SetHightlightedButton(outcome);
                continue;
            case LeaderboardProfileOutcome.Weekly:
                SetActiveWidget(weeklyWidget);
                SetHightlightedButton(outcome);
                continue;
        }
        break;
    }

```
