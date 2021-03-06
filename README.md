# Summary Gist (TL;DR)

Google provides a set of [client](#client_libraries) 
and [server](#server_libraries) 
code libraries of many languages to access all the many APIs they run in their cloud.
(still in alpha or beta)
Google's libraries hide the mathematics of [calculating date/time stamps, signatures, and Base64](#Keygen_dataflow) 
needed to authenticate computers requesting information Google holds on behalf of its users. In 2013, Google spent $1.6bn on its datacentres in three months, which is $6bn a year. That figure is nearly one-quarter of the whole venture capital expenditure in the USA for one year – rather a lot of storage. And it's just the start.

Google's service around [short URLs](#short_url_services) is used as an example that does not require more server setup. 

Credentials for access are obtained for a particular project created within Google's Developer Console. 
Obtain from the Credentials page a Public API Key so Google can track usage.
Some requests can be made with just the public key alone, and without need to go through a server.
They API Key is called public because it is included in client JavaScript, so it can be stolen and used by others.

For more secure processing:

1. Spin up a node.js server (locally or on Heroku or nodejitsu, etc.).
2. Sign into the [Google Developer Console](#Get_service_account) to define, then activate a project.
3. From the Google Developer Console download the .p12 file for the user who will own short URL data generated.
4. Within a developer terminal, convert the .p12 file to a .pem file using the generic "notasecret" password.
5. Upload the service account and .pem file to your custom server.
6. Code node.js to receive requests from a client (such as the demo client).
7. Make requests to Google servers by calling Google library functions with the service account address and .pem file.
8. Forward responses back from the Google API servers to the client.

Additionally, custom servers may want to [(reduntantly) obtain data from Google](#Why_capture_data) 
in order to have more flexiblity of presentation (with other data Google does not have)
and to ensure that all data are under house control.


# Objectives of This Repo
This repo provides a guided exploration for developers to quickly learn, in a hands-on way, 
an example of how to perform server-to-server communication with Google's many web services on behalf of specific Google users.

Sample [interactive client webpage](#Interactive_demo_client) code
  (with [tests](#Client-side_test_code)) to 
  [back-end node.js server code](#Nodejs_code)
  (also with [test code](#Server-side_test_code))
  is provided to demo how to use Google APIs (using API Key and OAuth2 with JWT).
	The example is the [various user activites](#user_activities) around [short URLs](#short_url_services).

The demo code is used to explain [the workflow to shorten URLs using Google APIs](#Workflow_diagram) 
	using [various clients](#Client_API_access) to format API calls;

This tutorial explains how you can make your own code, which first requires you to 
[obtain your own credentials](#Obtain_credentials) used for making API calls.

This tutorial also explores [OAuth2 JWT construction internals](#JWT_internals) (base64, signing, etc.) 
used in the library.

In the future, we will show additional variations on use of Google APIs, 
  such as the [Reports API](#Reports_API) 
  to collect and display analytics data Google collects.

TECHNICAL NOTES: 
JavaScript are coded following [Douglas Crockford's conventions](http://javascript.crockford.com/code.html), 
run through [JSHint](http://infohound.net/tidy/).
HTML are run through [HTMLTidy](http://infohound.net/tidy/).
CSS are run through [CSSLint](http://csslint.net/).

HTML, CSS, and JavaScript presented publicly on websites have been minified
and pass the [W3C Validator](http://validator.w3.org/).
So this repository is the place to review code.

Client-side coding makes use of 
jQuery 
and Bootstrap responsive theme library.

# <a name="from_goo.gl_anon"></a> Obtain short URLs manually from goo.gl as an anonymous user
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

> Notice in this example, Google provides both the front-end and back-end processing under its own account (not that of a Google user).


# <a name="short_url_services"></a> Short URL Services and QR Code Images

Up until recently, short URLs were needed to not waste any of the 140 characters allowed in a single tweet on 
[Twitter](http://www.twitter.com/). Long URLs would be first be shortened using a utility website and then pasted into Twitter.

But since Twitter now shortens long URLs (to just 22 characters) **automatically** using their own http://t.co host name, 
another utility website is really not required. However, Twitter does add

PROTIP:
Nevetheless, I personally use the Google service so I can include the QR code image created as a by-product of the
shortening. Visitors can scan the QR image on their smartphone to read the page on their mobile device without needing 
to type in the URL.

...   ![QR code for this page](http://www.merc.tv/img/fig/goo-node.readme.qrcode.png "QR code for this page")

Google began public use of its API URL Shortener 
[in 2009](http://googleblog.blogspot.com/2009/12/making-urls-shorter-for-google-toolbar.html).

The latest news about Google's URL Shortener is posted to the **forum** at
https://groups.google.com/forum/#!forum/google-url-shortener.


### <a name="Shorteners_perf"></a> Competitors and API Response Time

Google is not alone in its offering. The most well-known URL shortening websites are listed in 
[this comparison](http://royal.pingdom.com/2010/10/29/is-goo-gl-really-the-fastest-url-shortener-chart/)
of how quickly URLs are shortend:

...   ![Speed test](http://www.merc.tv/img/fig/goo.speed_tests.png "Speed test")



# <a name="Workflow_diagram"></a> Workflow to shorten URLs using Google APIs

NOTE: The following is the narration to video http://www.youtube.com/v/DEX3KdZD4tg 
which dynamically reveal the illustration below with spoken text.

![Google API workflow](http://www.merc.tv/img/fig/goo.workflow_2015.02.04a.png "Google API Workflow")

In this tutorial, we use Google's **URL Shortener** web service that converts **long** URLs into **short** URLs 
for quicker typing on keyboards and for smaller QR codes that Google generates.
This URL shortener service is a good way to explore the technology behind the many web services Google provides
because we don’t need to stand-up a custom server to use it.

Google provides a public [goo.gl](http://goo.gl/) website to generate short URLs and QR codes using its own
**API Key** used to track and limit usage.
**Custom** websites can get their own API Key from Google's **Developer Console**.

When a shortened URL is requested, the browser used provides data about itself and thus
**where and when** each Urls were used. This information Google saves and makes available
through its [Reports analytics API](https://developers.google.com/admin-sdk/reports/).

**Custom web servers** powered by Node.js or other language can get to that data if it has 
**service accounts** which stand-in for real Google users.
A key pair is generated for each account for use in creating electronic signatures
presented to Google's **Authentication server**.

To help developers figure out how to make calls to Google API servers,
Google created an [API Explorer](http://developers.google.com/apis-explorer/?hl=en_US#p/urlshortener/v1/) 
and the more involved [Playground](https://developers.google.com/oauthplayground/).

Google makes use of the OAuth2 standard.
So an **authentication token** assembled according to the "jot" standard 
is sent to Google's authentication server to obtain **access tokens** that are sent along with each server request. 
When an access token **expires**, the **refresh token** is used to obtain more access tokens.

<hr />

# <a name="Client_API_access"></a> Hands-on Exploration of API Services - Google URL Shortener :

The remainder of this tutorial takes a "deep dive" into the different ways Google's URL Shortener service can be accessed. 
Along the way, we consider several technical details. 
That is the reason why steps covered in other documentation are repeated here.

1.	[Obtain short URL manually from goo.gl as an anonymous user](#from_goo.gl_anon) was presented earlier.

	* [Detailed Statistics](#Details_stats)
	* [QR code](#QR_code)

2.	[Obtain short URL manually from goo.gl as a signed-in Google user](#from_goo.gl_signed_in) 
3.	[Obtain short URL manually from custom client as an anonymous user](#Interactive_demo_client)
4.	[Obtain short URL manually using Node.js command line library](#from_node_cli)

	* [Repeated requests in batch](#Repeated_batch_requests)

5.	[Obtain short URLs from Google API Explorer for known user](#from_api_console)

	* [API Methods](#Methods)
	* [Google Discovery API](#Discovery_API)
	* [Manual Authentication within API Explorer](#Manual_Auth)

6.	[Obtain short URL manually using Google's API Playground](#from_playground)
7.	[Obtain short URL manually using Google Code](#from_devtools)
8.	[Obtain short URL from Google's Node.js program](#from_google_node_js)
9.	[Obtain short URL from C-language LoadRunner script](#from_lr_script)


<hr />


## <a name="from_goo.gl_signed_in"></a> Obtain short URLs manually from goo.gl as a signed-in Google user
Now let's see what happens when the API request occurs by a user signed into Google:

1.	Type in the browser address box Google's URL Shortener home page at http://goo.gl/.
2.	Click Sign in and provide your Google password.
...   ![goo.gl Langing page](http://www.merc.tv/img/fig/goo.gl_landing.png "goo.gl Langing page")

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
12.	Click the **Details** link associated with the URL you just used. 

Since the link you just created won't have any hits yet, let's take a look at one that has lots of hits over time.

### <a name="Details_stats"></a> Detailed Statistics

An example of such a link is http://goo.gl/#analytics/goo.gl/l6MS/all_time created December, 2009.
	An alternate form of it is to type in your internet browser's address bar 
http://goo.gl/l6MS.info constructued by adding **.info** to the code generated.

> Notice that codes generated are case-sensitive. More characters are needed as needed.

...   ![Referrers](http://www.merc.tv/img/fig/goo.gl.l6MS.referrers.2015.02.02.png "Referrers")

> Hits are included in the "Not Provided" or "Unknown" category when https (secure URLs) are used for client requests.

> Because Google servers resolves short to long URLs, it can track meta information from clients used to view the link. IP addresses of clients can be asociated with the country of origin. The browser, and operating system platform are also provided by client browsers.

> Notice at the upper right corner the QR (Quick Response) graphic image for mobile smartphone readers to obtain the URL.

Older version of details, such as [this one captured by Mashable](http://mashable.com/2010/09/30/goo-gl-url-shortener/) 
do not have the circle graph and the world map.

...   ![Countries](http://www.merc.tv/img/fig/goo.gl.l6MS.countries.2015.02.02.png "Countries")

...   ![Platforms](http://www.merc.tv/img/fig/goo.gl.l6MS.platforms.2015.02.02.png "Platforms")

...   ![Browsers](http://www.merc.tv/img/fig/goo.gl.l6MS.browsers.2015.02.02.png "Browsers")

### <a name="QR_code"></a> QR Code Image

13.	To obtain the QR code, change the address by adding *.qr* after the generated code (l6MS) to
http://goo.gl/l6MS.qr.

...   ![QR code for this page](http://www.merc.tv/img/fig/goo.gl_l6MS.qr.png "QR code for this page")

To save the file from your internet browser, right-click on it and select Save image.

> Notice this QR codes is 100 x 100 because only 4 characters were used When goo.gl started. New QR code files may be 150 x 150 or more since codes need to increase over time as more characters are needed to uniquely index a larger number of URLs shortened.


## <a name="Interactive_demo_client"></a> Interactive Demo Client to Shorten Long URLs
The sample app shown at https://wilsonmar.github.com/gapi-node 
makes calls to the Google API URL Shortener
based on code at https://developers.google.com/api-client-library/javascript/start/start-js
but with additional code for jQuery, Bootstrap styling and icons.

1. Type out a URL you would like to shorten and click "Shorten".
	The response is the shortened URL.
3. The QR code image generated by Google.
4. The URL is added to the list of URLs in the drop-down.
5. Statistics presented is for the URL selected from the list.


The sample is similar to other web pages created:
* http://gaigalas.net/lab/googl
* [Chrome web store app](https://chrome.google.com/webstore/detail/googl-url-shortener/iblijlcdoidgdpfknkckljiocdbnlagk)
* http://hayageek.com/examples/urlshortener-api/index.php

> Notice here the client-side JavaScript library https://apis.google.com/js/client.js used is documented at https://developers.google.com/api-client-library/javascript/start/start-js.
The advantage to using this is as more websites use it, the visitors will have it cached on their browser already.
The library supports [many Google APIs](https://developers.google.com/apis-explorer/#p/) (Calendar, etc.).
But the library, https://github.com/google/google-api-javascript-client, 
is still in beta with 90 open issues as of 2015.02.01. We are using it nonetheless.

> QUESTION: Is the client.js library hosted among others at https://developers.google.com/speed/libraries/?csw=1

The functionality of these web pages can be implemented as a plug-in within UI frameworks:
* https://wordpress.org/plugins/googl/

### <a name=client_libraries"></a> Sample client code in various other programming languages have been published by Google at
https://developers.google.com/url-shortener/libraries
and used by others:

* PHP : https://code.google.com/p/php-url-shortener/
* PHP : http://davidwalsh.name/google-url
* Apple Objective-C : https://code.google.com/p/google-api-objectivec-client/
* Go language : https://godoc.org/code.google.com/p/google-api-go-client/urlshortener/v1
* .NET : https://developers.google.com/api-client-library/dotnet/apis/urlshortener/v1
* .NET C# : http://www.jphellemons.nl/post/Shorten-your-URL-with-Googl-or-Bitly-in-AspNet-C
* .NET C# : https://zavitax.wordpress.com/2012/12/17/logging-in-with-google-service-account-in-c-jwt/
* Java
* Python: https://github.com/parthrbhatt/pyShortUrl supports several shortener services (and provides a comparison)

Making data available as a REST API means any platform can have an app for that: 

* Android:
https://play.google.com/store/apps/details?id=com.mattiamaestrini.urlshortener&hl=en
* Apple iOS 
https://itunes.apple.com/us/app/url-shortener-for-googles/id622377001?mt=8


## <a name="from_node_cli"></a> Obtain short URL manually from a Node.js command line library
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

### <a name="Client-side_test_code"></a> Client-side Test Code
TODO



### <a name="Repeated_batch_requests"></a> Repeated requests in batch

This command can be invoked repeatedly by a calling program processing a list of long URLs. 
The short URL output can be captured into logs by node.js modules such as https://github.com/bevry/caterpillar.

> TECHNICAL NOTE: Batch processing of Google APIs using Windows server PowerShell scripts is covered at:
http://powershellnotebook.com/2014/10/31/powershell-shorten-urls-with-googles-api/

> FEATURE: A list of URLs can be processed by the front-end client?


### <a name="Server-side_test_code"></a> Server-side Test Code
TODO


## <a name="from_api_console"></a> Obtain short URLs from Google API Explorer for known user
1.	Sign in to your Google account.
2.	Go to Google's API Explorer page: http://developers.google.com/apis-explorer/

3.	Scroll down the list of Google's APIs to select **URL Shortener**, currently at version 1:

http://developers.google.com/apis-explorer/#p/urlshortener/v1/

...   ![Google URLS API Auth](http://www.merc.tv/img/fig/goo.api.auth.png "Google URLS API Auth")


### <a name="Methods"></a> API Resource Methods

> Notice there are three resource methods provided by the API:

	* [insert](https://developers.google.com/url-shortener/v1/url/insert) a new short URL, 
	* [get](https://developers.google.com/url-shortener/v1/url/get) the long URL from the short URL, and 
	* [list](https://developers.google.com/url-shortener/v1/url/list) the URLs generated and resolved by the public


### <a name="Discovery_API"></a> Google Discovery API

The most up-to-date (canonical) definition of API end points is by 
[Google's Discovery API](https://developers.google.com/discovery/)
which maintains a JSON file of all its public APIs at 
https://www.googleapis.com/discovery/v1/apis.
TOOL: You can search with a formatted display of this JSON file at
https://www.jsoneditoronline.org/?url=https://www.googleapis.com/discovery/v1/apis
The Discovery service requires https but not an API key.

The [video presenting this at Google I/O 2011](https://www.youtube.com/watch?v=lQbT1NrxpUo)
used as an example the [URL Shortener service](https://developers.google.com/url-shortener/?csw=1)
[10:12 into the video](https://www.youtube.com/watch?v=lQbT1NrxpUo&t=10m12s):

```
{
   "kind": "discovery#directoryItem",
   "id": "urlshortener:v1",
   "name": "urlshortener",
   "version": "v1",
   "title": "URL Shortener API",
   "description": "Lets you create, inspect, and manage goo.gl short URLs",
   "discoveryRestUrl": "https://www.googleapis.com/discovery/v1/apis/urlshortener/v1/rest",
   "discoveryLink": "./apis/urlshortener/v1/rest",
   "icons": {
    "x16": "http://www.google.com/images/icons/product/search-16.gif",
    "x32": "http://www.google.com/images/icons/product/search-32.gif"
   },
   "documentationLink": "https://developers.google.com/url-shortener/v1/getting_started",
   "preferred": true
  },
```

The file's schema follows specs at http://json-schema.org.

TOOL: The discoveryRestUrl in the JSON file at https://www.googleapis.com/discovery/v1/apis/urlshortener/v1/rest
can be viewed at 
https://www.jsoneditoronline.org/?url=https://www.googleapis.com/discovery/v1/apis/urlshortener/v1/rest
which structures sections about icons, parameters, auth, schemas, and resources for the object.

The scope URI used to request authentication, 
https://www.googleapis.com/auth/urlshortener,
is specified under <tt>auth > oauth2 > scope</tt>.



### <a name="Manual_Auth"></a> Manual Authentication within API Explorer

4. <a name="Authorize_API"></a> Authorize requests using OAuth 2.0 by clicking/sliding the OFF switch to turn it ON.

...   ![Google URLS API Auth](http://www.merc.tv/img/fig/goo.oauth2.scopes.png "Google URLS API Auth ")

5.	Check to select the scope https://www.googleapis.com/auth/urlshortener.
6.	Click Authorize.

...   ![Google URLS API Auth On](http://www.merc.tv/img/fig/goo.api.auth.on.png "Google URLS API Auth On")

7.	Now that OAuth is ON, click on urlshortener.url.list (without inputting any parameters).
8.	Click Execute.
9.	Scroll down the see Responses containing shortened URLs, such as:
 
...![Google URLS API Auth Json](http://www.merc.tv/img/fig/goo.api.auth.json.png "Google URLS API Auth Json")

10.	Click on URL Shortener API v1 > to return to the list of functions.
11.	Double-click on urlshortener.url.get.
12.	Paste the URL (such as http://goo.gl/maps/QO5Lp).
 
...   ![Google URLS API Auth Execute](http://www.merc.tv/img/fig/goo.api.auth.execute.png "Google URLS API Auth Execute")

13.	Press Execute for the long URL for a response containing both short and long URL.


> Notice the 3 steps involved above outlined in the [workflow diagram above](#Workflow_diagram):
1)	Select and Authorize API with a scope of access
2)	Exchange authorization code for access tokens (which occurs internally)
3)	Configure request to API

> TECHNICAL NOTE: Code behind Google's API Explorer is at https://code.google.com/p/google-apis-explorer/

## <a name="from_playground"></a> Obtain short URL using Google OAuth 2.0 Playground
Google's OAuth 2.0 Playground works only with active (default) versions of services, and includes some services not listed within the API Explorer.

1.	Sign in to your Google account.
2.	Go to Google's OAuth 2.0 Playground (aka API Console) at: 

https://developers.google.com/oauthplayground/

Alternately, you can instead launch the Google Chrome plug-in:
https://chrome.google.com/webstore/detail/oauth-20-playground/fcjholccjchiplkbibepfimlaapdaiih

...   ![Google API Playground](http://www.merc.tv/img/fig/goo.play.api_list.png "Google API Playground")

3.	Scroll down in the list and click **URL Shortener API v1**.
4.	Click to select the scope "https://www.googleapis.com/auth/urlshortener".
 
...   ![Google API Scope](http://www.merc.tv/img/fig/goo.play.scope.png "Google API Scope")

> Note that the scope is not used like public URLs, but internally to uniquely identify which Google API is being used.

5.	Click the blue **Authorize API** button to access the scope selected.
 
...   ![Google API Scope](http://www.merc.tv/img/fig/goo.play.manage.png "Google API Scope")

### <a name="Accept_API"></a> Authentication Token

6. As the user who owns the list, click **Accept**.

...   ![Google API Exchange](http://www.merc.tv/img/fig/goo.play.exchange.png "Google API Exchange")

How this authorization code is created will be described in the next section.

> Notice the authorization code is exchanged only *once* for access tokens which are used on an on-going basis. 
Lukas White [describes it this way](http://www.sitepoint.com/using-json-web-tokens-node-js):
Think of the token like a security pass. You identify yourself at the front desk of a restricted building on arrival (supply your username and password), and if you can be successfully identified you’re issued a security pass. As you move around the building (attempt to access resources by making calls to the API) you are required to show your pass, rather than go through the initial identification process all over again.

7.	Click Exchange authorization code for tokens.
 
...   ![Google API Exchange](http://www.merc.tv/img/fig/goo.play.refresh.png "Google API Exchange")

The Access token is added to communications from the client to establish its authenticity with Google servers. The 3578 seconds shown in this example is considered "short-lived", the time period when the access token remains usable. After that time, the Refresh token needs to be sent to obtain a fresh access token. The format of the packet containing this information is this:

...   ![Google API Access Token](http://www.merc.tv/img/fig/goo.play.access_token.png "Google API Access Token")

> Notice the token_type is "Bearer" (carrier) of the access token as defined in the 
OAuth2 spec.

> Notice also the "expires_in" : 3600 because the default maximum time is used.
Google allows this to be set during JWT requests.

8.	Click *List possible operations* for the pop-up, then expand **Insert Url**:
 
...   ![Google API Send Request](http://www.merc.tv/img/fig/goo.play.list_ops.png "Google API Send Request")

9.	Click on **Insert Url** to auto-populate the Request URI.
10.	In another browser window, view https://developers.google.com/url-shortener/v1/getting_started?csw=1#actions,
which defines this json-formatted request:

```	
POST https://www.googleapis.com/urlshortener/v1/url
Content-Type: application/json
{"longUrl": "http://www.google.com/"}
```

Also note what a successful response looks like in the sample page.

But instead of www.google.com, you would specify the long URL of your choice.

11. Click **Enter request body** to specify the long URL before pressing Close. For example:

```	
{"longUrl": "http://www.wilsonmar.com/"}
```

...   ![Google API Send Request](http://www.merc.tv/img/fig/goo.play.send_req_2015_02_01.png "Google API Send Request")

12. Click the blue **Send the request** button to obtain a successful (200 OK) response such as this:

```	
{
  "kind": "urlshortener#url", 
  "id": "http://goo.gl/p0eVli", 
  "longUrl": "http://www.wilsonmar.com/"
}
```

<hr />

> Notice authorization to API calls above were obtained from Google manually by [clicking the OFF switch to turn it ON](#Authorize_API)
or [clicking "Accept" Google managing short URLs](#Accept_API).

But when the user is not present to manually do that, 
Google would need something to prove that user acceptance was unreputably agreed to ahead of time.
That acceptance occurs by the user providing a private unique fingerprint key he/she generated
which the computer uses to "sign" requests made on the user's behalf.


# <a name="#Obtain_credentials"></a> Obtain Google API Credentials for automated API calls


## <a name="Keygen_dataflow"></a> Token Generation Dataflow

...   ![Token Generation Dataflow](http://www.merc.tv/img/fig/goo_dataflow_2015.02.04a.png "Token Generation Dataflow")

Let's recap. Instead of annonymous calls to goo.gl,
credentials or manual input of Google account passwords, 
if we want to generate and manage our own short URLs using Google's API,
We would login under our own Google account, and go into
**Google's Developer Console**
to define a specific **project**, and 
generate an **API key** for pasting into client-side JavaScript code.

That API Key is passed to the server along with the **longURL** to be shortened.
After we give the **shortURL** Google generates to people and they use it,
Google tracks those hits as analytics.
But Google considers analytics private information private to each user.

So a [**service account**](#Get_service_account) can be defined for a project 
so that custom **servers** can assemble requests for access to data of an owner.
Instead of passing on the user's secret password, when a user's service account is created
in the Developer Console, Google also generates and downloads a file of secrets called **.p12**.
BTW, the name p12 comes from the "PKCS" public standard number 12 on which the format of the file is based.

That standard defines very clever mathematics to create separate public and public keys such that
eliminates the need to transmit passwords which can be exposed to interception during transit.

A program written to the spec, **Openssl**, extracts a .pem file containing the private key
that the server provides to algorithms which generates the JWT token,
conveniently called "jot" perhaps because, internally, 
a dot separates each of the 3 parts of the token combined so the server can 
I say internally because these few steps are done inside a pre-defined library that takes care
of the underlying math I'll cover next.

The service account is combined with the Current Time and Expire Time of the token to get the JWT Body.
This is necessary because otherwise the token can be reused.
Google allows the server to specify how much time before a token expired, with a maximum of **30 minutes**.

The text is then encoded Base64 to make all characters unambious to send over the internet.
The private key is "signed" by passing the private key through an algorithm such asRSA SHA256,
which Google has specified.

The result is a duplicate set of **signatures** that is used **two** different ways.
One of the identical signatures is run through an unescape function to yield the **claim set**.

If all is well, Google's authentication server returns an **Assess Token** along with a **refresh** token.

The access token is reused until it expires.
Then the refresh token is used to authorize more access tokens.

Like most other major internet sites -- Amazon, Twitter, Facebook, LinkedIn, Yelp, etc. -- Google implements some form of the [OAuth 2.0 standard](http://tools.ietf.org/html/rfc6749) that defines the use of authorization tokens and access tokens. 
But web service uses a slightly different approach.

Now let's dive into the coding to implement the above.


## <a name="Get_service_account"></a> Get service account email for project

1.	Use an internet browser to bring up Google's Developer Console (also called API Console) at  https://console.developers.google.com/project

2.	Define a **project** Google uses project as a container for certificates related to a particular application.
Click on Create Project. Google suggests a PROJECT NAME such as "My Project 1" and a globally unique Project ID consisting of three sets of random words and numbers, such as "applied-algebra-825".

BEST PRACTICE IDEA: Specify a version number after each project name you define.

3.	Click the blue **Create** button.
4.	It may take a few seconds for this activity to complete. When the Project Dashboard appears, click the "X" Activities pop-up to dismiss it.
5.	The URL to reach the Project Dashboard for the project contains the Project ID, such as:
	https://console.developers.google.com/project/applied-algebra-825

This page can be reached again by clicking the APIs link within the APIs & auth menu at the left of the screen.

6.	Bookmark the URL in your browser for each specific API project.
7.	Click the blue **Enable an API** button for a list of Google services enabled. 
8.	Scroll down the list to click on the link to **URL Shortener API**. The response is:

...   ![Console authorize](http://www.merc.tv/img/fig/goo-console.authorize-api.png "Console authorize")

9.	As before, click to the left of the OFF switch to turn it ON.

...   ![Console authorize](http://www.merc.tv/img/fig/goo-console.accept.png "Console authorize")

> Notice there are additional [Terms and Conditions]() for this action is special not shown for other actions.

The API is moved to the top section of active APIs.

10.	Click the checkbox, then the blue Accept button.

...   ![Console accept form](http://www.merc.tv/img/fig/goo-console.quota.png "Console accept form")

11.	Click on the QUOTA tab.

> Notice there is a FREE QUOTA of 1,000,000 requests/day. Google needs authentication to determine who specifically are making calls so that such limits can be monitored and enforced.
Purchase additional capacity as Python API from https://cloud.google.com/appengine/pricing.

> Some have notice "403 rateLimitExceed" errors after 300 insert requests within an hour.
So this may be a show-stopper for you using this Google service.
Competing services include bit.ly, awe.sm, and others.
Twitter now has its own t.co service automatically applied to URLs in tweets.

12.	Click on the **Credentials** link within the API & auth section at the left menu.

**OAuth** is for accessing data. 

13.	Scroll down to the **Public API** access section.

The API Key is used to track Quota used.


## <a hre="Public_API_Access"></a> Public API Access 

The Public API access option generates an API Key used to perform anonymous URL shortener lookups because it is "not used for authentication". This is because the API Key is sent to Google in plain text, so it can be intercepted for reuse by another.

## Covert .p12 file to .pem format

method for sending small amounts of information over unsecure lines
signing not encryption

The technical standard is at 
https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32
http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html
http://openid.net/specs/draft-jones-json-web-token-07.html

A JSON Web Token is three separated by a dot. Thus the recommended pronouciation of "jot".
Debugger http://jwt.io/#


To address the issue at https://github.com/google/google-api-nodejs-client/issues/326
Conversion of .p12 keys to .pem keys is done by adopting the library at 
https://github.com/ryanseys/google-p12-pem
designed to be a command-line such as: gp12-pem myfile.p12 > output.pem




The statistics that Google provides by default displays data for only one line at a time.

What if we want to see the trend of hits for several URLs combined/overlaid on one graph?

Start by reading Google's [Tutorial: Hello Aanlytics API](https://developers.google.com/analytics/solutions/articles/hello-analytics-api)

[Google's Analytics Core Reporting API](https://developers.google.com/analytics/devguides/reporting/core/v3/)
https://github.com/rayshan/ga-extractor


## JWT vs. JWS vs. JWE

### <a name="JWT_definition"></a> JWT
JWS is an example of JWT defined in the IETF draft at https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32.

JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties.  The claims in a JWT are encoded as a JavaScript Object Notation (JSON) object that is used as the payload of a JSON Web Signature (JWS) structure or as the plaintext of a JSON Web Encryption (JWE) structure, enabling the claims to be digitally signed or MACed and/or encrypted.

### <a name="JWS_definition"></a> JWS
(https://github.com/brianloveswords/node-jws)
implements the creation of a
JSON Web Signature (JWS) as defined in http://self-issued.info/docs/draft-ietf-jose-json-web-signature.html
JSON Web Signature (JWS) represents content secured with digital signatures or Message Authentication Codes (MACs) using JavaScript Object Notation (JSON) based data structures. Cryptographic algorithms and identifiers for use with this specification are described in the separate JSON Web Algorithms (JWA) specification and an IANA registry defined by that specification. Related encryption capabilities are described in the separate JSON Web Encryption (JWE) specification.

### <a name="JWE_definition"></a> JWE
???

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

## <a name="from_lr_script"></a> Obtain short URL from C-language LoadRunner script

See github.com/wilsonmar/gapi-lr

<hr />

## <a name="Loopback_use"></a> Obtain short URL from a Node.js custom program 

an example of a node.js program running loopback.io 
calling Google APIs.

StrongLoop provides an API Explorer much like Google's API Explorer.
But Strongloop puts a nice color key for the various methods.
More importantly, custom methods are shown in the same explorer.

The Set Access Token at the upper right corner establishes
credentials for programs to create authorization tokens.

Loopback has a office supplies sample app

Access tokens are defined

user id from data 

Loopback

The JWT is calculated
within modules > strongloop > node_modules > loopback > lib > models > 
	access-token.js, acl.js
	access-context.js

	user.js line 127 has accessTokens class create method to createAccessToken using auth token inside userModel
	
user.login instantiates the accessToken
	
middleware > token.js
	
trace through the chain of calls

callbacks jump to functions defined in various libraries


# <a name="Why_capture_data"></a> Why Capture Data from Google?

[While viewing scroll of API Explorer from top to bottom]
...![Google API Explorer sample](http://www.merc.tv/img/fig/google.explorer_list_v01.png "Google API Explorer sample")

Why would one want to take the extra effort to extract data from Google when one has Gmail, Google maps, and other websites that Google provides? 

Let's take a look at this line chart showing corresponding data series across time.

...   ![Data series from several sources](http://www.merc.tv/img/fig/goo.combined_data_v01.png "Data series")

If you want to add event flags or additional data series **not** in Google servers you might need to have all the data on your **own** server. Google can automatically purge data on its severs anytime it wants. And Google has cancelled many services it has provided. So you need a way of keeping your data where **you** can really control.


# <a name="Using_analytics"></a> Making Use of Analytics from Google

Below is a description of the default response from Google when ".info" is added as a suffix to the short URL.

### <a name="Click_history_periods"></a> Click History Periods

The default download of data consists of several groupings of time:

Clicks for the past: two hours | day | week | month | all time

This is so that viewers have the information readily available for quick display upon click.

FEATURE REQUEST: 
Is there a "star schema" database behind the scenes so that we can see?


### <a name="Countries_desc"></a> Country Id Descriptions
A description lookup file is needed for the value of id's in the countries section.

```
 countries: [
            {
               count: "10903",
               id: "US"
            },
            {
               count: "1072",
               id: "CA"
            },
```

### <a name="OS_metrics_version"></a> Operating System Metrics No Version

FEATURE REQUEST: 
In platforms, it would be good to have the version of each operating system.
Such information is available from browsers.



# <a name="Tasks_here"></a> Tasks To Build This Repo
- [ ] Objective 2 - Explore libraries (done first to not waste time on objective 1)
	Decision to use https://github.com/google/google-api-javascript-client with documentation at
	https://cloud.google.com/compute/docs/tutorials/javascript-guide#authorization

- [ ] Objective 1 - Call Google API
  - [x] Create github repo (Wilson)
  - [x] Create github.io/gapi-node (wilson)
  - [x] Provide API key (Wilson)
  - [ ] Code running skeleton client form and node.js express with no formatting, no retrieval of QR code (Abdul)
  - [ ] Describe loopback skeleton (Wilson)
  - [ ] Add test code (Wilson)

  - [ ] Add library to call Google API (Abdul)
  - [ ] Add calling code in node server code (Abdul)
  - [ ] Code client QR code retrieval and display client side (Abdul)

  - [ ] Transfer server code to Heroku?
  - [ ] ~~Store credentials in MongoDB~~

- [ ] Objective 3 - Explain JWT
- [ ] Objective 4 - Reports API calls


