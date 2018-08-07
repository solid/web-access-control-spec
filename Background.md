Notes

# Authorizing Web Apps: Discussion

## Background

The assumption that the browser makes is that whenever the user is using a web app, that the web app is trying to attack the user. The web app is by default not trusted, and so the browser constrains the web app to be limited in anything it can do, and also provides the web app with limited error messages or information about the way it is being disabled. This is therefore a very hostile environment for which to program, unless the app is something rather trivial which does not access data on the web.

## Cross site scripting attacks

The original attack which caused a lot of the current design of the browser is a cross-site scripting attack. In this attack, a malicious person Mallory sends a normal user Alice for example an email with a link to Mallory’ web site. Alice follows the link and loads the web page M. We allow JavaScript scripts to run in any web page by default, and M has a script which runs on Alice’s machine, accesses some data which Alice has access to,and then secretly sends the data back to some system Mallory controls. The private data maybe accessed by the script in various ways. One way is that the computer Alice is using may be inside a firewall which gives her implicit access, to say a webcam in her house, or a journal library in her university. It may be done, then, without any interaction with Alice, or it can also be done by asking her to log in to a system to get the script some credentials. Web site M could be a fake version of her bank, B, and ask her to authenticate to the bank, or to her social network, on some pretense. The data can typically be returned to Mallory say be encoding it in a URL on a site M controls and going a GET. There are many variations of the Cross Site Scripting (XSS) attack, but that is the general idea.

## CORS

The first impulse of a browser developer may have been to just prevent a web page script to do any web access at all, but clearly they need to be able to do network access. For example, a bank needs to load a script which can display a persons. Bank accounts by going back to the server B for more data about different accounts at different dates. Clearly that had to be enabled. But Mallory had to be blocked. This let to the Same Origin Policy that so long as a script running in a web page which came from the same intrenet domain name (like www.bankofamerica.com) then the script could interact as much as it liked with any server address in the same domain. The protocol and domain part of the URI aRe known as the Origin. Hence, Same Origin Policy (SOP). In general all data from a server Origin and form any script it runs is kept separate from any data from any other Origin. This allows banks to work pretty well.

Were there any use cases where the SOP does now work? Well, yes — anything which relies on a script being able to access other domains. One example is a service which provides for example a JavaScript program to validate, test,or assess another web page: it can’t access the data in the page to do its job.
Another example is a data mashup. When a large amount of open public data started to become available from governments and so on, there was a flourishing of sites which loaded data from many different sites and provide a “mashup” of the data — a combined visualization of data from many sources which would typically l	was to insights that you would not get from one data source alone. A typical client-based data mashup site in fact does not work nowadays

What could be done? The browser manufacturers implemented some hooks to allow data to be shared across different origins, and called it Cross Origin Resource Sharing, or CORS. The core problem was — how in the browser to distinguish between data like a domestic webcam which was private, and open government data which was public? Not being able to change the webcam, they decided to make the data publishers change. They required that they add special CORS headers to their HTTP responses for any data which was public.

Access-control-allow-Origin: *

At the same time they added a feature to allow the data publisher to specify other a limited set of other origins which would be allowed access. This makes running a bank easier if also the credit card company code can access your customers data.

Access-control-allow-Origin: credit card company.example.com

This meant that anyone publishing data has to add
@@TBC

## The CORS twist

## CORS on a solid server

## Adding trusted web apps



Ted Nelson talks about the two gods of Literature, the reader and the writer. They are each invincible: the writer can write whatever they want, and the reader can if they chose read nothing the writer has written. Here we have though three gods, the reader, the writer and the spy, and none of them are omnipotent.
