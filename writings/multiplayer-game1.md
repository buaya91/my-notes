## The problem in multiplayer game
In multiplayer game, one of the most complex issue is to keep all player's state in sync with server state.
There are a few good articles around this topic on the internet. However, some details are missing here and there, which may be confusing for beginners in field of game programming, I hope I can clear things up in this article.

I'll present a few techniques commonly used in this problem space.

**Before** we jump into the problem, let's have an overview on how multiplayer game generally works.

Typically, a game program needs to simulate

> the changes in an environment with respect of time and players input

Game is stateful program, it depends on time (be it real or logical time), for example, PACMAN is simulating an environment that ghosts will keep moving.

A multiplayer game is no exception, just that the complexity is higher due to interaction between multipler players.

Let's use the classic Snake Game as an example, assume we use a server-client settings. The core game logic works like this

1. Read user inputs which can be one of [←, ↑, →, ↓], to change the direction of snake.
2. Apply user input if any, which change the direction of snake.
3. Move snake by 1 unit space
4. Check if any snakes bump into enemy/wall/self, remove them from the game.
5. Repeat

This logic will be run at a fixed interval on server side, as below, each loop is a called a `frame` or a `tick`

```scala

def main(): Unit = {
   while (true) {
      /**
      * 1. Read user inputs which can be one of [←, ↑, →, ↓], to change the direction of snake.
      * 2. Apply user input if any, which change the direction of snake.
      * 3. Move snake by 1 unit space
      * 4. Check if any snakes bump into enemy/wall/self, remove them from the game.
      * 5. Broadcast the new game state to all clients
      */
      Thread.sleep(100)
   }
}
```

And the simplest client would just listen to server update and render every frame received to players.

```scala

class Client {
   def onServerUpdate(state: GameState) = {
      renderGameState(state)
   }
}
```

**The problem**:

How do we sync the game world between server and different players in real time?

## Lockstep State update

### Concept
To make sure all client are in sync, the simplest way is to let client send update to server in a fixed interval,
let's say every 30ms. The update would contains user input, it can also represent `no user input`.

Once server gather input from **all user**, it can then proceed with next tick using those inputs.

