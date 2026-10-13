1. ==**Access Control theory and labs**==  
# Lab: User ID controlled by request parameter
- was supposed to get the API key of user CARLOS through the user WIENER (the credentials were provided)
- After logging in as WIENER , API key was provided of WIENER 
- Vulnerability found in this lab is Horizontal Privilege Escalation , and was able to get API keys of user CARLOS by changing the URL /my-account?id=wiener to my-account?id=carlos 
# Lab: User ID controlled by request parameter, with unpredictable user IDs
 - Same objective as the above lab , except on this scenario GUIDs were used instead of actual usernames or guessable userIds
 - Logged in as WIENER , couldnt find anything helpful to this case , navigated through the provided blogs , found the blog posted by CARLOS
 - Identified the unique GUID through burp , logged in as WIENER, replaced the WEINER's GUID with CARLOS's GUID , was successfully able to log in as CARLOS and obtain his API keys
 - Vulnerability : Horizontal Privilege Escalation

# Lab: User ID controlled by request parameter with data leakage in redirect
- Same objective as the above labs which is obtaining the API keys of CARLOS
- Logged in as user WIENER , checked with the usual patterns and then replaced the GET /my-account?id=wiener HTTP/2 to GET /my-account?id=carlos HTTP/2 in burp repeater
- Forwarded the req and checked with the response page , obtained sensitive info about the CARLOS (HTTP/2 302 Found-redirect)
- Retrieved the API keys 
- ![[Pasted image 20260419160911.png]]

 
# Lab: User ID controlled by request parameter with password disclosure
- This lab has user account page that contains the current user's existing password, prefilled in a masked input.To solve the lab, retrieve the administrator's password, then use it to delete the user `carlos`
- Logged in as WIENER , changed the URL from /my-account?id=wiener to /my-account?id=administrator
- Logged in successfully , then sent the req to BURP Repeater , examined the response page after forwarding the req
- Retrieved the ADMINISTRATOR pass , logged in as ADMINISTRATOR and deleted the user CARLOS
- Horizontal to Vertical privilege escalation

# Insecure direct object references (IDOR)
- Insecure direct object references (IDOR) are a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly.
- IDOR vulnerabilities are most commonly associated with ==horizontal privilege escalation==, but they can also arise in relation to ==vertical privilege escalation.== 
- ex - Consider a website that uses the following URL to access the customer account page, by retrieving information from the back-end database:`https://insecure-website.com/customer_account?customer_number=132355`,the customer number is used directly as a record index in queries that are performed on the back-end database. If no other controls are in place, an attacker can simply modify the `customer_number` value, bypassing access controls to view the records of other customers. This is an example of an IDOR vulnerability leading to horizontal privilege escalation.An attacker might be able to perform horizontal and vertical privilege escalation by altering the user to one with additional privileges while bypassing access controls. Other possibilities include exploiting password leakage or modifying parameters once the attacker has landed in the user's accounts page, for example.
 
# Lab: Insecure direct object references
- This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.Solve the lab by finding the password for the user `carlos`, and logging into their account.
- Found a live chat option in the webpage , after chatting with the bot and downloading transcript,sent the req to repeater 
- Changed the the URL as /1.txt to check prior users chat with the bot 
- logged in as the user using the password in the transcript 


# Lab: Multi-step process with no access control on one step
- This lab has an admin panel with a flawed multi-step process for changing a user's role.Familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.To solve the lab, log in using the credentials `wiener:peter` and exploit the flawed access controls to promote yourself to become an administrator.
- Logged in as admin and upgraded user CARLOS to admin and sent the req to repeater and logged out from admin acc
- Logged in as user WIENER and copied the session cookie and replaced the cookie of admin session with user WIENER's session cookie 
- Was able to upgrade roles of user WIENER by accessing the req of admin roles through admin panel 
- Multi-step process with no access control 

# Lab: Referer-based access control
- This lab controls access to certain admin functionality based on the Referer header.Familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.To solve the lab, log in using the credentials `wiener:peter` and exploit the flawed access controls to promote yourself to become an administrator.
- Logged in as admin and upgraded user CARLOS and sent the req to BURP repeater 
- Logged in as WIENER and changed the session cookie in repeater to the session cookie of user WIENER 
- Was able to breach due to weak access control 



