[Home](../README.md) > [Online](README.md)  
Related Pages : [Multiplayer](../Multiplayer/README.md)
# Online
## Online Service Interface
Whether you use an existing backend or you want to use your own custom backend, you should always implement your API using the [IOnlineServerInterface](https://dev.epicgames.com/documentation/en-us/unreal-engine/online-subsystem-in-unreal-engine).
| Interface        | Description                                                                                                                                                                                |
|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Achievements     | Listing all achievements in a game, unlocking Achievements, and checking your own unlocked Achievements and those of other users.                                                          |
| External UI      | Opening the built-in user interfaces for specific hardware platforms or online services. In some cases, services grant access to certain core features exclusively through this interface. |
| Friends          | Anything related to friends and friends lists, such as adding users to your friends list, blocking and unblocking users, and listing players you have recently met online.                 |
| Leaderboard      | Accessing online leaderboards, including registering your own scores (or times), and checking leaderboards for scores from your friends list or players around the world.                  |
| Online User      | Collecting metadata about a user.                                                                                                                                                          |
| Presence         | Setting the way that a user's online status will appear to other users, such as "Online", "Away", "Playing a game", and so on.                                                             |
| Purchase         | Making in-game purchases and reviewing the history of past purchases.                                                                                                                      |
| Session          | Creating, destroying, and managing online game sessions. Also includes searching for sessions and matchmaking systems.                                                                     |
| Store            | Retrieving the categories and specific offers available for in-game purchase.                                                                                                              |
| User Cloud       | Provides the interface for per user cloud file storage. You could want to store player inventory in a cloud file for instance.                                                             |
| Voice Chat (EOS) | Using Epic Online Services as a Voice Chat provider.                                                                                                                                       |

## Using Async Action to easily implement online services logic
By using AsyncActions, you make sure to separate all server logic from game logic, while still making it very easy to call these in gameplay code:

```c++
UAsyncAction_GiveItem* GiveItemAsync = UAsyncAction_GiveItem::GiveItem(PlayerController->GetPlayerState<ACustomPlayerState>(), ItemId);
GiveItemAsync->OnComplete.AddUObject(this, AYourClass::OnItemGiven);
GiveItemAsync->Activate(); // Or better, add to a queue.
```

Never call these from client executable. By having this code run on the gamemode, with WITH_SERVER_CODE preprocessors and error handling, you can make sure that these exposed online service async functions will never be misused. Whenever possible, you should also filter incoming and outgoing IP requests from within the backend infrastructure to only allow these calls from official dedicated servers. Use a request queue to make sure to limit the amount of active UAsyncAction.

## Epic Online Services (EOS)
EOS is a great free backend to develop on and publish with. You can follow the configurations writing in this [documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/online-subsystem-eos-plugin-in-unreal-engine) and use the Dev Auth Tool provided with the EOS SDK to automatically login to the backend.

You can use the online server interface like any other backend service and you can upload or read data directly from the portal where you configure your EOS project.

## CommonUser plugin
You can use the [CommonUserSubsystem](https://dev.epicgames.com/documentation/es-es/unreal-engine/common-user-plugin-in-unreal-engine-for-lyra-sample-game) to access the online service interface in a consistent manner across the project and it also exposes it to blueprint usages. Note that this does create multiple online subsystem based on the provided context.


© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).  