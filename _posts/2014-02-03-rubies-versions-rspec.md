---
layout: post
title:  "RSpec testing with different rubies"
date:   2014-02-03 12:00:00
comments: true
categories: ruby
image: /images/004-databox.png
---

I'm working on [Ruby gem for Databox API](https://github.com/otobrglez/databox) and I needed a way to test it locally under different versions of Ruby.

# What is Databox?

Databox is the mobile dashboard for executives and the most advanced business intelligence tool for smart-phones. Basically they provide endpoint to witch you can push your data and their wonderful client will display it perfectly on your smart-phone. Check out [their homepage](http://databox.com) for more information.

# RVM to help

The easies way is to use different Rubies with different gemsets and then execute RSpec inside it.

1. Install Rubies that you want to use

  ```
  rvm install ruby-1.9.3
  rvm install ruby-2.0.0
  ```

2. Create RVM gemsets

  ```
  rvm use --create ruby-1.9.3@databox
  rvm use --create ruby-2.0.0@databox
  ```

Then write scripts that looks somethings like this.

```bash
#!/usr/bin/env bash

# Define different Gemsets
declare -a SETS=(\
  'ruby-1.9.3-p484@databox' \
  'ruby-2.0.0-p353@databox' \
  # Add others here
)

# Loop over Gemsets and exec RSpec
for r in "${SETS[@]}"; do
  set -o verbose
  echo "Testing $r"

  rvm $r exec bundle install --quiet
  rvm $r exec bundle exec rspec --fail-fast --format=progress
done
```

The Bash script will loop over ```SETS``` and execute RSpec for each set. The output will look something like this.

```bash
Testing ruby-1.9.3-p484@databox
....................

Finished in 0.16668 seconds
20 examples, 0 failures
Testing ruby-2.0.0-p353@databox
....................

Finished in 0.15869 seconds
20 examples, 0 failures
```

**And thats it. :)**

P.s.: If you are using [Travis](http://travis-ci.org) for your CI. You can easily add this to your ```.travis.yml.``` and Travis will run your code with different versions of Ruby each time you push your code. Read more about this functionality under [choosing Ruby versions and implementations to test against](http://docs.travis-ci.com/user/languages/ruby/#Choosing-Ruby-versions-and-implementations-to-test-against).

```yaml
rvm:
  - 1.9.3
  - 2.0.0
```



