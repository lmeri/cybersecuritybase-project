# Cyber Security Base – Course Project I

The application's repo can be found [here](https://github.com/lmeri/cybersecuritybase-project) and it is based on provided starter code. The project currently has (at least) five different security flaws from the 2013 OWASP Top 10 -list. 

This application gives logged in users functionality to sign up to an event. You can also see names of others who have signed up to that event. 
The application does not currently support registering new user accounts.

Existing test accounts are: 
*ted/ted123
*ned/ned123
*fred/fred123
*zed/zed123

## Getting started: running the application

This is a simple spring boot application, and can be run like any other application in this course. Project can be imported to NetBeans (or other similar IDE) and then build and run. 

Project can also be run from terminal by navigating to project repository and executing:
mvn spring-boot:run
However, prerequisite for running the project that way is having Maven installed.

Afterwards the application can be accessed by typing localhost:8080 into browser's address bar. 

–
## Security vulnerabilities

### Cross-Site Request Forgery / CSRF (A8)
#### Issue: 
Currently users can be fooled into visiting/loading malicious websites containing code that sends a request to the application. If the user is authenticated, then that request could, for example, be used to create a signup entry without the user's consent.

#### Steps to reproduce:
1. In the github repository there should be file called example-csrf.html. Make sure you have either downloaded that file, or created a new .html one with the same content.
1. Open localhost:8080/ in the browser
2. If not logged in, type ”ted” (without quotes) in the User field and ”ted123” (without quotes) in the Password field.
3. After you have been redirected to the “/form” page, open the example-csrf.html file in browser.
4. You should be automatically redirected to the “/done” page where the signup created by CSRF request can be seen. If you are not automatically redirected, just type in “localhost:8080/done” (without quotes) to browser’s address bar and hit enter.

#### How to Fix?
1. Go to src/main/java/sec/project/config/ and open file SecurityConfiguration.java
2. From method configure() remove or comment out 
```http.csrf().disable();```

–

### Cross-Site Scripting / XSS (A3)
#### Issue: 
Cross-Site Scripting vulnerability occurs whenever a user can feed questionable data to the application without proper validation. Currently it is possible to pass malicious content to this application by entering it to the form’s email field. It could then be executed by other users with unpleasant consequences.

#### Steps to reproduce:
1. Open localhost:8080/ in the browser
2. If not logged in, type ”ted” (without quotes) to the User field and ”ted123” (without quotes) to the Password field.
3. On the page ”/form” in the Email field type in
```<script>alert("XSS");</script>```
4. Click Submit. Page redirects to ”/done”.
5. An alert pop-up that reads ”XSS” appears. This could be used to pass malicious content to a site that could then be executed by other users.

#### How to Fix?
1. Go to src/main/recources/templates
2. Open file done.html
3. Change all instances ”th:utext” to ”th:text”. If text is defined as escaped, the script can't be executed.

–

### Missing Function Level Access Control (A7)
#### Issue: 
Missing Function Level Access Control means that one or more of the application’s functionalities can be used without proped authentication. E.g. currently it is possible for guests (not logged in) to see the details of people who have signed up the the event, and even delete their signups. This is a serious security flaw, since possibly sensitive data should be protected from outsiders. This is also a reference to Sensitive Data Exposure vulnerability (A6).

#### Steps to reproduce:
1. Open localhost:8080/ in the browser
2. If not logged in, type ”ted” (without quotes) in the User field and ”ted123” (without quotes) in the Password field.
3. On the ”/form” page fill in anything you want in the field Email. For example ”testi@testi.fi”.
4. Click Submit.
5. On the ”/done” page you should see at least the signup you just submitted. Now click logout.
6. Next type ”localhost:8080/done” (without quotes) into the browser’s address bar. You should be able to see the signup created by ted.
7. Guests can even delete users’ signups, just by entering ”http://localhost:8080/done/clear/id” (without quotes and id replaced with signup’s id) into the browser’s address bar.

#### How to Fix?
1. Go to src/main/java/sec/project/config/ and open file SecurityConfiguration.java
2. Find method configure().
3. Replace the line that reads
```
.anyRequest().permitAll();
```
With
```
.anyRequest().authenticated();
```

–

### Insecure Direct Object Reference (A4)
#### Issue: 
Insecure Direct Object Reference flaw means that it is possible to access data in the application that should not be available to the user, for example via a path variable. Currently the application allows users to delete any signups, even from other users, just by changing the URL path variables.

#### Steps to reproduce:
1. Open localhost:8080/ in the browser
2. If not logged in, type ”ted” (without quotes) in the User field and ”ted123” (without quotes) in the Password field.
3. On the ”/form” page fill in anything you want in the field Email. For example ”testi@testi.fi”.
4. Click Submit.
5. On the ”/done” page you should see at least the signup you just submitted. Now click logout.
6. Type ”ned” (without quotes) in the User field and ”ned123” (without quotes) in the Password field.
7. On the ”/form” page fill in anything you want in the field Email. For example ”testi2@testi.fi”.
8. Click Submit.
9. You can now see both signups. In the browser’s address bar write ”http://localhost:8080/clear/id” (without quotes and replace id with ted’s signup’s id) and click Enter.
10. Now the signup was deleted. This means that normal users can delete other users’ signups, even though that functionality should require authentication of the user that submitted the signup in question.

#### How to Fix?
1. Go to src/main/java/sec/project/controller/ and open file SignupController.java
2. Find method submitDelete().
3. Replace the line that reads
```
signupRepository.delete(id);
```
With
```
if(signupRepository.findById(id).getName().equals(accountRepository.findByUsername(auth.getName()).getUsername())) { 
signupRepository.delete(id);
}
```

–

### Security Misconfiguration (A5)
#### Issue: 
The application has a security misconfiguration vulnerability regarding h2-console. H2-console lets you access SQL database with browser. It is important to know and control those who have access to the database server, so while it might be useful to have h2-console connection enabled in development, it is a serious security risk when the application is in production.

#### Steps to reproduce:
1. Open localhost:8080/h2-console/ in the browser
2. H2-console should now appear, even without logging in.

#### How to Fix?
1. Navigate to src/main/java/sec/project/config/ and open file SecurityConfiguration.java
2. After 
```
http.authorizeRequests()
``` 
add line
```
.antMatchers("/h2-console/*").denyAll()
```
This line of code prevents anyone, even authorized users, from accessing h2-console.
