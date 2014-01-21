---
layout: post
title:  "Bower + Rails on Heroku and S3"
date:   2014-01-21 12:00:00
comments: true
categories: rails
---

My recent project is single-page application that has front-end build with [Marionette.js](http://marionettejs.com/) and back-end written in [Ruby on Rails](http://rubyonrails.org/). Since I want to use top front-end packages, as easy as possible I used "new" package manager for front-end called Bower.

# Enter Bower

[Bower](http://bower.io/) is front-end package management system, written in JavaScript and it uses [Node](http://nodejs.org/) and [npm](https://npmjs.org/) in its base. Packages are managed via Git and you can use any type of transport with it (AMD, CommonJS,...)

Visit [Bower](http://bower.io/) to learn more about and [browse packages](http://sindresorhus.com/bower-components/).

# Pre-requirements

- Install [Node](http://nodejs.org/) and [npm](https://npmjs.org/)
- Install [Ruby](https://www.ruby-lang.org/en/) & [Rails](http://rubyonrails.org/)
- Regular setup of your Heroku instance

# Setup

1. Update your ```Gemfile``` with ```execjs``` and ```asset_sync```.

  ```ruby
  gem 'rails'
  # ... whatever u need here
  gem 'execjs'
  gem 'asset_sync'
  ```

  [asset_sync](https://github.com/rumblelabs/asset_sync) Gem will synchronize assets between Rails (on Heroku) and your S3. Please configure asset_sync **before** going on. [Instructions are on Github](https://github.com/rumblelabs/asset_sync)

2. Enable assets precompilation on Heroku

  ```bash
  heroku labs:enable user-env-compile -a myapp
  ```

3. In top folder of your Rails project create file ```package.json```.

  ```json
  {
      "name": "myapp",
      "version": "0.0.1",
      "engines": {
        "node": "v0.10.x",
        "npm": "1.3.x"
      },
      "dependencies": {
        "bower": "1.2.x"
      },
      "scripts": {
        "postinstall": "bash prepare_bower.sh"
      },
      "repository": {
        "type" : "git",
        "url" : "http://github.com/me/myapp.git"
      }
  }
  ```

  [package.json](https://npmjs.org/doc/json.html) is standard "package" format for npm. Notice the ```postinstall```. ```postinstall``` is script that will be called after npm has installed package. In our case; after all packages from npm ware installed we want to prepare bower and its dependencies.

4. ```prepare_bower.sh``` is simple script that will check if bower_components ware already installed. If they ware; it will remove folder. *I came up with this script due to bug that coused problems on Heroku.*

  ```bash
  #!/usr/bin/env bash

  echo "Prepare Bower"

  if [ -d vendor/assets/bower_components ]; then
      echo "Removing components"
      rm -rf vendor/assets/bower_components
  fi

  node_modules/bower/bin/bower install
  ```

5. Create ```bower.json``` and define your dependencies

  ```json
  {
      "name": "myapp",
      "dependencies": {
        "sugar": "*",
        "marionette": "*",
        "backbone.stickit": "*"
      }
  }
  ```

6. Tell Bower where he should install all dependencies in ```.bowerrc```

  ```json
  {
      "directory": "vendor/assets/bower_components"
  }
  ```

7. Configure Rails so that it know where to look for Bower components. Update your ```application.rb```

  ```ruby
    # ....
    config.assets.paths << Rails.root.join('vendor', 'assets', 'bower_components')
    # ...
  ```

8. Use bower components in your ```application.js```

  ```coffee
  #= require sugar
  #= require backbone/backbone
  #= require backbone.stickit/backbone.stickit
  ```

9. Heroku by default support only one "kind" of app. Either Ruby or Node but not both at the same time. Since, out project behaves as Ruby and Node app we need to use custom build pack to run it.

  Add buildpack-multi to your Heroku instance

  ```bash
  heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
  ```

  Create new file ```.buildpacks``` and add this two lines to it

  ```
  https://github.com/heroku/heroku-buildpack-nodejs
  https://github.com/heroku/heroku-buildpack-ruby
  ```

# Summary

If everything is configured correctly only thing that you should do when you want to deploy is

```bash
git push origin master
```

And Heroku will take care of the rest.

And on your local machine you now run this command each time you add packages to bower

```bash
npm install
```

Is't that nice? :)
