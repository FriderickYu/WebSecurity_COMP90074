## Question 1 and Question 2
* IDOR
* MFA
* Information disclosure
* CSRF


## Question 3 Threat modelling for Scenario A STRIDE
### Spoofing

**Identification**: The attacker might use IDOR vulnerability to access unauthorized webpage or other user's account

**Remediation**
* Forbidden parameters such as id appear in URL
* Creating a permission mechanism to limit users to view others' privacy
### Tampering
**Identification**: The attacker might use CSRF vulnerability to modify users' password or email, and access system with other users' account

**Remediation**
* Adding verification code during modifying password and email sections
* Generating `Token` in server, once client `POST` a `HTTP Request`, server will respond a message contain `Token`
### Repudation

**Identification**: Employees of `Outsourced Call Centers Pty.Ltd.` might steal data in back-end, or writing malicious code in source code. After that, these employees will deny such actions since there is no proof

**Remediation**
* Setting a log or version control module to record every action of database and source code


### Information disclosure
**Identification**
* External attacker might steal sensitive information or source code due to information disclosure vulnerability
* Internal employees of `Outsourced Call Centers Pty.Ltd.` might steal original data in database

**Remediation**
* Adding multiple-level permission management system, to regulate who can read/write which module
* Use generic error messages as much as possible. Don't provide attackers with clues about application behavior unnecessarily.
### Denial of service
**Identification**: Attacker sends too many request in the same time

**Remediation**
* Setting maximum number of requests in a short time
* Increasing bandwidth of network communication
### Elevation of privilege
**Identification**
* Attacker might use IDOR and MFA vulnerabilities to login as admin or read content which should only be assessed by admin

**Remediation**
* Setting more complex authentication system, or adding an auxiliary security protection in system 


## Question 4 Risk Statement
通用格式

_[Something might occur or not occur or is present] **due to** [a particular event or situation], **which leads to**
[consequences with reference to particular objectives]._

1. User's pasword might be leaked due to IDOR vulnerability, which leads to that personal information might be stolen.
2. Source codes in back-end might be leaked due to horrible source code file management, which leads to that attacker might utilize this vulnerability to attack this system.
3. The admin account might be stolen due to MFA vulnerability, which leads to special privilege leaked.
4. User's email or password might be modified due to CSRF vulnerability, which leads to that attacker might modify users' authentication information, and login system with victim identification.

## Question 5 Risk Appetite
For bank, espeically it involves lots of money, users' sensitive information,
therefore bank will not any vulnerabilities which might cause lost of money and sensitive information.
Because these vulnerabilities will eventually ruin the reputation of bank, not to mention that bank may involve
some national interests, commerical secrets, etc.

But for startup, some vulnerabilities could be accepted after all size of company is relatively small, few data and application can be damaged.
If the scale of entire development reaches to a certain size, then CISO should be worried about some critical vulnerabilites.

## Question 6 Evalution of risk statement
通用格式: Bank

An increase in _low value_ fraudulent transactions may occur **due to** globally enabling contactless
transactions on credit cards, **which leads to** the bank incurring financial loss **due to** lowering the barrier
charging fraudulent transactions to a stolen credit card.

**Likelihood**:Almost certain

**Consequence**:Minor

**Risk**:High
..........

<font color=red>先陈述risk statement,然后根据矩阵列出Likelihood, consequence,最后得出Risk等级，并分析出这些都是咋来的</font>

## Question 7 


