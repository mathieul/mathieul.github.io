---
layout: post
title: "Build a simple web application with Elixir -- Part 1 of many"
description: "First post in a serie of tutorials to build a simple web application with Elixir"
category: Elixir
tags: elixir tutorial
---
{% include JB/setup %}

## Preambule

I love open source software. I have an enormous respect for all contributors
to open source software. The world (and more selfishly my personal world)
would be quite different without the talent, time and dedication people like
Richard Stallman, Linus Torvald, Larry Wall, Yukihiro "Matz" Matsumoto,
David Heinemeier Hansson, John Resig, Yehuda Katz, Aaron Patterson, José Valim
and so many more, made the choice to give away graciously for the world to benefit.

I'm so thankful to all of them, I thought the least I could do was to start
giving back a little by sharing some of my learnings through blog posts and
tutorials. Maybe it will save people some time, maybe it will give
ideas, and maybe some of my enthusiasm for Elixir, Ruby and other software
goodies will be contagious.

## Elixir

I find Elixir to be an exceptional language, and the joy I am feeling learning
it and playing with it feels a lot like what I felt as I dug into Ruby and Rails
and started learning Ruby back in 2006. Thank you José Valim for bringing to us
all the power of Erlang and OTP, and making it so enjoyable and productive!

There are already a lot of great resources to get hooked and start learning
the language, like:

