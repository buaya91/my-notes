## <span style="text-decoration: underline;">Recap</span>

In [part 1 of the series](/teamblogs/how-does-multiplayer-game-sync-their-state-part-1), we discussed the following:

*   Challenges faced in multiplayer game
*   How to solve unresponsive UI by client predictions

I did however gloss over essential server implementation details, which
we will focus on in this article.

**Disclaimer: I am not a professional game developer, most of
the knowledge shared is based on what I read and my experience of
small hobby projects. The main goal of this article is to provide
an easy to understand introduction for networking in a multiplayer 
game.**

## What's the role of server?

Let's start by defining what a server should do, typically a server should serve as

a) Connection point for players

In a multiplayer game, players need to access a common endpoint to reach
each other, this is one of the roles of a server program, even in the
P2P communication model, there will be a connection point for players
to exchange their network information before a P2P connection can be
established.

b) Processing unit

In many cases, the server runs the game simulation code, process all
inputs from player and updates the game state. Note that this is not
always the case, some modern games offload lots of processing to the
client side. In this article we will assume it's the server's
responsibility to process the game, ie. Make the game tick.

c) Single source of truth on game state

In many multiplayer games, the server program also has authority on 
game state, the main reason is to prevent cheating, and it's also
easier to reason about when there's is a single point to get the
correct game state.

## Naive Server Implementation

Let's start to implement the server in the most straightforward fashion,
then improve from there.

The core of the game server is a loop that keeps updating the GameState
using a player's input, commonly know as a TICK, the function signature
is as follows:

> <span style="color: #ff6600;">(STATE<sub>n</sub> , INPUT<sub>n</sub>) => STATE<sub>n+1</sub></span>

A simplified server code snippet would looks like this

### <span style="text-decoration: underline;">Discussion</span>

I hope the code snippets look intuitive and straightforward,
the server simply take all inputs from the buffer and
<span style="color: #000000;">applies them in the next `TICK` function</span>
to get new GameState. Let's call this approach `Greedy Game Loop`, as it
tries to process things as fast as it could. It is all good, until
we think about our lovely universe where sunlight takes 8 minutes to 
reach the earth. 

<span style="color: #ff0000; background-color: #ffff99;">Latency strikes again</span>

