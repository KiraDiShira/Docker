# Building a Secure Swarm

## The Big Picture

Swarm is absolutely still at the heart of everything Docker is doing. The reason is Swarm has two parts.

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss1.PNG" />

What is a **Secure Swarm** cluster?

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss2.PNG" />

Well at the highest level, it's a cluster of Docker nodes. We've got managers and workers, and everything secure. So we've got Mutual TLS where workers and managers mutually authenticate each other, and all of the network chat is encrypted. Plus, the cluster stores encrypted, and it gets automatically distributed to all managers. And we can use labels to tag nodes and customize the cluster how we want it. Anyway, once we've got the cluster, then we can start scheduling containers to it. So instead of running individual Docker container run commands against specific nodes, and every time having to think about which node we should be running them on, well instead of that, we just throw commands at the cluster, and we let Swarm decide. 

So Swarm does all of the workload balancing and the likes. Great well, we can run two types of work on the cluster. Native Swarm work, and Kubernetes. Though, at the time of recording, right, you can only run Kubernetes work on Docker Enterprise edition.

## Swarm Clustering Deep Dive

Docker's had this notion of **Single-engine mode**, and **Swarm mode**.

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss3.PNG" />

Single-engine's where you install individual Docker instances, and you work with them all separately. Swarm mode, though, that's where you bring them all together as a cluster.

So let's go through building a cluster. We've got a node here, Docker installed. Throw a simple command at it

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss4.PNG" />

and we've got a Swarm. 

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss5.PNG" />

And that node now, is flipped into Swarm mode.

Behind the scenes there's a ridiculous amount of magic just happened.

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss6.PNG" />

This node's now the **first manager** of the Swarm. And the first manager in any Swarm is automatically elected as its **leader**. And that makes it a bit special. For starters, it's the **root CA** of the Swarm. I mean, you can configure external CAs if you need, just parse in the --external CA flag. But if you don't, this one gets the job. It's also issued itself a client certificate, built a secure cluster store, which by the way is ETD, and that's automatically distributed to every other manager in the Swarm and it's encrypted. And all of this for free, right? As in, we didn't have to do anything. It's done for us, including a default certificate rotation policy. Epic. Oh good grief, there's more. It's also created a set of cryptographic join tokens. One for joining new managers, and another for joining new workers. 


So let's join a new manager. We take another node running Docker in Single-engine mode, and we run a Docker Swarm join command on it. We give it the cryptojoin token for managers

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss7.PNG" />

and it's part of the Swarm.

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss8.PNG" />

And obviously operating in Swarm mode. And see how the cluster store's been extended to it. Oh, and it's been issued its own client certificate, right? That identifies who it is, the Swarm it's a member of, and its role as a manager. Sweet. Same again for a third.

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss9.PNG" />

Swarm managers are automatically configured for high availability, so any of these can fail, right? And the cluster keeps going.

Behind the scenes, though, only one of them is truly active, and that's the leader, which is important, right? Every Swarm has a single leader manager. The others, we call them **follower** managers. 

When you issue commands at the cluster you can issue them at any of the managers. It's just if you hit a follower manager, it's going to proxy commands to the leader.

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss10.PNG" />

If and when a leader fails, we have an election. One of the followers gets elected as a new leader. But all of this is handled by **Raft**, so within a Swarm, okay, all that distributed consensus stuff is done by Raft. The same as Kubernetes, actually. Raft's the go-to solution for distributed consensus these days. 

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss11.PNG" />

Now then, the ideal number of managers is three, five, or seven. Any more than seven, and you can end up spending more time thinking about decisions than actually acting on them. It's like deciding what to eat. If there's a handful of you it takes two minutes. If there's 20 of you, man, you can kiss goodbye to half of your night arguing. And it's the same with Swarm. So three, five, or seven for managers. Just make sure it's an odd number. That increases the chances of achieving quorum, and therefore avoiding split brain.

Now, as cool as Raft is, right, it's not a fan of slow or unreliable networks. I mean, who is, right? But this is important. Connect your managers over decent, reliable networks. 

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss12.PNG" />

So for instance, okay, and this is just an example. If you're in Amazon Web Services, don't go putting them in different regions. Across availability zones within a region? That's probably alright. But going cross region is just asking for pain. 

Okay, well look, adding workers is the same. Docker Swarm join again, only this time give it the Worker join token. And we can have a mix of Linux and Windows. Great if you're running hybrid apps.

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss13.PNG" />

Now when you join a worker, it does not get access to the cluster store. That's just for managers. But what each worker does get is the full list of IPs for all the managers. So if one of them dies, the workers can just talk to the others. Oh, and they all get their own certificates.

Anyway, like with the managers, the certificate identifies who the worker is, the Swarm that it's a member of, and what its role is: worker, yeah? Now then, the workers do all the application work. 

That's a Swarm. A cluster of managers and workers, a full-on PKI where the lead manager is the root CA, and it issues every node with a client certificate that gets used to a mutual authentication, role authorization, and transport encryption. I love it. And on top of that, we've got a distributed encrypted cluster store, cryptographic join tokens, and loads more. And the best bit, it's all just built in and works out of the box. Take a minute to let that sink in. I kid you not, it's something special. Secure out of the box, and easily configurable.