* [Meet Elixir](https://peepcode.com/products/elixir), the great PeepCode screencast by José himself
* [Programming Elixir](http://pragprog.com/book/elixir/programming-elixir), the book from Dave Thomas
* [Elixir getting started guide](http://elixir-lang.org/getting_started/1.html), a great and rich guide on Elixir's website
* [Great blog post serie](http://benjamintanweihao.github.io), by Benjamin Tan
* [Introduction to Elixir](http://www.youtube.com/watch?v=41PvAPSX0wg), a presentation from José Valim on Youtube
* [Elixir cheat sheet](http://media.pragprog.com/titles/elixir/ElixirCheat.pdf)

Without any more talking, let's get to it and start with building this application.

## Building a web application

I would like to show you how to build a simple CRUD application (create, read,
update and delete) which publishes events such as a record creation, update,
deletion, to all the browsers opened on the application. If you've been reading
on how to do that with Rails 4 (i.e.: [Railscast/ASCIICast on ActionController::Live](http://railscasts.com/episodes/401-actioncontroller-live?view=asciicast)), or tested it yourself,
I believe you'll be impressed at how simple and effective it is to do it with
Dynamo, the Elixir web framework.

The application will allow to manage UML sequence diagrams, also known as
pingpong diagrams, using a [nifty JavaScript library](http://bramp.github.io/js-sequence-diagrams/)
to draw the diagrams using SVG in the browser. It will use Mnesia to persist
those diagrams, Sass, Compass, Bootstrap to make the pages look pretty
and CoffeeScript for adding front-end behaviors.

## Introduction: setup the development environment and render a test page

In this first post we will just go through setting up the development environment
and render a test page to make sure we start on a good basis.

The first thing to do is to install Erlang R16B and Elixir. You should follow
the [installation section in Elixir's getting started guide](http://elixir-lang.org/getting_started/1.html).
If you're installing on Ubuntu, start with this [blog post from Avdi Grimm](http://devblog.avdi.org/2013/07/05/installing-elixir-on-ubuntu-13-04).

Once you have Elixir installed properly (you can start ```iex```, the Elixir REPL),
you can create a new Elixir project like so:

    $ mix new pingpong_diagrams
    * creating README.md
    * creating .gitignore
    * creating mix.exs
    * creating lib
    * creating lib/pingpong_diagrams.ex
    * creating test
    * creating test/test_helper.exs
    * creating test/pingpong_diagrams_test.exs

    Your mix project was created with success.
    You can use mix to compile it, test it, and more:

        cd pingpong_diagrams
        mix compile
        mix test

    Run `mix help` for more information.

The command creates a project scaffold, ready to be used to include libraries,
start testing and writing code, really convenient.

The [dynamo README on the github repository](https://github.com/elixir-lang/dynamo)
advises to use the master branch of Elixir and to clone the dynamo repository
and install from there. I had issues using the master branch of Elixir with
certain libraries we'll use later, so instead let's use the current stable
version of Elixir, 0.10.0, create a new project and install dynamo in this new
project.

We edit **mix.exs** to insert the dynamo dependency, so we can install it and
use its dynamo generator. Just replace this section:

    defp deps do
      []
    end

with this one:

    defp deps do
      [ { :cowboy, "0.8.6", github: "extend/cowboy" },
        { :dynamo, github: "elixir-lang/dynamo", ref: "38d7b6afdf2e232062a11c1f408c376d0e306ec3" } ]
    end

And we use the ```mix``` command to fetch and compile dynamo:

    $ mix deps.get

    * Getting dynamo [git: "git://github.com/elixir-lang/dynamo.git"]
    Cloning into 'deps/dynamo'...
    [...]
    ==> Leaving directory `/Users/mathieu/Documents/Development/Perso/pingpong-diagrams/deps/ranch'
    ==> cowboy (compile)
    * Compiling dynamo

Now that dynamo is installed we can now use the ```mix dynamo``` command to
generate a dynamo (web server) in our application.Answer **y** each each time
you're asked to overwrite a file.

    $ mix dynamo ../pingpong_diagrams

    * creating README.md
    /Users/mathieul/Documents/Development/Projects/pingpong_diagrams/README.md already exists, overwrite? [Yn] y
    * creating .gitignore
    /Users/mathieul/Documents/Development/Projects/pingpong_diagrams/.gitignore already exists, overwrite? [Yn] y
    * creating mix.lock
    /Users/mathieul/Documents/Development/Projects/pingpong_diagrams/mix.lock already exists, overwrite? [Yn] n
    * creating mix.exs
    /Users/mathieul/Documents/Development/Projects/pingpong_diagrams/mix.exs already exists, overwrite? [Yn] n
    * creating web
    * creating web/routers
    [...]
    * creating test/routers
    * creating test/routers/application_router_test.exs

I've ran into compilation issues of the dynamo web framework passed a certain
commit when using Elixir 0.10.0. So for now I lock the dynamo commit to the last
one I know to work fine with 0.10.0, until a new release of Elixir is coming out.

As we've seen previously, you specify your dependencies in the methods ```deps```
of your project **mix.exs** file. So let's replace the section specifying which
versions to depend on for cowboy and dynamo, to instead require the version
0.8.6 of cowboy and the commit 38d7b6af for dynamo. Replace this section:

    defp deps do
      [ { :cowboy, github: "extend/cowboy" },
        { :dynamo, "0.1.0.dev", github: "elixir-lang/dynamo" } ]
    end

with this one:

    defp deps do
      [ { :cowboy, "0.8.6", github: "extend/cowboy" },
        { :dynamo, github: "elixir-lang/dynamo", ref: "38d7b6afdf2e232062a11c1f408c376d0e306ec3" } ]
    end

Let's refresh the depencies and test running our brand new web application:

    $ mix deps.unlock
    $ mix deps.get

    * Getting dynamo [git: "git://github.com/elixir-lang/dynamo.git"]
    [...]
    ==> cowboy (compile)
    * Compiling dynamo

    $ mix server

    Compiled lib/pingpong_diagrams.ex
    Compiled lib/pingpong_diagrams/dynamo.ex
    Generated pingpong_diagrams.app
    Running PingpongDiagrams.Dynamo at http://localhost:4000 with Cowboy on dev

Now open a new browser window and go to ```http://localhost:4000```. You should
see a welcome page, all is working!

That's it for this first post. In the part 2 we will start building the pages
to list and create pingpong diagrams. Until next time...
