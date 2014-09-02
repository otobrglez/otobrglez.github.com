---
layout: post
title:  "Downloading a track from SoundCloud with Ruby"
date:   2014-09-02 12:00:00
comments: true
categories: ruby
image: /images/010-soundcloud.png
---

I came across a [StackOverflow question](http://stackoverflow.com/questions/25622120/downloading-a-track-from-soundcloud-using-ruby-sdk/25623734) on how to download a track from [SoundCloud], this is my explenation on how to do it.

# Assumptions

Things that you have to understand before doing this thing.

 - Not every track on [SoundCloud] can be downloaded! Only tracks that are flagged as **downloadable** can be downloaded - your code has to consider that option!
 - Your track URL has to be "resolved" before you get to download_url and after you get download_url you have to use your client_id to get the final download URL.
 - Tracks can be big, and downlowding them requires time! You should never do tasks like this straight from your Rails app in your controller or model. If the tasks runs longer you always use some background worker or some other kind of background processing "thing" - [Sidekiq] for example.

# Command-line client example

This is example of working client, that you can use to download tracks from [SoundClound]. Its using official [Official SoundCloud API Wrapper for Ruby][sc-ruby], assumes that you are using Ruby 1.9.x..

    # We use Bundler to manage our dependencies
    require 'bundler/setup'

    # We store SC_CLIENT_ID and SC_CLIENT_SECRET in .env
    # and dotenv gem loads that for us
    require 'dotenv'; Dotenv.load

    require 'soundcloud'
    require 'open-uri'

    # Ruby 1.9.x has a problem with following redirects so we use this
    # "monkey-patch" gem to fix that. Not needed in Ruby >= 2.x
    require 'open_uri_redirections'
    
    # First there is the authentication part.
    client = SoundCloud.new(
      client_id: ENV.fetch("SC_CLIENT_ID"),
      client_secret: ENV.fetch("SC_CLIENT_SECRET")
    )
    
    # Track URL, publicly visible...
    track_url = "http://soundcloud.com/forss/flickermood"
    
    # We call SoundCloud API to resolve track url
    track = client.get('/resolve', url: track_url)
    
    # If track is not downloadable, abort the process
    unless track["downloadable"]
      puts "You can't download this track!"
      exit 1
    end
    
    # We take track id, and we use that to name our local file
    track_id = track.id
    track_filename = "%s.aif" % track_id.to_s
    download_url = "%s?client_id=%s" % [track.download_url, ENV.fetch("SC_CLIENT_ID")]
    
    File.open(track_filename, "wb") do |saved_file|
      open(download_url, allow_redirections: :all) do |read_file|
        saved_file.write(read_file.read)
      end
    end
    
    puts "Your track was saved to: #{track_filename}"

Also note that files are in [AIFF (Audio Interchange File Format)](http://www.digitalpreservation.gov/formats/fdd/fdd000005.shtml). To convert them to mp3 you do something like this with [ffmpeg].

    ffmpeg -i 293.aif final-293.mp3

[SoundClound]: https://soundcloud.com/
[sc-ruby]: https://github.com/soundcloud/soundcloud-ruby
[Sidekiq]: http://sidekiq.org/
[ffmpeg]: https://www.ffmpeg.org/
