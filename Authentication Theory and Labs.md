2. ==**Authentication Vulnerabilities theory and labs**== 
 
- Authentication is the process of verifying the ==identity of a user or client==. Websites are potentially exposed to anyone who is connected to the internet. This makes robust authentication mechanisms integral to effective web security.
- Something you **know**, such as a ==**password**== or the answer to a security question. These are sometimes called "knowledge factors".
- Something you **have**, This is a physical object such as a ==mobile phone or security token==. These are sometimes called "possession factors".
- Something you **are** or do. For example, your ==biometrics or patterns of behavior==. These are sometimes called "inherence factors".


# Difference between ==Authentication and Authorization==
- Authentication is the process of verifying that a user is who they claim to be. Authorization involves verifying whether a user is allowed to do something.
- For example, authentication determines whether someone attempting to access a website with the username `Carlos123` really is the same person who created the account.
- Once `Carlos123` is authenticated, their permissions determine what they are authorized to do. For example, they may be authorized to access personal information about other users, or perform actions such as deleting another user's account.

# Impact of vulnerable Authentication
- The impact of authentication vulnerabilities can be severe. If an attacker bypasses authentication or brute-forces their way into another user's account, they have access to all the data and functionality that the compromised account has. If they are able to compromise a high-privileged account, such as a system administrator, they could take full control over the entire application and potentially gain access to internal infrastructure.
- Even compromising a low-privileged account might still grant an attacker access to data that they otherwise shouldn't have, such as commercially sensitive business information. Even if the account does not have access to any sensitive data, it might still allow the attacker to access additional pages, which provide a further attack surface. Often, high-severity attacks are not possible from publicly accessible pages, but they may be possible from an internal page.
# Vulnerabilities in password-based login
- For websites that adopt a password-based login process, users either register for an account themselves or they are assigned an account by an administrator. This account is associated with a unique username and a secret password, which the user enters in a login form to authenticate themselves.
- In this scenario, the fact that they know the secret password is taken as sufficient proof of the user's identity. This means that the security of the website is compromised if an attacker is able to either obtain or guess the login credentials of another user.
## Brute-force attacks
- A brute-force attack is when an attacker uses a system of trial and error to guess valid user credentials. These attacks are typically automated using wordlists of usernames and passwords. Automating this process, especially using dedicated tools, potentially enables an attacker to make vast numbers of login attempts at high speed.
### Brute-forcing usernames & Brute-forcing passwords
- Usernames are especially easy to guess if they conform to a recognizable pattern, such as an email address. For example, it is very common to see business logins in the format `firstname.lastname@somecompany.com`. However, even if there is no obvious pattern, sometimes even high-privileged accounts are created using predictable usernames, such as `admin` or `administrator`
- Passwords can similarly be brute-forced, with the difficulty varying based on the strength of the password. Many websites adopt some form of password policy, which forces users to create high-entropy passwords that are, theoretically at least, harder to crack using brute-force alone. This typically involves enforcing passwords with:
      - A minimum number of characters
      - A mixture of lower and uppercase letters
     -At least one special character
# Username enumeration
- Username enumeration is when an attacker is able to observe changes in the website's behavior in order to identify whether a given username is valid.
- Username enumeration typically occurs either on the login page, for example, when you enter a valid username but an incorrect password, or on registration forms when you enter a username that is already taken. This greatly reduces the time and effort required to brute-force a login because the attacker is able to quickly generate a shortlist of valid usernames.

While attempting to brute-force a login page, you should pay particular attention to any differences in:  
- **Status codes**
- Error messages
- Response times
# Lab: Username enumeration via different responses
- This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:- [Candidate usernames]- [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)
  To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.
- Used BURP intruder for this case , the username and password wordlist were provided , set the payload type to SIMPLE LIST and used SNIPER in this case , found username AE 
- Was able to find it since all other usernames had INVALID USERNAME in the response messages , username AE contained INCORRECT PASS
- Same steps were followed for password ,password was retrieved due to the STATUS CODE [302] , others all had 200 as their code

# Lab: Username enumeration via subtly different responses
- This lab is subtly vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, the wordlists were provided
- Enumerate a valid username, brute-force this user's password, then access their account page.
- Sent a valid fake username and pass to check the response , analysed common responses from all requests 
- One of the usernames contained distinguishable keys , and repeated the same process from intruder for passwords 
- Password was identified due to the STATUS CODE being 302