## Question 8 Method of testing an application
[安全测试常用方法](https://baijiahao.baidu.com/s?id=1639738035018300361)

1. Recon and analysis
   1. Map application content
      1. Explore visible content
         1. visit all the functionality you can in the application
         2. use tools to lg every call made, including background and websocket calls
         3. make sure to take manual notes while you are at it, as you will need to prioritise what functionality you
            want to test further and in-depth later on
      2. Consult public resources
         1. Use things like Google dorks to your advantage
         2. Read the documentation of the application
         3. Use the Way Back Machine (or any archives) to find hidden and deprecated assets
         4. Attempt to find WADL/WSDL and swagger files (or any documentation leaking API endpoints)
      3. Discover hidden cotent
         1. Directory brute force using something like dirbuster / gobuster / recursebuster / dirsearcher
         2. Spidering the application
         3. Looking into things like route files
      4. Discover default content
         1. Use plugins or any tools (like nikto) to find default / well-known file paths and headers
      5. Enumerate identifier specified functions
         1. Attempt to brute force different functionality in parameters such as the ones below: http://example.com/hello.php?action=editUser
         2. This can help uncover different functions and use functions that may not be accessible otherwise
      6. Test for debug parameter
         1. test for parameters such as the ones below:
            1. ?debug=true
            2. ?debug=1
            3. ?test=true
            4. etc
   2. Analyze the application
      1. Identify functionality
         1. Crawl the application and identify functionality (again after you have learnt more)
      2. Identify data entry points
         1. What kind of input can the user provide?
         2. How is this data transmitted?
         3. Is there any out-of-band channels of communication? Can the user control any of this functionality? (may
             lead to SSRF)
            1. Things like testing for authorisation of URLs
            2. Things like loading of images via URLs
      3. Identify the technolgies used
         1. Fingerprint the application
         2. Attempt to find out what kind of technologies being used
         3. Use something like wappalyzer to identify any kind of software packages in use
         4. This helps target your attacks towards those specific software packages / languages
         5. E.g. You don’t use python or ruby attacks on a PHP application, as they likely won’t work!
      4. Map the attack surface
         1. After identifying all the functionality within the application, attempt to list what types of vulnerabilities
            need to be tested
            1. Could the function be vulnerable to XSS? If yes, what types of XSS?
            2. Could the function be vulnerable to path traversals?
            3. etc
   3. Test client-side controls
      1. transmission of data via client
         1. hidden fields
         2. cookies
         3. preset parameters
         4. ASP.NET ViewState
      2. Client-side input controls
         1. length limits
         2. JavaScripts validation
         3. Disabled elements
      3. Browser extensions
         1. Java applets
         2. ActiveX controls
         3. Flash objects
         4. Silverlight objects
   4. Test the authentication mechanism 
      1. Understand the mechanism
      2. Data attacks
         1. test password quality
         2. test for username authentication
         3. test for password guessing
      3. Special functions
         1. test account recovery
         2. test "remember me"
         3. test impersonation functions
      4. Credential handling
         1. test username uniqueness
         2. test credential predictability
         3. check for unsafe transmission
         4. check for unsafe distribution
         5. check for insecure storage
      5. Authentication logic
         1. test for fail-open logic
         2. test multistage processes
      6. Exploit vulnerabilities
   5. Test the sesssion management mechanism
      1. Token generation
         1. test for meaning
         2. test for predictability
      2. Token handling
         1. check for insecure transmission
         2. check for disclosure in logs
         3. test mapping of tokens to sessions
         4. test session termination
         5. test for session fixation
         6. check for CSRF
         7. check cookie score
   6. Test access controls
      1. understand the requirements
      2. test with multiple accounts
      3. test with limited access
      4. test for insecure methods
   7. Test for input-based vulnerabilities
      1. SQL Injection
      2. XSS response and Injection
      3. OS command Injection
      4. Path traversal
      5. Script Injection
      6. File inclusion
   8. Test for function-specific input vulnerabilites
      1. SMTP injection
      2. Native code flaws
      3. SOAP Injection
      4. LDAP Injection
      5. XPath Injection
      6. Back-end request Injection
      7. XXE Injection
   9. Identify key attack surface
      1. Multistage processes
      2. Incomplete input
      3. Trust boundaries
      4. Transaction logic
   10. Test for shared hosting vulnerabilities
       1. Test segregation in shared infrastructures
       2. Test segregation between ASP-hosted applications
   11. Test for application server vulnerabilites
       1. Test for default credentials
       2. Test for default content
       3. Test for dangerous HTTP methods
       4. Test for proxy functionality
       5. Test for virtual hosting misconfiguration
       6. Test for web server software bugs
       7. Test for web application firewalling
   12. Miscellaneous checks
       1. Test for DOM-based attacks
       2. Test for local privacy vulnerabilites
       3. Test for weak SSL ciphers
       4. Check same-origin policy configuration


## Question 9 Describe the process, from identifying a risk to entering it into the enterprise risk register

* Establishing the context
  * Identifying the scope, context of risks
  * Meanwhile deciding which criteria will be used to identify the risk
* Risk Appetite
  * Estimating whether vulnerabilities in the system are critical or not
  * Then deciding how much risk are we willing to taking, considering which field system will be used in, whether data is important or not, etc.
* Risk Assessment
  * Risk identification
    * What is the risk and risk will eventually cause what kind of error or warning?
  * Risk analysis
    * What is the likelihood that risk will eventually happen?
    * The consequence if risk is happened
    * Based on above two criteria, making a risk rating according to consequence/likelihood matrix
  * Risk evaluation
    * Based on risk analysis, decide something
      * Like remediation, how to control the risk, or do nothing if possibility is low
* Risk Treatment
  * Treat each risk according to above several criteria 
* Recording and Reporting 
  * Record and report all risks in the documentation(like penetration report)
* Monitoring and Review<font color=red>(Risk Register之后的步骤)</font>
  * Enter all suitable endpoints into a monitoring system for monitoring and alerting
    * Use a SIEM tool (Security Information and Event Management)
    * Use vulnerability scanners
    * Check on Operating System and Application patch-levels
  * Agree on a set schedule to review all risks in the risk register(Monthly or Quarterly)