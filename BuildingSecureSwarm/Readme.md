# Building a Secure Swarm

## The Big Picture

Swarm is absolutely still at the heart of everything Docker is doing. The reason is Swarm has two parts.

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss1.PNG" />

What is a **Secure Swarm** cluster?

<img src="https://github.com/KiraDiShira/Docker/blob/master/BuildingSecureSwarm/Images/bss2.PNG" />

Well at the highest level, it's a cluster of Docker nodes. We've got managers and workers, and everything secure. So we've got Mutual TLS where workers and managers mutually authenticate each other, and all of the network chat is encrypted. Plus, the cluster stores encrypted, and it gets automatically distributed to all managers. And we can use labels to tag nodes and customize the cluster how we want it. Anyway, once we've got the cluster, then we can start scheduling containers to it. So instead of running individual Docker container run commands against specific nodes, and every time having to think about which node we should be running them on, well instead of that, we just throw commands at the cluster, and we let Swarm decide. 

So Swarm does all of the workload balancing and the likes. Great well, we can run two types of work on the cluster. Native Swarm work, and Kubernetes. Though, at the time of recording, right, you can only run Kubernetes work on Docker Enterprise edition.