# Lab: Username enumeration via response timing
- This lab is vulnerable to username enumeration using its response times. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.
- Since this lab restricts users to 3 attempts , X-FORWARDED-FOR has been used. Pitch-fork attack type is suitable since i  have to keep changing the fake IP so that i don't get banned for 30 mins 
- After logging in as user WEINER , sent the request to repeater and analysed it.Added X-FORWARDED-FOR in the request and sent it to BURP intruder.Set the payloads both type 1 and type 2 and started the attack
- Discovered the username with high response time and repeated the same process for passwords , was able to breach in
# Lab: Broken brute-force protection, IP block
- This lab is vulnerable due to a logic flaw in its password brute-force protection. To solve the lab, brute-force the victim's password, then log in and access their account page
- After analysing the responses from multiple login requests , was able to identify that system resets the attempts [after 2]if the user logs in successfully
- Created payloads list that suits this scenario and was able to retrieve the credentials
# Lab: Username enumeration via account lock
- This lab is vulnerable to username enumeration. It uses account locking, but this contains a logic flaw. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.
- Had to use FFUF due to the fact that BURP intruder has throttling in community version
- Server locks account after X failed attempts but error message is DIFFERENT when username exists - "You have made too many incorrect login attempts"
- Due to this major change , was able to find username , after repeating the same process with password , was able to retrieve it since it had completely different size then the rest of them 
- ==Basic POST brute force: ffuf -w LIST.txt -X POST -d "param=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -u URL== 
- ==Filter wrong responses: -fr "error message" Match right responses: -mr "success message" Speed control: -t 10 -rate 50 Two parameters: -w list1.txt:W1 -w list2.txt:W2 then use W1 and W2 instead of FUZZ==
# Flawed brute-force protection
- It is highly likely that a brute-force attack will involve many failed guesses before the attacker successfully compromises an account. Logically, brute-force protection revolves around trying to make it as tricky as possible to automate the process and slow down the rate at which an attacker can attempt logins. The two most common ways of preventing brute-force attacks are:
 - Locking the account that the remote user is trying to access if they make too many failed login attempts
 - Blocking the remote user's IP address if they make too many login attempts in quick succession
### Account locking
- One way in which websites try to prevent brute-forcing is to lock the account if certain suspicious criteria are met, usually a set number of failed login attempts. Just as with normal login errors, responses from the server indicating that an account is locked can also help an attacker to enumerate usernames.
Locking an account offers a certain amount of protection against targeted brute-forcing of a specific account. However, this approach fails to adequately prevent brute-force attacks in which the attacker is just trying to gain access to any random account they can.

For example, the following method can be used to work around this kind of protection:

1. Establish a list of candidate usernames that are likely to be valid. This could be through username enumeration or simply based on a list of common usernames.
2. Decide on a very small shortlist of passwords that you think at least one user is likely to have. Crucially, the number of passwords you select must not exceed the number of login attempts allowed. For example, if you have worked out that limit is 3 attempts, you need to pick a maximum of 3 password guesses.
3. Using a tool such as Burp Intruder, try each of the selected passwords with each of the candidate usernames. This way, you can attempt to brute-force every account without triggering the account lock. You only need a single user to use one of the three passwords in order to compromise an account.
# User rate limiting
Another way websites try to prevent brute-force attacks is through user rate limiting. In this case, making too many login requests within a short period of time causes your IP address to be blocked. Typically, the IP can only be unblocked in one of the following ways:

- Automatically after a certain period of time has elapsed
- Manually by an administrator
- Manually by the user after successfully completing a CAPTCHA

User rate limiting is sometimes preferred to account locking due to being less prone to username enumeration and denial of service attacks. However, it is still not completely secure. As we saw an example of in an earlier lab, there are several ways an attacker can manipulate their apparent IP in order to bypass the block.

As the limit is based on the rate of HTTP requests sent from the user's IP address, it is sometimes also possible to bypass this defense if you can work out how to guess multiple passwords with a single request.
# Lab: Broken brute-force protection, multiple credentials per request
- This lab is vulnerable due to a logic flaw in its brute-force protection. To solve the lab, brute-force Carlos's password, then access his account page.
- This case had a vulnerability where i could insert multiple passwords in just one request , because of this , i was able to get STATUS CODE 302 
- After getting the CODE 302 , i just looked for the option in BURP where for this particular request i get response from browser , copied url and pasted it , was able to access the user 

