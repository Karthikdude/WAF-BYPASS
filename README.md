


# Bypassing Web Application Firewalls (WAF)

**Author:** Karthik S Sathyan  
**Date:** 20 October 2024  
**Project Description:** This project dives deep into **Bypassing Web Application Firewalls (WAFs)** using advanced techniques like fingerprinting WAFs, exploiting blacklisting flaws, leveraging browser bugs, and various encoding strategies. By understanding how WAFs operate, ethical hackers and penetration testers can uncover vulnerabilities and improve application security.

---

## Table of Contents

- [Overview](#overview)
- [Fingerprinting a WAF](#fingerprinting-a-waf)
  - [Cookie Values](#cookie-values)
  - [Citrix Netscaler](#citrix-netscaler)
  - [F5 BIG IP ASM](#f5-big-ip-asm)
  - [HTTP Response Codes](#http-response-codes)
  - [ModSecurity](#modsecurity)
  - [WebKnight](#webknight)
  - [Automatic Fingerprinting with Wafw00f](#automatic-fingerprinting-with-wafw00f)
- [WAF Bypass Techniques](#waf-bypass-techniques)
  - [Bypassing Blacklists](#bypassing-blacklists)
    - [Brute Forcing](#brute-forcing)
    - [Regex Reversing](#regex-reversing)
    - [Browser Bugs](#browser-bugs)
  - [Cheat Sheet for Bypassing WAFs](#cheat-sheet-for-bypassing-wafs)
    - [Initial Tests](#initial-tests)
    - [Encoding Techniques](#encoding-techniques)
    - [Context-Based Filtering](#context-based-filtering)
- [Advanced Techniques](#advanced-techniques)
  - [Charset Bugs](#charset-bugs)
  - [Null Bytes](#null-bytes)
  - [Parsing Bugs](#parsing-bugs)
  - [Unicode Separators](#unicode-separators)
  - [Missing X-Frame Options](#missing-x-frame-options)
  - [Window.name Trick](#window-name-trick)
  - [DOM-Based XSS](#dom-based-xss)
- [Conclusion](#conclusion)
- [References](#references)

---

## Overview

### 🔥 Introduction

Web applications are increasingly becoming prime targets for cyber-attacks, and Web Application Firewalls (WAFs) are often the first line of defense against these threats. However, many WAFs rely on signature-based filters, often using blacklists, which can be exploited. This project explores methods for bypassing these WAFs to improve overall security, including detailed examples and proof-of-concept code.

WAFs generally rely on **blacklisting** or **whitelisting** approaches. While blacklisting attempts to block "known bad" input, it is often flawed due to its reactive nature. This repository provides methods to bypass WAFs and prove how insecure blacklisting can be.

---

## Fingerprinting a WAF

Before bypassing a WAF, it’s crucial to identify the type of firewall in use. **Fingerprinting** a WAF involves examining HTTP responses, cookies, and headers. Identifying the WAF helps to craft specific payloads tailored to evade its defenses.

### Cookie Values

Some WAFs append unique cookies to HTTP requests. For example:

- **Citrix Netscaler**: Adds a cookie like `ns_af`.
- **F5 BIG IP ASM**: Returns specific cookies in the HTTP request, which can be identified using tools or manually inspecting the response.

```bash
# Example: Fingerprinting F5 BIG IP
GET / HTTP/1.1
Host: target.com
User-Agent: Mozilla/5.0
Cookie: F5-TrafficShield=abcd1234;
```

### Citrix Netscaler

**Citrix Netscaler** can be identified by inspecting cookies. For example, when querying an application running behind Citrix Netscaler, look for the presence of `ns_af` cookies in HTTP requests.

### F5 BIG IP ASM

F5 BIG IP ASM, one of the most popular WAFs, can also be fingerprinted through the use of unique cookie values (`F5-TrafficShield`) or by examining the specific HTTP response codes.

### HTTP Response Codes

WAFs often return specific error codes when a potential attack is detected. Common response codes include:

- **403 Forbidden**: Denied access due to a rule violation.
- **406 Not Acceptable**: Returned by **ModSecurity**.
- **999 No Hacking**: Typically seen in **WebKnight**.

### ModSecurity

ModSecurity is an open-source WAF primarily designed for **Apache** servers. It can be easily fingerprinted by observing `406 Not Acceptable` errors in response to malicious payloads.

### WebKnight

WebKnight, specifically designed for **IIS servers**, responds with a `999 No Hacking` status when a malicious request is detected. This WAF works on a blacklist, blocking known attack patterns like SQL Injection and XSS.

### Automatic Fingerprinting with Wafw00f

**Wafw00f** is a powerful Python-based tool designed for automatically detecting and fingerprinting WAFs.

```bash
# Detecting WAF using Wafw00f
wafw00f http://target.com
```

---

## WAF Bypass Techniques

WAF bypass techniques involve crafting payloads that evade filters set by blacklists, encoders, or pattern-matching rules. Below are some of the most common bypass strategies.

### Bypassing Blacklists

Blacklists block known malicious patterns, but by altering the payload’s structure, it's possible to bypass them.

#### Brute Forcing

Brute forcing involves sending various attack payloads to see which ones are blocked or allowed. Tools like **Burp Suite** or **FuzzDB** can be used to automate this process.

#### Regex Reversing

Regex filters are prone to bypass if the payload can be crafted to avoid matching the regular expression. By **reverse engineering** these regex filters, valid payloads can be constructed.

```html
<scr<script>ipt>alert(1)</scr<script>ipt> <!-- Nested tag trick -->
```

#### Browser Bugs

Older browsers, like Internet Explorer, have unpatched vulnerabilities that can bypass even modern WAFs. These bugs include charset handling, null byte injections, and parsing quirks.

---

## Cheat Sheet for Bypassing WAFs

This cheat sheet provides a practical methodology for bypassing blacklist-based WAFs.

### Initial Tests

Start with harmless payloads like `<b>`, `<i>`, and `<u>` tags to see if they are filtered. Then, try known attack vectors like:

```html
<script>alert(1)</script> <!-- Basic XSS -->
```

If basic payloads are blocked, attempt case variation, tag nesting, and encoding.

### Encoding Techniques

Different encoding techniques can bypass filters that block specific characters or patterns. Examples include **HTML entity encoding**, **Base64 encoding**, and **Hexadecimal encoding**.

```html
%3Cscript%3Ealert%281%29%3C/script%3E <!-- URL-encoded XSS -->
```

### Context-Based Filtering

WAFs often fail to analyze the context in which input is reflected. For example, if input is reflected inside JavaScript tags, it's possible to bypass filters by injecting code without closing the existing tags.

```html
<script>var x="test";alert(1)//</script> <!-- Inject code inside script tags -->
```

---

## Advanced Techniques

### Charset Bugs

Certain WAFs can be bypassed by manipulating the **charset** parameter in HTTP headers. For example, encoding a payload in **IBM037** charset can bypass poorly configured WAFs.

```python
# Example in Python
import urllib
payload = 'alert(1)'
encoded_payload = urllib.parse.quote_plus(payload.encode("IBM037"))
print(encoded_payload)
```

### Null Bytes

Null byte injection involves appending `%00` to parts of the payload, exploiting older browsers like **Internet Explorer 9**.

```html
<scri%00pt>alert(1);</scri%00pt>  <!-- Null byte injection -->
```

### Parsing Bugs

Older versions of browsers like **IE7** can be tricked into executing malformed HTML by inserting unexpected characters such as `%`, `//`, or `!`.

```html
<%div style=xss:expression(alert(1))> <!-- Works up to IE7 -->
```

### Unicode Separators

Certain WAFs fail to filter **Unicode separators** like `x0B` that act as spaces. These can be used to bypass event handlers that are normally blocked.

```html
<a/onmouseover[\x0b]=alert(1)> <!-- Using a Unicode

 separator -->
```

### Missing X-Frame Options

Exploiting missing **X-Frame-Options** headers allows attackers to use clickjacking techniques or iframe injections to execute malicious code.

### Window.name Trick

The `window.name` property in iframes can be exploited to inject code when the WAF restricts normal methods of JavaScript injection.

```html
<iframe src="http://target.com" name="javascript:alert(1)"></iframe> <!-- Window.name injection -->
```

---

## DOM-Based XSS

DOM-based XSS occurs when JavaScript takes user input and inserts it into the DOM without proper sanitization. WAFs often fail to block these because they don't intercept client-side processing.

```javascript
var input = location.hash;
document.write(input); <!-- Vulnerable to DOM-based XSS -->
```

---

## Conclusion

WAFs provide a valuable layer of defense, but blacklisting alone is not a sustainable solution. This project highlights the importance of using **whitelisting** and **proper encoding** to secure applications effectively. Developers and security teams should prioritize patching vulnerabilities and ensuring that WAFs are regularly updated with new signatures.

---

## References

- [OWASP WAF Bypassing](https://owasp.org/www-pdf-archive/OWASP_Stammtisch_Frankfurt_-_Web_Application_Firewall_Bypassing_-_how_to_defeat_the_blue_team_-_2015.10.29.pdf)
- [Hacken Blog on WAF Bypassing](https://hacken.io/discover/how-to-bypass-waf-hackenproof-cheat-sheet/)
- [ModSecurity XSS Evasion Challenge](http://blog.spiderlabs.com/2013/09/modsecurity-xss-evasion-challenge-results.html)
- [HTML5 Security](http://html5sec.org)
