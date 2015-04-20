---
layout: post
title:  "Using Grunt for S3 deployment / sync"
date:   2015-04-20 12:00:00
comments: true
categories: front-end
image: /images/019-grunt-logo.svg
---

For my recent [AngularJS](https://angularjs.org/) project [SNAGR](http://snagr.io) I've decided that I'm gonna automate deployment process as much as possible.

Because, who has the time to do everything manually this days - right? :)

## Project setup

- Production app is served directly from [Amazon S3](http://aws.amazon.com/s3/) bucket
- [AngularJS](https://angularjs.org/) app managed with [Yeoman](http://yeoman.io) and [GRUNT](http://gruntjs.com/).
- Tasks are defined in `Gruntfile.js`
- Local development is done with `grunt serve`.
- Once I'm happy with my product I use `grunt build` to build / minify everything into `./dist` folder.
- `./dist` folder is then self-contained production ready single page app, that you can copy to front-end server...

## Hello Amazon Simple Storage and s3cmd

After `grunt build` produces `./dist` folder I use [s3cmd](http://s3tools.org/s3cmd) to [sync](http://s3tools.org/s3cmd-sync) new content with my S3 bucket.

    s3cmd sync -c s3.conf -r ./dist/* s3://<my bucket>

`s3.conf` is configuration file that holds all credentials information. You can configure your own with following command:

    s3cmd --configure -c s3.conf

Sync command has couple of options. With `-P` your MIME types will be preserved with `--delete-removed` files that are no longer present locally will be removed from your S3 bucket. Check documentation about [more flags](http://s3tools.org/s3cmd-sync).

## Now GRUNT it!

Manually running s3cmd sync after build is just boring. So lets add it to GRUNT "workflow" with two tasks. First task that actually syncs `./dist` folder

```javascript
grunt.registerTask('deployToS3', 'Deploying to S3 bucket', function(target){
  var done = this.async();
  var command = "/usr/local/bin/s3cmd";

  var child = grunt.util.spawn({
    grunt: false,
    cmd: command,
    args: 'sync -c s3.conf -r ./dist/* -P s3://<my bucket>'.split(" "),
    opts: {
      stdio: 'inherit'
    }
  }, function(error, result, code){
    if(code != 0 && error !== null) grunt.fatal("Error sycing with S3 bucket");

    grunt.log.writeln(String(result).trim());

    done();
  });
});
```

Second task that first builds project and then invokes syncing.

```javascript
grunt.registerTask('deploy', [
  'build',
  'deployToS3'
]);
```

Now, all you have to do is...

```
grunt deploy
```

... and share this. ;)

Cheers!

