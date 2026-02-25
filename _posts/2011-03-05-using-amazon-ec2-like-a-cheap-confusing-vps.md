---
title: "Using Amazon EC2 like a cheap, confusing VPS"
date: 2011-03-05
author: Jonathan Owens
---

Let's say you wanted a VPS-like server somewhere that runs all the time, gives you root access, and lets you install whatever you want. Let's say you also wanted to attach tons of space, a CDN, a database server, and anything else. Well you'd want to use some of the Amazon AWS products but you'd probably not want to have your VPS service hosted somewhere else, so you can centralize.

So you'd think, EC2 lets you run servers, and EBS lets you use them like they have regular hard drives attached, let's just buy a yearlong EC2 reservation and get a fat server for an amortized $75/month! It's a great plan, until you start to try it and realize everything in AWS is designed in little tiny pieces, not systems.

So then you try something like RightScale, or Judo, or Scalr, and you think, I don't need this autoscaling clouding magic scripty crap, I just want a server! Is that so hard? Well no, but it's $250/month from Slicehost, and you don't want to pay that.

Let's do something different. Let's get as close to a VPS as we can in EC2 using the simplest tools possible - just the [console](https://console.aws.amazon.com/ec2/) and the [ec2 API tools](http://docs.amazonwebservices.com/AmazonEC2/gsg/2006-06-26/setting-up-your-tools.html).

First, some concepts. AWS has its own language that is not very familiar. I strongly recommend taking an hour or two and reading the [EC2 User Guide](http://docs.amazonwebservices.com/AWSEC2/latest/UserGuide/) so you can get a handle on what's going on here. I tried to avoid it but really it's best to just read it straight through. This isn't a quickie project.

The core units we'll be focusing on are EBS volumes and EBS-backed AMIs. These are going to form the core of your server's identity.

EBS volumes can start as either a Snapshot, which is basically  a tape backup of a drive at a point in time, or as an empty drive. Once created, they can then exist either as a detached ("available") volume, which is just like an unplugged hard drive sitting on the table, or an attached ("in-use") volume, which is like a drive plugged into a server. (Note that the server they are attached to doesn't have to be running.) You can leave EBS volumes sitting around detached as long as you like, and take snapshots of them whenever.

EBS-backed AMIs are awesome, because we can take any Snapshot and make it the root device of an AMI. Then whenever you start the AMI, it would be like like taking that Snapshot (remember, tape backup), buying a server, copying the tape data onto the server's main hard drive, and starting it up. You still have the tape, and the server now has the tape data (as copied to its hard drive, a new EBS volume) as its starting point.

The thing about AMIs is they don't really exist as servers, they exist as the idea of a server. Sort of a specification, not a saved state. They have a particular set of disks (or snapshots) to attach, a kernel to run, and an architecture, but that's about it. In most cases they're designed to destroy all the data they create during their lifetime, because AWS likes you to run things on-demand, not forever. Well in a VPS you want forever, so we're going to do some tricks to get there.

What we're going to create in AWS is an AMI that, when you create an instance from it, creates and boots from an EBS volume containing the data in a Snapshot. When the instance is stopped (shut down) or terminated (deleted), the EBS volume it made will sit there detached in your list.

This is important because while you could use an EBS-backed instance as a VPS simply by never terminating it once it's running, if for some reason it was terminated, you'd lose all the data on it unless you took a snapshot, but even then you'd be restoring from your last snapshot, not the moment the instance stopped. This is why termination protection exists, but a checkbox on a webpage is not enough to protect production data.

Let's do some work. Look for a the AMI you want in the [AMI list](https://console.aws.amazon.com/ec2/home?region=us-east-1#s=Images). At the time of writing `ami-3202f25b` (Ubuntu 10.04 20110201.1) is a good one. Launch it, and go to *Instances*. When it's running, right-click it and select *Create Image (EBS AMI)*. You'll see an AMI go to Pending in your list of AMIs Owned By Me, and a Snapshot go Pending as well. Get some paper and write down the Snapshot ID, then go back to AMIs and write down the Kernel ID.

Now drop into your terminal where we'll be using some of the ec2 tools (you installed them, right?). You can't do what we want from the web console, which is to set up a server that keeps all its disks around when you terminate it. Use `ec2reg` like so:

```bash
ec2reg -n 'Ubuntu 10.04 20110201.1 Base' -d 'Basic ubuntu server configuration' --root-device-name /dev/sda1 -b /dev/sda1=your-snap-id:8:false -a x86_64 --kernel your-kernel-id
```

What you just did was make an AMI (again, the *idea* of a server) that runs Ubuntu 10.04. What's special about it is that when you make instances out of it, those instances don't destroy the data they made over their lifetime. The `false` at the end of the `-b` argument is the trick.

So how many servers do you want? Just launch as many of those AMIs as you like and even if you terminate them, the stuff they do will still exist. You probably never actually _want_ to terminate them, only stop them, but you would be safe even if you did. Now you can safely install software right on your instances without running launch scripts or pasting in a bunch of userdata.

Once you start running your instances, you'll still want to take snapshots of their volumes every so often. As long as you don't terminate your instances, only stop them, they'll work just like servers with hard drives. If you do terminate one, you'll need to do a little runaround. You'll have a detached volume that was that instance's root device. You can't boot a server from a detached EBS volume, only a snapshot. So take a snapshot of the detached root device, then run `ec2reg` again with that snapshot as the snap-id. You now have a new 'rescue' AMI that really only represents that one particular instance, so when you launch a replacement instance from it, you should be right back where you left off. You can keep that AMI around if you want, the important thing is the snapshot. You can always create another AMI based on a standing snapshot that has the data you want, just make sure you pick the right kernel and mountpoints. Documentation helps here.

So this is great for software on a server's root drive, but let's say you want to run a data storage engine on your VPS-like EC2 setup. You wouldn't be getting much of the benefits out of EC2 by having all that data stored right on the boot drive, far better to use a separate EBS volume for your data, or even a couple of them RAIDed together. This allows you to take data snapshots separately from system snapshots.

There's 2 ways you could do this. First, we'll design a setup where all the instances you want will have a data volume mounted when they are launched.

We'll need to make a different AMI to represent this, because you can't edit the launch block device configuration of an AMI once you've made it. We'll use  `ec2reg` again:

```bash
ec2reg -n 'Ubuntu server with 10GB at sdf1' -d 'Ubuntu 10.04 with new 10GB volume at /dev/sdf1' --root-device-name /dev/sda1 -b /dev/sda1=your-snap-id:8:false -b /dev/sdf1=:10:false -a x86_64 --kernel your-kernel-id
```

This will create a new 10GB EBS volume at `/dev/sdf1` when this AMI is launched. It won't actually be mounted in the OS, or even formatted, because the root device snapshot we're working with has no idea that it exists. If you're launching these for the first time this should be fine, because you can format it, declare it in `fstab`, etc. and your changes will be safe due to the persistent root device. If you terminate an instance and have to reattach its snapshot to a new AMI later, you'll need to snapshot both the root and data volumes and include them in the rescue AMI:

```bash
ec2reg -n 'Redis slave server rescue AMI' -d 'Rescuing the redis slave from snapshots' --root-device-name /dev/sda1 -b /dev/sda1=instance-snap-id:8:false -b /dev/sdf1=instance-data-snap-id:10:false -a x86_64 --kernel your-kernel-id
```

This will make an AMI that will let you relaunch that instance with both its root device and its data intact.

The other way to do this is to simply create some EBS volumes and attach them to the instances you made from the first AMI. They won't be destroyed if you terminate the instances because they were attached after the instances launched. Just don't forget to reattach them if you have to rescue the instances, or include them in the rescue AMI as shown above.

Taking into account the costs of data transfer, snapshot and EBS storage, and instance type runtime costs, this may or may not actually be cheaper than a standard VPS. You get a lot more headroom though, and easy integration with other AWS products.
