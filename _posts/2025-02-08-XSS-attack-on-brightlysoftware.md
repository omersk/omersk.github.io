# The Art of Hacking: A New Beginning

Hey there, fellow internet adventurers!

So, my first blog post? Yeah, let’s just say it ended with a bang—but not the kind I was hoping for. I found a reflected XSS vulnerability, only to be thwarted by the most annoying of enemies: a lack of full exploitation. It’s like finding a treasure chest, only to realize you don’t have the key. So, no bug bounty glory for me... yet.

I was about to pass out into a blissful nap after a hard day of slaying secure programming assessments (sounds fun, right?). But just as my eyelids were staging a peaceful protest, a thought struck me—BAM! Google Dorks! Those magical search queries that unlock the hidden backdoors of the web.

And just like that, my nap plans were squashed by the siren call of hacking. I knew what I had to do.

My mission? Use Google Dorks to find my next victim. Target locked. First dork: inurl:/About Us. A simple query. A quick browse through the results, and—BOOM! I stumbled upon a website that had the potential to be a goldmine. But don’t take my word for it—let me show you.

![Brightlysoftware About Us](/assets/img/Brightlysoftware/Brightlysoftware_about_us.jpg)

# Research
So I clicked on this website, and guess what I saw? The holy grail of web exploitation—a search button. Seriously, is there anything more thrilling to a hacker than that?

![Brightlysoftware Search Button](/assets/img/Brightlysoftware/Brightlysoftware_search_button.jpg)

Feeling like a digital Sherlock Holmes, I decided to test my skills. I typed “AAA” into the search bar, hoping it would trigger a reflected XSS vulnerability. You know, the kind where your input comes back at you like a boomerang. Guess what happened?

![Brightlysoftware Search Button AAA](/assets/img/Brightlysoftware/Brightlysoftware_search_button_aaa.jpg)

Yes! The text came back to me just like I hoped! It was like I had found a secret passage in a video game. Things were looking up. So, naturally, I got cocky.
Time to test some real XSS attacks. Let's start simple: bold text.

![Brightlysoftware Search Button Bold](/assets/img/Brightlysoftware/Brightlysoftware_search_button_bold.jpg)

Nice. It worked. The website understood bold text. I was feeling like a god of the digital realm.

But wait. What about a script? You know, something with an alert box?

![Brightlysoftware Search Button Script Alert](/assets/img/Brightlysoftware/Brightlysoftware_search_button_script_alert.jpg)

...Blocked. Seriously? That’s a hard pass from the site. Access Denied, my old enemy.

At this point, I was about to just throw in the towel and take that nap. But then... a lightbulb moment! I Googled “Access Denied” and found this gem:

![Brightlysoftware Search Button Script Alert](/assets/img/Brightlysoftware/Brightlysoftware_stackoverflow_question.jpg)
![Brightlysoftware Search Button Script Alert](/assets/img/Brightlysoftware/Brightlysoftware_stackoverflow_answer.jpg)

Could this be a sign? I was about to find out. With my hacking instincts tingling, I decided to check whether the website was using a blacklist to block certain queries. I tested a few things, like this harmless little query:

```
<img src=aaa>
```

It worked! The website was like, “Sure, we’re fine with images... mostly.”

But then I tried something a little more mischievous:
```
<img src=aaa onerror=alert(1)
```
Blocked. The blacklist was awake. But here’s where it gets interesting: I searched for the magic words—alert(1)—and BOOM! Guess what I got?

That’s right—Access Denied again! So, I had cracked the code. The website was using a blacklist. The stakes were high, and I was on the verge of a breakthrough. There was no turning back.

## Time to Break Out the Big Guns
With a cheeky grin, I grabbed my trusty Python script and started making some beautiful chaos. Here’s the code I used to test the XSS queries against the site’s blacklist:

```python
import requests

BASE_URL = b'https://www.brightlysoftware.com/search?field_industries=All&search='

potentially_work = []

with open("xss.txt", "rb") as f:
    lines = f.readlines()

for line in lines:
    result = requests.get(BASE_URL + line).text
    if "Access Denied" not in result:
        potentially_work.append(line)

    print(potentially_work)
```

I ran the script, and—drumroll—boom. Some queries worked. This was my moment. One worked so well, I thought I might’ve just unlocked the Matrix.

![Brightlysoftware Search Button Script Alert](/assets/img/Brightlysoftware/Brightlysoftware_poc.jpg)

With that, I was ready for my nap. The hacking gods were pleased with me. 😎


At this moment, I was ready to take my nap :)

# The Epic Conclusion:
So, to summarize: I found a website with a sweet little reflected XSS vulnerability, successfully exploited it, and then sent them a polite ticket (you know, the “hey, you might want to fix this” kind of ticket).

And just like that, I was off to dreamland. Sweet, sweet hacking dreams.

Takeaways:

*** Google Dorks = Power. Use it wisely.
*** XSS vulnerabilities? They're everywhere, just waiting for you to say, "Hello!"
*** Blacklists are not invincible. Keep digging, and you'll find the cracks.
*** And most importantly... naps are essential. 💤

Hope you enjoyed the ride! Don’t forget to check for those security vulnerabilities while you're Googling your next pizza order.
