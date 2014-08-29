---
layout: post
title:  "From Sinatra to Amazon Simple Queue Service (SQS)"
date:   2014-08-29 12:00:00
comments: true
categories: ruby
image: /images/009-aws.png
---

Amazon provides Simple Queue Service also knows as SQS. It is a fast, reliable, scalable,
fully managed message queuing service in the cloud that you can use from your applications.
I've decided to write a simple Sinatra application that pushes messages to queue and then on the other side worker that reads that messages.
This is how I did it...

## Example application

Create some common **boot.rb** file where you put your "initialisation" code in - something like this will work:

```ruby
require 'bundler/setup'

# Use this to load my '.env' that contains AWS credentials
require 'dotenv'; Dotenv.load

# We are using Amazon Ruby SDK
require 'aws-sdk'

# We configure AWS Ruby SDK
AWS.config(
  access_key_id: ENV.fetch('AWS_ACCESS_KEY_ID'),
  secret_access_key: ENV.fetch('AWS_SECRET_ACCESS_KEY')
)

# We get SQS Web Service
sqs = AWS::SQS.new

QUEUE = sqs.queues[ENV.fetch('AWS_QUEUE_URL')]
```

Simple Sinatra web **app.rb** that sends message yo your Amazon SQS if you visit /smoke.

```ruby
require_relative 'boot'
require 'sinatra'

get '/smoke' do
  message = QUEUE.send_message({email: "my@email.com", at: DateTime.now}.to_json.to_s)
  "Message was sent to Amazon SQS with id #{message.id}.\n"
end
```

Create another file - name it **worker.rb** - that will subscribe to your Amazon SQS queue and pull messages.

```ruby
require_relative 'boot'

QUEUE.poll do |msg|
  object = JSON.load(msg.body) rescue {}
  puts "Got your email #{object['email']} at #{object['at']}."
end
```

My local **.env** file contains credentials and URL to SQA queue.

```bash
AWS_ACCESS_KEY_ID=secret
AWS_SECRET_ACCESS_KEY=secret
AWS_QUEUE_URL=https://sqs.us-west-1.amazonaws.com/...
```

## Running it

Then you simply startup your [Sinatra] app and worker like so

```bash
ruby app.rb # in one terminal
ruby worker.rb # in another terminal
```

If you then visit `/smoke` you will see something like this - meaning that job was scheduled.

```
Message was sent to Amazon SQS with id 788e5e28-8055-4c8f-bb51-c634c327a021.
```

And in your worker terminal you'll see

```
Got your email my@email.com at 2014-08-29T15:20:38+02:00.
```

Now, you don't have to limit yourself to just one worker... You can have as many as you like, one message will be processed only on one worker by default.
Another note here is that we are sending JSON as message thats why I've used JSON.load in worker. Amazon SQS sends/receives plain text messages.

And thats how you use Amazon SQS. ;)

P.s.: This is code that I also wrote on [SO for someone](http://stackoverflow.com/questions/25567349/run-ruby-task-whenever-amazon-sqs-queue-is-updated).

[Sinatra]: http://www.sinatrarb.com/
