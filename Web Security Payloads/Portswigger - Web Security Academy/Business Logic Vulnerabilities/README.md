# Business Logic Vulnerabilities

## Summary
* [General Recon for Business Logic Vulnerabilities](#recon)
* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)

## Recon

* Logic flaw vulnerabilities are different than other vulnerabilities such as SQL Injection, there isn’t a common signature to identify Logic Flaws, as there is for SQLi. 

* It requires walking through the entire application and all it’s functionalities and determine if there is a defect in the logic that was implemented in the application.

* Generally, a programmer may have reasoned, “If A happens, then B must be the case, so I will do C.” But there wasn’t any consideration for, “But what if X occurs?”.

* There are 12 different Logic Flaw examples on the Web Hacker’s Handbook 2, that shows some real-world examples where logic flaws can happen.  An overview of some of those examples will be below.


### Logic Flaw Examples / Concepts

* Remove parameters/cookies/query strings in each of the requests one at a time and analyze the responses.  This ensures that all relevant code paths within the application are reached.

* In multistage processes attempt to “force browse” and submit requests in different orders and analyze the responses.  Think of the assumptions that may have been made.

* In a funds transfer application for example, try submitting negative values or very large values.  Analyze the affects it has on the application.

* In any situation where sensitive values are adjusted based on user controllable criteria, analyze if this is a one-time process or if these values change based on further actions by the user. 
    * (Example: Getting a discount after a minimum total is reached, then removing items to lower that amount, while still using the discount)

* Sometimes an escape character is used to escape malicious characters to protect against certain vulnerabilities such as command injection.  We can inject our own escape character \ to neutralize the escape character that is used for defense.  (For example, injecting the following payload ( ; ls ) results in ( \; ls ).  If we inject the following payload ( \; ls ) then the application will still insert an escape character, but it will be neutralized ( \\; ls ).

* Identify any instances where the application either truncates, strips out, encodes or decodes user supplied data.  Determine if malicious strings can be derived.

* Identify cases where the application is storing information in a static manner as opposed to per-thread/per-session based.
    * (Examples:  There is an endpoint that holds error messages that contain user info (not session based), so a user can potentially see details for another user.  Race conditions in login functionalities, where static values are used to determine the user in the backend, so its possible for a user to see another user’s data upon logging in.)


---
---

## Cheat Sheet

* Use a Web Proxy to bypass any client-side restrictions implemented on the application.  We can change the values for critical parameters and potentially break the logic of the application.  
    * For example, changing the price of an item on our shopping cart to an arbitrary value.  Another example, change the quantity value of an item purchased to a negative number, which may bring the total price of our shopping cart order down.


* Manipulate a numeric input field so that its value reaches a very large number.  Analyze how the application responds, maybe there is a limit and once it is reached, it may be reverted back to zero or a negative number.  Depending on the context, this logic flaw can be very critical.


* Similar to the previous point, by submitting a very large to an input field, the application may truncate the value to a certain character limit.  Depending on the context this can be used to bypass some restrictions on the application. 
    * For example, registered email gets truncated to something@adminUser.com


* After registering an account on an application.  Identify if there are any “Update email” functionality available.  Use this functionality and verify if the application requires verification on the new email address specified before fully updating our email address.  If it does not, then we can update our email to an arbitrary value and potential bypass some access controls.


* Remove parameters completely from requests and analyze how the application responds.  This can potentially bypass some restrictions or logic that the application is using.  
    * For example, in a “change password” functionality, remove the “current-password” parameter if there is one.


* When going through a workflow/functionality, skip a step and see how the application responds.  We may be able to bypass a critical step in the process.  
    * For example, in a “Cart checkout” workflow, skip the checkout step and go straight to the “order confirmation” step. Another example, after logging into an application you must select a “role”.  Drop all of the requests after logging into the application and analyze how the application responds, we may be able to bypass some access control related functions, since our “role” was never selected.  This is essentially a “force browse” bypass.


* If the application has 2 Coupons that can be used to get a discount on an order, but only allows to use each one of those once per order, try submitting them one after another and analyze how the application responds.  This may bypass some flawed logic on the application.


* If the application offers a coupon code that can be used when submitting an order, check if this coupon can be used an infinite number of times once per order.  We can purchase a gift card and use the coupon code when purchasing it, and when redeeming the gift card, we earn a profit.  An “Infinite money logic flaw” can be exploited here.

