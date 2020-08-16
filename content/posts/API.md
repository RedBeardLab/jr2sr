---
title: "API"
date: 2020-08-16T15:19:28+02:00
draft: false
---

In this post we are going to study the widely use term API. You will see how this connect quite nicely with the previous article about [web requests]() and tangentially to the one about [DNS]() and [processed]().

Let's get started.

## The term `API`

API stand for `Application Programmatic Interface`, the term is very wide and includes all the way a program/application can interact with something else.
Indeed the term is so wide that does not really help.
An API can be used to make some actions happens, retrieve data, change some internal state, whatever.
Strictly speaking whatever expose an interface callable by a software, it is exposing an API.

They way you call a library in python or javascript or any other programming languange is an API.
Indeed every library defines an API.
All the classes, methods, functions, constants and underlining absubtion that a library define and relies on, are part of the library API.

Similarly, as we will see when we will study, sockets, files and I/O, the operative system (either Linux, Windows, Mac, etc...) defines an API.
Indeed there is an interface to display stuff on screen, read files, make web requests, and all the other things that the operative system does for us.

Softwares might defines other API, for instance the browsers, like Chrome or Firefox, defines an API.
The javascript that is executed by the browser must indeed follow the API of the browser.
Actions like changing a `div` class, or add a CSS property must be done following the API of the browsers.

Now that we have understood how wide the term API is, we should leave aside these technicalities and focus on what is colloquialy called an API. 
The Web APIs.

## Web APIs

Up untill now we discussed API that were used inside a programming language.
When we discuss the API of a python library, that API is consumed using Python.
When we talk about the APIs of a browser, those API are consumed using Javascript.
(With the operative system APIs, those API are consumed using `C` arguably the language of the OS. Of course this is a semplification.)

The idea behind web API is simple, and it starts with a question. 
What if the way to consume the API is not language specific, but it is transmitted in a widely used protocol, like HTTP?
In this way all programming languages that can talk HTTP (basically all) would be able to consume the API.

As we studied in [`What is a web request`](), HTTP is very convenient, not only because virtually everywhere, but also because it a text based protocol.
Which means that it is simple to inspect it and see what is going on underneath.

We understand that the most common way to expose Web API is an HTTP server, but it is far from the only way.
There are other possibilities, each for different tasks.
For instance, we could expose a RabbitMQ, or MQTT endpoint, to allow our user to receive notification.
We could expose a (g)RPC server, that uses the protoc protocol, to allow fast transmission of binary data.
Another common possibility is to expose databases over the internet (usually over a private fraction of the internet) in this way applications can store and retrieve data.

We now have an understaing of API.
From consuming an API inside our favourite programming language we move into consuming it over an agnostic protocol.
We saw that the most common protocol to use is HTTP, but there are dozens upon dozens of other way to expose an API.

The next step will be to explore more throughly how the use of HTTP for exposing API.

## Advanced HTTP

We studied in [`What is a web request`]() the foundantation block of an HTTP request.

As you might recall an HTTP request is composed of:

1. Verb
2. Path
3. Headers (Host is mandatory)
4. Optional Body

The response is composed of:

1. Status Code
2. Headers
3. Optional Body

Now we will focus on the less obvious part of both the requets and the response in order to understand the details.

As always, we will study general, usually applied principle, however each application is different and for different reason, the API of some application might deviante from the best practise we are studying now.
Howeve, as a developer, we should try to follow the best practises as much as possible.

### HTTP Verb

The verb specify what is the intention of the request.

There is a verb to getting / downloading a resource, the `GET` verb. 
There is one for uploading a request `POST`.
More advanced verbs are the one used to delete some resource `DELETE`, updated a resource `PATCH`, upload a resource of know identifier `PUT` or to check if a resources exists without downloading it `HEAD`.

Using the correct verb helps a lot while designing a web API.

For instance, suppose we are designing a simple API to manage a website that shows images.
A ammateur interface would be something like:

```
POST /new-image
GET /image/<id>
POST /delete-image/<id>
POST /update-image/<id>
GET /image-exists/<id>
```

You can see that we embed the action in the path of the request, `/new-image`, or `/delete-image` or `/image-exists` while we don't make use of the HTTP verbs.

A more professional interface will look like:

```
POST /image
GET /image/<id>
DELETE /image/<id>
PUT /image/<id>
HEAD /image/<id>
```

