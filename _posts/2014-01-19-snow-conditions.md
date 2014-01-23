---
layout: post
title:  "Snow conditions in Slovenia"
date:   2014-01-19 12:00:00
comments: true
categories: hacking
image: /images/001-pushover.png

---

Since the winter is here and snow season is about to open in Slovenia I wanted to create a little project that would notify me when [Slovenian Environment Agency (ARSO)][arso] publishes new so called "snow report".

Just to make this challenge more interesting I wanted to get push notification to my phone as soon as new report is published.

This is how I solved the challenge.

## Analyze "the source"

[ARSO][arso] publishes reports on their [web page][arso-sneg-raz] with some mystery frequency. Some times they publish reports on Monday, some times on Thursday and on some occasions even on Saturday. Time is even stranger, sometimes early in the morning and some times in the evening. That basically means that there is no real schedule that I can rely on to check for new content. Great.

## Analyze "content" and write parser

Content on [web page][arso-sneg-raz] is just regular HTML that can easily be parsed with simple [XPATH][xpath] expression. No big problems there. Here is hypothetical parser, I've used [HTTParty](httparty) and [Nokogiri](nokogiri) - complete parser can be found in [lib/snow.rb][snow.rb].

{% highlight ruby %}
require "bundler/setup"
require "httparty"
require "nokogiri"

# URL of source
url = "http://www.arso.gov.si/vreme/"
url += "napovedi%20in%20podatki/snegraz.html"

# Request with HTTParty
response = HTTParty.get(url)
# New Nokogiri::HTML
document = Nokogiri::HTML(response.body)

# Content parsed with XPATH
content = document.xpath("//td[@class='vsebina']/p")
puts content

{% endhighlight %}

## Periodical scraping

Since the site has no schedule on when they release reports I had to write simple mechanism - I call it [bot.rb][bot.rb] - that fetches content every 30 minutes. For this task I used [Clockwork Gem](https://github.com/tomykaira/clockwork)

{% highlight ruby %}
# Require parser
require_relative "snow"
require "clockwork"

module Clockwork

  handler do |job, time|
    if job == "snow_parse.job"
      # Call parser here. Parser outputs key (for logs)
      puts Snow.process.key
    end
  end

  # Schedule it to 30 minutes
  every(30.minutes, 'snow_parse.job')
end
{% endhighlight %}

Since the code runs on [Heroku][heroku]. I've written simple [Procfile][heroku-procfile] with following definition.
{% highlight bash %}
bot: bundle exec clockwork lib/bot.rb
{% endhighlight %}

## Handling duplicated reports

Parser works so that it generates [SHA1](http://en.wikipedia.org/wiki/SHA-1) hash from parameters in report. Then it checks [Redis](http://redis.io) if content with existing hash already exists. If no content with hash is found it dispatches push and stores hash for next time.

{% highlight ruby %}
# SnowInformation is abstraction over content.
SnowInformation = Struct.new(:date, :level, :details) do

  # Parse date
  def date
    @date ||= Date.parse(self[:date])
  end

  # Generate SHA1 hash
  def sha
    @sha ||= Digest::SHA1.hexdigest (values * "-")
  end

  def key
    "#{date}-#{sha}"
  end

end
{% endhighlight %}

Processing is done in snow.rb and it looks like this:

{% highlight ruby %}
  class Snow
    # ...

    def process
      # Get SnowInformation from "state" variable
      info = state

      # Check if "key" exists in database
      unless redis.exists info.key

        # Store key in Redis
        redis.set info.key, info.value #, {ex: 604800} # 7 days

        # Notify with push
        notify info
      end

      # Return SnowInformation
      info
    end
  end
{% endhighlight %}

# Push notifications

For dispatching notifications to my phone I used [Pushover](https://pushover.net/). [Pushover](https://pushover.net/) is simple push service that makes it easy to send real-time notifications to your [Android](https://play.google.com/store/apps/details?id=net.superblock.pushover&ts=1390224423) and [iOS devices](https://itunes.apple.com/us/app/pushover-notifications/id506088175?ls=1&mt=8).

This is how I wrapped their service with [HTTParty][httparty].

{% highlight ruby %}
class Pushover
  include HTTParty
  base_uri 'https://api.pushover.net/1'

  def initialize params={}
    @params = params.merge!({
      token: ENV["PUSHOVER_KEY"],
      user: ENV["PUSHOVER_USER"]
    })
  end

  def push message, options={}
    self.class.post("/messages.json", body: @params.merge!({
      message: message
    }).merge!(options))
  end

end
{% endhighlight %}

And then in code I can just do
{% highlight ruby %}
def notify info
  pushover.push(info.level, {
    title: "Snezne razmere",
    url_title: "Snezne razmere #{info.date}",
    url: ...
  })
end
{% endhighlight %}

# The result

When everything was put together and [properly tested](https://github.com/otobrglez/snow-conditions/tree/master/spec). I started getting snow reports as soon as parser detected changes on site. This is how my [Pushover](https://pushover.net/) client looks on a busy month.

![snow-conditions screenshot](https://photos-5.dropbox.com/t/0/AAAoY1C_pJOivd514c0fe_nCeNk3X1863EiosumpAA2J7Q/12/697441/png/1024x768/3/1390230000/0/2/snow-conditions.png/CtU659fhKGR2aZ3aS5G3L9nPdkpeq81dv8iR1UQ44ew)

Whole [source code](https://github.com/otobrglez/snow-conditions) of snow-conditions project and setup process for [Heroku](https://heroku.com) can be found on my [GitHub profile](https://github.com/otobrglez).


[arso]: http://arso.gov.si
[arso-sneg-raz]: http://www.arso.gov.si/vreme/napovedi%20in%20podatki/snegraz.html
[xpath]: http://www.w3schools.com/xpath/
[httparty]: https://github.com/jnunemaker/httparty
[nokogiri]: http://nokogiri.org/
[bot.rb]: https://github.com/otobrglez/snow-conditions/blob/master/lib/bot.rb
[snow.rb]: https://github.com/otobrglez/snow-conditions/blob/master/lib/snow.rb
[heroku]: https://heroku.com
[heroku-procfile]: https://devcenter.heroku.com/articles/procfile
