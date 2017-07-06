---
layout: post
title: "Same-origin on saperated frontend and backend"
categories: gsoc
---
This is my 3rd week into the GSoC program under Mozilla and today I will write about how I achieved same-origin on [science.mozilla.org](https://science.mozilla.org) and its api-server. 

<!-- more --> 

Some Background Context
=======================
I am doing GSoC project under Mozilla. As part of this project I have to implement a number of features on Mozilla Science website. The website frontend (written in react) runs on science.mozilla.org and talks to an api server (written in Django). So one of the features I was trying to implement was to implement authentication. These were the possible ways:

Token Based Authentication
--------------------------
An elegant method, which would have been ideal, except for the fact that the api server was also used to serve django's admin interface and we wanted a single authentication for both admin and frontend. Since django used session authentication for admin we were left with session authentication.

Session Based Authentication
----------------------------
Session Authentication is a pain when authenticating across applications. The major problem is xsrf attack. If you haven't heard of it, read about it in an excellent [article](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) by OWASP. Go on, we will wait.

So normally how developers avoid the attack is by making sure that the client send a csrf token along with their request. The csrf token is normally stored in a browser cookie or inside HTML sent by server. Since an attacker cannot access these locations he cannot access the token.

But the same problem applied to us, since we could not have access to browser cookies (because the api-server runs on different domain).

Solution was to somehow bring client and server on same-origin, to make cookies set by server accessible to client.

We could do this in following ways:


Integrate Fontend and backend
-----------------------------
Very messy, as it would involve integrating two repositories and making sure react and django play nice with each other

Use a proxy server
------------------
Setup something like a proxy-server which would route requests based on its request path. For example, `/api/*` requests would be routed to api-server and everything else to frontend server. One disadvantage was the proxy-server code would be saperate from frontend and backend and routing information would be split across three codebases.

So we integrated the the proxy-server code with our frontend code using [http-proxy-middleware](https://www.npmjs.com/package/http-proxy-middleware). Now all the requests would be directed to frontend-server and if its an `/api/` request it would be proxied to api server.

This approach was not perfect and still left something to be desired namely:

- The same routing info was in two repositories
- Proxying adds some additional overhead to api requests

But I think we can all live with that.