In this other interface we are making the correct use of the HTTP verbs.
Once the meaning of each verb is being codified and respected, the advatanges are clear.
The URL are much simpler to follow, there is just one `/image` with or without the identifier.
It is impossible to make mistake. 
You will never need to ask if the url to upload a new image is `/new-image` or `/upload-image`.
Or if the one to delete the image is `/remove-image` or `/delete-image`.

Overall, the use of the correct HTTP verbs make the interface much nicer to use and simpler.

### HTTP Headers

The headers is an HTTP call are the place that provide the `metadata` for the call.
They usually stores information like the authentication credentials, or information about the paging if you are returning an huge number of elements.

Important information that are stored in the headers are the caching policy.
Can the HTTP request be cached by any proxy or middleware? For how long? All this information should be stored in the headers.
The cache information are provided by the `Cache-Control` header.

Other important information in the header are the type of content that the caller expects.
For instance, does the caller expect to receive data as JSON or as XML?
This can be specified by the `Accept` header.

Parsing and understading the content of ann HTTP response is complex, the body can be big, the content encoded in some weird format, etc... hence all the information that are somehow interesting to manage the request itself, should be provided as headers.
This allow smarted proxy and middleware to optimize the messages.

### Status code

Another fundamental piece of professional developed API is the correct use of status codes.

As HTTP verbs helps in simplify the requests, status codes help in simplify the responses.
An user of the API can immediately understand what happend to a request just from the status code without actually reading the body.
This allow the provider of the application to don't provide a body for all possible errors or mistake (even though one is always a good idea).

The status codes are integer numbers between 100 and 599, not all of them are used.

They are divided into 5 big categories on the most signficant digit.

#### 100 to 103

Protocol related, hardly used in every day code.

#### 200 to 206

Succesfull request, if you get one of these status code, it means that you created the requests correctly and the server was able to reply.
Everythng worked as expected and you can consume the result.

When creating a web API, it is very useful to distinguish between `200 OK`, `201 Created`, `202 Accepted`, `204 No Content`.

The first, `200 OK` is a generic positive response, it does not provide much information and it should be avoided when possible.

`201 Created` is a more specific response, it let the client know that the request was successfully and that a new resource was created.
This is usually the correct status code for a `POST` request, or even to a `PUT`.

`202 Accepted` is more complex, it means that the request was succeffully, the server got the data and understood what the client asked.
However the server has not yet finish to act upon the request, it may not even have started.
It is useful if acting on the request is extremelly slow or costly, this let the client know that eventually the request will be worked on, but not now.
This means that the client does not need to keep the connection with the server alive.
When you return a `202` status code, you should also return some way to check the status of the request.
In this way the client can come back to your service, at a later time, and check the progresses.

`204 No Content`, it means that no content is included in the body. It make sense when you are deleting something or when you are updating an old resources.

There are more `2XX` status codes, but the one mentioned are the most used.

#### 300 to 308

These status codes are not widely used when developing a Web API. They might in very complex scenarios but it is safe to omit them in our studies and just kwnoing that they exists.

The most used one is the `301 Moved Permanently`, a redirect, to inform the clients  that the location (URL) of a resources having changed and point the client to the new location.

#### 400 to 451

These status codes are errors or problems on the client side.

Whil writing an API, if you catch an error on the client side and you are forced to return a 4XX status code, you should try very hard to include some information on how to fix the error.

Maybe the authentication is missing? Let the user know.
Maybe the user provided XML while you were expecting JSON? Let the user know.
Maybe the user is not authorized? Let the user know.

The more information you provide the easier will be to develop an application against your API, and developers will thank you for it.

`400 Bad Request` is a quite general status code, the server is not able to process the request and the client should send a different request.

`401 Unauthorized` the client did not provide correct authentication, so for the server was impossible to know if the request should be satisfied.

`403 Forbidden` the client did provide correct authentication, but its is not allowed to actually make the request.
For instance, some user may comment to a blog post, but they cannot modify it, at least they are the authors.
Hence a request to modify the blog post will return `403` to all users but the post author.

`404 Not Found` the most notorious status code. The resorce is not available.

There are more 4XX status codes, but I would like to focus, on the last one, `451 Unavailable For Legal Reason`.
This status code is to be used when some piece of data cannot be served for some legal reason, copyright infringement, embargo or something similar.
The specific number was choosen as reference to Fahreneith 451 a novel about an dystopian universe in which books were burned.
451 Fehreneith is the temperature at which paper burns.

#### 500 to 511

These status codes are errors or problem on the server side.
