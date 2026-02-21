# The Monolith Destruction Toolkit

This was a talk given 7 Dec 2016 at FutureStack, the New Relic Developer Conference, in San Francisco. Jose and I had worked on breaking down monolithic services.

## Video URL

https://www.youtube.com/watch?v=zjZsZvZr7QY

## Transcript (Auto-Generated)

thank you thank you good afternoon thank
you so much for staying for the whole
conference and getting your money's
worth as ashley said I'm Jonathan Owens
I've been doing site reliability work
for about 10 years now four of them at
New Relic I'm Jose Fernandez I've been
around like for a long while I think or
five years now and i love the jvm that's
a rare quality mm-hmm so we're going to
present some service architecture
patterns that our team at New Relic uses
to destroy monoliths one piece at a time
if you agree to these terms and
conditions of the talks AI wonderful
Jose and I have had the wonderful
privilege of being here with our
families though as you can imagine
traveling with kids under six on a plane
staying in a hotel is pretty hairy
things might get a little punchy bear
with us who works with software like
this big square no no not square shear
you can't do anything with it you don't
know how it works inside its you can't
like put anything on it nothing sticks
to it it's just miserable you don't know
why it's breaking you don't know why it
works does anyone even know how this
thing got here so heavy so my team has
one of these and we're in the process of
breaking it up now if you're lucky
enough to have one of these it's
probably actually a good problem to have
it means your company has been
successful enough to actually build a
piece of software to be big you know you
either die a micro service or live long
enough to see yourself become the
monolith right and our monolith grew up
right along with the company which is to
say really fast we've seen superlinear
traffic growth for straight and the code
base is grown at what feels like the
same rate so the business itself is a
success working on the software maybe
isn't and everybody that touches the
thing gets sad this is kind of a not a
good situation this is a very money
centric point of view about software
this is my son on his first airplane
ride two days ago the feeling in his
heart right here the thrill of seeing
something new and endless and fresh we
so rarely get this as adults but
sometimes when we're building software I
feel it something new something that is
in the world now that wasn't there
before and something that is uniquely me
is expressed in it monoliths destroy
this feeling they put you in a box where
nothing is possible we are better than
this but how do we get there how do we
escape this trap well let's make a list
you need a few things we here's one
patterns worked for us and here's the
things that we did to do it we'll assume
you have new relic concer here and we
believe these tools and this pattern can
be used by any of you working on any
kind of software will start with the
specifics of our story and maybe that
will help sound familiar to you storing
and collecting the data that customers
send us about their software is one of
the hardest jobs that we have it's so
hard that we didn't really like make
something big at first it was small but
it kept getting harder and harder and
harder we have this giant JVM to do it
and so as the company grew like you saw
in that chart before this process just
grew and grew till the code base started
to look like this and everybody who
touched it caught the sad and everybody
knew we had to break it up because we
couldn't add things to it anymore it was
too much but we couldn't break it up all
at once because as satisfying as it is
to work on something and make it better
it doesn't actually help anybody except
for you and the company is a lot bigger
than you and the people working on this
problem so what we learned was that we
had to make this thing smaller just one
chunk at a time and then do that again
and again and again until it's rubble
and the whole new system is running and
our world is better so let's start with
what our code base actually does oh crap
okay it's just a lot of stuff this is
why working on it is so tricky even for
our most experienced programmers in fact
nobody of the company even really
understands all of its behavior you say
collector and people kind of shake it's
a four-letter word
so okay it does all this stuff how do we
pick the right things to extract we have
to be careful if you pick the wrong
thing you're going to introduce too much
risk and the whole model with comes down
in a bad way if you pick the wrong thing
you're going to break off too much risk
and I'll take you forever to build it if
you pick the wrong you get the idea so
let's start with the fact that more
services isn't the goal actually making
people happier is making our customers
happier is making our customers pay us
more money so they can get their data
faster is so let's look at the thing
that causes the most pain here in along
one dimension or another so if we start
with developer pain we're probably not
going to do ourselves well because
that's going to be hard to extract so we
found the right angle to be scale pain
so the biggest scale thing in here was
time series data or a new relic parlance
this is time slices a single time slice
to a relic is one of those red dots on
the screen that is one minute of data
along one metric from one host we have a
lot of these hundreds of millions a
minute and so the collector our model
with it processes this data is extremely
busy working on these things all the
time it has to store both at excuse me
it has to both aggregate this data so
take all the processes that send this
data in one minute and glue it all
together into a single minute worth of
data and then it has to write it all to
MySQL this is a very big job that is
probably suited to more than just one
JVM per account doing it the other nice
thing about time slices is they're quite
isolated there's really only one system
that writes this data are monolithic
collector there's one system that stores
the data a whole lot of MySQL and
there's one system that queries the data
this thing called MTS that just is the
guts behind the chart you see here so
while we're pulling this out let's pull
out the storage system to this might be
a little more than we should have bit
off but we decided you know mysql is
kind of causing us a lot of pain it's
kind of a junior monolith so let's use
cassandra because it's actually scaling
to be big enough and the nice thing
about doing that is it lets us build the
system on the
so with Jose is going to talk about how
we're going to build that system in the
dark okay so the first thing that our
managers told us when we were building
this we're sitting down with the
original team was like it has to have no
customer facing impact so Syria
customer-facing impact but then how do
you how do you access this this data
that's coming into the monolith so you
can like work on it right like it's you
know it's it's in the JVM how do we grab
it and pull it out and then I mean in
the company we've had some experience so
using Kafka so somebody in the team had
this bright idea hey why don't we
distract the data and put it into Kafka
there you go so just this is the basic
streaming architecture with Kafka on on
the left you have events horses which
Kafka calls producers so in our world
these are your apps running you Alec so
they're sending us on all these time
information about your appt the metrics
time slice data and then in the middle
you have these services you know it
could be like monoliths like the
collector or smaller microservices but
in terms they are consumers so these
consumers are consuming the data that
the producers are producing and these
consumers can also produce more data and
you got this chain of happiness until it
gets over to the to the data store so at
the edge on the far right you have the
consumers that actually write the data
out to the data store so what I'm going
back so we need it to her access to this
data that was coming into the mana late
and we're like okay let's put it into
Kafka so this is actually a really
really easy change if you work on the
JVM the the Catholic client is actually
really easy and a lot of fun so yeah so
we have the data and our mating a copy
of that time size data into Kafka and
the cool thing is that once we got this
in Kafka we didn't have to touch them
online I mean there
some minor changes but it was like we're
done and now we can start working on
that data stream so in the end what you
see here it's it's not very complicated
so you got the monolithic collector on
the left it's producing all these events
or the time slice as events and it's
basically making its way towards the far
right which is the data store Cassandra
and along the way I mean you know we
could maybe modify the data append the
data filter out the data aggregated you
know you can do whatever you want to it
but the key part of this is that so the
brown bids that I've highlighted there
those are the cupcake you so that's
where the your data is sitting between
consumers right and if you think about
it if this was in the monolith this
would be in the memory right so it's a
memory and you have to restart it what
happens to the data it's gone so we're
like sweet that's that's a really useful
idea now we can like iterate on this and
not affect the monolith or like hey what
what if we can actually do that on the
read side right so if you think about it
like the data that we're storing the
model is storing through the legacy
database which is my sequel and then the
new pipeline that we build this touring
to the new database which is Cassandra
right so the data that's in both
databases have has to be about the same
right if you query it has to come the
same results so on the left here you
have a new relic APM so when it when you
pull up a chart and you rarely chart
it's actually fetching that data the
time slice data that Jonathan mentioned
from this query service that internally
we call MPs and then the query serve is
like going to my sequel give me all the
time slices for this chart and that
brings it back to the to APM and
displace it to you so what we did it's
like okay whoa cool let's add a few
lines of code to the query service so
that when a query comes in we're going
to also record the results and the
performance information about the result
so how long it took to prepare the
database and then we're going to put
that into another calf cook you right
and you can see that down there and then
we had this service that's consuming
from this
the cue and executing the same query to
cassandra and also recording the results
and the query time so now we have the
query information with a legacy system
and the new system how it performed in
what it returned right so it would have
to be the same so we we grab that
information and send it over to our
insights product that's events and the
cool thing about that over there that's
inside so if you haven't used insights
you should you know after they just go
up a start an account then start sending
events so the cool thing is that here we
can chart how well the legacy system is
performing against the new system right
and we saw right away the Cassandra was
like just kicking ass it was way faster
and then the the queries were the same
so we knew that we didn't have a problem
in a new pipeline we were in sorting the
wrong data or maybe like doing something
to it wrong but the cool thing about
insight is that we could also show this
data to the executives and and the
senior managers are kind of like saying
like are we going to do this project or
not are we going to rewrite the model in
life and it was just like an easy way to
show this hey here you go you can look
at this this horse race and you can see
that you know the results are pretty
evident so I was really cool that was
really cool so now that we had this dark
environment we could iterate in we had
the freedom to experiment with new new
limits of performance and new ways we
could architect for processing on this
data so let's make things smaller and
see what happens so the high machines
running full speed on microservices
guilty as charged so rarely does it work
out we do have a success story here
though there was for us some bound north
of hello world where two small was
actually a reality but you know smaller
what did actually work out when we
started emitting those time slices from
the collector like cloned sheep we found
we had one service this minute
aggregator component that consumed it
aggregated the data and then wrote out
the finished data to Cassandra we
figured simply doing this outboard of
the collector would probably be enough
but it really wasn't it didn't
form nearly as well as we expected and
it put a lot of memory pressure on that
JVM and while it was a nice simple
extraction of the logic from the
existing model it wasn't quite up to the
task so since we were working in the
dark since we had this environment that
we could prove was faster and correct we
could iterate freely and say well what
if we split up writing and aggregation
turned out our service really had two
jobs those aggregation components and
those writing components were radically
separate and because we had that kafka
component that we could just stuff data
into freely it was not too big a deal to
split them up by splitting them we could
optimize those tasks separately
aggregation is really memory and CPU
intensive and writing was I owe a
network intensive Lord sends data out to
a remote system those are really
different performance profiles and we
could learn all this safely without
affecting customers because again this
whole system was behind the curtain
another interesting thing about this
architecture was that the more finally
we split those stages up the more data
ended up in Kafka and this actually
yields really interesting opportunities
to experiment with that data that we
didn't have before so the thing that
those the thing that comes after the
minute aggregator there that cute turns
out to be really valuable that's the new
relic data firehose the one we've had
all along hiding inside the collector
that everyone wanted to have access to
we can actually grow on this
architecture much more freely than we
could before here's an idea for another
thing we can extract something called
summary records these are what power the
traffic lights and some forms of
alerting in our system this is another
job the collector does in its monolithic
space and there's not really a need for
it to be in there if we have all the
data needed to evaluate that in another
queue or in kafka parlance a topic you
can actually create a new outboard
system following the same pattern take
the same specification re implement it
in a new scalable service off that same
data pipeline and then just go delete
the code out of the model with and
you're done it's really powerful nothing
in this pipeline needs awareness of this
new component you don't have to get
inside the aggregators or the writer
to extract this data and do something
with it nothing in the collector has to
change it can just stay there doing what
it does until we delete the code and
swap out for the new service and again
the fact that we didn't have Kafka
before that this data wasn't on the
outside is what led to us having such a
monolith if you put all the valuable
data in system inside one process and
one service that's things just going to
metastasize you have to expose it you
have to yield it in a way that's going
to be useful to other services so they
can grow organically the more you do
this the more you get a feel for it it
starts to feel like these things are
separate and yet compose one product
right so it each of these can be great
at their own thing and then together
they form like a nice meal into
something that actually hangs together
is coherent and each service can become
way simpler it's easier to optimize it's
easier to monitor it's easier to deploy
the effects of an outage or narrower
it's just much nicer in general but how
do you find the boundaries because one
of the problems in many mano lists is
that they can be very interconnected all
these different components depend on
each other to operate it's going to be
hard to know exactly how to pull them
out we found two rules that are easy to
say and kind of hard to follow the first
one is simply Conway's law the
boundaries of the service had matched
the communication boundaries of your
company there is a session happening
right now opposing this one that I don't
go to it go watch the recording that
talks about how we made this happen at
New Relic by actually structuring the
company in such a way that the services
all have a single team owner and that
that team represents that services
interface to every other team and so in
our case since we owned the monolith
completely we can say we're taking out
this part no one else you know is going
to be touching that unless we're
standing in the way and we're going to
decide exactly which parts get pulled
out and who's going to have them so we
had one team steering that whole thing
we still do that's been really helpful
because prior to this organizational
change our monolith was owned and
modified by a whole bunch of different
people and they all had commit and
pretty soon it was as we saw a ball of
snakes that's not very fun at least have
one person that owned
is the ball of snakes so they can say
you get a snake and you get a snake and
look under your seat and everybody gets
snake and someone can know where all the
snakes are it's give me the other one we
found so once you've got all that
established between teams individual
teams may want to break up smaller
services within themselves so in our
case this was the minute aggregator
became two services that had to be
experimentally proven half the team at
the time that that was proposed rolled
their eyes and said why in the world
would that make any difference you're
doing the same amount of work the other
half of the team that thought it was a
good idea said well I'm going to do it
I'm going to prove it to you and they
went and did it and it was pretty easy
because it was small service and we saw
the charts and went oh my gosh that was
such a good idea I don't know why I
doubted you data is really powerful here
sometimes it doesn't work out those
sometimes you break that service up and
you go well I don't really do anything I
got another thing to monitor forget it
you know put it back in it's got to
prove its mettle but it's not always
performance sometimes it can be this is
way easier to work on or this breaks it
up in such a way that when our team
grows we can actually hand the service
off to another team there can be just
simple ergonomics matters here again
having a lot of services since isn't the
goal or the point of micro services
that's not why we're even breaking up
this model if the goal is to make this
thing scale in a humane way so that you
can continue to enjoy working on the
software just don't make it look like
this you know we want to avoid these
monoliths just like we want to avoid
having concerns mixed just like we want
to avoid eating our food and a big bowl
all blended together you know pizzas
good so is chilly but don't make
yourself a
pizza-crepe-taco-pancake-chili bag
separate concerns when it's obvious and
the the thing that makes this possible
in an easy way is having Kafka or some
other service bus there if all your
services have to communicate over a
custom-designed api's with different
payloads and different agreements and
you've got cross team api version
disagreements it can start to get pretty
hairy but if your stream processing
between these services it's a lot
simpler each one can just define its
payload and say I'm going to write this
data to this topic and here's how you do
serialize it and go
figure it out it gets a lot easier they
start to plug together in a more uniform
way so there is real cost to be sure
more services does mean more work but
sometimes it means the right kind of
work and sometimes it's easier than you
think now that we had this structure we
had to actually figure out how to make
it reliable at scale so Jose tell us
about DevOps DevOps so at this point we
have Kafka we got microservers really
happy about on the left side of the
project right it's fast it's easy to
work on we can iterate it's awesome but
the collector was a nightmare to work
out it actually burned out a lot of
Engineers you know they made lasted a
year and the team and then they they're
gone right so and this was because the
the the project was done by just all
developers right and we we really didn't
we wanted to avoid this happening on the
new pipeline so we decided to put some
ops in in the dev and make it all better
there we go so DevOps you're all
probably been airing about it a lot this
is a google transport on the popularity
of DevOps in the last five years being a
developer I really didn't understand
what it meant you know I'm just coding
away and it just keeps happening out
here and Twitter and conferences people
talking about it so you know I went
ahead and ask alice you know hey Alice
what what is DevOps so I'm actually glad
that she did our talk before this
because you know I'm said like a segue
to it so this is how Alice explain
step-ups DevOps means empathy and
communication over the wall until there
is no more wall so now you're asking so
wait wait wait what is this wall that
alice is talking about so let me explain
this wall through an example so in a
traditional engineering org you have
developers in the product team and ops
people in the site inside relatability
team right so let's say that me as a
developer I need I need some some
hardware
so I'm like hey Jonathan I need three
Cassander service ASAP and I'm being
very vague right so I'm not used to
working with operations people and I'm
like I have a deadline I need to meet
and you know there's pressure so I need
this fast and Jonathan he's you know
traditionally he's in a good
organization that serving other teams
and multiple products and he's working
through a ticketing system right so my
ticket went in and it's probably like
you know third in line so but i have no
visibility to this so i'm waiting and
waiting and waiting and getting
frustrated cuz i'm thinking hey you know
what's going on the operations people
are so slow what's going on and then
eventually Jonathan came back say hey
Jose you know I'm so sorry it took so
long there's a big big q But Here I got
your three attempts or keep in mind that
you know my goal is to to to ship this
product you know that I need Cassandra
for but Jonathan's goals are worth
through the ticket stream like help help
as many things as you can so since I was
very vague I didn't specify hey I
actually needed Cassandra 30 and he gave
me like whatever the default was so now
the whole cycle repeats itself yeah this
actually happened we used to work
together another team so so in the
DevOps world the idea is like why don't
we just go back to like scrappy you know
New Relic your Wan routes where you're
like sitting together working on the
same team you know New Relic t1 that's
the idea of that office yeah
so what this means is that rather than
having s3's as like consultants that
come or help you out or or or sorry not
because he'll just better like ticketing
machines they actually join your team
right so once you're on your team they
have the same goal so it's ship you know
whatever product you're working on right
so once they're on your team there's
faster communication you know there's
less back and forth where you're more
productive because you're working with
him on the same thing and also not only
that but you know they bring like a
broader perspective because
traditionally you know like thats coat
it up throw it over the wall and then
yessiree is going to like run the
software and in the example before
rather than like you know sending a
ticket we basically just kind of book a
room sit you know working it together
parent it might take a day but that's
like way better than weeks that would
have taken in the traditional model so
that's great you know we're more
productive we can get you know ship
stuff faster but there's there's other
benefits that I've seen that happen when
you when you when you introduce them
absolute to to a team culture i sorry
and so teen cultures think that changes
right so i'm an engineer i'm a developer
my vision is very narrow it's like
what's the next feature i gotta i gotta
do what's a deadline you know i'm just
going to try and get it it's no use my
own judgment of like you know writing
test or not or or you know but then I
saw reasons they're more used to like
running the software they have a broader
perspective right and its really good to
have them in your team and in those
meetings where decisions are made right
because I've been in teams were it's all
developers and I you know we obviously
made the wrong choices because then the
the software the rebuilding becomes
these model it's that they were you know
trying to crush now so how did we do it
as Jonathan mentioned there's this
session going on in parallel right now
that talks exactly how we did it but the
gist of it is is this event that we had
cold upscale awkward upscale was is that
we you know grabbed all the product
organization developers
Esther ease and we just ask them pick a
product right so they were like tables
where you would just you know it was the
browser product insights product and we
just walk in to whatever like you know
you were interested in but what it did
is that now that team into own it's an
entire stack you know the UI the
services the the back end so and then we
have like we went from from a team of
like you know maybe one s3 to a team of
a 50-50 split you know three developers
three histories and honestly like the
the results have been great you know I
was a little hesitant about it was like
hey we're going to lose our velocity
what's going on but and actually we
ended up being a lot faster because if
you think about it when we were hitting
these roadblocks there because we're
kind of waiting on hard work or you know
try it like sending it take it over to
the sre or getting waiting for that so I
highly suggest you watch the the talk I
think it's called inversion inversion of
control version of control so what's
your recording so to wrap this this
section we we'd like to introduce a new
a new role that we think should replace
es cerebral because traditionally a
series are you know to pull them they
kind of get handed out two teams for
like you know maybe like 3-4 months or
maybe once a feature is launched but we
don't we don't think that works that
well so what we'd like to introduce is
the idea of a product reliability
engineer so rather than having an SOE
that just joins the team for temporarily
it's a necessary that permanently on the
theme so what this does is that they
they are you know they owned the product
as well so they're interested in the
products goals and they you know they
want the product to ship so there you
know working with you together now what
we've noticed is that in teams that own
their entire stack so the UI to the back
end if they don't have an SRE it's it's
you know I we can see that it's they
struggle a bit and you know some teams
are lucky that they they have like
developer that's kind of like interested
in like operations and databases and
they kind of you know take on that role
like but they're like anna balancing
their their coding duties and ops 2 t's
so what we suggest is rather than like
having them balance it just like train
up the person to become like your s3
right brother you know and just make it
like that that's a responsibility we
honestly think sometimes like themes
that own their entire stack are too def
heavy they should have like at least one
essary all right so now that we've
dubbed some ops we're going to talk
about how containers made this even
better so docker is everywhere at New
Relic it is the fastest way to get
something done we've got a big
infrastructure to run it when our team
was getting started we was like
obviously we're going to build it in
docker that's going to be the fastest
way to get going the thing we were
replacing certainly didn't run in dr. it
was a huge pain to deploy if it had run
in dr. it probably wouldn't need
replacing because of an interesting
property of docker that I don't think
it's quite enough attention and it's the
docker is opinionated about software
architecture in a way that encourages
more robustly crafted applications think
about that for a second what this really
means is that it puts guard rails a
little bit around what your app can do
and shouldn't do it makes the right
thing much easier in some cases it
forces you to do the right thing the
power of doctor is not so much in its
technology which is interesting in its
own right but the the see the germ of
what makes this interesting to an
organization especially as it relates to
bringing ops and Dev together in
alignment has to do with how it surfaces
operational concerns to application
developers in a way that's
understandable to them and again forces
them to do the right thing when you give
too much leeway to how people develop
software you kind of end up with stuff
that gets grown organically and no
matter how much documentation you add
the thing is never really going to be
useful or fun or interesting to use it's
very different from craft and is very
different from from creativity within
constraints when you have designs that
are more carefully thought through you
get unuseful property the doctor has
called affordance
the service presents its interface in a
way that matches our capabilities to the
need you're capable of making an API
request to something your need is to get
the logs why not make the logs available
over an API request doctor makes that
happen for you and it gives it to you
for free dr. presents standard
operational interfaces for almost all
interesting service operations restarts
deployment again logging signals shells
all that stuff is just there no matter
how the service is built for our team
because we built everything in dr.
anybody that comes onto the team needs
to only learn one way to package one way
to deploy one way to interact with the
logs it's not a surprise you don't have
this model with well oh you have to
learn how to read its logs and this
thing way it does database thread pools
it's different and the way that it does
deployments is different we can't afford
to have that anymore the system is too
complex it needs to be simpler have more
framework around at more rules so that
they can work together dynamically we
were so taken with this property that we
decided well you know what doesn't run
in dr. yet is Cassandra when we started
the process we had doctor of running all
this stream processing stuff but
Cassandra was very traditionally managed
because that's how we did all the
databases at New Relic what we found
though is that no matter how fast and
how hard we wanted to iterate on
Cassandra it still just took too long it
would take us hours to get a simple
config change out to do performance
testing the reason for that was well
it's a database and we manage databases
in configuration management and as soon
as you touch configuration management
you might be touching all the servers
and you better just go get coffee
because this is going to take a while
where's that kind of time I'm now
spoiled because all my other services
are in dr. and I can employ those things
in 10 minutes without asking permission
why can my database work the same way
well in our case it was good because we
were already working in the dark we
already had a system that customers
weren't seeing so if we break it
horribly by running it in docker were
the only ones at risk here so great
opportunity oh crap it's a database you
know what Dockers not designed to run
databases
guess what it doesn't care about what's
on disk so we had some work to do we
actually spent about six or eight weeks
and we did some serious change to our
deployment system Centurion which is
actually you can use yourself it's a
little bit like a Capistrano for docker
and since it's just Ruby we were able to
teach it how to interact with Cassandra
over jmx and push all the buttons we
wrote some config management stuff to
make services that run Cassandra and
docker just just that so that we could
have persistent disk presented to the
Cassandra services lots of work it's big
job but we were able to go from taking
hours and three teams to deploy a change
to Cassandra to taking 30 minutes and
one team whenever we want the payoff was
tremendous we may have spent you know
six to eight weeks just making this work
but a year what is it 15 months on we
still get tremendous benefit out of this
because again that affordance is so
powerful we don't need to cut up a bunch
of different build scripts we just make
dr. images it's not a surprise it also
forces us to make better software again
Cassandra's properties about how you
have to be on the server to interact
with it about how much it cares about
disks aren't necessarily great it's just
sort of how it evolved but cramming it
and doctor actually helped us out
because it makes it easier to work with
we have a system that anybody can run so
you know we've got optimized builds of
cassandra and dr. now other teams can
actually use that and it's pretty
powerful so how in the world do you do
this right so like you've got cool story
bro but you know my motto this doesn't
like your monolith well for us it was
start small and i hope this could work
for you get that wedge in the door start
with the new services build them in
containers building the new way the
stuff available to do this is getting
better all the time Centurion like I
said can automate your deployments Kuber
Nettie's mesas and the like are all
getting pretty mature you can run this
stuff in the Google cloud platform and
in ecs on Amazon the stuff is pretty
solid now it's been around for a while
it may seem scary and weird and it is
but it can really help out with all the
things that you do so there's your
shopping list
treat data intake as a stream put it in
a stream processor break up your model
this one stream at a time put some ops
in your dad that will taste delicious
and do it in containers because it makes
things so much better and I got to tell
you given that monolith torn down we're
really does so much to open up your work
to the joy of creation the things you
learn tearing one of these down will
help you learn how to never build one
again thank you thank you for staying we
have about 10 minutes for QA so if
you'll have any questions please stand
up and our helpful MC Ashley will come
to you with a microphone you mentioned
midway through your talk that a
collector was a big problem and burned
out a lot of debs but you kind of talked
about replacing stuff kind of downstream
from the collector did you ever get back
around to actually replacing the
collector or with something
microservices so the way we're tackling
it is as I said one stream at a time so
it's put a downstream we're gonna have
to do that a dozen more times it's not
about right so there's there's a lot of
things in there as I showed so the goal
is eventually to just not have it but to
tear it out one piece at a time and
right now it's kind of a hot potato like
whichever team has the most services
still in there has to own it so you know
we're sort of racing like okay we've got
these three things left we've got a tear
out like let's get it going but yeah
we're still still in process with that
it's not done by any stretch
the company they work for actually the
state of California is currently going
through a journey on dev ops and we're
at the point where we're trying to scope
out what the DevOps engineers are going
to do so from us and our perspective
that's continuous integration and
facilitating the developers site
reliability engineer role I've read
about that and the way that Google uses
it sounds like you guys are using it the
same way at New Relic do you guys have
the product reliability engineer or I'm
kind of thinking that's what we're
thinking of as a devops engineer today a
product reliability engineer no Jose and
I just made that up yeah a sec so we we
continue to have the title site
reliability engineer you can imagine it
as analogous to another specialty like
QA engineer prior to the upscale reorg
site reliability what that title site
reliability engineer was the exclusive
domain of a site engineering
organization and if you had that title
you were necessarily attached to that
organization in some form or another but
we sort of blew that up and made like a
caterpillar liquefied and rearranged all
the everybody's and so now that's just
like teams that own a product or a pile
of services and those teams have what
engine whatever engineers with whatever
titles are most appropriate for working
on that thing so we don't treat those
specialties as areas of responsibility
we treat them as areas of knowledge or
expertise so sort of were you getting at
so so I I sort of rankle at the idea of
like oh you're a devops engineer
therefore you only do these things I
feel like that's an artificial boundary
and the idea is that oh you're you're a
reliability engineer that means you're
really good at understanding reliable
systems and I'm not going to expect you
to like get really good jvm tuning
unless you're interested in it or get
really good at understanding enterprise
software patterns or optimizing for
loops but if you know how to make a
database sing like awesome that's how
you advance within that title I was I
didn't know I think I mean the
difference that I wanted to make with
the the when I'm over here for people
with a product related engineers that in
the past when we had worked with the the
Google way of doing the masteries like I
felt like as a developer that they
the s-series were on loan to us and on
top of that they're also working with
other teams right so there was still
kind of like a mini a mini ticket queue
rather than a bigger ticket queue and
then when we did upscale and jonathan
and alice and leslie another co-worker
join the team like they were part of the
team right so they their goal was to
seeing this product this new pipeline
launched and you know they were not
going to leave six months out or
whatever so that's kind of like the the
gist of like what a product liability
engineer is at we actually Jose and I
work together when I was a site
reliability engineer in the Google like
parlance and so I was attached to five
other teams in addition to his working
with all those groups on their
reliability and while it was great and a
helpful form of that long-term vision
for like I'm gonna make this thing and
then I'm gonna be supporting it 12
months from now it was sort of missing
and so there are teams that have work
that I did for them a year ago under
that model I'm no longer attached to
that team or involved in any way but
they have to support it and that sort of
creates an awkward transition period
that with product reliability I don't
that that's sort of a issue expectations
are different on the dock arising the
database the TLDR short version of
docker izing your databases
there is no short version yes I guess I
mean the short version is that a queue
manage a database now as a developer
because it's just soccer deploy I need
to look at the long soccer logs in the
past I have to like SH in the box and
then asked Johnny hey what's the command
to get luck where the logs at we are we
didn't hear the question is are we just
changing the app and not the datastore
is that the question can you give me a
kiss thank you are you just changing the
Cassandra binary and not the datastore
no no we're just you know how do you
also so there exists Cassandra dr.
images that the the Cassandra foundation
publishes that's great that was a really
helpful start so technologically what we
did was we made this concept of like a
doctor eyes Cassandra host so you know
which man am I imagine and so it runs a
sort of specialized docker that just
does that and so it will do things like
volume mounts so we'll expose the native
disks into the container will use house
native networking so we don't remap the
ports we let Cassandra map whatever
ports it wants we actually tune some of
the disk subsystem things in ways that
are friendly to Cassandra so rather than
using docker for its multi-tenancy
properties we use it as a container
deliverable just a deployment artifact
so that's that's the difference there
thumbs up okay cool that's you were
getting at so yeah so we treated as a
service rather than a datastore yeah we
know how to deploy software so let's
deploy our databases like their software
let's go to the inside there another
question back
kind of link to that one with Kafka like
when you went into that dark environment
the wealth and customer-facing Kafka
looked like an initial part of
infrastructure so that came in docker
native did you have to solve persistent
issues with that system that you learned
something in that process sounds like
where did Kafka come from like that what
are we learning that process so Kafka
pre-existed when we started this project
it's actually run by another team we
joke on our team that kafka's like our
electricity if it goes out like nobody
gets any work done they don't run Kafka
and dr. on no so yeah Kafka is the only
thing I think that's not running in
docker but we could well there are
plenty of other non doctor eyes services
still it's not anybody strength to kool
aid yet right there's no reason you
couldn't run Kafka and dr. similar to
how we run Cassandra but that is a very
high risk change for that team they
don't have a dark environment to operate
Kafka in so they just haven't made the
jump yet I mean we might think of doing
that if they're like hey we're not gonna
have this big kefka cluster that the
entire company can use but you guys have
to make your own capital cluster and
then we might put it in darker or your
fun so if you don't have the dark
environment then how do you microservice
say well Madeleine well you can still
use the same pattern well you you really
need that dark in there you got to have
something I mean I would you're shaking
your head like I can't possibly do that
I'd you think hard about why like how do
you do QA without something that's not
customer-facing somewhere we found cough
gonna be super powerful for making these
dark environments because it lets you
duplicate traffic even read traffic like
we saw was was really really powerful
traffic duplication is a really
important way to get sort of live QA
environments and Kafka's pretty good way
to do that if you don't have anything
like Kafka there must be something you
have either continuous deployment
continuous integration I hope so 2016 I
mean there is no easy answer here my
answer is that it's essential like it
this is something you have to have on
the ground because how else are you
going to know that what you're
extracting does the right stuff it's a
tip
property of a monolith that the thing is
untested the thing is legacy code that
no one really understands the complete
specification for you need to run some
kind of environment that you can test it
with that complete specification which
in our case we found was the actual
traffic time for one last question
gutting all your answers wonderful well
thank you again for joining us thank you
for attending future stack and see you
at the keynote tonight have a safe trip
home everybody yes thank you both the
keynote is at four thirty in about 15
minutes downstairs
