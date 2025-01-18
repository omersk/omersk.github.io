## Intro
Hi there! This is my very first blog post, and I couldn’t be more excited to share it with you! 😊

The story behind this post might seem like it begins in the dullest way imaginable—homework for my M.E. studies at the prestigious Technion Institution. But don’t let that fool you; what starts as a routine academic assignment takes a fascinating turn into the thrilling world of cybersecurity.

As a passionate tech enthusiast with a deep interest in uncovering security vulnerabilities, I always gravitated toward the most challenging and relevant courses the Technion had to offer. Though these courses were often the hardest and most time-consuming, they were always worth the effort.

From Reverse Engineering (236496) to Operating Systems (236376) to Introduction to Learning Systems (236501), my journey through these courses ultimately led me to what I consider the crown jewel of cybersecurity studies: Secure Programming (236491). This course, while notoriously difficult, turned out to be my playground, as I was already familiar with many of its core topics. My experience competing in CTFs like pwnable.kr and hackthissite.com had equipped me with the skills to tackle most of the material with ease.

But then came an unexpected twist. Today, I encountered an assignment on XSS attacks, the one topic I hadn’t explored before. Within minutes of diving into it, I was blown away! Completing the assignment in record time, I felt an unstoppable urge to channel my newfound knowledge into something bigger.

I immediately googled the first bug bounty program I could find on HackerOne and stumbled upon Lightspark’s challenge:

![Lightspark BugBounty](/assets/img/Lightspark-BugBounty.jpg)

And that’s where the real adventure began...

## Research
I opened the website and searched for a common XSS structure: a text box that takes input and displays it somewhere on the screen.

After browsing through several pages, I found the following page:
![Lightspark SupportPage](/assets/img/Lightspark-SupportPage.jpg)


Cool! Let's try entering some random input to see if it echoes back our text as "Not Found."
![Lightspark BoldText](/assets/img/Lightspark-NotFoundPage.jpg)


Amazing!

Now, let's experiment by inserting some common XSS attack patterns:

Test 1: Basic HTML Tags
Input:
\<b\>BoldText\</b\>

Result:
This didn't work. The \<b\> tags were removed when I checked the output:

![Lightspark BoldText](/assets/img/Lightspark-BoldText.jpg)

Interestingly, examining the HTML revealed that the text was wrapped under:
\<strong\>“BoldText”\</strong\>

Alright, this is intriguing. Let's proceed with more common XSS payloads.

Test 2: JavaScript Injection
Input:
\<script\>alert(1)\</script\>

Result:
The displayed output was:

![Lightspark BoldText](/assets/img/Lightspark-Alert.jpg)

And the HTML contained:
\<strong\>“”\</strong\>

Unfortunately, the script tags were completely sanitized. No alert was triggered.

Test 3: Encoded Characters
Next, I attempted to bypass the filters by encoding the script tags, like this:
&lt;script&gt;alert(1)&lt;/script&gt;

Result:
The output looked like:
\<strong\>“\<script\>alert(1)\</script\>”\</strong\>

But again, no alert was triggered. Most likely, this is because the script was treated as plain text within quotation marks.

Test 4: Using Alternative Payloads
I decided to step up the game and use a more sophisticated approach. Based on ChatGPT's suggestion:

"Sometimes, certain event handlers or elements like \<img\> or \<svg\> might be overlooked by filters. For example:
\<img src="x" onerror="alert(1)"\>"

Let's give this a try:


Result:
![Lightspark BoldText](/assets/img/Lightspark-XSS.jpg)
It worked!!!

