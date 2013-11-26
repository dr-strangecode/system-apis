
There has been a lot of talk about "infrastructure as code" over the past couple of years and how we can use various configuration management tools to work like programmers.  The idea is great but the practice leaves a lot to be desired.

We can keep the definitions for our infrastructure in source control.  We can iteratively develop our infrastructure.  We can even practice the ops version of TDD and use tests to drive out our infrastructure.  But are we really programming?

If we think about object oriented programming from an operations perspective, the analogies are fairly easy.  Say you have a fairly simple app that has a couple of web servers, a couple cache servers, and a database.  We might think about a collection of web server objects, a collection of cache server objects, and a database object.  Not hard to conceptualize.  But what happens in practice and how does it relate to good programming principles?

First we set up our configuration management, defining the roles that our servers will perform.  The web servers get apache.  The cache servers get memcached.  The database gets sqlite because you still hate your high school english teacher... but that's another story.

Things go well when you first set everything up.  Then, like every app ever written, you have to log in to the machine to do a little bit of maintenance.

Stop right here.  If we remember our desire to treat things like a programmer would his application: why are we messing with the internal state of the object by logging in to the system?

