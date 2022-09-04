# Authentication

## Summary

* [General Recon For Authentication Vulnerabilities](#recon)
* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)

## Resources

* https://portswigger.net/web-security/authentication
* https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
* https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/04-Authentication_Testing/README
* https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/

## Recon

* Walk-through the application and identify any processes that pertain to user identification ( e.g. login, registration, reset/forgot password, etc. )

* Analyze the source code of the application and identify any files or comments or any other resources that may leak information that helps identify valid users.

* Use enumeration tools to discover more content such as hidden directories or files which may leak information about valid users or other authentication related processes. (Tools = Burp Pro’s (Discover Content), gobuster, ffuf , etc.)

* Determine if there is a consistent account name/email structure ( John Doe = jdoe@test.com ) this can help to identify potential users of the application.

* All this information can be helpful when performing authentication related testing, such as the cases described in the labs here.


---
---

## Cheat Sheet

### Verbose Messages:  
* Submit invalid credentials to an authentication related function and analyze the response.  Identify if there is a way to enumerate valid credentials based off any error messages the application is returning.  

* For example, if the username is incorrect the application returns the error message “Invalid username”, however, when the password is incorrect the application returns the error message “Invalid password”.


### Verbose Messages:  
* Similar to the previous point, the responses may have a very minor difference that can be difficult to identify just by looking at them.  We can use Burp Intruder’s “Grep-Match” function to see which response does not contain the known message.  

* For example, when the username is incorrect the message will be “**Invalid username or password.**”, but when the password is incorrect the message changes to “**Invalid username or password**”. (The period . at the end is missing in the 2nd message)


### Misconfiguration Bypass:  
* Sometimes the application will not validate the password field unless the submitted username exists on the application.  

* We can take advantage of this logic and submit requests that contain a very long password, and brute force for valid usernames.  The valid username should have a notable difference in the time taken to receive the response.


### Brute Force Protection Bypass:  
* The application may be blocking our requests after a certain number of invalid requests that are submitted.  We can potentially bypass this restriction by using the **X-Forwarded-For** header in our requests to spoof our IP address.


### Brute Force Protection Bypass:  
* The application may be blocking our requests after a certain number of invalid requests that are submitted.  

* We can try submitting valid credentials before that limit is reached and see if we can bypass this protection.  If this works, we need to include the valid credentials in our brute force requests to prevent from being blocked and bypass this logic.


### Logic Flaw Bypass:  
* The application may be blocking our authentication requests after a certain number of invalid requests are submitted.  However, sometimes the application will only lock out accounts that actually exist.  

* We can use this logic to gather valid accounts on the application, if our requests get blocked or similar, the username may be a valid one.


### Logic Flaw Bypass:  
* It may be possible to submit multiple passwords with just one HTTP request by using an Array of Strings -  [“test123”, “password”, …].


### 2FA Bypass:  
* When directed to a page where the 2FA code needs to be submitted, force browse to another page.  This may bypass the 2FA code requirement.


### 2FA Bypass:  
* Strictly analyze the authentication process and identify if there are any parameters/headers/endpoints that are explicitly used to determine which user the 2FA code will be created for.  If we can tamper these values, we may be able to log into the application as another user without needing to know their authentication credentials.  

* For example, after logging into the application with valid credentials, there is a request that is submitted to the application which contains the following Cookie –> Cookie: verify=user1.  Change that cookie value to another user.  

* Then when submitting the request when entering the 2FA code include/change the same cookie value and either brute force the 2FA code or include the code you obtained and see if this bypasses any logic flaws.


### 2FA Bypass:  
* If the application is terminating our valid session cookie after a certain amount of incorrect 2FA code attempts, then we can use a Burp Macro to create a new session after every request that is sent through Burp’s Proxy, Repeater, Intruder, etc.  

* Check out the following lab for an example – “Lab: 2FA bypass using a brute-force attack”   -   *pending quick write up on Burp Macro to include on here*


### Brute Force Protection Bypass / Insecure Design:  
* Use all the application’s functionality that is available.  If the application has a “Remember me” feature after logging in, use it and identify how the application is implementing this feature.  If this feature is insecure, we can exploit it to potentially log into the application as another user or enumerate valid accounts.  

* For example, after clicking on the “Remember me” functionality, the application sets a new cookie on the client.  The cookie is generated by hashing the user’s password with an insecure hashing algorithm with no salt and appending their username to it.  If there aren’t any brute force protections on this Cookie, we can use this vector to brute force the credentials of another user.


### Authentication Bypass:  
* Use all of the applications functionality related to authentication such as Forgot password, Reset password, Update password, etc.  Analyze the entire processes and identify any interesting headers/parameters that are potentially being used to identify which user the requests are being initiated for.  Change these values to another account and analyze how the application responds.  

* For example, when using the “Forgot Password” functionality, there is a body parameter called username.  It may be possible to update the password of any valid user on the application by simply changing the value of this parameter.


### Authentication Bypass:  
* When using the “Forgot password” functionality on an application, identify if it is possible to manipulate the domain used in the password reset link.  We can use headers such as (Host, X-Forwarded-Host, or X-Host) to do this.  If this is possible then we may be able to steal another user’s reset token to update their password.  

* For example, if we can manipulate the password reset link to be https://ATTACKER-SERVER.COM/reset?token=xxx, when the user clicks this link that they receive in their email, the request will be sent to the attacker's server and the query parameter will be logged.  Now we can use this token in the correct application domain to reset the victim user's password. 

* Check out the following lab for an example - Lab: Password reset poisoning via middleware


### Verbose Messages:  
* The login page is not the only place that can contain overly verbose messages indicating that a username or password is correct/incorrect.  If the application has a change password functionality, we can potentially use this function to brute force the correct password of another user by using the application’s verbose responses.

* For example, in a change password function, supplying different combinations for correct/incorrect values in the currentPassword, newPassword, confirmNewPassword parameters, may result in a method to enumerate user credentials.
