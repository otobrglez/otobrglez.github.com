---
layout: post
title:  "Snow conditions in Slovenia"
date:   2014-01-20 12:00:00
categories: hacking
---

Since the winter is here and snow season is about to open in Slovenia I wanted to create a little project that would notify me when [Slovenian Environment Agency (ARSO)][arso] publishes new so called "snow report".

Just to make this challenge more interesting I wanted to get push notification to my phone as soon as new report is published.

This is how I solved the challenge.

## Analyze "the source"

[ARSO][arso] publishes reports on their [web page][arso-sneg-raz] with some mystery frequency. Some times they publish reports on Monday, some times on Thursday and on some occasions even on Saturday. Time is even stranger, sometimes early in the morning and some times in the evening. That basically means that there is no real schedule that I can rely on to check for new content. Great.

## Analyze "content" and write parser

Content on [web page][arso-sneg-raz] is just regular HTML that can easily be parsed with simple [XPATH][xpath] expression. No big problems there. Here is hypothetical parser, I've used [HTTParty](httparty) and [Nokogiri](nokogiri).

{% highlight ruby %}
require "bundler/setup"
require "httparty"
require "nokogiri"

# URL of source
url = "http://www.arso.gov.si/vreme/napovedi%20in%20podatki/snegraz.html"

response = HTTParty.get(url)                # Request with HTTParty
document = Nokogiri::HTML(response.body)    # New Nokogiri::HTML

# Content parsed with XPATH
content = document.xpath("//td[@class='vsebina']/p")
puts content

{% endhighlight %}


## Periodical scraping

Since the site has no schedule on when they release reports I had to write simple "periodical" mechanism (I call it bot) that fetches content every 30 minutes.


[arso]: http://arso.gov.si
[arso-sneg-raz]: http://www.arso.gov.si/vreme/napovedi%20in%20podatki/snegraz.html
[xpath]: http://www.w3schools.com/xpath/
[httparty]: https://github.com/jnunemaker/httparty
[nokogiri]: http://nokogiri.org/
