---
title: Don't Click Submit Twice!!
layout: page
permalink: /DontClickSubmitTwice/
order: 0
---

<button type="button" onclick="location.href='https://github.com/michaellperry/Mathematicians';">Download Sample</button>

Have you ever used a site that warned you that clicking Submit twice would double-charge your credit card?

Have you ever written one?

If you've disabled the submit button to prevent this problem, you're not solving the complete problem. Messages on the Internet are not guaranteed. The client needs the ability to retry the request. Subsequent attempts should not duplicate the results.

This is a feature known as Idempotency. Here's how to make an idempotent Web API.

## Generate a unique request ID

The trick is to get a unique ID for the request *before* you attempt it. Most APIs generate unique identifiers during the insert. Maybe it's a database-generated auto-incrementing ID. That's too late. You don't have an opportunity to detect duplicates at that point.

When the user first brings up the form, generate a unique request ID. This request might never been used. But if it is, we can ensure that it is only used once. A good choice for this is a GUID, but a timestamp will usually work fine as well. You want to stay away from a database-generated ID for this purpose.

If you are writing a web app with a standard HTML `FORM` element, then generate the GUID on the server and send it down as a hidden `INPUT`. But if you are creating a Web API for a rich client application, then provide an additional endpoint for getting a new request ID.

For example, this API adds mathematicians to a database. The client starts with a call to `GET api/mathematicians/new`, which returns the following JSON payload:

```json
{
    "unique": "4e5f8dff-bdd8-48d9-9c10-4eab38d0fab3",
    "name": {
        "ids": [],
        "firstName": null,
        "lastName": null
    },
    "_links": {
        "self": {
            "href": "http://localhost:4409/api/mathematicians/4e5f8dff-bdd8-48d9-9c10-4eab38d0fab3"
        }
    }
}
```

The server generates this GUID and simply returns the object. It doesn't store anything. So if you were to `GET api/mathematicians/4e5f8dff-bdd8-48d9-9c10-4eab38d0fab3`, it would result in a `404 Not Found`. This resource hasn't been created, but the request GUID is ready for you.

## Post the request with the unique ID

The rich client application fills out the object and sends a `POST api/mathematicians`. The body of the object carries with it the unique request ID generated earlier.

```json
{
    "unique": "4e5f8dff-bdd8-48d9-9c10-4eab38d0fab3",
    "name": {
        "ids": [],
        "firstName": "Leonhardt",
        "lastName": "Euler"
    },
    "_links": {
        "self": {
            "href": "http://localhost:4409/api/mathematicians/4e5f8dff-bdd8-48d9-9c10-4eab38d0fab3"
        }
    }
}
```

If something goes wrong, the client can always `POST` the object again. It will have the same unique request ID as the first time.

What you do **not** want to do is to `GET` a new request ID before each `POST`. That would defeat the purpose.

## Check the request ID

The unique request ID should become the external ID of the object that you are creating. In the example above, it is even part of the object's URL. This makes it easy to see if you already have this object. If you don't insert it, and return HTTP status `201 Created`.

If you do have the object, then you've detected a duplicate request. What should the response be? Is this an error?

Nope. This is precisely what the client should do. Presumably it didn't hear the first response. So the correct thing to respond with the second time is `201 Created`.

## Never expose the internal ID

When you insert an object into your relational database, it will increment a primary key. That is, if you set up the primary key to be an `INT IDENTITY(1,1)`. This is great, because it gives you a very efficient way to link this object to its children with foreign keys.

However, this generated integer ID should **not** be returned to the client. This is not the identity of the object. Only the unique request ID generated *before* the initial `POST` should be exposed to the client.

Why? Well to start with, since this ID is generated on the server, the client has to wait for a full round trip before it can create any child objects. If you want to build a rich client application that works offline, then you need to be able to queue up more requests before the first one ever made it to the server.

But more importantly, the database-generated ID is location dependent. It is only meaningful on that specific database. If you decide to migrate your database to a different server, you will need to be careful to preserve the database-generated IDs. And if you decide to merge two databases together, they may have already used the same ID to represent different objects.

## Retry until successful

Once you've generated the request ID *before* creating the object, and then checked for an existing object on the back end, then you can safely retry submissions. Go ahead and let the end user click submit twice! Or better yet, do it for them! Write your rich client application to put the request in a queue. If you don't get a response from the server, keep the request in queue and try it again. Keep trying until you get a `201 Created`, or a `400 Bad Request`. Any other response code (especially a `500 Internal Server Error`) means that you need to try again.

You can download a full working example of an API that doesn't mind if you click submit twice. The [Mathematicians](https://github.com/michaellperry/Mathematicians) example used above is on GitHub. Please fork it and give it a try.