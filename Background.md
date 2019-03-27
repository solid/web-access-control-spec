Notes

# Authorizing Web Apps: Background
This short article is a very much simplified backgrounder on the protocols
which web browser manufacturers have implemented to control access to data by web apps on different web sites.

## Background

*Ted Nelson talks about the two gods of Literature, the reader and the writer. They are each invincible: the writer can write whatever they want, and the reader can if they chose read nothing the writer has written. Here we have though three gods, the reader, the writer and the spy, and none of them are omnipotent.*

The assumption that a web browser makes is that whenever the user is using a web app, that the web app is trying to attack the user. The web app is by default not trusted, and so the browser constrains the web app to be limited in anything it can do, and also provides the web app with limited error messages or information about the way it is being disabled. This is therefore a very hostile environment for which to program, unless the app is something rather trivial which does not access data on the web.

## Cross site scripting attacks

The original attack which caused a lot of the current design of the browser is a cross-site scripting attack. In this sort of attack, a malicious person Mallory sends a normal user Alice for example an email with a link to Mallory’ web site. Alice follows the link and loads the web page M. We allow JavaScript scripts to run in any web page by default, and M has a script which runs on Alice’s machine, accesses some data which Alice has access to, and then secretly sends the data back to some system Mallory controls. The private data maybe accessed by the script in various ways. One way is that the computer Alice is using may be inside a firewall which gives her implicit access, to say a webcam in her house, or a journal library in her university. It may be done, then, without any interaction with Alice, or it can also be done by asking her to log in to a system to get the script some credentials. Web site M could be a fake version of her bank, B, and ask her to authenticate to the bank, or to her social network, on some pretense. The data can typically be returned to Mallory say be encoding it in a URL on a site M controls and going a GET. There are many variations of the Cross Site Scripting (XSS) attack, but that is the general idea.

## Cross-Origin Resource Sharing "CORS"

The first impulse of a browser developer may have been to just prevent a web page script to do any web access at all, but clearly they need to be able to do network access. For example, a bank needs to load a script which can display a person's
bank accounts by going back to the server B for more data about different accounts at different dates. Clearly that had to be enabled. But Mallory had to be blocked. This let to the Same Origin Policy that so long as a script running in a web page which came from the same internet domain name (like www.bankofamerica.com) then the script could interact as much as it liked with any server address in the same domain. The protocol and domain part of the URI aRe known as the Origin. Hence, Same Origin Policy (SOP). In general all data from a server Origin and form any script it runs is kept separate from any data from any other Origin. This allows banks to work pretty well.

Were there any use cases where the SOP does now work? Well, yes — anything which relies on a script being able to access other domains. One example is a service which provides for example a JavaScript program to validate, test, or assess another web page: it can’t access the data in the page to do its job.
Another example is a data mashup. When a large amount of open public data started to become available from governments and so on, there was a flourishing of sites which loaded data from many different sites and provide a “mashup” of the data — a combined visualization of data from many sources which would typically l	was to insights that you would not get from one data source alone. A typical client-based data mashup site in fact does not work nowadays work any more.

What could be done? The browser manufacturers implemented some hooks to allow data to be shared across different origins, and called it Cross Origin Resource Sharing, or CORS. The core problem was — how in the browser to distinguish between data like a domestic webcam which was private, and open government data which was public? Not being able to change the webcam, they decided to make the data publishers change. They required that they add special CORS headers to their HTTP responses for any data which was public.

```
Access-control-allow-Origin: *
```
At the same time they added a feature to allow the data publisher to specify other a limited set of other origins which would be allowed access. This makes running a bank easier if also the credit card company code can access your customers data.
```
Access-control-allow-Origin: credit card company.example.com
```
This meant that anyone publishing public data has to add

```
Access-control-allow-Origin: *
```
in any response.  This meant a huge amount of work for random open data publishers
all over the web, an effort which in many cases for many reasonable reasons was not done, leaving the data available to web apps, but unavailable to browsers.

The browser actually looks for these headers not on the request itself, but in
on a "Pre-flight" OPTIONS request which is inserted before the main request.  So while the developer may see in the browser console only the main request, the number of round trips has in fact increased.

### Header blocking

