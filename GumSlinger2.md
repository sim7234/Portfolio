# Gumslinger 2 : [Back](https://github.com/sim7234/Portfolio/blob/main/README.md)

* Full time internship 2025 December - 2026 June
* Engine: Unity
* Genre: Mobile, 3D, Physics based duel game against bots with player stats.
* Released on January 20 2026 for Android and IOS.
* Worked with 10 other people.

## Link to [Itatake and Gumslinger2](https://itatake.com/portfolio/gumslinger-2-ducks-nukes/)

## Concepts i have worked with

Some concepts links to an example with code in it or further explaination, otherwise examples are further down.

* ### [Systems that support remote controlled balancing and rebalancing once live and timed (de)activation](https://github.com/sim7234/Portfolio/blob/main/Gs2Remote.md)

* ### [Refactored existing code with data that was already live and could not be modified](https://github.com/sim7234/Portfolio/blob/main/Gs2Refactor.md)

* ### Followed code structure, standards and practices of an existing project

* ### Optimized new features to work for mobile platforms

* ### Worked with async, tasks and cancellation tokens

* ### [Dynamic UI for any aspect ratio with localization support](https://github.com/sim7234/Portfolio/blob/main/Gs2UI.md)


## Code Structure

At Itatake i learnt to follow their best practises and naming conventions, things like always have namespaces, use camelCase for variables that are not static or properties etc. I also adapted my coding style to project specific standards such as utilizing ready made classes that gathers objects instead of using GetComponent or serialization.

Also learnt to use the Service pattern over the Singleton pattern.


## Optimization for mobile platforms

I learnt to use Unitys Addressables package to load and unload assets on demand, using this together with Atlases to only have neccesarry sprites loaded at any time.

Some minor coding practices such as caching property access like transform.position to avoid Unity having to convert the access to C++.

Use of lazy initialization so that nothing gets initialized unless its neccesarry.

The use of DateTime instead of timers to comnpare how long something has lasted to avoid unnecesary update loops.

## Async, tasks and cancellation tokens

GS2 required plenty of Async operations, at first i had barrely used async but by the end i was completly comfortable with it, here is a few simple things i learnt.

using functions with async Task to allow a function to be awaitable or run in the background with fire and forget.

Here is an example of me using TaskCompletionSource to navigate UI  

<details>
<summary>Async outcome task code</summary>

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
</details>

## What i have done

* Bug fixes, everything from some projectiles not making a sound to softbodies sometimes spawning under ground or getting moved around when they should be stationary, i have worked with a multitude of bug fixes

* I have made a large amount of features, most small some medium sized and a few large scale features.

* Code refactoring, i have refactored plenty of code to restore bugged features, make room for new requirements or just to make a part of the project be easier to work with