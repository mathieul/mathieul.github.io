---
layout: post
title: "Pingpong Diagrams, a simple Elixir web app -- Part 2 of many"
description: "Second post in a serie of tutorials to build a simple web application with Elixir"
category: Elixir
tags: elixir tutorial
---
{% include JB/setup %}

## Previously on Pingpong Diagrams...

In the first post we setup a new project and generated a dynamo, a web
application inside our project. The [dynamo README file on the github page](https://github.com/elixir-lang/dynamo) describes every component of a dynamo in details.

Before getting into the meat of building the front-end part of the application,
let's setup the asset components.

## Stylesheets: Sass, Compass and Bootstrap

Managing assets can be easily decoupled from the backend, so we can very well
build up a solution integrating several strong tools from a different platform.
We will use the Sass stylesheet processor with the Compass CSS authoring
framework, along with Twitter Bootstrap front-end tool suite. Instead of using
the command line that comes with each tool, we will use [guard](https://github.com/guard/guard)
to control them all, and generate updated assets automatically as you update
source files.

Those tools require a [Ruby runtime](http://www.ruby-lang.org/en/downloads/) to run.
I like to use [rbenv](https://github.com/sstephenson/rbenv) to install Ruby as
it allows to have different versions installed on the same machine. If you
don't know Ruby or want a very easy solution to install it and you develop on
a Mac, you should follow [Tokaido](https://github.com/tokaido/tokaidoapp/releases/).

Once you have Ruby installed, you need to install the bundler gem (a gem is
a Ruby library) which we will use to install front-end tools and manage their
dependencies for us. Let's install ```bundler``` and create a Gemfile file to
describe the tools we want to install.

    $ gem install bundler

    Successfully installed bundler-1.3.5
    1 gem installed

    $ cd /path/to/pingpong_diagrams
    $ vim Gemfile

    # Gemfile
    source "https://rubygems.org"
    gem "compass"
    gem "bootstrap-sass"
    gem "guard-compass"
    gem "guard-copy"

And we trigger the installation:

    $ bundle update
    Fetching gem metadata from https://rubygems.org/............
    Fetching gem metadata from https://rubygems.org/..
    Resolving dependencies...
    Installing sass (3.2.10)
    [...]
    Using guard-compass (0.0.8)
    Using bundler (1.3.5)
    Your bundle is updated!

Out of the box, Dynamo expects the assets to be located in ```./priv/static```.
We will locate all assert sources in ```./priv/assets``` and configure the
different tools to generate assets and store them into ```./priv/static```.
Starting with Compass.

    $ mkdir config
    $ vim config/compass.rb

    # compass.rb
    require "bootstrap-sass"

    http_path = "/"
    css_dir = "priv/static/stylesheets"
    sass_dir = "priv/assets/stylesheets"
    images_dir = "priv/static/images"
    javascripts_dir = "priv/static/javascripts"
    add_import_path File.expand_path("../../priv/vendor/stylesheets", __FILE__)

    # output_style = :expanded or :nested or :compact or :compressed
    output_style = :expanded
    relative_assets = true
    preferred_syntax = :sass

We start setting up ```guard``` by creating the ```Guardfile```:

    # Guardfile
    require "fileutils"

    %w[priv/assets/images priv/vendor/images priv/static].each do |path|
      FileUtils.mkdir_p(path) unless File.exists?(path)
    end
    FileUtils.cp "priv/assets/favicon.ico", "priv/static"

    guard :compass, configuration_file: "config/compass.rb"

    guard :copy, from: "priv/assets/images",
                 to: "priv/static/images",
                 mkpath: true,
                 run_at_start: true

    guard :copy, from: "priv/vendor/images",
                 to: "priv/static/images",
                 mkpath: true,
                 run_at_start: true

    FileUtils.mkdir "priv/static" unless File.exists?("priv/static")
    FileUtils.cp "priv/assets/favicon.ico", "priv/static"

And we run the compass command to copy bootstrap source assets in our project
and create the first sass files. After compass is done we move javascripts and
images which were copied to ```priv/static``` to ```priv/vendor``` where we
will store 3rd party files.

    $ compass create . -r bootstrap-sass --using bootstrap

    directory ./priv/static/images/
    directory ./priv/static/javascripts/
    directory ./priv/static/stylesheets/
       create ./config.rb
       [...]
       create ./priv/static/stylesheets/styles.css

    $ rm -f config.rb
    $ mkdir -p priv/vendor
    $ mv priv/static/images priv/vendor
    $ mv priv/static/javascripts priv/vendor

Finally we start the ```guard``` command in a separate terminal. As long as the
command is running, any time you save changes to one of the asset source files
it triggers regenerating the dependent asset. For instance saving a Sass file
will automatically generate the CSS stylesheets.

    $ cd /path/to/pingpong_diagrams
    $ touch priv/assets/favicon.ico
    $ bundle exec guard

    00:11:38 - INFO - Guard is using TerminalTitle to send notifications.
    [...]
    00:11:38 - INFO - Guard is now watching at '/Users/[...]/pingpong_diagrams'

Note that running guard triggered generating a CSS stylesheet in
```priv/static/stylesheets/styles.css``` which contains a compilation of all
bootstrap stylesheets already.
