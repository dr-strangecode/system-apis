## Are We There Yet?

There has been a lot of talk about "infrastructure as code" over the past couple of years and how we can use various configuration management tools to work like developers.  The idea is great but the practice leaves a lot to be desired.

We can keep the definitions for our infrastructure in source control.  We can iteratively develop our infrastructure.  We can even practice the ops version of TDD and use tests to drive out our infrastructure.  But are we really programming?

If we think about object oriented programming from an operations perspective, the analogies are fairly easy.  Say you have a fairly simple app that has a couple of web servers, a couple cache servers, and a database.  We might think about a collection of web server objects, a collection of cache server objects, and a database object.  Not hard to conceptualize.  But what happens in practice and how does it relate to good programming principles?

First we set up our configuration management, defining the roles that our servers will perform.  The web servers get apache.  The cache servers get memcached.  The database gets sqlite because you, madam, are evil and love to see them cry.

Things are going well.  Everything is up and running.  You're browsing the bofh archives, reminiscing about the glory days.  Then it happens.  Like every app ever written, you have to log in to the machine to do a little bit of maintenance.

Stop right here.  If we remember our desire to treat things like a developer would his application: why are we messing with the internal state of the object by logging in to the system?  We habitually log in to our systems to perform routine tasks but by doing this we have thrown out any similarity to programming.  We decry the practice of developers making a bugfix on the production server, yet we don't think twice when we log in to restart a service.

This raises the question, "What would our systems look like if we were to treat them like developers treat their objects?"


## Do It Like They Do On The IRC Channel

Good developers don't mess with the internal state of their objects.  They define behaviors, interfaces, and interactions.  It's the sanest way to deal with large systems.

If you need an object to do something or look a certain way, you don't poke around in its guts: you define an interface to achieve the desired behavior.

Take the simple case of managing a service.  Sometimes you need to turn a service off or on without the need to change a config file.  Configuration management is a poor choice for the periodic or non-normal state.  I don't want to deploy a new configuration for something that's only temporary.  I want to be able to "manage" the service.

If we extend that thought a little more: wouldn't it be interesting to have a program that we can deploy that models the functionality of our system?  A program that lets us perform those routine tasks we're so used to doing by hand?  A program that's in source control?  A program that we can have tests for?  A program that can handle the little bits that configuration management is weak at?

One of the easiest entry points to treating systems more like programs is to put an actual API on the system, focused on system functionality.


## My API Is Happy To See You

An [API](http://en.wikipedia.org/wiki/Application_programming_interface) provides a clean, well defined interface for systems to interact with each other.

That interface provides the perfect mechanism to ensure that system operations are performed the same way *every* time.  Configuration Management lets us standardize what our systems look like; API's allow us to standardize how we manage our systems.

There are a lot of different kinds of API's.  One of the simplest, and the kind that we'll be focusing on here, is a [web API](http://en.wikipedia.org/wiki/Application_programming_interface#Web_APIs).  They're easy to interact with on the command line with tools like curl; or when you want to start putting together some more complex behavior, you can make a command line client with your language of choice.

Let's look at what the interfaces for our system API's might look like with our earlier simple app example:

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

If we want to get the status of our database, we don't need to log into the system.  Instead, we can do a simple curl:

```
curl http://database.server.company.internal.dns.com/database_status
```

An example of more complex behavior you might find in a custom client (Ruby'ish psuedo code):

```
def weekly_downtime_maintenance
  # stop web servers
  [web-servers].each do |server|
    post "#{server}/stop_web"
  end

  # db maintenance
  post "#{database-server}/clear_old_stuff"
  post "#{database-server}/stop_database"
  post "#{database-server}/compact"
  post "#{database-server}/start_database"

  # clear cache
  [cache-servers].each do |server|
    post "#{server}/clear_cache"
  end

  # start web servers
  [web-servers].each do |server|
    post "#{server}/start_web"
  end
end
```

Now we have a fairly complex task broken down into steps that anyone can run with our client.  It also ensures that things happen in the correct order every time, with no typos.  This is a lot easier to hand off to new admins or even pass back to the developers to run.

**Warning:**  Be careful about what you expose and how: especially if your systems are publicly accessible.  Firewalls, SSL, authentication, and all the other goodies associated with secure web apps are in order here.

## The DevOps Win

One of the core principles of DevOps is culture.  Put another way, it's a sense of community.  How can you have a sense of community with someone you don't talk to?

Interface design presents a great opportunity to collaborate with developers.  It provides developers an opportunity to work with you on good programming techniques.  It brings up questions like "Should we break this one API call up into smaller pieces and what's the best way to do that?".

These kinds of conversations can be very helpful in developing a sense of community.  You're able to talk about something developers love in a setting that interests you.  They provide a safe topic when things have gotten a little tense because of system downtime or bugs in production.


## API Quick Start

You've got two options for starting out with API's: write your own or use a tool that makes it easy.


#### PyJoJo

I like to start with [pyJoJo](https://github.com/atarola/pyjojo).  It basically turns bash scripts into API calls.  Don't expect it to tie into your authentication system; that's what a custom API is for.  What it's great at is easily getting an API onto your system that lets you start exploring what we've been discussing.

Word to the wise: be specific.  If you write a little method that lets you install any package in the world, don't be suprised when someone has used it to install something stupid.  If you want to try writing a method to install the latest version of the software package that the developers have been working on: consider hard coding the script to the name of the software, but making the version a parameter you can pass.


#### Write Your Own

Eventually you're going to want things out of the API that pyjojo won't do.  It's time to start writing some code.  Better yet, get the developers to do it!  Make a demo of what you're thinking in pyjojo and then let the developers show you "how it's *really* done."


## Parting Thought

System API's are more of a first step than a destination.

The real power comes when we realize what we've just done.  We've put *API's* on our *systems*.  I don't know about you, but I think I know a guy or two at the office that messes with this "web stuff."  Maybe one of them could make, I dunno, a dashboard or something?

Something that would tell us who did what, when, and what happened.

Something that provides an easy interface to complex functionality.

Something that has a deploy pipeline and tests.

Now *this* is starting to sound like programming.
