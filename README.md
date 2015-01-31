# Objectives of this repo : gapi-node
This repo was created to meet these objectives: 

1. house code for the [interactive client website](#Interactive_demo_client) developed to demo
  how to call a Google API (using JWT) from within a node.js server to shorten a URL for use in Twitter messages;
2. explore [alternatives from Google and others to make those calls](#Altertives_to_call);
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

# Why?
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

# <a name="Interactive_demo_client"></a> Interactive Demo Client to Shorten Long URLs
The sample app shown at https://wilsonmar.github.com/gapi-node 
is structured like an app that calls the Google API.

1. Type out a URL you would like to shorten and click "Shorten".
2. The response is the shortened URL 
3. The QR code image generated by Google.
4. The URL is added to the list of URLs in the drop-down.
5. Statistics presented is for the URL selected from the list.

The sample is similar to other web pages created:
* http://gaigalas.net/lab/googl
* http://hayageek.com/examples/urlshortener-api/index.php
* [Chrome web store app](https://chrome.google.com/webstore/detail/googl-url-shortener/iblijlcdoidgdpfknkckljiocdbnlagk)

The functionality of these web pages can be implemented as a plug-in within several frameworks:
* https://wordpress.org/plugins/googl/

The functionality of these web pages can be implemented in various other programming languages:
* ???

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


# <a name="Altertives_to_call"></a> Alternatives to use Google to shorten URLs

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



## <a name="from_goo.gl_anon"></a> Obtain short URLs manually from goo.gl as an anonymous user
As an introduction to how Google’s URL Shortener API service works, let’s look at the human user interface which web services APIs replace.

1.	Use the Google Chrome browser. If you do not have one installed, install it from http://google.com/chrome.
2.	Sign out of your Google account.
3.	Type in the browser omni bar Google's URL Shortener home page at http://goo.gl/.
4.	Type or paste a URL such as maps.google.com in the box under "Paste your long URL here". 
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
Notice that the URL processed recently does not appear on the list of URLs because it was created under Google's account. URLs previously shortened using your Google account should show up on this page. 
3.	Type or paste URL maps.google.com (or other URL of your choice) in the text box under "Paste your long URL here".  (http:// is not needed). 
4.	Double-click on the long URL and press Ctrl or command+C to save it in your clipboard for use later.
5.	Press Shorten URL. 
6.	Paste the long URL and shorten it again.
7.	Press Ctrl or command+C to save it to your clipboard.

## <a name="from_node_cli"></a> 

## <a name="from_google_node_js"></a> 
## <a name="from_api_console"></a> 

## <a name="from_playground"></a> 
## <a name="from_devtools"></a> 


## Get Your Google API Key

## Convert Your Google API Private Key

## Client libraries
https://developers.google.com/api-client-library/dotnet/apis/urlshortener/v1

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
