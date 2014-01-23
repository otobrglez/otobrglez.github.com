---
layout: post
title:  "Practical example of dependency injection in Ruby"
date:   2014-01-20 12:00:00
comments: true
categories: hacking
image: /images/002-di-ruby.png
---

Today while working on my next great and amaizing project I used programming pattern called dependency injection.

Dependency injection is software design pattern that allows the removal of hard-coded dependencies and makes it possible to change them in "run-time" or "compile time".

We have a class/model called ```Flow``` that after person is added to ```Flow``` sends email to person who was added. Since ```Flow``` needs ```FlowMailer``` to send emails that means that there is dependency between ```Flow``` and ```FlowMailer```.

This is how the code looks without pettern.

```ruby
class Flow

  def add_access! user
    access = accesses.create!(user)
    FlowMailer.access_created access
  end

end
```

And this is how the class looks with dependency injection pattern.

```ruby
class Flow

  attr_writer :mailer
  def mailer
    @mailer ||= FlowMailer
  end

  def add_access! user
    access = accesses.create!(user)
    mailer.access_created access
  end

end
```

# Why is this better?

Because now among other things you can test your code decupled and you can swap mailer anytime you want with whatever you want. Like with double for example.

```ruby
describe Flow do
  let!(:flow){ create :flow }
  let(:user){ flow.creator }
  let(:other_user){ create :user }

  context "#add_access!" do
    # Create double
    let(:mailer) { double("FlowMailer") }

    # "Inject" double into object
    before { flow.mailer = mailer }

    it do
      expect(mailer).to receive(:access_created).once

      expect { flow.add_access!(other_user) }
      .to change(flow.flow_accesses, :count).from(1).to(2)
    end

end
```

To find more about dependency injection I suggest that you also look int this resources:

- [Wikipedia - Dependency Injection](http://en.wikipedia.org/wiki/Dependency_injection)
- [Stackoverflow - What is dependency injection?](http://stackoverflow.com/questions/130794/what-is-dependency-injection)
- [Kresimir Bojcic - Dependency Injection In Ruby](http://kresimirbojcic.com/2011/11/19/dependency-injection-in-ruby.html)
