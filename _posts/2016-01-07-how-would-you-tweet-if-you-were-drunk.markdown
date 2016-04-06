---
layout:     post
title:      "How would you tweet if you were drunk?"
subtitle:   "An experimental bot that drunk tweets using your own twitter vocabulary"
date:       2016-01-07 17:12:38
author:     "Vinay"
header-img: "img/post-bg-drunk-tweet-bot.jpg"
---

### Why did I make a drunk twitter bot?

There are plenty of things I love about being a Londoner. One of those is hanging out with my pals in one of the many amazing pubs. Not because I love to drink, but because my friends turn hilarious when they are too drunk.

It's always fascinating to me, as someone who's never been drunk, to see the language capabilities of my friends deteriorate after every drink; it's been the cause of many a drunk whatsapp text messages that were hilarious the next day.

Got me thinking though, Can I teach a bot to construct sentences in a way drunk people would? Well, that was the start of a few fun days of programming, and the result is this <a href="https://twitter.com/drunktweetbot" target="_blank">Drunk Twitter Bot</a>

#### Okay... So what does it do?

Exactly what the above section summarises. The bot goes through your tweets, dynamically constructs a lexicography of your twitter vocabulary, gets drunk, constructs a sentence, and tweets it back to you. Result is a tweet that *could* be similar if you tweeted while you were drunk.

Here are some drunk tweets that are constructed this way: 

<blockquote class="twitter-tweet" data-lang="en-gb"><p lang="en" dir="ltr"><a href="https://twitter.com/Ed_Johnson">@Ed_Johnson</a> <a href="https://twitter.com/hashtag/IfIWereYouAndDrunk?src=hash">#IfIWereYouAndDrunk</a> : The Hawks have been holding back for some cheesecake....nah man</p>&mdash; Drunk Tweeter (@drunktweetbot) <a href="https://twitter.com/drunktweetbot/status/602685532452343808">25 May 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet" data-lang="en-gb"><p lang="en" dir="ltr"><a href="https://twitter.com/dzhdzhdijidiji">@dzhdzhdijidiji</a> <a href="https://twitter.com/hashtag/IfIWereYouAndDrunk?src=hash">#IfIWereYouAndDrunk</a> tfw you come from the person in front of the pit lord</p>&mdash; Drunk Tweeter (@drunktweetbot) <a href="https://twitter.com/drunktweetbot/status/602705663769354240">25 May 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet" data-lang="en-gb"><p lang="en" dir="ltr"><a href="https://twitter.com/SenoyaDriskell">@SenoyaDriskell</a> <a href="https://twitter.com/hashtag/IfIWereYouAndDrunk?src=hash">#IfIWereYouAndDrunk</a> YIPPEE!! Thanks for the most importantly... No expectation…</p>&mdash; Drunk Tweeter (@drunktweetbot) <a href="https://twitter.com/drunktweetbot/status/602856685015846912">25 May 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet" data-lang="en-gb"><p lang="en" dir="ltr"><a href="https://twitter.com/maggi_nnn">@maggi_nnn</a> <a href="https://twitter.com/hashtag/IfIWereYouAndDrunk?src=hash">#IfIWereYouAndDrunk</a> : भाग बहनचोद। : The most unselfish love involves the worst way round.</p>&mdash; Drunk Tweeter (@drunktweetbot) <a href="https://twitter.com/drunktweetbot/status/601979877860999168">23 May 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


### The making

One of the first challenges for me when I started working on this idea was to figure out a way to construct a lexicography / dictionary of all the words in English language, and then figure out the symantics and relationship between words so I can construct gramatically valid sentences. As a first prototype, I tried doing that with <a href="http://neo4j.com/" target="_blank">Neo4j</a>; a NoSql graph database built exactly for this kind of scenarios. 

However, as soon as I started the tedious process of normalizing and converting all the symantics into nodes and relationships for graph database, it became clear to me that this method is probably not very feasible. It would take a lot of manual work on my part to keep it upto date. What if I wanted to expand it to other languages? Oh, let's not forget all the made up words that people use these days.

I wanted to be a bit smart about it and make it dynamic so that the bot would learn the language constantly and adapt to the habits of the people it's trying to immitate. This is also one of the reasons I chose to go the twitter route. Where else am I going to find inexhaustible number of people talking about various things constantly? 

With twitter, Not only I had a ready stream of words pouring in from all over the world, but it was also in various languages, should I chose to expand the bot to other languages.

Once I had these conclusions drawn, I quickly abandoned the idea of manually constructing the symantics for the language as natural language processing was a much smarter idea.

### Enter Markov's Chain

The twitter bot now looks at trending stream, picks up random user, constructs a dictionary of all the words in all the tweets in that account simply by counting all the occurance of a word w.r.t. the count of words that follows the given word, and the word that is followed by the given word.

For example, consider this sentence: `"It was a nice day. All the nice days had plenty sun."` Looking at the word `day`, there are two occurances. (I <a href="https://en.wikipedia.org/wiki/Stemming" target="_blank">stem</a> the words so the comparision is more accurate). `Day` is followed by `all`, and `had`. It is followed by `nice` twice though. So when the bot is constructing a sentence and comes across the word `nice`, the probability of the next word being `day` is higher than other words.

For this to happen, I construct a "spinning roulette wheel", and select the next word in sequence for the sentence based on the weight of the word, proportional to the times the next word occurred. The code for this block is: 

	(defn wrand
	  "given a vector of slice sizes, returns the index of a slice given a
	  random spin of a roulette wheel with compartments proportional to
	  slices."
	  [slices]
	  (let [total (reduce + slices)
	        r (rand total)]
	    (loop [i 0 sum 0]
	      (if (< r (+ (slices i) sum))
	        i
	        (recur (inc i) (+ (slices i) sum))))))

I loop through the dictionary and construct an array of words repeated proportional to the times they occured in the dictionary and select a random word from the array. Additionally, I mix up the proportion a little bit to make it seem like a "drunk" sentence.

Since the sentence is constructed based entirely on the <a href="https://en.wikipedia.org/wiki/Markov_chain" target="_blank">markov's chain</a> algorithm, It has an added benefit of the sentence being in any language.

<blockquote class="twitter-tweet" data-lang="en-gb"><p lang="th" dir="ltr"><a href="https://twitter.com/NuDdy_YH">@NuDdy_YH</a> <a href="https://twitter.com/hashtag/IfIWereYouAndDrunk?src=hash">#IfIWereYouAndDrunk</a> ก้อเดินหน้าต่อไปนะ ทุกคนทำหน้าที่ตัวเองให้ดีที่สุด มีความสุขกับที่เปน จบ</p>&mdash; Drunk Tweeter (@drunktweetbot) <a href="https://twitter.com/drunktweetbot/status/602066454993903616">23 May 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Looks like I offended someone with that tweet! The original tweeter had a few choice words to say about that tweet when he read it!

### Room for improvement

Though the basics of the sentence generation is in place, it could be made faster. It constructs the dictionary for every user by placing them in memory. Perhaps changing the dictionary to a better data structure and using redis is a way to make it faster, it needs a bit of research.

I'd also like to add some interactive-ness to the bot by making it respond to users. If someone wanted to see their own drunk tweet, they could tweet the bot and ask for a drunk tweet. I'll have to find some time to make these improvements, and when I do, I'll update this blog post.
