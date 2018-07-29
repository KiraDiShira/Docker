- [index](https://github.com/KiraDiShira/Docker/blob/master/README.md#docker)

# What are containers?

## The Bad Old Days

The business needs a new application, so IT needs to go and procure a new server to run it on, and of course that server's got an upfront CapEx cost plus a bunch of OPEX costs that kick in later, things like the cost of powering and cooling it, administrating it, all of that stuff.

What kind of server does this application require? We don't know.

And in that case, IT did the only reasonable thing. They erred on the side of caution and went big and fast because the last thing anybody wanted, including the business, was dreadful performance, that inability to execute and potential loss of customers and revenue. 

We ended up with a bunch of massively overpowered physical servers running at like 10% of what they're capable of, maybe 20% on a great day. However you look at it though, it was a shameful waste of company capital and resources.

## Hello VMware

Then along came VMware, or I should say along came the hypervisor because there's not just VMware out there.

Instead of dedicating one physical server to one lonely app, suddenly we could safely and securely run multiple apps on a single server. 

But, and there's always a but, it's not a perfect solution. Of course it's not.

## VMwarts

<img src="https://github.com/KiraDiShira/Docker/blob/master/WhatAreContainers/Images/wac1.PNG" />

In the hypervisor virtualization model we take a single physical server. It's got processes, memory, disk space, all of that stuff, and we know we can run loads of apps on it. 

Now I'm just going to go four here because it keeps the diagram nice and clean, but to accomplish this we create four virtual machines, or virtual servers. Now these are essentially slices of the physical server's hardware. So, for example, virtual server one here, we might have allocated it, just for arguments sake, we'll say 25% of the processing power of the physical server. Remember we're big picture stuff here, so maybe 25% of CPU, 25% of memory, and 25% of disk space. And maybe we did the same for the other three virtual machines. Now these virtual machines are all slices of the real resources in the physical server below.

Then each of these virtual machines needs its very own dedicated operating system, so that's four installations of usually Windows or Linux, each of which steals a fat chunk of those resources, CPU, memory, disk just in order to run. No applications yet. This is just to run the operating systems. 

And then a lot of the time these operating systems need licenses. I mean red hot enterprise Linux isn't free, and Windows certainly isn't either, so there are costs right there in both resources and budget and costs that, I don't know, it just feels like a shame that we have to have them. Because you know what? None of us are in the business of seeing how many operating systems we can run, manage, and pay for.

It's not just the cost of licensing these operating systems. Each and everyone needs feeding and caring for, so admin stuff like security patching, maybe antivirus management. There really is a whole realm of operational baggage that comes with each one. And VMware and other hypervisors, as great as they absolutely are, they don't do anything to help us with these kind of problems. So yeah, VMware and the hypervisor model change the world into a better place, but it's still got its issues. There still are efficiencies that can be gained by moving to better technologies and better methodologies.

## Containers

<img src="https://github.com/KiraDiShira/Docker/blob/master/WhatAreContainers/Images/wac2.PNG" />

Result? More space for containers, therefore more apps, and apps spin up in usually less than a second.
