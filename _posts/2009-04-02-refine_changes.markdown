---
layout: post
title:  "Step Two: Refine Changes"
date:   2009-04-02 13:01:00 -0600
categories: examples
permalink: /examples/giftcards/refine-changes.html
redirect_from: /book/examples/giftcards/refinechanges/
---

You've identified the changes that a user can make. Now these changes need to be refined. This is an iterative process in which we group changes, pull out entities, and document prerequisites.

The first refinement to the gift-card changes is to group the Purchase, AddValue, and Redeem changes. These three changes all have the same parameters. More importantly, these three changes are all specific types of the same thing: Transactions. Let's pull that out.

	-Transaction(Card card, Store location, money increaseInCardValue) :
		-Purchase()
	
		-AddValue()
	
		-Redeem()

	
	-VoidTransaction(Transaction voidedTransaction)

Purchase, AddValue, and Redeem all inherit the parameters of Transaction. The money parameter has been changed to an increase in card value, so Redeem would expect a negative number. ServiceFee did not make the list because it is not applied within the context of a store.

Next, I looked all all the entities that appeared in the parameters. Each entity needs to be created. Creating an entity is itself a change. A change that creates an entity should have the same name as the entity it creates, just like a constructor. So I renamed "NewStore" to just "Store", and added changes for the other entities:

	-Vendor(string vendorName)

	-Store(Vendor vendor, string storeName)

	-Card(Vendor vendor, string cardNumber)

Finally, I looked for prerequisites. A prerequisite is one change that has to occur before another change. The prerequisite becomes a parameter to its successor.

We already have some good examples of prerequisites. For example, a Card must be created before a ServiceFee is applied, and a Store must be opened before it is closed. Since these are parameters, they are prerequisites

	-ServiceFee(Card card, money fee)

	-CloseStore(Store closedStore)

Examining the business requirements, I found one prerequisite that was not captured in the model. A card has to be purchased before a customer can either add value or redeem, or before the vendor can levy a service fee. So I moved Purchase back out of Transaction and made it a prerequisite:

	-Purchase(Card card, Store locationOfPurchase, money purchaseAmount)

	-Transaction(Purchase initialPurchase, Store location, money increaseInCardValue) :

		-AddValue()
	
		-Redeem()
	
	-VoidTransaction(Transaction voidedTransaction)

	-VoidPurchase(Purchase voidedPurchase)

	-ServiceFee(Purchase initialPurchase, money fee)

Transaction no longer takes Card as a parameter, since the Card is implied by the Purchase.

After these refinements, we have the following changes:

	-Vendor(string vendorName)

	-Store(Vendor vendor, string storeName)

	-Card(Vendor vendor, string cardNumber)

	-Purchase(Card card, Store locationOfPurchase, money purchaseAmount)

	-Transaction(Purchase initialPurchase, Store location, money increaseInCardValue) :

		-AddValue()
	
		-Redeem()
	
	-VoidTransaction(Transaction voidedTransaction)

	-VoidPurchase(Purchase voidedPurchase)

	-ServiceFee(Purchase initialPurchase, money fee)

	-CloseStore(Store closedStore)

The graph of these changes is becoming more organized. You can see as you flow down the graph the order in which changes have to take place.

![Refine Changes]({{ site.baseurl }}/images/refinechanges.jpg)

We've defined the changes that the user can make, and the relationships among those changes. But we are not quite ready to call this a historical model. Next we need to query it.

