---
Title: "Single Sign On (SSO)"
Description: Single sign on is a single key to multiple resources or locks on a platform.
Date: 2019-06-21
Lastmod: 2019-06-21
Author: Wickstargazer
Image: /post/images/sso.png
Cover: https://wickstargazer.com/post/images/sso.png
Categories: ["Security", "Single Sign On"]
Tags: ["Security", "Single Sign On", ".Net Core"]
Draft: false
MinuteRead: 4 MIN READ
PhotoBy: Nguyen Nguyen from Pexels
---

*Single sign on is a single key to multiple resources or locks on a platform.*

<!--more-->

Single sign on is a single key to multiple resources or locks on a platform. Similar to a google account can access all of google products so is a FlowAccount account can access multiple products and services from the Single Sign On (SSO).

We choose to use .NET core for the development and the identity server provioded by the framework is at it’s 4th implementation. Here is the [github link.](https://github.com/IdentityServer/IdentityServer4 "github link.")

Ok lets talk about the Concept. There is one and only way to request for resources through SSO and that is introspection of the client Token. Now there are two type of tokens that is commonly used.
A JWT token (where the user info/user claims is hashed into the token and kept with the client.
OR a reference token where the token is just a reference but user profile and information is kept in the server-side. This case, the user claims are kept in the database.

To request for a token there are many a ways. Each client can be granted to authorize the user with these ways. Hence the ways are called grant types. These are the basic grant types that are common :

1. Password
2. Client Credentials
3. Authorization Code
4. Hybrid
5. Implicit

Now there are a few grant types that cannot co-exists with others for security purposes. Such as Hybrid and Authorization Code.

Now we always hear the magic word, “Open ID”, what is it? It is the most common used standard in the web world to authorize a user and return an access token. It is used by Google and Facebook and many other platforms that allow a third-party app to request for a token and conduct actions on different resources and services on their platform.
![authorization code flow](/post/images/authorization_code_flow.png)

We utilizes the Authorization code flow. OpenID requests a .well-known url that tells the client where is the IdentityServer, what is the token endpoint, and other relevant endpoints like introspection, profile etc.

What happens in the Authorization code flow is that the user comes into the resource, the resource asks if the user has a token either by the Authorization Header or Cookie in the HTTP request. If not, it redirect the user to the Identity Server with a Redirect URL telling the server that once complete please post back to me with user’s credentials.

The resource then gets the token and saves it someone to make sure when it requests other actions the token is sent to the server either through the Authorization header or cookie. The security of sending the token can be drill down and talked about in more details.

This happens with several security checks like using the nonce cookie, and certain information also can be passed using the state object.

Usually the OpenID flow is done on a front-end app. While any API resources just introspect the token and does not get involve in the login or requesting the token.

Coming to the last point. Since a platform can consists of a gazillion resources and we would not want all the client to access to all of it, so we do have a concept of limiting scope to the clients. These scopes are often shown to the user that it is being requested and user chooses to allow all or parts of it. Some system are simple so we do not list them or force the user to click allow access.

There are few basic tricks that is easy to discover and knowing them does take you a long way. These are ….

1. When generating clients, these are the objects that a client usually look for to be complete. 
	- Choose between JWT/Reference token
	- Setup your grant types and do not forget the refresh_token type if you need. If you allow the client to refresh tokens, do not forget to give them the offline scope.
	- Using SharedSecret in the ClientSecret, if you misspell it will not work and will not give an understandable error message.
	- Add the scope to the api resource on which that client needs.
	- Always have a postman collection to do an introspection and use those tokens manually for easy testing of the setup.
    - Always check the redirect url, even one “/” can make the request invalid and cause the redirect to fail.
2. The main difference between client_credentials and OpenID/Authorization Code flow is one to one and one to many. Being client_credentials is for one user to request for all resources. While the other is for the client to authenticate many user to all resources.
3. When generating client_credentials do not forget to add the scope claims to specify which user in the system is the IdentityServer client for.
4. Last and not least, do have your introspection collection ready so you can trouble shoot quickly.

These are the absolute basic of the Identity Server Single Sign on (SSO) please stay tuned for more details elaboration of the implementation.

 [Second part -- Angular Starter Kit with Single Signon (Authorization Code Flow)](http://https://wickstargazer.com/angular-starter-kit-with-single-signon-authorization-code-flow)