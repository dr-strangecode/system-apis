## Are We There Yet?

There has been a lot of talk about "infrastructure as code" over the past couple of years and how we can use various configuration management tools to work like programmers.  The idea is great but the practice leaves a lot to be desired.

We can keep the definitions for our infrastructure in source control.  We can iteratively develop our infrastructure.  We can even practice the ops version of TDD and use tests to drive out our infrastructure.  But are we really programming?

If we think about object oriented programming from an operations perspective, the analogies are fairly easy.  Say you have a fairly simple app that has a couple of web servers, a couple cache servers, and a database.  We might think about a collection of web server objects, a collection of cache server objects, and a database object.  Not hard to conceptualize.  But what happens in practice and how does it relate to good programming principles?

First we set up our configuration management, defining the roles that our servers will perform.  The web servers get apache.  The cache servers get memcached.  The database gets sqlite because you, madam, are evil and love to see them cry.

Things are going well.  Everything is up and running.  You are browsing the bofh archives, reminiscing about the glory days.  Then it happens.  Like every app ever written, you have to log in to the machine to do a little bit of maintenance.

Stop right here.  If we remember our desire to treat things like a programmer would his application: why are we messing with the internal state of the object by logging in to the system?  We habitually log in to our systems to perform routine tasks but there are a number of problems with this.

#### Who's Murphy?

One good system deserves another, and another, and another.  Systems are left in our "Please keep working" backlog because we have ten new apps to get running yesterday.  No one wants to think about what might happen if they were to all have a little "issue" at the same time.  Like a leap second or something else that would *never* happen...

#### It's Three In The Morning And I Forgot My Keyboard Two Bars Ago

Ever been working on something when you should have been asleep a day ago?  Ever get that sinking feeling when you fire off a command and then start wondering "Which database was this again?"

#### It's Not You It's Me

The benevolent overlords have heard your feeble pleas and finally decided to get you some help.  Just one problem: little Johnny thinks `/etc` is a really cool place to buy stuff for his mom.  Does he know the right order to restart services?  Does he know that you should really double check that one little thing if you don't want to spend two days picking up the little pieces?


## Do It Like They Do On The IRC Channel

Good programmers don't mess with the internal state of their objects.  They define behaviors, interfaces, and interactions.  It's the sanest way to deal with large systems.

If you need an object to do something or look a certain way, you don't poke around in its guts: you define an interface to achieve the desired behavior.

Take the simple case of managing a service.  Sometimes you need to turn a service off or on without the need to change a config file.  Configuration management is a poor choice for the periodic or non-normal state.  I don't want to deploy a new configuration for something that's only temporary.  I want to be able to "manage" the service.

If we extend the thought a little more: wouldn't it be interesting to have a program that we're deploying to our systems?  A program that's in source control?  A program that we can have tests for?  A program that can handle the little bits that configuration management is weak at?  A program that models the functionality of our system?

## My API Is Happy To See You

One of the easiest entry points to treating systems more like programs is to put an actual API on the system, focused on system functionality.  Let's go back to our earlier example of web servers, cache servers, and a database:

#### Web Servers
```
/current_user_count
/start_web
/stop_web
/web_status
```

#### Cache Servers
```
/clear_cache
/start_cache
/stop_cache
/cache_status
```

#### Database Servers
```
/compact
/clear_old_stuff
/start_database
/stop_database
/database_status
```

These are somewhat contrived examples of things that configuration management is a poor fit for: things that cause us to log on to the systems.

## API Quick Start

You've got two options for starting out with API's: write your own or use a tool that makes it easy.

#### PyJoJo

I like to start with [pyJoJo](https://github.com/atarola/pyjojo).  It basically turns bash scripts into API calls.  Don't expect it to tie into your authentication system; that's what a custom API is for.  What it's great at is easily getting an API onto your system that lets you start exploring what we've been discussing.

Word to the wise: be specific.  If you write a little method that lets you install any package in the world, don't be suprised when someone has used it to install something stupid.  If you want to try writing a method to install the latest version of the software package that the developers have been working on: consider hard coding the script to the name of the software, but making the version a parameter you can pass.

#### Write Your Own

Eventually you're going to want things out of the API that pyjojo won't do.  It's time to start writing some code.  Better yet, get the devs to do it!  Make a demo of what you're thinking in pyjojo and then let the developers show you "how it's *really* done."


## Where Do We Go From Here?

This is more of a first step than a destination.  API's on the systems is great, but the real power comes when we realize what we've just done.  We've put *API's* on our *systems*.  I don't know about you, but I think I know a guy or two at the office that messes with this "web stuff."  Maybe one of them could make, I dunno, a dashboard or something?

Something that would tell us who did what, when.  Something repeatable and three-in-the-morning-proof.  Something little Johnny can wipe his nose with.