![img](https://raw.githubusercontent.com/buaya91/image-storage/master/multiplayer-game-networking/lockstep-single.png)

The image above show how one client interact with server, I hope the problem is obivous, client will stay idle from **T0** to **T1**, waiting for server update to proceed. The latency can range from 50ms to 500ms, depending on network quality, and human will notice any delay over 100ms, so freezing the user interface for 200ms can be big problem for some games. This is not the only issue with lockstep approach.

![img](https://raw.githubusercontent.com/buaya91/image-storage/master/multiplayer-game-networking/lockstep-multi.png)
This image is slightly more complicated, showing multi-client interaction with server, you can see that client B have a slower network connection, thus although both A and B send input to server at **T0**, update from B reach server at **T2** instead of **T1**, server only proceed once it receive all updates which is **T2**.

This means

> the latency of the game is now the latency of the most lagged player

we are punishing all players because one of them is lagging, eventually all players will leave your game ....

Not to say there's a possiblity that client B might be disconnected, thus will block the server until connection timeout.

### Discussion
There are some problems including 2 of which we just mentioned :

1. Client will not be responding until receive state update from server, horrible user experience.
2. Game responsiveness depends on the most lagged players, playing with a friend with DSL connection? Have Fun
3. The connection would be really chatty, clients need to send some useless heartbeat data regularly so that server can confirm it have got all information needed to step forward, which is not efficient.

First of all, certain kind of games are immune to these problems, most `Turn-based` game actually use some variant of
such model, as client are supposed to wait. For slow-paced game, small amount of delay is acceptable too, for example Farm Ville.

A great example is **Chess**, where 2 players take their own turn, assuming each turn takes 10 secs

1. user are expect to wait each other for 10 secs, they can wait
2. 2 players take turn alternatively, so lagged player does not affect other player
3. each turn takes on average 5 secs, 1 request every 5 secs is fine

But for fast-paced game, like all FPS, all of these problems make lockstep approach not suitable for them.
We will see how we can solve these problem in the rest of articles.

## Client Predictions
Let's first solve the problem of user-responsiveness, game response after 500 millis after user press a key destroy
the gaming experience. How to solve this problem?

### Concept
Some of you might have already have the answer, instead of waiting on server update, client can actually emulate the game by running game logic locally (ie. on the client's machine).

Let's assume to produce game state at `Tn`, we need state at `Tn-1` and all user input at `Tn-1`.

![img](https://raw.githubusercontent.com/buaya91/image-storage/master/multiplayer-game-networking/clientprediction-single.png)

The idea is simple, let's have a fixed update rate, which is `1 unit of time` in this example, client send input to server at **T0**, and emulate the game state at **T1**, so client can then render the game without having to wait the state update from server, which only arrive at **T3**.

This approach only works if

> 1. Game state update logic is deterministic, ie. no randomness, or in some way, referentially transparent, so that server and client produce the same game state given the same input.
> 2. Client have all information required to run game logic

1 is not always true, but we can try to make it as similar as possible, and ignore the small differences, ie. floating points computation of different platform, use the same seed for pseudo-random algorithm.

2 is also not true, for example

![img](https://raw.githubusercontent.com/buaya91/image-storage/master/multiplayer-game-networking/clientprediction-multi.png)

In the image above, Client A still try to emulate game state at **T1** using the information it has from **T0**, but Client B also submitted input at **T0**, which Client A is not aware of, this means Client A's prediction of **T1** will be wrong. Luckily, since Client A still receive state of **T1** from server, at **T3**, client have the chance to correct it's mistake.

> Client side need to figure out if the previous emulation is correct, and how to resolve the conflicts.

The resolution of conflicts is normally called **Reconcilation**.

Implementation of **Reconcilation** varies depending on use case, I'll show a simple one, which we just throw away our prediction and replace it with the correct state from server.

1. Client need to maintain 2 buffers, one for predictions, one for user input, which can be used to compute predictions.
   Remember, **State Tn** is computed using **State Tn-1** and **Input Tn-1**, it will be empty at first.
![img](https://raw.githubusercontent.com/buaya91/image-storage/master/multiplayer-game-networking/clientprediction-initbuffer.png)

2. When player press an arrow key, the input in stored in InputBuffer, and client will also produce predictions which is then used to render the view, the prediction is stored in PredictionBuffer.
<div>
<img src="https://raw.githubusercontent.com/buaya91/image-storage/master/multiplayer-game-networking/clientprediction-buffer1.png"/>
</div>

![img](https://raw.githubusercontent.com/buaya91/image-storage/master/multiplayer-game-networking/clientprediction-buffer2.png)

3. When server State0 is received, and does not match with the client Prediction0, we can replace Prediction0 with State0, and recompute Prediction1 using Input0 and State0.
<div>
<img src="https://raw.githubusercontent.com/buaya91/image-storage/master/multiplayer-game-networking/clientprediction-reconciledbuffer.png" style="display:block;" />
</div>

4. After reconcilation, we can safely removed State0 and Input0 from the buffer, as we can confirm we got it correct.
![img](https://raw.githubusercontent.com/buaya91/image-storage/master/multiplayer-game-networking/clientprediction-droppedbuffer.png)

Note: this reconcilaton comes with a drawback, there might be glitches on view if server state and client prediction differ too much, for example if we predict enemy is moving south on **T0**, but at **T3** we realize it move towards north, and reconcile by simply using state from server, the enemy will bounce from towards north to reflect it's correct position. There are ways to handle this problem, but it will not be in this article.

### Discussion
Client prediction technique gives us a big benefit

> Client run on it's own update rate, independent to server update rate, so that if server is having hiccups, it does not affect client side frame rate.

It inevitably comes with some complexity :

1. We need to handle more state and logic on client side, (Prediction buffer, state buffer, prediction logic).
2. We need to decide how to handle conflict between prediction and real state from server.

And it still leave us with some problem :
1. View glitches due to wrong predictions
2. Chatty connection


## Conclusion
In this article, we went through 2 ways of approaching multiplayer game networking :

1. Lockstep state update
2. Client prediction

Each comes with it's own set of trade off, so far we havent get a closer look on the server side, which will be covered in next article some time later.

Thanks for reading !
