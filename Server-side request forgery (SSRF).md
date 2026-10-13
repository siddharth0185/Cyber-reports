

# Lab: Basic SSRF against the local server
- This lab has a stock check feature which fetches data from an internal system. To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.
- The stock checker accepted a user-controlled URL and the server made the request, making it vulnerable to SSRF. 
- Instead of requesting the stock API, the request was changed to `http://localhost/admin`, allowing access to an internal admin interface that was inaccessible from the internet.
- After discovering the admin endpoint, a request was sent to delete the user `carlos`, completing the SSRF attack. http://localhost/admin/delete?username=carlos

# Lab: Basic SSRF against another back-end system 
- This lab has a stock check feature which fetches data from an internal system.To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`.
- The vulnerable `stockApi` parameter allowed the server to make requests to arbitrary internal hosts. 
- Burp Intruder was used to enumerate the internal `192.168.0.1–255` range on port `8080` until a host returned `200 OK`, revealing the admin interface. 
- After identifying the live host, the SSRF request was modified to `/admin/delete?username=carlos`, allowing deletion of the user.    stockApi=http%3A%2F%2F192.168.0.255%3A8080%2Fadmin%2Fdelete%3Fusername%3Dcarlos


# Lab: SSRF with blacklist-based input filter
- This lab has a stock check feature which fetches data from an internal system.To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.
- The application blacklisted both `localhost` and `/admin`, returning `400 Bad Request` when either appeared directly.
- The hostname filter was bypassed using the alternative localhost representation `127.1`, while the path filter was bypassed by partially URL-encoding the letter `a` in `/admin` (`/%2561dmin`). [ Double url encoding ]
- The blacklist checked the raw input before decoding, allowing the request to reach the internal admin endpoint after URL decoding.


# Lab: SSRF with whitelist-based input filter
- This lab has a stock check feature which fetches data from an internal system.To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.
- Unlike blacklist filters, whitelist filters only allow specific hosts/values. The bypass relies on making the filter and the backend parse the same URL differently. 
- `@` can separate userinfo from the actual hostname, while `#` starts a URL fragment and is not normally sent to the server. 
- URL encoding/double encoding can create parser differences (`%2523` → `%23` → `#`), and multiple techniques can be combined to bypass a whitelist.
- http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos



# Lab: SSRF with filter bypass via open redirection vulnerability
- This lab has a stock check feature which fetches data from an internal system.To solve the lab, change the stock check URL to access the admin interface at `http://192.168.0.12:8080/admin` and delete the user `carlos`.
- The stock checker only allowed the local application URL, so first found an open redirect in `/product/nextProduct` using the `path` parameter, which could redirect to an arbitrary URL. 
- Used the trusted local URL as the `stockApi`, then set the redirect `path` to the internal admin interface at `192.168.0.12:8080/admin`. 
- The `&` inside the nested URL had to be encoded as `%26` so the `path` parameter stayed inside `stockApi`; this chained the SSRF → open redirect → internal admin request and allowed deleting `carlos`.


# Blind SSRF 
-  Blind SSRF occurs when the server can be forced to make a request to a supplied URL, but the response is not returned to the attacker. 
-  OAST tools such as Burp Collaborator are used to detect the request by generating a unique domain and monitoring for DNS/HTTP interactions. 
- Blind SSRF can still be used for internal probing, and in advanced cases can be chained with other vulnerabilities to achieve serious impact such as RCE.
- With responses not being visib
