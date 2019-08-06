---
Title: "Angular Starter Kit with Single Signon (Authorization Code Flow)"
Date: 2019-06-21
Lastmod: 2019-06-21
Author: Wickstargazer
image: /post/images/angular_starter.jpg
Categories: ["Angular", "OpenID", "TypeScript", "SSO"]
Tags: ["Web Development", "OpenID", "Angular", "TypeScript", "SSO"]
Draft: false
MinuteRead: 4 MIN READ
PhotoBy: Markus Spiske temporausch.com from Pexels
---

*In this article, we will talk about how to startup a universal angular app that integrates with a Single Signon Server using the authorization code flow. The github repo is available inside. It is the second part of the SSO series that focus on the front-end app connectivity*

<!--more-->

> Disclaimer: I assume the reader understands how to write html/scss and start a basic angular-app or any webpack based site like Vue or Reactjs before reading this blog. If not please visit [getting started with angular before going on!](https://angular.io/start "getting started with angular before going on!")Disclaimer: I assume the reader understands how to write html/scss and start a basic angular-app or any webpack based site like Vue or Reactjs before reading this blog. If not please visit [getting started with angular before going on!](https://angular.io/start "getting started with angular before going on!")

Choosing a technology to develop is as confusing and as hard as choosing where to take your partner on an exiting date that will close the deal! After all the hard research and trials and errors, if you end up with angular typescript then you are reading the correct article on how to get started!

First off, angular framework is a Dependency Injection oriented framework where everything is dependent on something! And that can be a brutal struggle when you try to get things off the ground. But thanks to their CLI, it makes everything so much easier. Before getting started, here are some recommendations.

1. Use **yarn** to manage your package instead of **npm** because it will delete the unused dependencies and re-install and keep your **node_modules** clean, not confusing versions for you when running into some dependency bugs.
2. Never ever `import * from` anything in your project because it causes the bundle to be extra large!
3. Always set up the paths in your tsconfig.json before importing relative paths. It saves a lot of string length and make your code **READABLE!** 
 	Example like so:
 	```
    "paths": {
	    "@/*": ["src/app/*"]
	}
    ```
8. If your project is public and there is no login gate....make god damn sure that its angular universal and it's content is indeed rendered from the server side.

And then happy coding! :smiley:

Ok, lets do the hard part. First off you need to initiate the project, mostly angular documentations helps you do that so I wouldnt ask you to read on if you haven't visited the [angular-cli getting started guide.](https://cli.angular.io/ "angular-cli getting started guide.")

Once you have `ng new && ng generate && ng serve` your first app, lets look at the other components inside that actually makes the project a solid starter kit for building a usable viable product!

To start off, lets look at how you run Angular normally. In a normal day, we use `ng serve` or `ng build` to run the angular app. Since we are going to be using the OpenID flow, we have modified the `server.ts` file to allow the expressjs and server-side of things to handle the flow. 

Therefore in this case we will also be running our Development environment on universal mode using the code in server.ts to help us with Authentication. So to make that able, I added the command to build and run dev into package.json

```
"build:dev": "ng build && ng run angular-starter:server && npm run compile:server"
 "compile:server": "tsc -p tsconfig.server.json"
 "serve:ssr-dev": "nodemon --watch src/ --exec node dist/server --verbose  -e ts,html,scss"
```
To run the app, use `yarn build:dev && yarn serve:ssr-dev`

Thats it! you have the app running.

***Note: This is not the best way to do a hot-reload but since angular-cli does not support hot-reload for universal app yet, we will go with this for now :smile:***

Ok, now we have started the app lets look at the structure of the project, the code to help understand how it works because most probably there are fine-tuning to do or bugs to fix :blush:

**Server.ts**
```
import * as cryptoRandomString from 'crypto-random-string';
import * as request from 'request';
import { config } from './server-config/config';
import { Issuer } from 'openid-client';
import * as cache from 'memory-cache';
```

These are the important imports to get things started with the authentication. The crypto helps generate the nonce state, while the config is the OpenId identity config. Example like so...

```
{
    baseUrl : 'http://localhost:3000',
    issuer: 'https://id.server.com',
    authorization_endpoint: 'https://id.server.com/connect/authorize',
    token_endpoint: 'https://id.server.com/connect/token',
    userinfo_endpoint: 'https://id.server.com/connect/userinfo',
    jwks_uri: 'https://id.server.com/.well-known/openid-configuration/jwks',
    endsession_endpoint: 'https://id.server.com/account/logout',
    post_logout_redirect_uri: 'https://example.com',
    CLOCK_TOLERANCE: 25300,
    client_id: 'server-front-server',
    redirect_uri: 'http://localhost:4200/callback',
    scope: 'openid server-api offline_access',
    response_mode: 'form_post',
    cookiesDomain: "localhost"
}
```
The Issuer module is the main library we use from openid-client npm. It helps with the redirecting, getting code, getting the token after callback. It wraps the standard checks and validations for the nonce and http request flow. The thing you have to lookout for is the Clock Tolerance and `Issuer.defaultHttpOptions = { timeout: 50000, retries: 3 };` The first allows a difference in the machine clock timing which sometimes is a pain if its cloud based and have discripancy across regions.

The magic function:
```
const requireLogin = async (req, res, next) => {
  if (req.cookies.your_token_name && req.cookies.your_refresh_token_name) {
      next();
  } else {
      const state = await stateGenerated();
      const postForm = client.authorizationPost({
          redirect_uri: config.redirect_uri,
          scope: config.scope,
          response_mode: config.response_mode,
          response_type: config.response_type,
          state: state
      });
      res.send(postForm);
  }
}
```
This function does what it says! :laugh: Add it as a middlewear to the routes and you make those routes secure!
It checks for cookie as a base. I have not configured the cookie to be http-only because of the different in architecture design as some system uses the Authorization header to send the Bearer token. While some api introspect the token from a cookie value.

**Security wise, both Bearer and Http-Only Cookie are secure because no external javascript can add Header to an ajax request or site request. however there are other ways to prevent XSS and Cookie fraud attacks. One such way is to use XRSF token to re-identify the request. Lets talk about that in another Blog! And for now, its secure enough to get started!**

I will not dig into the `/login`,`/callback`,`/logout` and `/register` at this moment since it re-uses the wrapper but just make sure the routes are handled by the server-side.

So this is the main entry point for your angular app.

```
router.get('*', requireLogin,  function (req, res) {
   // res.sendFile(path.join(__dirname + '/index.html')); // non universal render
   res.render(join(DIST_FOLDER, 'browser', 'index.html'), { req, res });
});
```

What it does is check for login and render the index.html the same way the universal render would! I would like to add some additional trick. Thats the transfer-state. What I do to help transfer states from server-side to client-side is by adding the provider to the render function like so. This makes the IsMobile flag have value when doing the server-side render in your component.ts. However i will dig into the TransferState later on.

***Basically it helps not to fetch http requests twice on the app or call certain function twice when rendering like trying to render the mobile site***

```
res.render(join(DIST_FOLDER, 'browser', 'index.html'), {
      req,
      res,
      providers: [
      { provide: 'IsMobile', useValue: req.device.type === 'phone' }
    ]
```


*The third part of the series will be showing the implentation of the actual identity server*


