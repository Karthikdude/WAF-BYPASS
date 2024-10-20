```markdown
# Bypassing Web Application Firewalls (WAF)

**Author:** Karthik S Sathyan  
**Date:** 20 October 2024  
**Project Description:** This project explores techniques to bypass Web Application Firewalls (WAFs) by leveraging fingerprinting methods, blacklisting flaws, browser bugs, and encoding tricks.

## Table of Contents

- [Fundamental Concept](#fundamental-concept)
- [Introduction](#introduction)
- [Fingerprinting a WAF](#fingerprinting-a-waf)
- [WAF Bypass Techniques](#waf-bypass-techniques)
  - [1. Fingerprinting WAFs](#1-fingerprinting-wafs)
  - [2. Bypassing Blacklists](#2-bypassing-blacklists)
  - [3. Browser Bugs and Charset Encoding](#3-browser-bugs-and-charset-encoding)
  - [4. Encoding and Entity Decoding](#4-encoding-and-entity-decoding)
- [Conclusion](#conclusion)
- [References](#references)

## Fundamental Concept

The fundamental concept behind Cross-Site Scripting (XSS) attacks revolves around the failure of input parameter validation in web applications. WAFs try to prevent these attacks by filtering malicious input, but there are ways to craft semantically equivalent payloads that bypass these filters.

---

## Introduction

Web Application Firewalls (WAFs) have become a critical part of cybersecurity as they protect applications from common vulnerabilities like SQL Injection and XSS. However, many WAFs rely on blacklist-based filters that are prone to bypass. This project focuses on fingerprinting WAFs and demonstrating various techniques to bypass them.

---

## Fingerprinting a WAF

Before bypassing a WAF, it is crucial to identify the specific WAF in use. WAF fingerprinting involves looking for clues in HTTP responses, cookies, and headers. Here are some techniques used:

### 1. Fingerprinting via Cookie Values

Many WAFs add unique cookies to the HTTP request. For example, Citrix Netscaler and F5 BIG IP ASM append specific cookie values, helping identify the WAF protecting a web application.

### 2. Fingerprinting Citrix Netscaler and F5 BIG IP ASM

Citrix Netscaler and F5 BIG IP ASM can be fingerprinted using cookies (`ns_af`, `F5-TrafficSheild`) and certain HTTP responses.

### 3. HTTP Response Codes

Common HTTP response codes such as 403, 406, and 419 are indicative of certain WAFs, like ModSecurity or WebKnight, which return `406 Not Acceptable` or `999 No Hacking` responses.

### 4. Automatic Fingerprinting with Wafw00f

**Wafw00f** is a popular Python tool for automatically fingerprinting WAFs. It performs several tests, including cookie analysis and HTTP response matching.

```bash
# Example of using Wafw00f for WAF detection
wafw00f http://target.com
```

---

## WAF Bypass Techniques

### 1. Fingerprinting WAFs

Fingerprinting helps identify the type of WAF, which is crucial before launching bypass attempts. This involves analyzing cookies, HTTP responses, and error codes as explained above.

### 2. Bypassing Blacklists

Most WAFs operate using blacklists that filter known attack patterns. These blacklists can be bypassed using techniques like brute force, regex reversal, or exploiting browser-specific bugs.

#### 2.1 Brute Forcing

Brute forcing involves sending a variety of payloads to see which ones are not blocked by the WAF.

#### 2.2 Regex Reversing

By reversing WAF's regular expression filters, you can craft payloads that bypass the filter while remaining valid JavaScript.

#### 2.3 Browser Bugs

Exploiting browser bugs, especially in older browsers like Internet Explorer, can be effective in bypassing blacklists. For instance, charset bugs and null byte injections can fool filters into executing malicious scripts.

```html
<scri%00pt>alert('XSS');</scri%00pt>  <!-- Example using null bytes -->
```

---

### 3. Browser Bugs and Charset Encoding

### 3.1 Charset Bugs

Certain WAFs may be bypassed by changing the charset used in HTTP requests. Modifying the `Content-Type` header to use an encoding like `IBM037` can allow bypassing WAFs that aren't configured to handle such encodings.

```python
# Charset encoding in Python
import urllib
payload = 'alert(1)'
encoded_payload = urllib.parse.quote_plus(payload.encode("IBM037"))
print(encoded_payload)
```

### 3.2 Null Bytes

Null bytes are commonly used as string terminators and can be used to bypass certain WAF filters, especially in older browsers like Internet Explorer 9.

---

### 4. Encoding and Entity Decoding

### 4.1 Entity Decoding

WAFs may decode certain HTML entities, which can lead to successful bypasses. Injecting encoded payloads that get decoded by the WAF can result in malicious script execution.

```html
&lt;script&gt;alert(1)&lt;/script&gt;  <!-- Encoded payload -->
```

### 4.2 Other Encoding Techniques

You can obfuscate payloads using different encoding schemes, including HTML, base64, and hexadecimal, to bypass WAFs.

```html
<a href="javascript&#x3A;alert(1)">Click me</a>  <!-- Hex encoding -->
```

---

## Conclusion

Blacklisting is not an effective long-term solution for securing web applications. It is a temporary fix that often leads to more vulnerabilities. Developers and administrators should focus on patching source code vulnerabilities and using whitelists rather than relying solely on WAFs. Regular updates to WAF signatures and manual configuration are essential to maintaining security.

---

## References

- [OWASP WAF Bypassing](https://owasp.org/www-pdf-archive/OWASP_Stammtisch_Frankfurt_-_Web_Application_Firewall_Bypassing_-_how_to_defeat_the_blue_team_-_2015.10.29.pdf)
- [Hacken Blog on WAF Bypassing](https://hacken.io/discover/how-to-bypass-waf-hackenproof-cheat-sheet/)
- [ModSecurity XSS Evasion Challenge](http://blog.spiderlabs.com/2013/09/modsecurity-xss-evasion-challenge-results.html)
- [HTML5 Security](http://html5sec.org)

