# Access Control - Portswigger

## Summary
* [General Recon For Access Control Vulnerabilities](#recon)
* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)

## Recon

* Map out the application and review the results of the mapping exercises
* We need to understand what are the application’s requirements for Access Controls  
    * Are there different levels of users in the application?
    * Are users only given access to a subset of data belonging to them?
    * Is there administrative functionality that can be accessed through the application?
    * Are there any identifiers that may be used to determine a user’s access? (Example: ?admin=yes)


* If there is only 1 type of User Role on the application.  Then we may not encounter Vertical Privilege Escalation, however, we may still discover Horizontal Privilege Escalation.  

* For example, a query parameter is used to determine which address to send your purchased items to ( ?addressId=7232 ).  The address is displayed on the screen for confirmation.  Is it possible to view other user’s addresses by changing the value of the parameter?



### Testing with Different User Account:
    * If you have access to a normal user & admin user.  Walk through the entire functionality that the higher privileged user can access, then attempt to access that functionality with the lower privileged user.  This requires strict comparison, functionality that both users should have access to is not Vertical Privilege Escalation.  We can use Burp’s “compare site maps” feature to help.

    * If you have access to 2 accounts of the same privilege level, identify if Horizontal Privilege Escalation is possible.

### Testing Multistage Processes:
    * Burp’s “compare site maps” feature will not work here, because the requests sent may be out of order from the standard process.  This can lead to false positives/negatives.

    * A way to perform this testing is to walk through a protected multistage process and user Burp to access each one of those requests by a lower privileged user.  Sometimes the initial request in the process is protected but the subsequent requests are not protected from unauthorized access.

### Testing with Limited Access:
    * If you only have 1 account to test with or no accounts at all, we can map out the application to identify any hidden sensitive/protected functionality.
    * When pages are identified that may return different data depending on the user, try adding parameters/cookies such as – admin=true, debug=true, etc.
    * Identify functionality where the application grants a user access to a subset of wider resources, such as emails, orders, documents, etc.  If these resources are retrieved through some predictable identifiers ( ?order=1234 ) try to determine values that reference other resources, we should not have access too. <br>

### Testing Restrictions on HTTP Methods:
    * An application’s access controls may be bypassed by platform level controls.
    * If there is a protected functionality that only a higher-level privilege user can access, test whether this functionality can be accessed with a different HTTP method.  Then determine if a lower-level privilege user can bypass access controls using this method.

---
---
<br>

## Cheat Sheet

### ROBOTS
* Go to the /robots.txt page on the application and see if it reveals any sensitive locations on the application.  We can potentially use these disclosed endpoints to perform actions that we should not be authorized to do.

### Source Code Leak
* Look through the source code of every page, there may be a client-side script or comments that discloses sensitive functionality or data.

### Sensitive Cookie
* Identify if there are clients-side Cookies that are being used to enforce your level of access on the application.  Change the value and analyze how the application responds. For example, a Cookie named Admin=false, change it to Admin=true.

### Mass Assignment
* Identify if the application is vulnerable to a Mass Assignment vulnerability.  For example, when updating your email address on the application, the response discloses a critical parameter called “roleID”.  Submit another request to update your email address, but this time also include the parameter “roleID” with a different value.  Analyze how the application responds, maybe this can bypass some access controls.

### HTTP Header Bypass
* Use non-standard headers to potentially bypass access control restrictions on endpoints.  For example, the X-Original-URL header can be used.

### HTTP Method Bypass
* Use different HTTP methods when requesting a resource.  This may bypass the access controls implemented on the endpoint.  For example, use GET, PUT instead of POST.

### IDOR
* IDOR vulnerability allowed us to specify a different resource belonging to another user by manipulating a query parameter.  For example, ?id=user123, change it to ?id=user456.

### IDOR
* IDOR vulnerability allowed us to view another user's account information.  This time the parameter value was unpredictable, however, there was some functionality on the application which exposed other user’s GUID value.  Which can be used to view their account information in an endpoint like ?id=xxxx-xxxx…

### Redirect Leakage
* When submitting an invalid account on the following endpoint /my-account?id= we receive a 302 HTTP response with no body.  If we submit a valid account on the endpoint, we still receive a 302 HTTP response, but the response body is disclosing sensitive information for this user.

### IDOR
* If there is a request similar to this one /download/2.txt, change the file to /download/1.txt, this may disclose valuable information that may belong to other users on the application.

### Multi-step Bypass
* When testing a multi-step process with an admin account, test to see if a lower privilege user can bypass the access controls on any of those steps individually.  If it is a 3-step process, the first 2 steps may be properly protected, but the last step may be left unsecured.

### Referer Header Bypass
* The Referer header may be used to prevent unauthorized access to certain endpoints on the application.  If we can figure out what the required value is, we may be able to bypass this restriction.