# Gumslinger 2 code refactor example : [Back](https://github.com/sim7234/Portfolio/blob/main/Gumslinger2.md)

One of the features that was created for this project was a event system that allows for remote creation of events with different rewards, requirements, durations and visual effects. This system was created by another colleague.

Using this system i setup a collectible event, but during this time both me and my collegue notices flaws with this system as each event was hardcoded due to being built from old existing hardcoded systems. He did not have time to refactor, so over the course of a few weeks i refactored the event system to be more abstract, easier to impliment and and work with.


This code used to be over 400 lines of code, but most of it is now moved into RemoteEventController and all event data is now abstracted, if a event needs to change any behaviour functions can be overwritten.


```CS

    public class CollectibleEventController : RemoteEventController
    {
        public CollectibleEventController()
        {
            EventStatisticsLocator.Service.SubscribeToEventProgress(EventTypes.Collectible, this);
        }

        private EventProgressData myData;
        internal override EventProgressData MyData
        {
            get
            {
                if (myData == null)
                {
                    myData = StorageLocator.Service.Progress.EventProgressManager.CollectibleData();
                }
                return myData;
            }
        }
        internal RemoteEventHubButtonController hubButtonController;
        internal override RemoteEventHubButtonController HubButtonController
        {
            get
            {
                if (hubButtonController == null)
                {
                    hubButtonController = new RemoteEventHubButtonController(this);
                }
                return hubButtonController;
            }
        }

        internal override void AddValidEvent()
        {
            Debug.Log("COLLECTBILE EVENT VALID");
            EventManager.AddValidEvent(this);
        }

        private List<EventRemoteData> GetTestData()
        {
            return new List<EventRemoteData>(){
                 new EventRemoteData(
                     "NameOfEvent",
                     new DateTime(2026, 05, 02, 05, 15, 0),
                     new DateTime(2026, 07, 25, 00, 25, 0),
                     3,
                     new string[]
                     {
                          "Reward1",
                          "Reward2",
                          "Reward3"
                     },
                     new int[]
                     {
                         3, 5, 10 //requirements for each tier, this data is setup on remote and is only here for testing.
                     },
                     "Tiered"

                     ),
           };
        }

        protected override List<EventRemoteData> GetFromRemoteService()
        {

#if !UNITY_EDITOR

            var remoteObject = DynamicContentLocator.Service.GetCollectibleEventRemoteObject();

            if (remoteObject == null) { return null; }

            var remoteDatas = remoteObject.CollectibleRemoteEventDatas?.ToList();

            return remoteDatas;
#else
            return GetTestData();
#endif
        }

        public override void EventProgressChanged(bool positiveValue = true, int amount = 0, Enum enumType = null)
        {
            MyData.eventProgress += amount;
        }
    }
    ```