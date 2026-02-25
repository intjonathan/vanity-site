# Kubernetes the Database

This was a talk given at KubeCon + CloudNativeCon 2018 in Seattle, WA. Me and Maryum Styles were working on a team operating a Mesos cluster and needed more advanced inventory and orchestration tools than Mesos could provide. We created a Kubernetes API server cluster with custom resources, along with inventory software to populate objects based on AWS and datacenter inventory. With the broad client support for first-class operation locking and object inventory in the Kubernetes ecosystem, we could accelerate our cluster management tooling dramatically.

## Abstract

In the operations world, one of the hardest problems is keeping track of your inventory: Which machines belong to which teams? Which machines are in service? How long have they been there? At New Relic, the ability to keep track of a massive inventory that runs across multiple providers quickly became an unbearable task so much so that it required designing a completely new central {tracking?} system that could scale with a large infrastructure. In this talk, you’ll learn how Jonathan Owens and Maryum Styles used the Kubernetes API server to jump-start this design and create a unified infrastructure description service. They will share how they defined resources, created controller services, and dramatically decreased the process of manual updates.

## Video URL

https://www.youtube.com/watch?v=eja7b3tahMg

## Video Transcript (auto-generated)

welcome welcome check check check hello
sounds like we are heard clearly thank
you so much for coming to our talk here
in one of the last sessions on the last
day and we're so happy to have you and I
really hope that what we have to say
today will really help you out in your
work when you return I am Jonathan Owens
I'm aim stylist I worked for New Relic
in the Portland Oregon office the main
engineering headquarters they work in
San Francisco because we're a public
company you get to take a look at our
safe harbors slide it's worth clarifying
that this is not a talk about running
databases on kubernetes there are many
excellent talks about that here what
this is actually about is running
kubernetes components themselves much
like a database in our case we use it to
store database of our hosts inventory on
our different cloud providers and
hardware providers from around the world
first a small question what's on your
kitchen counter right now you probably
haven't been home for a few days you
might not remember exactly you might
have a cleaner kitchen than I do
but when it comes to answering questions
about what you have it gets harder when
that thing is even further away again
you're probably not close to your
kitchen right now but what if you had a
rental property say in Germany could you
tell us what was on that counter could
you tell us what your rental what's your
tenants we're doing in there not so fun
so we had a similar problem on our team
at New Relic what team is that we work
on a container fabric team that's our
internal platform for deploying and
writing containerized services
developers build and deploy excuse me
building and push up code from their
laptop as containers we then take those
containers and run them on our platform
that platform runs DCOs
which is a mezzo space application
scheduler as this actually runs the
services we don't have kubernetes
running services just yet this is a
fairly old platform so similar to ubers
the story we picked DCOs and mezzos at a
time when it was stable and served as
well and it's a pretty decently sized
deployment we have about a thousand
machines most of which are hardware
and it's running and now in two regions
on three different infrastructure
providers again including our own
bare-metal data center and the purpose
of all this of course is to run user
services make them available and make
them scale so the way we actually manage
this thing is a lot of ansible in
terraform before we added this inventory
concept and used the kubernetes
components there was a lot of well we
got to startup machines with terraform
tara forms going to talk directly to the
infrastructure provider once those
machines are created by the
infrastructure provider we turn around
and take ansible ask those
infrastructure providers again for the
things that we just made
then go and orchestrate the changes on
the machines that we can put them to
work
for all this to function we need a total
network connectivity across these
components so from the orchestrating
machine that was running ansible and
jenkins that might be a laptop that
might be another machine in the CI
system we would need to connect over a
VPN to the infrastructure provider to
talk to it about what it built and then
also over that same VPN to the actual
machines so that we could actually do
work on them a funny thing happened on
the way to scale we added a European
region last year this is the first new
separate region that we'd had to build
and with that came a whole new
networking set up including another VPN
so all of a sudden this gets awkward
right which VPN do I beyond do I have to
be on a separate one to look at the
European ones versus the North American
ones and then when we talk to the people
that were running this project they
described a future that looks a little
bit like this this made us a little
uncomfortable as it should you that's a
lot of lines that's a lot of networks we
can't really operate comfortably in a
world where every single time we want to
touch something there's some central
point that has to work on it and we have
to figure out just which cluster do we
talk to at all so wouldn't it be nice if
these things could be more independent
wouldn't it be nice if we could just
sort of push this intelligence closer to
the edge so we looked hard at the
problem and realized that the problem in
fact was not one of connectivity it
wasn't essential for us to talk to all
these machines directly all the time we
just needed to know what was there we
had an inventory problem and not
necessarily a connectivity
problem if we can find out what's there
and what's going on and have that be
updated as a reflection of what's real
on the ground then we can turn back
around and go over the whatever VPN is
appropriate to region and clusters where
there's something interesting going on
and actually make the changes this
turned out to be a pretty good idea the
more we thought about it and well what
else could we do with this we could
actually if we had a place where the
state of the clusters was easily
available and always up to date we can
hang more parts off of it we can work a
straight monitoring off of this thing we
can have more self driven orchestration
if it's easy to answer the question of
what's going on where but what could we
do that with well we thought about using
terraform State so if you've used
terraform you might be familiar with the
file it makes this TF state file and
it's really useful because it lets you
look at all the things that terraform
did and what state it believes the world
to be in but it only updates when you
actually run a plan or when you there's
a resynchronization operation you can do
and that only happens on demand it's not
really live on top of that it's just a
file so you can find it in s3 but you
have to like read and write this entire
file when you want to do anything so
just really clumsy we looked at
commercial datacenter inventory
management tools this would be something
like the vice 42 or n light these are
marketed for towards people that are
running their own data centers which we
are but they're really deeply concerned
with things what's the Machine what rack
is it in what are your PD use how many
hard drives do you have on hand and not
really anything higher level than that
and we really wanted the ability to
express like I've got a cluster and
here's what's important to me about a
cluster and here's its regen and it
wasn't really gonna extend into that
world as I said we're running on DC OS
so you might be asking yourself well
what about that doesn't that have
anything you could use here sadly the
answer is no do you cos because it's
built on mezzos is really built on a
responsive pattern mesos itself waits
for offers it waits for agents it waits
for things to happen in the
infrastructure and then asks for things
to happen when those have occurred
there's not really a declarative world
inside DCs and all the components that
come with
it and that was really what we wanted we
wanted this pattern of declaring state
so we looked at what didn't work what
did we learn from that and what do we
actually need well it needs to be a
single source of truth we can tell that
much that's the point of this whole
thing that's why we don't wanna have all
those lines we wanted the ability to
update it from anywhere so you know the
European region and then future regions
or future separate networks could just
go and talk to this thing without having
to tunnel to it it needed to be
compatible with the ansible inventory so
we want to make this one step at a time
we want to change the inventory but
we're not yet ready to change all the
orchestration and so we just need to be
able to plug this thing into the ansible
script and have it go it needs to be
production quality of course if we're
gonna hang monitoring and other things
off of it and then going back to the
thing we learned from the DCIM tools we
want the ability to put higher level
abstractions this is a cluster this is a
log host there's what's going on that I
want to have happen more so than just
like where's my stuff so being operators
of a cluster and being in this space we
started reading all kinds of websites
including the kubernetes one and if you
look at the kubernetes website you might
find a list something like this and you
know we looked at that and went well we
have most of these things okay that's
cool you kind of need those things to
run a cluster but there's one thing on
here that DCOs notably really doesn't
have and that's this centralized
configuration thing that it sounds
pretty neat how could we get just that
well it turns out you totally can so the
API server is one of the first things
that you provision when you build a
cluster and it's the thing that holds
the declarative state of all the objects
that represent the clusters behavior but
it can do a little more than that it
comes with a bunch of stock objects like
deployments and stateful sets and
services but you can extend it with
custom objects that can really represent
just about whatever you want they're
designed for orchestrating more complex
application deployments on kubernetes
but they don't have to be that so what
would we need to do to make an inventory
system that would consisted entirely of
CR DS well we would need a basic object
structure so we need to tell
the API server here are the objects I
want to talk about things like houses
things like clusters that's just a blob
of llamo you'd give that a thing we need
something that lives inside these remote
clusters to talk to the infrastructure
providers and see what do you have what
hosts are under management what has
changed and then we just need an adapter
so an tubulin work with this thing and
find out what is available and what
should it do so to give you more detail
about how we built those components I'm
going to turn it over to Miriam so we
need a piece of code that would query
our provider API it's a push data into
our cube server we run a pretty naive
kubernetes controller that would manage
this process and then we call them
fetchers because they fetch data they
also push data we still call them
fetchers so we're gonna go over how they
work this is the overview and we start
by making our API call to our provider
we have one fetcher per provider and the
we are our providers our AWS software
and then we also have our data center we
take our provider response and store
that in a ghost truck internally so we
can work with it
we then think that struck and convert
into a host object our host object is a
custom resource in kubernetes customer
expressions are really awesome because
of how you to extend the kuving's
framework with their own custom restarts
you get to use the same powerful api's
along with the crud you get a caching
you get informers you get Watchers and
you can take all of that and do amazing
things in order to create a custom
resource you can follow these steps I
mean start by doing your custom resource
definition mo file you then create like
a ghost truck that you use in your code
you update the register file with your
new object name and then you run your
cogeneration your code generations where
the real magic happens that's where you
get all of the files I will do the
Informer's the like create the updates
the deletes all of that is included with
you and then you apply your CRD file
to your API server so that way the API
server says I
hostess almost I could look at a host
see idml file this is what ours looks
like it's pretty minimal says hey server
look out for a custom resource
definition and look it's going to be a
host and so it says like hey this is
what a host is gonna look like
well TRD file so you have the option to
use validation and validation can say
hey for some fields we don't want to
have any numeric characters we'd had
chosen to use that and we chose to just
with the option I think can be really
great in certain situations not in ours
this is what our host looks like and go
and it sort of programming with
internally and the type meta stores the
kind of API version we saw earlier and
then our object meta stores and
annotations and labels which we look
into you have respect which represents
the desired state of yourself object and
then your status which represents the
status status of your host object we use
our labels and annotations a lot for our
inventory service labels are really
awesome you can use them to search and
filter for example if you were like hey
I want to see all of my hosts that are
log machines that are running in AWS you
can say you can use the command line or
the API request to do that annotations
are really great we using the store data
about the hosts that we then use in
downstream systems or in scripts so now
we got our host object we can store it
in our database via an update or create
API call to our cube API start right and
it is going to a real database it's
going to at CD one of the things you
have to make sure you do is like if
you're using an update call you can
possibly clobber your data and so on our
case we use patch to make sure that we
don't drop any fields in our struct
object how has it all worked together in
real life like where's our Q&A living so
we use terraform to create our ec2
instance in our elastic load balancer we
then use ansible
to add our certs
and then use install cube API and the
controller manager so know about the
note about the certs our certs are added
by hand and currently if we have any
human or a new service that wants to
talk to her cube cube API server we need
to approve the new certs and that's a
manual process or something we hope to
automate in the future and the only
reason we're installing the controller
manager is because it allows you to the
cert approval process if you heard a
kubernetes the hard way by Kelsey
Hightower we're doing kubernetes the
easy way because we're only doing the
first three steps
we're not to point any pods or any
workloads we're not worrying about any
pod networking so if you're interested
in playing around with this model I
would say it's pretty simple and really
easy we're still deploying our fetchers
in DCOs because as our existing platform
in the future we plan to transition over
to kubernetes and so now how do we use
our data so this is what a host looks
like when when it gets the command line
and we can seal it in the yellow file
let me take this data and we can
transform it into JSON for instable and
to do this process we use a Python
script that goes through makes api calls
for a cube api server and then it'll we
take that and spit it into a JSON file
one of the things we do in our Python
script is filter out machines that are
out of service or machines that are not
working anymore and you there's actually
a way to manage this life cycle within
kubernetes using finalized errs
finalized errs are defined in your hosts
your host object and what happens is you
add finalized errs to your hosts and
then when you make a delete call there's
a deletion timestamp added to your host
object and then you'll have a controller
that watches your deletion timestamp and
then do any steps needed to delete the
host and it will remove those finalized
errs and then the hosts will probably be
deleted we're not currently doing this
we have yet so we have the final average
of our hosts we've got to write the code
that would actually remove the final
answers and watch for deletions
timestamp
but it's highly recommended to implement
that so let's talk about how we're
expanding these patterns
Thank You Miriam so we found this
pattern to be already really useful
right we've spent a lot of time on how
we use it for inventory but that was
really just like the first thing as soon
as we had this place where we could
easily define declarative desired States
for our cluster a whole new set of
opportunities opened up you know we went
from this world where we had this sort
of phased manual back-and-forth it was
very iterative it was very hand ribbon
and it really couldn't take us into a
multi region future with the API server
in the middle just the way it's made
it's made to run in this really
decomposed way with lots of clients that
don't necessarily know about each other
working on objects together and the two
kind of big aspects that make this
possible is the TLS mutual off over
that's suitable for use over the
internet so that lets us get rid of the
VPNs that lets us get rid of the sort of
network connectivity requirement and
then the other aspect is the way it does
consistency ordering for clients so when
clients make update requests and then
they try to post it back to the server
if another client has already changed
that object they get told and so it's
very easy to coordinate lots of things
that work on the same host this lets us
add more stuff so if if we've got a
place where we can declare how things
should work we can then use that for
other things so we used to do monitoring
I'll pick out that as an example when we
added hosts to our monitoring system
before it used to be get driven we would
have a script that would go and do
basically what ansible does go and ask
for all the servers and then create a
config blob in a git repo and then the
monitoring server would go and actually
pull from that repo and update its
configs based on the repo well that's
just like kubernetes with extra steps
right like this is a big hassle so
instead we get to do that dynamically we
were able to lift up all that monitoring
logic into the API server so now as soon
as the inventory changes the host
objects in the API server the monitoring
server can pick that up right off the
data plane because they're listening for
new hosts and go reconfigure itself
right on the monitoring container way
faster
no poll requests required no gate
connectivity it's just much simpler to
think about really anything that needs
updates about what goes where can do
this you know monitoring is a great
start we really want to go further than
that so you know it's not yet
declarative the host objects that we
have are just really status there's not
really a spec that we rely on for much
yet and creating a new one only happens
when a new one actually appears in the
infrastructure provider so terraform is
still making direct calls to the
infrastructure there's no controller for
building new instances like a cluster
autoscaler might do
similarly ansible has to just like go
reach directly out to these machines
it's not running right on the machines
though we have managed to actually build
on this a little bit and we now have a
component that lives on the actual hosts
that monitors for when changes need to
be a run by ansible and they can go ask
for ansible to be run on them so it's
like a little half step towards having
self-managed clusters and we expect to
continue to evolve that please ask for
more details after if you're curious or
contact us online so this is how he came
to think about kubernetes you need a lot
of parts to make kubernetes go all those
features you saw earlier each one of
those is like an entire software project
in and of itself it's like a hamburger
right you need a lot of stuff to make a
hamburger you need something from the
bakery something from the ranch
something from the dairy something from
the garden and they all come together
and make something sort of greater than
the sum of the parts but you know if you
just take the G's G's good just on its
own and you know we took this sort of
cheesy part of kubernetes this sort of
thing that's delicious on its own and
made a sandwich out of it you know and
still cheese in there it's just it it's
sort of differently shaped when you eat
it all together so you know we encourage
you it was so valuable for us to look at
these parts of kubernetes look at the
opinions about how distributed systems
work that are baked into the code that
are encoded in it and then think about
how you can apply that to your own
systems we've struggled for years with
this pattern of how do we converge lots
of stuff and the folks that build
kubernetes have too but they've come up
with ideas that are better than we
and we've had a lot of enjoyment
applying them to our world thank you so
much we'll take questions at this time
and we're so glad that you came thank
you please shout out your questions we
will repeat them for the benefit of the
audience
no questions come on you have to ask me
like what I had for breakfast this
morning or something yes do we think
this will last the test of time given
the changes happening in kubernetes
absolutely so custom objects and the
custom resource definition are
continuing to be expanded I see movement
happening especially in the API
machinery sig to expand on what custom
objects can do ideas being thrown around
include things all the way up to and
including converting all of the core
kubernetes objects to CR DS themselves
that get loaded when you install
kubernetes or even generally just
teasing apart more of the API component
from kubernetes core objects you know I
don't think we're running at
cross-purposes with what they're trying
to do with the API server at all and the
more we talk to them about it the more
kind of useful things are emerging from
that dialogue so yeah we think this will
totally go yes
you know things into CRD
yes that was the the core insight was
that like what if we didn't write an
inventory service what if we just used
CRTs as the inventory service and then
wrote the machinery around it to make
that CRT useful yeah
what put kubernetes on the radar for
this purpose and not others so we were
looking at moving off DCs and to
kubernetes and still are and so this
forms sort of the camels nose under the
tent to go like well what if we just use
this part how would that go and it gave
us an opportunity to learn the system
we've seen other teams inside New Relic
look at what we've done here and go I
want that and so we've already got a
team using it to manage the load
balancers any place where I want to
declare state in some flexible way and
then serve that and reify that in in
actual machines it's proven to be pretty
useful I don't see any reason why you
couldn't apply this to non containerized
systems is this open source no we would
like to I think the fetchers especially
for things like AWS and IBM cloud are
pretty like you could make those generic
in a meaningful way right now we're
working with another team inside New
Relic to get them to use this as well
for their large cluster orchestration
and I think as part of that process will
look at teasing that apart because I
don't there's no reason why we couldn't
make these open they're pretty pretty
powerful so yes we're working on that
it's poem API periodically we want to do
something more smart like that but since
we don't have a ton of hosts like we
only like a dozen right now we're just
pulling time for a few more questions
yes
mm-hmm yeah so the question was how do
we deal with are we thinking of
replacing this terraform flow or - we're
terraform still has to go and reach out
and make things without being queued by
changes to the CRT that reflect the
question so because of our migration
plans off DCOs I don't think we're gonna
make that change on this system but when
we start building kubernetes clusters we
absolutely want to do that and so we're
trying to get really plugged in with the
cluster API folks because they're
talking about a design for managing
kubernetes clusters that is totally this
idea you make a cluster object then you
make controllers that create that actual
cluster in the real world that's the
world we want to operate in because we
expect at our scale to have many
clusters and I don't want to manage many
clusters with like a whole bunch of pull
requests - all over the codebase I want
to make just an object and an API server
and how to go so yeah that's that's
where we want to be I hope to come back
next year and be like check it out
cluster API is the thing yes
so do we expect to continue using this
when we're off DC LS is that
and why is it difficult to move off DC
us to excellent questions I do expect he
I don't know what do you think should we
still use the host objects to manage
cuber nineties clusters I mean I don't
know if I think what the Machine like
from cluster AP I have a machine-type we
probably won't have to worry about it as
much yeah the cluster API folks have a
whole set of CRTs that they've defined
that are pretty close to what we're
doing here so they define a machine
object and a machine set we're gonna
look real hard at that because you know
we may have to straddle two worlds we
may have a host inventory that has both
machines that are running DCOs and also
machines that are running kubernetes and
those may take different representations
honestly we don't know we'll see as for
what's hard about moving off DCOs i
think in our case it's not actually
going to be that hard because all we run
on DCOs are stateless services and those
are pretty easy to migrate to kubernetes
i think what's going to be hard is is
pulling on the stateful services which
we do want around kubernetes and we're
very early in that process still so
we're not certain yet we've made it
easier on ourselves it's not shown in
the architecture diagrams but we do have
a piece of software that sits in the
middle so when teams deploy their
services that go through a piece of
software called Grand Central that's
what actually talks to DCOs and creates
the applications we expect that same
flow to happen when they deploy to
kubernetes for the most part and so we
just changed that part in the middle the
grand central part and have it just
deploy to another cluster and have that
go which makes it easy for the service
owners and hard for us because we have
to make sure that both of those things
work well alright any more questions
yes
there aren't any conflicts because
they're not really talking to each other
the fetchers are deployed in DC waves
but they're just communicating on the
cube API server via API request it's
just over it like HTTP request so
there's not really any like fighting
that would happen
there's no situate space for that yeah
we're only running like just the small
part of kubernetes right there's no CNI
there's no cubelet there's no you know
service objects no IP mapping and so all
the parts of kubernetes that sort of do
interesting work that might be in
conflict with DCOs simply aren't running
and when we stand up communities
clusters we expect them to be on
separated equipment we're not gonna try
to like co-locate those I think that
would end in tears
all right one more question oh yes sorry
mm-hmm so the question is like what's
the source of truth and it seems like
yeah so how like who asserts what a host
should do and what it looks like
yeah I think it's mostly terraform right
like when we make a host and we we give
it a role or we give it labels that
mostly comes from terraform right I
guess it depends right cuz we're having
AWS and we have our data center so for
our data center we would just change the
role like in the API calls and then one
cube API queries that provider I
wouldn't know all of that was
terraformed yeah I guess you would just
it would end up in AWS the API somehow
and that's API would be I guess the API
is really a source of truth and then
cubic cubic Lee I saw like leaning on
that and that's where it's going with
this data from so I wouldn't say I was
it's the provider and not really a
terraform much of the intelligence about
what a host does in our system is
determined by the host name so that gets
set on creation of a host and then
almost every tool that sort of is
downstream of that is like oh you're one
of these I'm gonna do that thing with
you and that's done not for any
particularly good architectural reasons
but it you could conceptually it's the
same as like putting a bunch of labels
on it which is how that's often done all
right that's all the time we have thank
you so much for joining us today we hope
you have a great conference and safe
trip home
you