As well as blocking the data, the CORS system blocks headers from the server to the web app.
To prevent this this, the server must send another  [header](https://www.w3.org/TR/cors/#access-control-allow-headers-response-header):
```
Access-Control-Allow-Headers: Accept, Authorization, User, Location, Link, Vary, Last-Modified, ETag, Accept-Patch, Accept-Post, Updates-Via, Allow, WAC-Allow, Content-Length, WWW-Authenticate
```
This must include things like the Link: header which are normal headers blocked by the browser, and also any new headers the app and server are using for any purpose.

### Method blocking

### Example

One solid server does CORS [this way](https://github.com/solid/node-solid-server/blob/master/lib/create-app.js#L26)

## The CORS twist

The twist is that in fact the designers of CORS make it even more difficult.

There was a feeling, I understand, that allowing the publishers to simply put `ACAO: *`
header on their stuff was too easy.  It was felt it was a "footgun", something it would be
too easy for users to shoot themselves in the foot with.
The "users" here are of course not ordinary users, they are system managers configuring
web servers.  The concern was that system managers would find that access to their data
was being blocked, and they would just throw on such a  header everywhere, even where the data
was in fact not public. Where, for example, they provided different versions of a page to
different users, it was important to preserve the wall between the users, and they would be tempted
to mak it with  `ACAO: *` which would mean it would be accessible

So they blocked the use  `ACAO: *` whenever the incoming request carries credential information.
Whenever the user is "logged in", if you like, and using their logged in identity.

But of course there are times when a system needs to access private user-specific information
from another site, just for the works of a bank.
For the case that the client is using credentials, *the access would only be allowed
if the Access-Control-Allow-Origin header specified explicitly the origin of the request exactly*.

The trouble is that the HTTPS: world is now divided into two webs, one in which credentials are passed, on in which they are not.  A problem with that is that if you are not an end user top level application, but in fact
some piece of middleware, a library within a code library, you just call the browser to do the web
operation, and the browser used to do password, or TLS, login if necessary.  The middleware
code is not aware


This meant that if you had a server with completely public data, like open government data,
which you wanted any code to be able to access, the rule of thumb among system managers
was that you should always echo back to any origin the same origin header. Instead of

```
Access-control-allow-origin: *
```
all public data servers should send

```
Access-control-allow-origin: $(RECEIVED_ORIGIN)
```
Where  `$(RECEIVED_ORIGIN)` is replaced with the actual value of the Origin header
on the request.

To the extent that adding a fixed field to all public data servers was a pain,
this was more of a pain.  It was a step up, requiring active code rather than a simple configuration change -- A lot to ask of every publisher of open data on the net.

 There was a campaign to explain this to data publishers, and code snippets were widely distributed.  In fact there is with Apache a way of doing this using
 internal environment variables. [@@ref]

Would not a better design have been to set up a static header such as
 ```
 Access-control-allow-origin: PUBLIC_AND_UNCUSTOMIZED
 ```
where the system manager would have no one else to blame for sing it on any resource
which was secret, or was customized with a user's identity?  One might of thought.
But that is not how CORS was done.

So, data publishers of the world went about putting CORS Origin reflector code
in their servers.

But once you are using the origin reflector technique, it becomes essential to allow add the `Origin` to
the `Vary:` header is you have one, or, if not add a new
```
Vary: Origin
```
to every outgoing response which has a reflected ACAO header.
Otherwise, this is the failure mode:

- A script on site A asks for the public data
- The server responds with the data with ACAO: A header
- The browser caches that response
- The user uses a different app on site B to look at the same data
- The browser uses the cached copy -- but the origin A on it does not match the requesting site B.
- The browser silently blocks the the request to the puzzlement of user and developer.

So in a properly running CORS-based system, the server sends the Vary: Origin header, and forces the browser to keep a different copy of the data for every web app which asks for it, which is very ironic, given that it may be completely public data.

The CORS design most go down in history as the worst piece of design in the whole web. But that is the background against which now those designing a system such as Solid now have to create a system which will be immune to XSS attacks, data will be
public and private under user's control, and web apps are be deemed trustworthy by various different social processes.

## Epilog: A second CORS twist

The second twist with CORS is the at the browser doesn't even actually implement the CORS with a twist
algorithm completely.

https://lists.w3.org/Archives/Public/www-archive/2017Aug/0003.html

Notwithstanding issues with the design of CORS, Chrome doesn’t in fact
do it properly.

If you request the same resource first from one origin and then from
another, it serves the cached version, which then fails cord because the
 Origin and access-control-allow-origin
headers don’t match.  This even when the returned headers have “Vary:
Origin”, which should prevent that same cached version being reused for
a different origin.
That was with Chrome Version 59.0.3071.115 (Official Build) (64-bit)

It seems also that Firefox showed the same behavior for  in 2018-07

## References

- [WXSS] [Wikipedia, "Cross-site scripting"](https://en.wikipedia.org/wiki/Cross-site_scripting)
- [CORS] [Cross-Origin Resource Sharing
W3C Recommendation](https://www.w3.org/TR/cors/)  16 January 2014
- [WCORS][Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)
- [WSOP] [Wikipedia, "
Same-origin policy"](https://en.wikipedia.org/wiki/Same-origin_policy)
- [W3C-SOP][W3C Wiki, Same Origin Policy](https://www.w3.org/Security/wiki/Same_Origin_Policy)
