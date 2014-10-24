---
layout: post
title:  "Aloo - GIST for data / RailsRumble 2014"
date:   2014-10-24 12:00:00
comments: true
categories: rails
image: /images/011-aloo.png
---

Previous weekend I've participated in annual [Rails Rumble 2014][ror14]. On this special hackathon I've build app/product named [Aloo]. This is the story behind it, some details and some problems that I've discovered on my journey.

## What is RailsRumble?

[Rails Rumble 2014][ror14] was hackathon that begun Saturday, October 18th at 00:00:00 UTC (midnight) and lasted for the next 48 hours, until Sunday, October 19th at 23:59:59 UTC. Teams could be sized between 1 and 4 members, nothing can be done previous to event and if you push changes after finish you are disqualified. Apps must be build with some kind of Ruby technology, they have to use something related to [Rack middle-ware][rack] and end product has to be deployed to [Heroku]. The test is imagination, speed and execution. Check out [FAQ][rrfaq] for more details.

## Aloo idea

I wanted to build something that will be related to **data**, **analytics** and would work in real or at least **near real-time**. I thought how nice it would be if there would be just this simple endpoint on which I could just send my data and the engine would build warehouse/cube on the fly. And this is how [Aloo] was born.

## Personal rules

Since I was doing this alone, I needed some kind of plan and strict rules on what and how to build. This rules that I've set for myself ware simple.

* Build solid core functionality with proper tests on first day.
* Push real data to business layer and test if it works.
* Build simple UI layer that would display real data in 1 way.
* Add more functionality in layers.
* Idea is complex, people need explanation! Think about UX!

## How Aloo front-end works?

If you visit [Aloo], on landing page you will see large call to action button - **Create new dashboard for free!**. If you click on it, modal form appears where you enter name of your dashboard; few example names are pre-filled - to give visitors idea what they can do with it.

<div class='center-img'><img src='/images/011-aloo_form.png'/></div>

After you create new board your are redirected to your new dashboard, where you will see empty charts, empty tables. Another modal box will appear. This one gives you instructions on how to push data to your board with `curl`. If you chose to execute given command with example data on your machine you will get visualization that will resemble this one,...

<div class='center-img'><img src='/images/011-aloo_dashboard.png'/></div>

Toggling between views will also display your data in quarters and with bars instead of line chart.

<div class='center-img'><img src='/images/011-aloo_dashboard_2.png'/></div>

Visualization on top of dashboard is done with [Chart.js](http://www.chartjs.org/), and it pulls data from table beneath the chart. There are 2 charts generated when you load the page, changing tabs toggles visibility of each chart. Its not ideal, but for prototype of this kind it works and it renders nicely and fast.

For layout I've used [Bootstrap] with some modifications, and I helped myself with [Compass].

## How Aloo back-end works?

Each board you create with [Aloo] will get its own credentials, credentials that can be passed as basic [HTTP basic authentication headers](http://en.wikipedia.org/wiki/Basic_access_authentication) with each request.

Everything in [Aloo] is stored into [Redis], board as [hash](http://redis.io/commands#hash), the rest is done in pure [key-value fashion](http://redis.io/commands#string). When [Aloo] receives your `[[date, value],..` array it will generate bunch of key-value paris that will be stored into Redis the [pipelined way](https://github.com/redis/redis-rb#pipelining).

```ruby
# Computing keys that will go into Redis
Kpi.new("sales","2014-12-10",100).keys
["sales:14", "sales:14:q:4", "sales:14:m:12", "sales:14:w0:49", "sales:14:d:344"]
```

This is snippet of code that generates them:

```ruby
  #...
  def year; @year ||= date.strftime('%y')                       end
  def quarter; @quarter ||= ((date.month - 1) / 3) + 1          end
  def month_in_year; @month_in_year ||= date.month              end
  def week_in_year; @week_in_year ||= date.strftime('%W').to_i  end
  def day_in_week; @day_in_week ||= date.wday                   end
  def day_in_year; @day_in_year ||= date.yday                   end
  def hour_in_day; @hour_in_day ||= date.hour                   end

  def keys
    @keys ||= [
      [key, year],                      # Yearly
      [key, year, 'q', quarter],        # Quarterly
      [key, year, 'm', month_in_year],  # Monthly
      [key, year, 'w0', week_in_year],  # Weekly
      [key, year, 'd', day_in_year]     # Daily
      # ...
    ].map {|k| k.join(":") }
  end
end
```

On the other side - when you want to get data back - I compute date range, build another set of keys and get the data back with [mget](http://redis.io/commands/mget). This works like this;

```ruby
Stats.new("sales").keys("2014-12-01","2014-12-07")
[
"sales:14", "sales:14:q:4",
"sales:14:m:12", "sales:14:w0:48",
"sales:14:d:335", "sales:14:d:336",
"sales:14:d:337", "sales:14:d:338",
"sales:14:d:339", "sales:14:d:340"
]
```

But since, I don't need all the keys that are suitable - I can also pass a little regular expression to my keys:

```ruby
Stats.new("sales",/\:d\:/).keys("2014-12-01","2014-12-07")
[
"sales:14:d:335", "sales:14:d:336",
"sales:14:d:337", "sales:14:d:338",
"sales:14:d:339", "sales:14:d:340"
]
```

So now all I have to do is display and sum all the values in right places and thats it.

## Lessons learned

This are things that I learned by doing rumble.

* I really wanted to combine this back-end with front-end with some real-time mechanism like [Web Sockets][ws] or [Server-sent events][sse] and do visualisation as the data gets in. I tried to do this with [Puma] and [ActionController::Live][acl], but for some reason, that I didn't have time to inspect further I got locks all over the place so I had to abandon that. It made me sad. I home that I'll be able to use it in some other project.
* My idea is quite complex to explain to general visitor, even if I add examples, using `curl` to send data to [Aloo] it just feels to hackish, I should do this with classical 'upload-your-CSV-file-here-button'
* Its hard to 'sell' analytics to anyone who doesn't understand or need it. ^^
* I did this all solo in ~35 hours. Imagine what could be done if I would connect with more people,... I should definitely find someone to join next year. Perhaps you, dear reader?
* Writing code is just part of the competition, 'selling' the idea to 'judges' is the second part; I suck at that. I should get someone to do more PR for me.

Thanks for reading. Please let me know if you feel that [Aloo] is the real deal, also let me know if you've build something nice this year!

See you guys next year!

P.s.: I'm also still waiting for the final results. You can [vote for Aloo here](railsrumble.com/entries/153-aloo-business-analytics-fast).

[Heroku]:http://www.heroku.com/
[Aloo]:http://www.aloo.io/
[ror14]:http://railsrumble.com/
[rack]:https://github.com/rack/rack
[rrfaq]:http://blog.railsrumble.com/about/
[Bootstrap]:http://getbootstrap.com
[Redis]:http://redis.io/
[Compass]:http://compass-style.org/
[sse]:http://en.wikipedia.org/wiki/Server-sent_events
[ws]:http://en.wikipedia.org/wiki/WebSocket
[acl]:http://api.rubyonrails.org/classes/ActionController/Live.html
[Puma]:http://puma.io/
