CTanks
======

This is a programming game, inspired by Crobots by Tom Poindexter
(http://www.nyx.net/~tpoindex/crob.html).  Players write programs for
tanks, with the intent to seek out and destroy other tanks.

Output is a JSON object, and some scripts are provided to wrap the
object up in a web page.


Included programs
-----------------

I tried to stick with the Unix philosophy of one program per task.  I
also tried to avoid doing any string processing in C.  The result is a
hodgepodge of C, Bourne shell, and awk, but at least each piece is
fairly simple to audit.


### round.sh tank1 tank2 ...

Runs a single round, awards points with rank.awk, and creates a new
summary.html with summary.awk.  This is the main interface that you want
to run from cron or whatever.
      

### forftanks tank1 tank2 ...

A program to run a round of tanks and output a JSON description of the
game.  This is what tanks.js uses to render a game graphically.
The object printed contains:

    [[game-width, game-height],
     [[tank1-color, 
      [[sensor1range, sensor1angle, sensor1width, sensor1turret],
       ...]],
      ...],
     [[
      [tank1x, tank1y, tank1angle, tank1sensangle, 
       tank1flags, tank1sensors],
      ...],
     ...]]

If file descriptor 3 is open for writes, it also outputs the results of
the round to fd3.  


### rank.awk

Processes the fd3 output of forftanks to award points and output an
HTML results table.


### summary.awk tank1 tank2

Creates summary.html, linking to all rounds and showing overall
standing.


### designer.cgi

Accepts form input and writes a tank.



Building from source
--------------------

You should be able to just run "make".  The C is supposedly ANSI C, and
might even compile on Windows.  I've built it on Linux, with glibc and
uClibc, big- and little-endian.



History
-------

This is a port of the "Tanks" program written by Paul Ferrell
<pflarr@clanspum.net> in 2009-2010.  Paul created the entire game based
off a brief description I provided him of Crobots and a vague desire to
"make something fun for high school kids to learn some programming."  We
ran Paul's Tanks as part of a 100-attendee computer security contest in
February of 2010 and by all accounts it was a huge success.  It even
made the nightly news.

Paul's version was written in Python and provided a custom language
called "Bullet", which looked like this:

    >addsensor(50, 0, 5, 1);        # 0
    >addsensor(100, 90, 150, 1);    # 1
    >addsensor(100, 270, 150, 1);   # 2
    >addsensor(100, 0, 359, 0);     # 3
    
    # Default movement if nothing is detected
                : move(70, 70) . turretccw();
    random(2, 3): move(40, 70) . turretccw();
    random(1, 3): move(70, 40) . turretccw();
    
    # We found something!!
    sense(3): move(0, 0);
    sense(1): turretcw();
    sense(2): turretccw();
    sense(0): fire();

Nick Moffitt played with this original version and convinced me (Neale)
that something like Forth would be a better language.  I added some code
to accept a scaled-down version of PostScript.  The IRC channel we
frequent collectively agreed to give this new language the derisive name
"Forf", which should ideally be followed by punching someone after it is
spoken aloud.  I wrote a Python implementation of Forf, which was slow,
and then Adam Glasgall wrote a C implementation, which was quick.

I decided to take Tanks to Def Con in July 2010, and just for bragging
rights, to have it run on an Asus WL-500gU.  This is a $50 device with a
240 MHz MIPS CPU, 16MB RAM, and a 4MB flash disk, along with an
802.11b/g radio, 4-port 10/100 switch, and an additional 10/100 "uplink"
port; it's sold as a home wireless router.  I had originally intended to
run it off a lantern battery just for fun, but eventually thought better
of it: doing so would be wasteful for no good reason.

Aaron McPhall <amcphall@mcphall.org>, my summer intern at the time, got
OpenWRT and Python onto the box and benchmarked it at about 60 seconds
for a 16-tank game, after he had profiled the code and optimized a lot
of the math.  That wasn't bad, it meant we could run a reasonably-sized
game at one turn per minute, which we knew from past experience was
about the right rate.  But it required a USB thumb drive to hold Python,
and when we used the Python Forf implementation, the run-time shot up to
several minutes.  I began this C port while Adam Glasgall, another fool
on the previously-mentioned IRC channel, started work on a C version of
a Forf interpreter.

This C version with Forf runs about 6 times faster than the Python
version with Bullet, on the WL-500gU.  A 1GHz Intel Atom runs a 16-tank
game in about 0.2 seconds.


What's so great about Forf?
---------------------------

Nothing's great about Forf.  It's a crummy language that only does
integer math.  For this application it's a good choice, for the
following reasons:

* No library dependencies, not even malloc
* Runs in fixed size memory
* Not Turing-complete, I think (impossible to make endless loops)
* Lends itself to genetic algorithms


Author
------

Neale Pickett <neale@woozle.org>
