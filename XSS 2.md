# # Lab: Reflected XSS into attribute with angle brackets HTML-encoded
- This lab contains a reflected cross-site scripting vulnerability in the search blog functionality where angle brackets are HTML-encoded. To solve this lab, perform a cross-site scripting attack that injects an attribute and calls the `alert` function.
- "onmouseover="alert(1) broke he html encoding and solved the lab

# Lab: Stored XSS into anchor `href` attribute with double quotes HTML-encoded
- This lab contains a stored cross-site scripting vulnerability in the comment functionality. To solve this lab, submit a comment that calls the `alert` function when the comment author name is clicked.
- Since its a stored XSS , the injection point is thee comment section , after insepcting all the comments from the blog , href is used and its used in the website field 
- So posted a random comment , name , mail , as for the website field since they recognize http , https and ftp and javascript , jus injected javascript:alert(1) and it executed 

# Lab: Reflected XSS in canonical link tag
 - This lab reflects user input in a canonical link tag and escapes angle brackets.
   To solve the lab, perform a cross-site scripting attack on the home page that injects an attribute that calls the `alert` function.
 - USed the access key conceps in this lab , did not have to break the attribute , instead was already in it 
 - USed cTRL U to get html source code and got the pattern and used single quotes instead of double quotes and then was able to modify the url in a way i wanted - https://0a0b00ed0404b7e5803903e400c20059.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)

# Lab: Reflected XSS into a JavaScript string with single quote and backslash escaped
-  This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality. The reflection occurs inside a JavaScript string with single quotes and backslashes escaped.
- The lab contained javascript string , so just had to figure out the tags used and see the pattern of those , the lab contained                    
- <script>
    var searchTerms = 'sid123';
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
    </script>
 - Broke out of the script with -
-  </script> <img src=1 onerror=alert(document.domain)>
 the second part is the payload , since there's no image source 1 , this fires / activates the oneerror and this event fires when there's an error of any sort and executes the alert lab


# Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded
- This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets are encoded. The reflection occurs inside a JavaScript string. To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the `alert` function.
- Same reflected XSS vulnerability , just need to inspect the webpage and figure out patterns 
- The lab contains js strings , so used '-alert(1)-' to break out of it and execute the alert function


# Lab: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped
- This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets and double are HTML encoded and single quotes are escaped.
-  The random string has been reflected inside a JavaScript string.
-  Tried sending the payload `test'payload` and observed that single quote gets backslash-escaped, preventing from breaking out of the string.
-  Tried sending the payload `test\payload` and observed that  backslash doesn't get escaped.
- Replaced input with the following payload to break out of the JavaScript string and inject an alert:
    \\'-alert(1)//`



# Lab: Reflected XSS in a JavaScript URL with some characters blocked
- This lab reflects your input in a JavaScript URL, but all is not as it seems. This initially seems like a trivial challenge; however, the application is blocking some characters in an attempt to prevent XSS attacks.
- The exploit uses exception handling to call the `alert` function with arguments. The `throw` statement is used, separated with a blank comment in order to get round the no spaces restriction. The `alert` function is assigned to the `onerror` exception handler.
- As `throw` is a statement, it cannot be used as an expression. Therefore need to use arrow functions to create a block so that the `throw` statement can be used.Then need to call this function,  assign it to the `toString` property of `window` and trigger this by forcing a string conversion on `window`.

# Lab: Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped
- This lab contains a stored cross-site scripting vulnerability in the comment functionality.To solve this lab, submit a comment that calls the `alert` function when the comment author name is clicked.
- The lab is vulnerable to stored XSS , hence the payload is injectable in the comments section of the blogpost 
- Inspecting the page aftermath of submitting random comment , found that i am directly inside of the onclick event , so i dont need onclick in my payload , also the event had the website in it making it the injectable field in the comments section
- http://foo?&apos;-alert(1)-&apos; (&apos; is html encoding for ' ) and it was solved 


# Lab: Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped
- This lab contains a reflected cross-site scripting vulnerability in the search blog functionality. The reflection occurs inside a template string with angle brackets, single, and double quotes HTML encoded, and backticks escaped. To solve this lab, perform a cross-site scripting attack that calls the `alert` function inside the template string.
- The lab has reflected XSS so injectable point is the search bar , aftermath of inspecting requests and source code when the search is used the following code is seen in the response : 
  var message = `0 search results for 'sid123'`;
  document.getElementById('searchMessage').innerText = message;
- So had to inject my payload : ${alert(document.domain)} 


# Lab: DOM XSS in `document.write` sink using source `location.search`
-  This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search`, which you can control using the website URL.
- After inpecting the element , my search string is being put inside img src , so just had to close the tags and inject my payload -"> < svg onload=alert(1) > 


# Lab: DOM XSS in `document.write` sink using source `location.search` inside a select element 
- This lab contains a DOM-based cross-site scripting vulnerability in the stock checker functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search` which you can control using the website URL. The data is enclosed within a select element.
- 