# Vulnerabilities in multi-factor authentication
- Many websites rely exclusively on single-factor authentication using a password to authenticate users. However, some require users to prove their identity using multiple authentication factors.
- Verifying biometric factors is impractical for most websites. However, it is increasingly common to see both mandatory and optional two-factor authentication (2FA) based on something you know and something you have. This usually requires users to enter both a traditional password and a temporary verification code from an out-of-band physical device in their possession.
- While it is sometimes possible for an attacker to obtain a single knowledge-based factor, such as a password, being able to simultaneously obtain another factor from an out-of-band source is considerably less likely. For this reason, two-factor authentication is demonstrably more secure than single-factor authentication. However, as with any security measure, it is only ever as secure as its implementation. Poorly implemented two-factor authentication can be beaten, or even bypassed entirely, just as single-factor authentication can.
- It is also worth noting that the full benefits of multi-factor authentication are only achieved by verifying multiple different factors. Verifying the same factor in two different ways is not true two-factor authentication. Email-based 2FA is one such example. Although the user has to provide a password and a verification code, accessing the code only relies on them knowing the login credentials for their email account. Therefore, the knowledge authentication factor is simply being verified twice.
# Lab: 2FA simple bypass
- This lab's two-factor authentication can be bypassed. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, access Carlos's account page.
- In this case , i had to log in as user WIENER , retrieve code 2FA code from my email and then was able to access my account
- Victim's ID&PASS was already provided but did not have the email to retrieve the 2FA code from , but was able to breach after changing the URL after logging in to ==/my-account== 
- The system could not detect this logic flaw 
# Lab: 2FA broken logic
- This lab's two-factor authentication is vulnerable due to its flawed logic. To solve the lab, access Carlos's account page.
- Logged in as per the provided credentials and navigated to the 4-digit security code portal and was successfully able to log in WIENER 
- Forwarded all the requests to BURP repeater , changed the first level login req name from WIENER to CARLOS , sent the req , got a response that contained new session cookie
- Replaced the session cookie with the newly assigned session cookie and then replaced WIENER with CARLOS
- Both parameters were replaced in a POST login 2 request , which also contained mfa-code , after forwarding the req , obtained response 200 which stated the mfaaa code was incorrect
- Then using FFUF , i brute forced the 4-digit mfa code and was able to enter CARLOS's page

# Lab: Brute-forcing a stay-logged-in cookie
 - This lab allows users to stay logged in even after they close their browser session. The cookie used to provide this functionality is vulnerable to brute-forcing.To solve the lab, brute-force Carlos's cookie to gain access to his **My account** page.
 - After logging in as WIENER , there was a need to inspect different requests to check stay-logged-in cookie 
 - After decoding the stay-logged-in cookie , found that it follows a pattern of base64(username+':'+md5HashOfPassword) 
 - Having to already know the victim's username , i had to brute force the password in the same format as mentioned above
 - Had to add these set of rules for payload processing to ensure proper successful brute force 
   -Hash: `MD5`
   -Add prefix: `carlos:`
   -Encode: `Base64-encode
- Then i was able to successfully access CARLOS's page

# Lab: Offline password cracking
- This lab stores the user's password hash in a cookie. The lab also contains an XSS vulnerability in the comment functionality. To solve the lab, obtain Carlos's `stay-logged-in` cookie and use it to crack his password. Then, log in as `carlos` and delete his account from the "My account" page.
- After logging in as WIENER , the lab was said to have a XSS vulnerability , under a blog posted a comment which secretly exported all users stay-logged-in cookie 
- Again , the stay-logged-in cookie was in base64 form and then had to convert the value to md5 [base64(username+':'+md5HashOfPassword)] 
- Was then able to extract victim's password and delete his account 
# Lab: Password reset broken logic
- This lab's password reset functionality is vulnerable. To solve the lab, reset Carlos's password then log in and access his "My account" page.
- After logging in as WIENER and inspecting the forgot password requests , there was a unique temp-forgot-password-token which was generated after clicking on the recovery email 
- Sent all those requests to burp REPEATER and then just to check , i started replacing id=wiener to id=carlos and forwarded the response , got the session cookie 
- In the POST request where the user had to post email/username to get an email for password reset , replaced wiener to carlos and sent it , response was generated and it says the email has been sent , but couldn't find any way to access carlos's email 
- In another request where the temp-forgot-password-token was given to user WIENER , i just replaced the userid from wiener to carlos , and it actually worked , in response to the change i got status code 302 
- So hence i was able to change carlos's password 
  ![[Pasted image 20260712105730.png]]

# Lab: Password reset poisoning via middleware
- This lab is vulnerable to password reset poisoning. The user `carlos` will carelessly click on any links in emails that he receives. To solve the lab, log in to Carlos's account. You can log in to your own account using the following credentials: `wiener:peter`. Any emails sent to this account can be read via the email client on the exploit server
-  Sent the `POST /forgot-password` request to Burp Repeater. Found out that the  `X-Forwarded-Host` header is supported 
- X-Forwarded-Host: MY-EXPLOIT-SERVER-ID.exploit-server.net
-  Changed the `username` parameter to `carlos` and send the request.
- After navigating to my access log i was able to find the victim's temp-forgot-password token , then successfully infiltrated CARLOS's id

# Lab: Password brute-force via password change
- This lab's password change functionality makes it vulnerable to brute-force attacks. To solve the lab, use the list of candidate passwords to brute-force Carlos's account and access his "My account" page.
- Logged in as WIENER , changed password , went through each request , tried giving incorrect passwords , the lab has a vulnerability in the password change request
- Victim's id was known , brute forced the password from the given list , had to follow a mechanism where the new password and confirm password were different , which provided the 200 status code response which says your current password is incorrect , tried with giving both new and confirm password the same word , but once the current password doesn't match , i was logged out from my account page , so the case 1 worked and was able to access victim's page
