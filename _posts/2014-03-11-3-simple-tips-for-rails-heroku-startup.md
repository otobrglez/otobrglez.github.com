---
layout: post
title:  "3 simple tips for Rails / Heroku startup"
date:   2014-03-11 12:00:00
comments: true
categories: startups
image: /images/006-heroku.png
---

Yesterday I visited old collage of mine who is working with his team at little startup called [ViaEmail](http://www.viaemail.me/). Like a lot of startups out-there he is using [Heroku](http://heroku.com) for his hosting platform of choice. Beginner on low-budget, but moving fast. After giving me nice demonstration I really wanted to help out this guy with some nice and useful tips. There are some of them.

## Your app is "sleeping"?

If you don't have any request to your Heroku instance it will go to "idle". Because of the [nature](https://devcenter.heroku.com/articles/dynos) how Heroku works, the app will be feeling really slow and sluggish. There are few ways that can solve this.

1. One way is to scale your dynos, to at least 2. Simple. But you have to [pay for extra dyno](https://www.heroku.com/pricing).

2. Second way - and probably the cheapest - is to create another Heroku instance and put a simple script that would periodical trigger requests to your app. This will prevent your app from going into idle.

  I wrote a little Ruby app that does exactly that. Its called [dontdieonme](https://github.com/otobrglez/dontdieonme). To use it; create new Heroku instance, and push this code to it. Set environment variable ```URL_TO_GET``` and I also suggest that you add add Rollbar to project so that you can see if there is something wrong. Restart the instance. The app will trigger request every 2 minutes.

3. **Update.** [Miha Rekar](http://mr.si/) also proposed 3rd way. Add [NewRelic](https://addons.heroku.com/newrelic) to your Heroku instance and configure [availability monitoring](https://docs.newrelic.com/docs/alerts/availability-monitoring). NewRelic will **ping** your app every 10 minutes or so.

## You are serving static assets from Heroku?

Second problem that most beginners don't know is that Heroku is NOT meant to be used for serving static assets and if you are serving "non-changing" JavaScript or CSS from your Heroku instance you are doing it wrong! Best place for your static assets is CDN; so put your files either on [Amazon S3](http://aws.amazon.com/s3/), [Google Cloud Storage](https://cloud.google.com/products/cloud-storage/), [Rackspace](http://www.rackspace.com/cloud/files/) or some other place. If your Heroku instance is serving static content its waisting computing power that could be better spent by your app.

Since manually compiling assets and moving them to cloud is a bit painful you can either automate the process with Rake script that looks something like [this](https://gist.github.com/otobrglez/1053855) or you can use [asset_async gem](https://github.com/rumblelabs/asset_sync). When using asset_async when you deploy your code to Heroku it will automatically compile all your assets and sync them with your cloud. Nice and easy.

## Should we write tests?

The third tip that I gave was more an answer than actual tip. The question was; *do you code with TDD principles, do you write test for everything?*

My answer was quite simple. If you are new to Rails, new to Ruby. Don't. First learn as much as you can to be comfortable around new "world". Get to know *everything*, try it out, learn by doing mistakes, try to debug the thing. Then - and only then - try to improve and extend your skills by writing test. Proper test should test the right things, the right way and by slowing down the test suit as less as possible. Writing tests properly also takes time in startup world - time next to team is everything.

I also recommend that you check out some of more popular open source Ruby / Rails project. Projects like [discourse](https://github.com/discourse/discourse), [monocle](https://github.com/maccman/monocle) and [others](http://www.opensourcerails.com/). Dig into their code, into their [test suites](https://github.com/discourse/discourse/tree/master/spec/models).


Hope that this tips can also save you some initial headaches. If not - let me know if I can help you out. :)