![](https://cdn2.hubspot.net/hubfs/323094/qingwei/speed_of_light_meme.jpg "speed_of_light_meme.jpg")

The fact that the server processes all input from buffer in every `TICK`,
means the GameState will depends on network latency. Diagram
below illustrates why this is a problem

![](https://github.com/buaya91/image-storage/blob/master/multiplayer-game-networking/server-side-latency-issue-single.png?raw=true)

The image shows 2 clients sending inputs to server, we observe 2
interesting facts.

1.  Requests took different time from different client to server, 
**1** unit of time from <span style="background-color: #fab9d9;">Client A</span> to <span style="background-color: #ccffcc;">Server</span>,  **1.5** unit of time from <span style="background-color: #ccffff;">Client B</span> to <span style="background-color: #ccffcc;">Server</span>
2.  Requests took different time from same client to server,
1st request took **1** unit of time, 2nd request took **2** unit of time.

In short, latency is inconsistent, even on the same connection.

Inconsistent latency combined with **`Greedy Game Loop`** gives several problems, let look at these further.

<table style="border-color: #000000; background-color: #ffffff;" border="3" cellpadding="5">
    <tbody>
        <tr>
            <td>
                <h4 style="padding-left: 30px;">Client Side Prediction will&nbsp;not work</h4>
            </td>
            <td>
                <p style="text-align: left;">
                    <span style="font-size: 13px;">
                    If&nbsp;we cannot predict when the&nbsp;server would
                    receive input (due to latency), we can't make any
                    predictions with high accuracy. (Forgot how Client
                    Side Prediction works? read&nbsp;
                    <a href="http://www.cakesolutions.net/teamblogs/how-does-multiplayer-game-sync-their-state-part-1#client-side-prediction">here</a>)
                    </span>
                </p>
            </td>
        </tr>
        <tr>
            <td>
                <h4 style="padding-left: 30px;">Low latency players&nbsp;get advantages</h4>
            </td>
            <td style="width: 642px;">
                <span style="font-size: 13px;">
                If input takes a shorter time to reach server, it will
                be processed sooner, creating unfair advantage for
                players with fast networks. eg. Two players
                shoot&nbsp;each other at the same time,
                they are supposed to kill each other at the same time,
                but Player B has a lower latency thus killed Player A
                before Player A's&nbsp;command is processed.
                </span>
            </td>
         </tr>
    </tbody>
</table>


There is a simple solution to mitigate inconsistent latency,
[Lockstep State Update](http://www.cakesolutions.net/teamblogs/how-does-multiplayer-game-sync-their-state-part-1#lockstep) that we discussed in the previous article. The idea is that server does not proceed until it received input from all players, it has 2 benefits:

*   it does not require client side prediction
*   all players will appear to have the same latency as the slowest
player, removing the advantage we mentioned

However, it does not work for fast-paced action games as the
responsiveness is low. (More details can be found on previous article,
thus I will not repeat here.)

Next section, we will talk about how to make the server side work for
fast paced games.

## <span style="text-decoration: underline;">Server Reconcilation</span>

To solve the problem of inaccurate client side predictions, 
we need to make the client-server interaction more predictable 
from the client point of view. When a Player presses a key on client
side, the client program needs to know when this input would being
processed on server side.

One possible way is to **let the client suggest when the input should
be applied**, this way, client side would be able to predict it
reliably. The term `suggest` is used as server might reject the
suggestion if it's invalid, for example trying to cast a magic when 
your magic power is empty.

The input should be applied shortly after user input, ie.
<span style="color: #993366;">**T<sub>input</sub> + X**</span>,
where X is the delay. The exact value depends on game, normally less
than 100ms to be responsive. Note X can also be zero, in this case
it should happen immediately after user provides input.

Let's say we choose X = 30ms, which translates into roughly 1 frame
for 30fps (frame per second), and it takes 150ms for input to travel
to server, there's a good chance when input reaches the server,
the target frame for input had passed.

![](https://github.com/buaya91/image-storage/blob/master/multiplayer-game-networking/server-side-independent-latency.png?raw=true)

Looking at the diagram, User A pressed a key at **T**, which supposed
to be processed at **T + 30ms**, but the input is received by server
at **T + 150ms**, due to latency, which already passed **T + 30ms**.
This is the problem we are going to solve in this section

> **How does server apply input that should happen in the past?**

### <span style="text-decoration: underline;">The Concept</span>

You might have recalled client side prediction has a similar issue of
incorrect predictions due to lacking information of opponents, and
the incorrect predictions will later be corrected by state updates
from server using **[Reconcilation](http://www.cakesolutions.net/teamblogs/how-does-multiplayer-game-sync-their-state-part-1#reconcilation)**.
The same technique can be used here, the only difference is that
we are correcting the GameState on server using user input from clients.

All user input needs to be tagged with a timestamp, this timestamp will then be used to tell the server when to process this input.

![](https://github.com/buaya91/image-storage/blob/master/multiplayer-game-networking/server-side-reconcilation1.png?raw=true)

<sub>Note: On the first dotted line, it's <span style="color: #000000;">**Time X**</span> on Client side, but <span style="color: #000000;">**Time Y**</span> on Server side, this is an interesting nature of multiplayer game (and many other distributed system), as client and server <g class="gr_ gr_201 gr-alert gr_gramm gr_inline_cards gr_run_anim Grammar multiReplace" id="201" data-gr-id="201">runs</g> independently, the time of client and server typically will be different,  our algorithm will handle the difference.</sub>

Diagram above shows interaction between 1 Client and the Server,

1.  Client sends an Input with timestamp telling server this input of
Client A should happen on <span style="color: #ff0000;">Time X</span>. 
2.  Server received the request on <span style="color: #008000;">Time Y</span>,
let's assume Time X is older than Time Y for the sake of discussion.
When developing our algorithm, we should not
assume <span style="color: #008000;">Time Y</span> is bigger or less
than <span style="color: #ff0000;">Time X<span style="color: #000000;">,
this will give us more flexibility.</span></span>
3.  <span style="color: #ff0000;"><span style="color: #000000;">
The <span style="color: #ff0000;">**RED BOX**</span> is where
reconcilation happens, the server needs to apply the Input X to the
latest game state so that it appears that input X happens
on <span style="color: #ff0000;">Time X</span>.
4.  GameState from server also includes timestamp, which is required 
for both server side and client side reconcilation.

#### Details of Reconcilation (the **<span style="color: #ff0000;">RED BOX</span>**)

1.  Server needs to maintain
    *   **<span style="color: #000000;">GameStateHistory        </span>**<span style="color: #000000;">- history of GameState within a time frame **P**, eg. all GameState since a second ago</span>
    *   **ProcessedUserInput ** - history of UserInput processed within a time frame **P**, ie. same value as time frame of GameStateHistory
    *   **UnprocessedUserInput** - UserInput received, but not processed yet, also within time frame **P**
        ![](https://github.com/buaya91/image-storage/blob/master/multiplayer-game-networking/server-buffer0.png?raw=true)
2.  When server received an input from user, it should be inserted into the **UnprocessedUserInput**.

    ![](https://github.com/buaya91/image-storage/blob/master/multiplayer-game-networking/server-buffer1.png?raw=true)

3.  Next, when server ticks, 
    1.  Check if there is any user input in the **UnprocessedUserInput** which is older than the current frame
    2.  If not, you are good, simply run the game logic with latest GameState and corresponding Inputs (if any), and broadcast to clients.
    3.  If yes, it means some of the game states generated previously are wrong due to missing information, we need to correct it
    4.  First we need to find the oldest unprocessed user input, let say it is on Time N, (Tips: this operation is fast if the **UnprocessedUserInput **is sorted).
    5.  Then we need to obtain the corresponding GameState on Time N from **GameStateHistory**, and the processed user input on Time N from **ProcessedUserInput**
    6.  Using these 3 pieces of data, we can create a new GameState which is more accurate.
        ![](https://github.com/buaya91/image-storage/blob/master/multiplayer-game-networking/server-buffer2.png?raw=true)
    7.  Then move the Unprocessed Input N to ProcessedUserInput, so that we can use it for reconcilation in the future.
    8.  Update the GameState N in **GameStateHistory**
    9.  Repeat step 4 to 7, for `N+1, N+2 ...`, until we get latest GameState.
    10. Server sends out the latest frame to all players. 

### <span style="text-decoration: underline;">Discussion</span>

Server side reconcilation suffers similar problems as client side
reconcilation, when we reconcile, it means we did something wrong,
and we are correcting by changing history. This means we cannot apply
irreversible outcomes, i.e, killing a players, such irreversible
outcomes will only be applied when it goes out of the
**GameStateHistory**, ie. when it cannot be rewriten anymore.

In addition, the incorrect GameState sometime causes awful UI jump.
Diagram below illustrate how it happens

![](https://github.com/buaya91/image-storage/blob/master/multiplayer-game-networking/server-error0.png?raw=true)

Entity starts at top left corner, it is moving toward
right hand side, 5 ticks later, it shifted towards right, but
then server received the user input saying that the entity changed
direction on Tick N, so the server reconciles the game state, and
now suddenly the entity **jumps** to the bottom left on the canvas.

I might be exaggerating the effect, sometimes entity does not move
that much, thus the jump would less obvious, but it is still noticeable
in many cases. We can control the jump by changing the size
of **GameStateHistory**, **UnprocessedUserInput** and **ProcessedUserInput**,
the smaller the buffer size, the less jump there would be, because
we would be less tolerant on input that arrives late, eg. If Input
that is late for more than 100ms is ignored, player with ping > 200ms
wont be able to play the game.

> We can trade **network latency tolerance** for more **accurate game state update**,
or vice versa.

One popular technique to overcome the problem of inaccurate Game State 
is **Entity Interpolation**, the idea is to smoothen the jump by
spreading it out within a short amount of time.

![](https://github.com/buaya91/image-storage/blob/master/multiplayer-game-networking/server-error1.png?raw=true)

I will not include implementation details of **Entity Interpolation**
in this article, however some references will be provided at the bottom of article.

## <span style="text-decoration: underline;">Wrapping Up</span>

We have talked about how both client and server might work in a multiplayer game.

![](https://github.com/buaya91/image-storage/blob/master/multiplayer-game-networking/wrapup.png?raw=true)In general, a multiplayer game has 3 loosely coupled loops, **Server Game Loop**, **<g class="gr_ gr_149 gr-alert gr_spell gr_inline_cards gr_run_anim ContextualSpelling ins-del multiReplace" id="149" data-gr-id="149">Client side</g> prediction Loop**, and **<g class="gr_ gr_150 gr-alert gr_spell gr_inline_cards gr_run_anim ContextualSpelling ins-del multiReplace" id="150" data-gr-id="150">Client side</g> UI Rendering Loop**. By having some sort of buffer in between them, their execution can be decoupled, giving us <g class="gr_ gr_205 gr-alert gr_gramm gr_inline_cards gr_run_anim Grammar multiReplace" id="205" data-gr-id="205">flexibility</g> to create better gaming experience.

## <span style="text-decoration: underline;">Conclusion</span>

Here ends my article on Multiplayer games, I learned much of this
knowledge from experts in this field, building a 
[simple multiplayer game](https://github.com/buaya91/scalajs-snake) also helps a lot.
I've only show one way to implement a multiplayer server,
there are more other ways, depending on what kind of game you're
building, I encourage you to explore some of those ideas by building a simple game.

Thanks for reading, happy hacking !

## <span style="text-decoration: underline;">**References and Further Reading**</span>

*   [Entity Interpolation] - [http://www.gabrielgambetta.com/fpm3.html](http://www.gabrielgambetta.com/fpm3.html)
*   [<span>[Entity Interpolation] - </span>http://gafferongames.com/networked-physics/snapshots-and-interpolation/](http://gafferongames.com/networked-physics/snapshots-and-interpolation/)
*   [Lag Compensation] - [https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking#Lag_compensation](https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking#Lag_compensation)
