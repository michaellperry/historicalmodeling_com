---
layout: post
title:  "Uniqueness"
date:   2009-04-07 13:01:00 -0600
categories: examples
permalink: /examples/giftcards/uniqueness.html
redirect_from: /book/content/uniqueness/
---

Historical Modeling is all about the changes made to a system, and only to a lesser extent about the objects in the system. It therefore has a different approach to identity.

In Object-Oriented Modeling and Design, James Rumbaugh defined identity as uniqueness independent of any identifying attribute. Historical Modeling, on the other hand, defines the uniqueness of a change by its type and its fields. A change is a historical fact. Repeating that fact doesn't make it a different fact.

In the gift card system, the Vendor fact type is defined as:

```
fact Vendor {
  string vendorName;
}
```

I could tell you that there exists a vendor called Starbucks by writing the following fact:

```
Vendor("Starbucks")
```

If I were to repeat that fact, it would not create a new vendor. It would just invoke the vendor that was already described.

A Card is defined by this fact:

```
fact Card {
  Vendor vendor;
  string cardNumber;
}
```

So I could describe a card as:

```
Card( Vendor("Starbucks"), "4646828279791313" )
```

That description is a statement of fact. It states that such a card exists. Every change in a historical model is a statement of fact. It is unique only in its type and fields. It is therefore impossible to change those fields. Every change -- or fact -- is immutable.

Sometimes a fact needs a little more information to make it unique. Let's look at a Purchase.

```
fact Purchase {
  Card card;
  StoreDateOfBusiness storeDateOfBusiness;
  money purchaseAmount;
}
```

I could describe a purchase like this:

```
Purchase(
  Card( Vendor("Starbucks"), "4646828279791313" ),
  StoreDateOfBusiness(
    Store( Vendor("Starbucks"), "23-305" ),
    DateOfBusiness( 2009-04-11 ) ),
  10.00 )
```

Even though I've invoked the vendor twice in that statement, it's the same Vendor.

But the important thing to realize is that there can be no other Purchase with exactly those fields. This could cause a problem. If I made a mistake and voided that purchase, I could not try it again. We want a second purchase for the same card at the same store on the same date for the same amount. The way Purchase is modeled, that second purchase is indistinguishable from the first.

To correct this, we need to add a field that makes the Purchase unique. There are many ways to do this: timestamp, random number, and GUID are three that work well. Rather than choosing one technique, I'll just call the new field "unique".

```
fact Purchase {
  unique;
  Card card;
  StoreDateOfBusiness storeDateOfBusiness;
  money purchaseAmount;
}
```

Going through the model, there are a few more facts that need the "unique" field. Here's the complete list:

- Purchase
- Transaction
- ServiceFee
	
Transfer does not need "unique" because it has sets of Transactions and Purchases as fields. These provide a strong business uniqueness, so no artificial uniqueness is required. If we added a business rule that only one ServiceFee is applied to a Purchase within a DateOfBusiness, then we could remove the "unique" field from ServiceFee, too.

In historical modeling, changes to the system are historical facts. They are unique only based on their type and fields. If you need two identical facts to be unique, look for ways within the business domain to make them unique. If you can't find something natural, add the "unique" field.

