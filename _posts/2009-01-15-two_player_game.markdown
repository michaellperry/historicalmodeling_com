---
layout: post
title:  "Two Player Game"
date:   2009-01-15 13:01:00 -0600
categories: examples
permalink: /examples/two-player-game.html
redirect_from: /book/examples/content/two-player-game/
---

People on different machines can use a historical model to communicate. Their models are synchronized so that a fact created by one person becomes visible to the other.

A game between two people can be modeled as a fact graph. At the top, a Person fact is created for each unique person. A GameQueue fact represents an on-line lobby. Each of the people submit GameRequest facts to the GameQueue.

![Two Player Game]({{ site.baseurl }}/images/twoplayergame.jpg)

A service monitors the GameQueue looking for outstanding GameRequests. This is a typical example of the Service pattern. The code for the GameQueue fact shows how the service finds outstanding requests:

```
fact GameQueue {
   string identifier;

   GameRequest* outstandingRequests {
      GameRequest r : r.gameQueue = this
         where r.isOutstanding
   }
}

fact GameRequest {
   Person person;
   GameQueue gameQueue;

   bool isOutstanding {
      not exists Game g : g.gameRequest = this
   }
}
```

The service creates a Game for each pair of GameRequests. The creation of that game causes both GameRequests to be removed from the queue: when a game exists, the request is no longer outstanding and hence does not match the query.

Once a Game has been created, each person creates a Player fact. This fact records the relationship between a Person and a Game. They then play the game by making Moves.

**Pivots**

At several points in this graph, a fact is created by one machine in response to a different fact created by another. The role that relates two facts from different sources is called a “pivot”. These roles are indicated in red.

The service creates the GameQueue. Each player creates a GameRequest. The role between the game request and the predecessor game queue is a pivot. It is the point in the model at which the player’s client communicates with the server.

Similarly, in response to the GameRequest, the server creates a Game. Since the game is a response to a game request and each fact is created on a different machine, this relationship is a pivot.

Finally, a Player is created in response to a Game. Whereas the Game comes from the server, the Player comes from the client. This pivot point sets up the communication between the two players.

All subsequent Moves are created as successors to the Player, both of which originate from the same client machine. This role is not a pivot. Each of the two clients is using their corresponding Player-Game pivot to communicate moves with the other client.