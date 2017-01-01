---
layout: post
title:  "Pairing Without A Matchmaking Service"
date:   2009-02-14 13:01:00 -0600
categories: examples
permalink: /examples/two-player-game.html
redirect_from: /book/examples/content/two-player-game/
---

In some circumstances, it is undesirable to require a matchmaking service to pair requests. Perhaps the frequency of requests is too low to justify the overhead of a service. Or perhaps we want to eliminate this single point of failure.

As an alternative to a matchmaking service, the clients can negotiate their own matches. They accomplish this through a conversation having three parts: request, bid, and accept.

One client initiates the conversation with a **Request**. It places this request in a rolling queue. Frames divide the queue into discrete time slices so that requests naturally expire. The model pivots at the relationship of the request with the queue, so that all clients observing the queue can see the request.

Other clients respond to the request by creating a *Bid*. This relationship again pivots the model so that the source of the request receives all bids.

The requestor selects one of the bids. It creates an *Accept* fact, simultaneously acknowledging the bid and closing the request. The relationship with the bid pivots so that the bidder receives the result. But the relationship of the accept with the request is not a pivot. All observers of the request must also be able to observe the accept.

**Rolling Queue**

No single service processes this game queue. Clients will come and go. If we opt for a simple queue, all clients will receive all requests, not just the ones that were recently added. Those requests would not age.

Instead, we split the queue into discrete frames over time. The desired timeout of the requests determines the duration of each frame. Requests will expire within one to two time frames. For example, a 15 minute time frame will expire requests in 15-30 minutes. 

![Matchmaking]({{ site.baseurl }}/images/matchmaking.jpg)

If clients are to communicate through a rolling queue, they must have an approximately consistent time source. Within a tolerance of a second or two, they must agree on the current time.

To find requests, each client queries the current time frame and the prior time frame. But to queue a new request, the client adds it only to the current time frame. This overlap ensures that requests queued toward the end of the time frame still have a full duration before expiration.

**The Algorithm**

A client wishing to join a game will first look for requests in the prior and current time frames. It obtains the current time from a time server, rounds it down to the beginning of the duration, and addresses that frame of the queue. Only requests that have not yet been accepted or canceled are considered. If a request is found in the current or prior frame, the client chooses one arbitrarily and places a bid on it.

The client observes the bid for an accept or reject message. If the bid is accepted, the game may begin. If the bid is rejected, or if a reasonable timeout has elapsed, then the client restarts the algorithm from the beginning.

On the other hand, if the client found no request in the current or prior frame of the queue, it adds a new request of its own. It will then observe this request for any bids. The client accepts the first bid that is received, but continues to observe the request. All further bids are rejected.

If the request expires having received no bids, the client restarts the algorithm from the beginning. Or if the client explicitly abandons the request, it issues a cancel fact.

By following this algorithm, several clients can pair up without the need of a matchmaking service. This algorithm is not as robust as a centralized service, but neither does it have the same overhead.