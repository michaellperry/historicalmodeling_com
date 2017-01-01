---
layout: post
title:  "Step One: Identify Changes"
date:   2009-04-01 13:01:00 -0600
categories: examples
permalink: /examples/giftcards/indentifiy-changes.html
redirect_from: /book/examples/giftcards/identifiychanges/
---

In a state-based model, like an object-oriented or a relational model, the first step is to identify entities. In a historical model, however, you identify changes. Changes are the operations that a user performs on your system that modify it in some way.

On a previous project, I helped to create a gift card system. These are the changes that we identified:

- A customer purchases a card
- A customer adds value to a card
- A customer redeems a card for merchandise
- A customer voids a purchase
- A customer voids an add value
- A customer voids a redemption
- A vendor applies a service fee
- A vendor opens a new store
- A vendor closes a store

There are other operations that are not changes, such as checking a balance or running a report. We'll get to these later, but for now we are just interested in the changes.

Assign a name to each change, and give it parameters. Parameters can be simple types, but they can also be entities. Parameters can even be other changes. These are the names and parameters assigned to the gift card changes.

- Purchase(Card card, Store locationOfPurchase, money purchaseAmount)
- AddValue(Card card, Store locationOfAddValue, money addedAmount)
- Redeem(Card card, Store locationOfRedemption, money redeemedAmount)
- VoidPurchase(Purchase voidedPurchase)
- VoidAddValue(AddValue voidedAddValue)
- VoidRedemption(Redeem voidedRedemption)
- ServiceFee(Card card, money fee)
- NewStore(Vendor vendor, string storeName)
- CloseStore(Store closedStore)

Visually, we can display each change as a bubble. The arrows indicate that a change takes another as a parameter:

![Identify Changes]({{ site.baseurl }}/images/identifychanges.jpg)

This is a good start, but it is not yet a historical model. The changes need to be refined.

