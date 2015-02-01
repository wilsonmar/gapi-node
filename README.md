# Objectives of this repo : gapi-node
This repo was created to provide you: 

1. house code for the [interactive client website](#Interactive_demo_client) developed to demo
  how to call a Google API (using JWT) from within a node.js server to shorten a URL for use in Twitter messages;
2. explore [the workflow to use Google APIs](#Altertives_to_call);
3. explain [JWT construction internals](#JWT_internals) (base64, signing, etc.) used in the code; and
4. show additional variations on use of Google APIs, such as the [Reports API](#Reports_API) 
  to collect and display analytics data Google collects.
 
# Tasks To Build This
- [ ] Objective 2 - Explore libraries (done first to not waste time on objective 1)

- [ ] Objective 1 - Call Google API
  - [x] Create github repo (Wilson)
  - [ ] Create github.io/gapi-node (wilson)
  - [ ] Provide API key (Wilson)
  - [ ] Code client form URL with no formatting, no retrieval of QR code (Abdul)
  - [ ] local node.js skeleton (Wilson with Abdul)
  - [ ] Add library to call Google API (Abdul)
  - [ ] Add calling code in node server code (Abdul)
  - [ ] Code client QR code retrieval and display client side (Abdul)

  - [ ] Transfer server code to Heroku?
  - [ ] ~~Store credentials in MongoDB~~

- [ ] Objective 3 - Explain JWT
- [ ] Objective 4 - Reports API calls

# <a name="Interactive_demo_client"></a> Interactive Demo Client to Shorten Long URLs
The sample app shown at https://wilsonmar.github.com/gapi-node 
makes calls to the Google API URL Shortener [first made available in 2009](http://googleblog.blogspot.com/2009/12/making-urls-shorter-for-google-toolbar.html).

1. Type out a URL you would like to shorten and click "Shorten".
2. The response is the shortened URL 
3. The QR code image generated by Google.
4. The URL is added to the list of URLs in the drop-down.
5. Statistics presented is for the URL selected from the list.

The sample is similar to other web pages created:
* http://gaigalas.net/lab/googl
* [Chrome web store app](https://chrome.google.com/webstore/detail/googl-url-shortener/iblijlcdoidgdpfknkckljiocdbnlagk)
* http://hayageek.com/examples/urlshortener-api/index.php

The functionality of these web pages can be implemented as a plug-in within several frameworks:
* https://wordpress.org/plugins/googl/

Sample client code in various other programming languages have been published :
* PHP : https://code.google.com/p/php-url-shortener/
* Apple Objective-C : https://code.google.com/p/google-api-objectivec-client/
* Go language : https://godoc.org/code.google.com/p/google-api-go-client/urlshortener/v1
* .NET : https://developers.google.com/api-client-library/dotnet/apis/urlshortener/v1

# <a name="Altertives_to_call"></a> Workflow to use Google for shortening URLs

This illustration shows how data is input into and output from Google's servers:

![Data flow](http://www.merc.tv/img/fig/goo.dataflow_v01.png "Data flow")

Among the many services Google provides, in this tutorial, we use Google's URL Shortener service  (at this scope address https://www.googleapis.com/auth/urlshortener/v1/) because one doesn’t need to stand-up a custom server in the Google Cloud to work with the URL Shortener service Google provides.

Google’s goo.gl website converts long URLs into short URLs for inclusion within tweets or for quicker typing in keyboards.
Google also provides an API Explorer, a Playground, and the code.google.com website for exploring calls made on behalf of a 
Google user signed in. These sites mock how approved sites can use Google's API to generate shortUrls.

Behind the scenes within each of these websites, there needs to be calls to Google’s **authentication service** so Google can enforce limits on how many calls are made.

Because Google maintain statistics on where and when each shortUrl is created and invoked by the public, Google can display statistics on its web pages but also offer statistics to custom web sites through its Reports API.
But first, a Node.js library such as this (which we'll be talking more about later) is used to authenticate with Google using credentials Google assigns through its Developer Console for a particular project associated with a Google account. 

The authentication token assembled are sent to Google's authentication server to obtain an access token that will be sent along with requests for conversion. When an access token expires, the refresh token provided by Google is used to obtain more access tokens.

<hr />
# Different ways to access a Google API

The remainder of this tutorial takes a "deep dive" into the different ways Google's URL Shortener service can be accessed. 

1.	[Obtain short URL manually from goo.gl as an anonymous user](#from_goo.gl_anon)
2.	[Obtain short URL manually as a signed-in Google user](#from_goo.gl_signed_in)
3.	[Obtain short URL manually using Node.js command line](#from_node_cli)
4.	[Obtain short URL from Google's Node.js program](#from_google_node_js)
5.	[Obtain short URL manually using Google's API Playground](#from_playground)
6.	[Obtain short URL manually using Google Code](#from_devtools)

Along the way, we consider several technical details:

1.  [Speed test of goo.gl vs. competing URL shortener services](#Speedtest_results)
http://royal.pingdom.com/2010/10/29/is-goo-gl-really-the-fastest-url-shortener-chart/
img/fig/goo.speed_tests.png

2.  [Copy shortened URL to clipboard](#copy_to_clipboard)
http://googleblog.blogspot.com/2011/04/beefing-up-googl-with-new-features.html

3.	[Get list of short URLs from Google's API Console for known user](#from_api_console)
g)	Public API access<br />
h)	Use Google's Developer Console to define projects, service accounts, and private keys
i)	.p12 file and cryptography
j)	Store and retrieve private keys securely
k)	Make web service (API) call to Google servers
l)	Manually retrieve report on short URL resolution history 
m)	Retrieve report on short URL resolution history using Node.js

<hr />

## <a name="from_goo.gl_anon"></a> Obtain short URLs manually from goo.gl as an anonymous user
As an introduction to how Google’s URL Shortener API service works, let’s look at the human user interface which web services APIs replace.

1.	Use the Google Chrome browser. If you do not have one installed, install it from http://google.com/chrome.
2.	Sign out of your Google account.
3.	Type in the browser omni bar Google's URL Shortener home page at http://goo.gl/.
4.	Type or paste a URL of your choice (such as maps.google.com) in the box under "Paste your long URL here". 
(http:// is not needed).
5.	Click the "I'm not a robot" checkbox. (This scheme activated September 2014 does not require input of random words, as described at http://www.wikiwand.com/en/ReCAPTCHA).
6.	Click the blue Shorten URL button.
7.	Press Ctrl or control+C to copy the shortened URL to your computer's clipboard. For comparison later, paste it into another document.
8.	In the browser's address bar, double-click to highlight the goo.gl and press Command+V to paste the shortened URL.
9.	Press Enter to resolve the URL.
In this example, Google provides both the front-end and back-end processing under its own account (not that of a Google user).


## <a name="from_goo.gl_signed_in"></a> Obtain short URLs manually from goo.gl as a signed-in Google user
Now let's see what happens when the API request occurs by a user signed into Google:

1.	Type in the browser address box Google's URL Shortener home page at http://goo.gl/.
2.	Click Sign in and provide your Google password.
![goo.gl Langing page](http://www.merc.tv/img/fig/goo.gl_landing.png "goo.gl Langing page")

> Notice that the URL you processed anonymously does NOT appear on the list of URLs associated with your Google account because it was created under Google's account. URLs previously shortened using your Google account would show up on this page. 

3.	Type or paste a URL of your choice in the text box under "Paste your long URL here".  (http:// is not needed). 
4.	Double-click on the long URL and press Ctrl or command+C to save it in your clipboard for use later.
5.	Press Shorten URL. 
6.	Paste the long URL and shorten it again.
7.	Press Ctrl or command+C to save it to your clipboard.

> Notice that the shortened URL created for the same long URL is different than the one created before. Also, different short URLs are created for the same long URL input. Other URL shortener services do not have such behavior. With Google, one can have private shortened URLs (to track links from different marketing channels, etc.). That is why authentication is necessary, for Google to report when and how many clicked on particular short URLs.

8.	Open a new browser window. Perhaps a Firefox browser.
9.	Paste the URL in the browser address bar and press Enter to retrieve the page.
10.	Repeat on a different browser or even different operating system.
11.	Return to the Google url shortener window. 
12.	Click the Details link associated with the URL you just used.

Details for a [popular link](http://goo.gl/#analytics/goo.gl/l6MS/all_time) looks like this:

![Short URL usage Statistics](http://www.merc.tv/img/fig/goo-gl-analytics.jpg "Statistics")
Image credit: [Mashable](http://mashable.com/2010/09/30/goo-gl-url-shortener/)

> Notice Google has generated a QR code for mobile smartphone readers to obtain the URL.

13.	Scroll down. 
	
> Notice Google tracks referrers, from which country the request was made, and what browser, and operating system platform was used to view the link.

## <a name="from_node_cli"></a> Obtain short URL from a Node.js command line
A custom program (not Google) can also provide a way to shorten URLs under its account.

1.	Install node.js with npm on your Windows, Mac OS, or Linux machine.
2.	Install module node-googl into your node.js server:

```
npm install -g goo.gl
```
The module comes from: https://github.com/kaimallea/node-googl

3. At the terminal command line, obtain a short URL by specifying the goo.gl with a host name (without the http protocol):

```
goo.gl     maps.google.com
```

The response would be something like this:

```
http://maps.google.com -> http://goo.gl/fbsS
```

> Notice this command can be invoked repeatedly by another program processing a list of URLs. The short URL output can be captured into logs by node.js modules such as https://github.com/bevry/caterpillar.


## <a name="from_api_console"></a> Get list of short URLs generated from Google API Explorer for known user
1.	Sign in to your Google account.
2.	Go to Google's API Explorer page: http://developers.google.com/apis-explorer/

3.	Scroll down to select URL Shortener, currently at version 1: http://developers.google.com/apis-explorer/#p/urlshortener/v1/

> Notice there are three functions presented (insert a new URL, get the long URL, and list).
 
Because Google manages shortened URLs with authentication, Google can report the creation time and other analytics when it expands short URLs.
4.	Authorize requests using OAuth 2.0 by clicking the OFF switch to turn it ON.
 
5.	Check to select the scope https://www.googleapis.com/auth/urlshortener.
6.	Click Authorize.
 
7.	Now that OAuth is ON, click on urlshortener.url.list (without inputting any parameters).
8.	Click Execute.
9.	Scroll down the see Responses containing shortened URLs, such as:
 
10.	Click on URL Shortener API v1 > to return to the list of functions.
11.	Double-click on urlshortener.url.get.
12.	Paste the URL (such as http://goo.gl/maps/QO5Lp).
 
13.	Press Execute for the long URL for a response containing both short and long URL.

> Notice the 3 steps involved above:

1)	Select and Authorize API with a scope of access
2)	Exchange authorization code for access tokens (which occurs internally)
3)	Configure request to API


## <a name="from_playground"></a> 

## <a name="from_devtools"></a> 


## <a name="from_google_node_js"></a> Obtain short URL from Google's Node.js program 

https://github.com/google/google-api-nodejs-client

```
npm install googleapis --save
```

The response:

```
googleapis@1.1.0 ../../../../../node_modules/googleapis
├── async@0.9.0
├── string-template@0.2.0 (js-string-escape@1.0.0)
└── gapitoken@0.1.3 (jws@0.0.2)
```

Create folder 

gapi_client_node.js

console.log(urlshortener);
	lists all google apis
	
console.log(urlshortener.url);
	{ get: [Function], insert: [Function], list: [Function] }

<hr />

## Get Your Google API Key

## Convert Your Google API Private Key

<hr />
# Why Capture Data from Google?
The following is the narration to this video ___ on YouTube.com and DailyMotion.com ___.
Videos dynamically reveal these illustrations with spoken text.

[While viewing scroll of API Explorer from top to bottom]
![Google API Explorer sample](http://www.merc.tv/img/fig/google.explorer_list_v01.png "Google API Explorer sample")

This tutorial is a guided exploration for developers to quickly learn, in a hands-on way, how to perform server-to-server communication with Google's many web services.
Google provides a large number of APIs (Application Programming Interfaces) so programs in servers can communicate with other computers directly without human interaction: Information from users of Google's Gmail, Calendar, Drive cloud, Google+, and YouTube videos.
Why would one want to take the extra effort to extract data from Google when one has Gmail, Google maps, and other websites that Google provides? 

Let's take a look at this line chart showing corresponding data series across time.

![Data series from several sources](http://www.merc.tv/img/fig/goo.combined_data_v01.png "Data series")

If you want to add event flags or additional data series **not** in Google servers you might need to have all the data on your **own** server. Google can automatically purge data on its severs anytime it wants. And Google has cancelled many services it has provided. So you need a way of keeping your data where **you** can really control.

<hr />

# Server
In order to not expose credentials in client code, calls to the Google API needs to be made from a server. In our case, from a node.js server. The sample server code is based on use of ???

# JWT vs. JWS vs. JWE

## JWT
JWS is an example of JWT defined in the IETF draft at https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32.
JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties.  The claims in a JWT are encoded as a JavaScript Object Notation (JSON) object that is used as the payload of a JSON Web Signature (JWS) structure or as the plaintext of a JSON Web Encryption (JWE) structure, enabling the claims to be digitally signed or MACed and/or encrypted.

## JWS
(https://github.com/brianloveswords/node-jws)
implements the creation of a
JSON Web Signature (JWS) as defined in http://self-issued.info/docs/draft-ietf-jose-json-web-signature.html
JSON Web Signature (JWS) represents content secured with digital signatures or Message Authentication Codes (MACs) using JavaScript Object Notation (JSON) based data structures. Cryptographic algorithms and identifiers for use with this specification are described in the separate JSON Web Algorithms (JWA) specification and an IANA registry defined by that specification. Related encryption capabilities are described in the separate JSON Web Encryption (JWE) specification.

## JWE
