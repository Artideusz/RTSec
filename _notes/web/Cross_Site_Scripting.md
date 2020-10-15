---
title: XSS Basics
layout: note
permalink: /notes/web/xss
name: XSS Basics
grouped_by: web
---
# Cross-site Scripting (aka XSS)

Cross-site scripting (or shortly called XSS) is a method of **injecting javascript code** to a web application **without permission**. The most dangerous thing about this is that XSS grants an attacker control over what other people **see** on the webpage and/or steal cookie information from a user that allows him to **"log in"** to his account, he can then for example send malicous links through your social media account to other people that trust the user (family, close friends). 

Luckily nowadays, session cookie theft is a bit harder to pull off, mainly because more and more developers take advantage of the [HttpOnly]() header and alot of frontend frameworks try to prevent XSS in general. But still, since the attacker has full control over the content of the page (and your browser), he might as well turn it into a login screen that looks **excactly** to the real one on the website. Many people would just think that it's just some kind of hiccup from the site and they attempt to log in again, giving sensitive information to the attacker by sending that information to the attackers server.

There are three main types of XSS:

## Reflected XSS

This type of XSS takes advantage of poor server-side sanitization of [query string](https://en.wikipedia.org/wiki/Query_string) input. An example would be:

1. The user sends the following GET request:
    - `https://example.com?keyword=123`
    - The `?keyword=123` is the query string.
2. The user recieves the following content from the server:
    ```html
        <p>Sorry, there is no keyword called 123.</p>
    ```
3. It looks like the server sent the query string input back to the user, meaning there can be a possibility of reflected XSS.
4. The user sends another GET request that looks like the following:
    - `https://example.com?keyword=<h1>123</h1>`
    - The user added a HTML header tag (h1) to the query string.
5. The user recieves the following content from the server:
    ```html
        <p>Sorry, there is no keyword called <h1>123</h1></p>
    ```
    - The content is rendered correctly, changing the size of `123`.
6. The user now tests if he is able to invoke XSS using the `<script>` tag or any tag or HTML attribute that allows javascript execution.
    - `https://example.com?keyword=<script>alert('XSS')</script>`

7. The server sends back the following content to the user:
    ```html
        <p>Sorry, there is not keyword called <script>alert('XSS')</script></p>
    ```
8. The user is greeted with an alert box saying '`XSS`' when the user recieves the content to the browser.
9. The user now knows that the webpage is vulnerable to reflected XSS.

This way the attacker can execute javascript in the victims browser, he could also mask the javascript code by encoding the characters in a way that doesnt look very suspicious. For example with [URL encoding](https://en.wikipedia.org/wiki/Percent-encoding):

`https://example.com?keyword=<script>alert('XSS')</script>` 

Would look like this: 

`https://example.com?keyword=%3c%73%63%72%69%70%74%3e%61%6c%65%72%74%28%27%58%53%53%27%29%3c%2f%73%63%72%69%70%74%3e`

### How to prevent it?

Always sanitize and validate user input. The easiest way to do that is to use a library that validates and sanitizes user input. I really recommend reading through the [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html).

## DOM-Based XSS

This attack is pretty similar to Reflected XSS, but the problem is not on the server-side, but on **client-side code**, on the javascript code that is already on the site. The way it differs from a reflected XSS attack is that the input is processed by client-side code that manipulates the DOM. For example:

1. User sends following request:
    - `https://example.com#lang=en`
2. User recieves following content:
    ```html
        <h1 id="greeting">Welcome!</h1>
        <p id="lang">Language is set to: English</p>
        <script>
            const lang = location.hash.match(/=(.*)$/)[1] // Content after the = in #lang=en
            const langP = document.querySelector("#lang") // The <p> tag
            const greetingH1 = document.querySelector("#greeting") // The <h1> tag

            switch(lang) {
                case "en":
                    greetingH1.innerHTML    = "Welcome!" 
                    langP.innerHTML         = "Language is set to: English"
                    break;
                case "pl":
                    greetingH1.innerHTML    = "Witajcie!"
                    langP.innerHTML         = "Język ustawiony na: Polski"
                    break;
                case "it":
                    greetingH1.innerHTML    = "Benvenuto!"
                    langP.innerHTML         = "La lingua è impostata su: italiano"
                    break;
                default:
                    langP.innerHTML         = `No language called ${lang}`

            }
        </script>
    ```
3. After inspecting the script tag content, the user sent the following request:
    - `https://example.com#lang=<img src=/ onerror="alert('DOM XSS')">`

DOM-based XSS attack vectors can also be **undetectable** if fragments are used on the site (#), because fragments never get sent to the server, whereas query strings are. This kind of attack is more and more common as web applications become more advanced. Bare in mind that even if a website has a completely secure backend, that does not mean that the frontend will handle user input safely.

### How to prevent it?

OWASP has an extension guide for DOM-Based XSS [here.](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)

## Persistent / Stored XSS

This is one of the more dangerous XSS attacks out there, since the malicious javascript is returned by the server itself. The way this is dangerous is that nobody really knows if the next page they load on the browser will be malicious or not, because the malicious code is not visible within the URL. This type of attack focuses on sending javascript code to a server, that later saves it in it's database and outputs it for everyone that views the page with that output from the database. A simple example would be:

- An attacker creates a post with javascript code as the content on a social media page.
- The server sends that input to a database.
- After some time, a user goes on a page where this post is fetched from the database.
- The server returns the post from the database with the malicious javascript to the victim.
- The malicious javascript code is executed.

### How to prevent it?

Similarly to the way reflected XSS is prevented.

## Tools for XSS Scanning

There are a ton of XSS scanning tools that I will describe in the next update.

## Useful Resource Links

- [My Node.js XSS challenge project](https://github.com/Artideusz/XSS-Challenges)
- [Excess XSS - a good explanation of XSS](https://excess-xss.com/)
- [OWASP - DOM-Based XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
- [OWASP - XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP - XSS Filter Evasion Cheat Sheet](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)
- [Kurobeats - XSS Vectors Cheat Sheet](https://gist.github.com/kurobeats/9a613c9ab68914312cbb415134795b45)
- [Google Gruyere](https://google-gruyere.appspot.com/)
- [DOM-Based XSS Explanation](http://www.webappsec.org/projects/articles/071105.shtml)
