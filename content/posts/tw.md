---
title: "Scheduled Tweets With Cloudflare Workers"
date: 2022-08-17
draft: false
author: "Austin Christiansen"
authorLink: "https://austinchristiansen.com"
description: ""
---

<!--more-->

### Making scheduled tweets with Twitter API v2 and Cloudflare Workers

&nbsp;

In this post, we'll explore how to use Cloudflare Workers to make scheduled tweets with Twitter API v2.

[Check out the GitHub repository with the code used in this article](https://github.com/austin-mc/TwitterWorker)

&nbsp;

#### Getting Started With Cloudflare Workers

If you haven't already, head over to the Cloudflare Workers [docs](https://developers.cloudflare.com/workers/) and read the [get started guide](https://developers.cloudflare.com/workers/get-started/guide/). Follow these to install wrangler and log in to your account.

To get the project started, we'll use the JavaScript quickstart provided by running the following code: `npm init cloudflare TwitterWorker https://github.com/cloudflare/worker-template`.

This will create the files needed for the project. We'll first get `wrangler.toml` updated by replacing the default code with the following:

&nbsp;

```toml
name = "twitterworker"
node_compat = true
main = "index.js"
compatibility_date = <CURRENT DATE>

account_id = <YOUR ACCOUNT ID>

[triggers]
crons = [ "0 0 * * *" ]
```

&nbsp;

To get your account ID, you can run `wrangler whoami`. The main thing to note here is the `[triggers]` section with our cron. A cron trigger specifies when the scheduled Worker will run. The [Cloudflare docs](https://developers.cloudflare.com/workers/platform/cron-triggers/) go into detail on the syntax here. In this case, the cron is set to run once at 00:00 UTC every day.


Next, open `index.js`, and modify the default event listener to listen for a scheduled event instead of a fetch (we'll write the `triggerPost` function later): 

&nbsp;

```js
addEventListener('scheduled', event => {
  event.waitUntil(triggerPost(event));
});
```
 
&nbsp;

Once this is done, we're ready to get started with the Twitter API. 

&nbsp;

#### Setting up the Twitter Developer Account

Now, we'll log in to the Twitter [developer portal](https://developer.twitter.com/en) and create a new project. Under the "User Authentication Settings" section for the project, we want to enable OAuth 1.0a. In OAuth 1.0a settings, we need to select "read and write" for app permissions so the app is able to post tweets. Next we'll go to the Keys and Tokens tab of the project and regenerate the Access Token and Secret, which is required after changing authentication settings. Finally, we'll write down the API Key and Secret, as well as the Access Token and Secret.

&nbsp;

#### Generating the Request Authorization

Now that we have our API keys, we can use them to generate the request authorization in our Worker. In the directory with the project, run `npm install oauth-1.0a --production` and `npm install crypto-js`. The OAuth 1.0a package makes generating the request authorization much easier, and crypto-js will be used for its SHA-1 hash in the request authorization signature. Much of the code for the authorization in this project comes from [docs](https://www.npmjs.com/package/oauth-1.0a) for the OAuth 1.0a package. At the top of `index.js`, import the two packages:

&nbsp;

```js
import { HmacSHA1, enc } from 'crypto-js';
import OAuth from 'oauth-1.0a';
```

&nbsp;

Next, we'll want to safely store the API keys. We can do this with `wrangler secret put <variable name>` and placing the key in the following prompt. This will encrypt and securely store our keys for use in the Worker, where they can be used just like any other variable. For this project, I used the following secrets:

- TWITTER_API_KEY
- TWITTER_API_SECRET
- TWITTER_AUTH_TOKEN
- TWITTER_AUTH_SECRET

Now we can begin generating our authorization. We'll initialize the OAuth with our API key and secret, and define a quick hash function to generate the HMAC-SHA1 signature required by OAuth 1.0a.

&nbsp;

```js
const oauth = new OAuth({
	consumer: { key: TWITTER_API_KEY, secret: TWITTER_API_SECRET },
	signature_method: 'HMAC-SHA1',
	hash_function: hashSha1,
});

function hashSha1(baseString, key) {
	return HmacSHA1(baseString, key).toString(enc.Base64)
}
```

&nbsp;

Next, we’ll add our request URL and HTTP method to a variable that will be added to the authorization, and in a seperate variable we'll store our Access Token and Secret:

&nbsp;

```js
// Will be added to request headers
const reqAuth = {
  url: "https://api.twitter.com/2/tweets",
  method: 'POST',
};

const token = {
	key: TWITTER_ACCESS_TOKEN,
	secret: TWITTER_ACCESS_SECRET,
};
```

&nbsp;

#### Using Cloudflare Workers KV

To store the text that we want to Tweet, we'll use Workers KV. Doing so allows us to store the tweet in a variable similar to a `wrangler secret`. For this case, using KV is beneficial because the value can be changed through wrangler or on the Cloudflare dashboard without having to change the actual code. KV also supports changing the value from within the code, but this will not be used for this project. Read more about KV [here](https://developers.cloudflare.com/workers/runtime-apis/kv/).

To get started, we first need to make a KV namespace with `wrangler kv:namespace create "TWITTER_WORKER"`. Once our namespace is generated, wrangler will give an output to be added to `wrangler.toml`. It will look something like this: 

&nbsp;

```toml
kv_namespaces = [
    { binding = "TWITTER_WORKER", id = "<NAMESPACE ID>" }
]
```

&nbsp;

Once this is added to `wrangler.toml`, we can add the KV pair for the tweet with `wrangler kv:key put --binding=TWITTER_WORKER "TWEET_CONTENT" "Sent from Cloudflare Workers"`. We can access this value within the code by using `await TWITTER_WORKER.get("TWEET_CONTENT")`. To update the value of the key we can use the same wrangler command used to create it, or it can be done through the Workers dashboard at any time.


&nbsp;

#### Making the Fetch Request

Now we're ready to begin formulating our POST request. We'll put everything together in our function `triggerPost` invoked by the scheduled event listener. First, create a JSON object using `json.stringify` with the `"text"` value being the content of the tweet. With Cloudflare Workers, `fetch` will be the function we use to send our HTTP request. For headers, we'll use  `oauth.toHeader` and `oauth.authorize` together with our request authorization and token variables, and set the `Content-Type` to `application/json`. Once we've done this, our Worker is finished. 

Here's the complete code for the `triggerPost` function:

&nbsp;

```js
async function triggerPost(event) {
  
  var reqBody = JSON.stringify({
    "text": await TWITTER_WORKER.get("TWEET_CONTENT")
  });
  
  const response = await fetch(reqAuth.url, {
    method: reqAuth.method,
    headers: {
      ...oauth.toHeader(oauth.authorize(reqAuth, token)),
      'Content-Type': 'application/json',
    },
    body: reqBody
  });
  
	return new Response(await response.json());
}
```

&nbsp;

#### Deploying the Worker

All that needs to be done now is to run `wrangler dev` for a test run and `wrangler publish` and Worker will be live. Once published, the Cron may take up to 30 minutes to take effect.

Heres the final result:

&nbsp;

![Final Result](/images/workers.png)

&nbsp;

Thanks for reading!