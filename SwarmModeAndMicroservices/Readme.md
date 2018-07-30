- [index](https://github.com/KiraDiShira/Docker/blob/master/README.md#docker)

# Swarm mode and microservices

## Swarm mode theory

A collection of Docker engines joined into a cluster is called a **swarm**. 

And then the Docker engines running on each node in the swarm are said to be running in **swarm mode**.

Swarm mode is optional. We don't have to, and, if we don't, Docker is just going to run like it always has in **standalone mode** or **single engine mode**.

Anyway, a swarm itself consists of one or more **manager nodes**, and then one or more **worker nodes**.

The manager nodes look after the state of the cluster and dispatch tasks to the worker nodes.

Mangers are highly available, meaning that if 1 or 2 or however many of them go away, the ones remaining will keep the swarm going. And it works like this behind the scenes. While you can have X number of managers, an odd number is highly recommended, and only one of them is ever the **leader** or **primary**. All managers maintain the state of the cluster, but if a manager that's not the leader receives a command, it's going to proxy that command over to the leader, and then the leader's going to action it against the swarm. And, you can spread managers across availability zones, regions, data centers, whatever suits your high availability needs, but of course, that's all going to be dependent on the kind of networks that you have, you're going to want reliable networks. Now, I suppose I should mention **Raft**, because while we can have multiple managers for redundancy and high availability, that naturally gives us complexities over agreeing on the current state consensus. Well, Raft is the protocol that's used behind the scenes to bring order to that chaos by ensuring we achieve a distributed consensus. 

Now, manager nodes are worker nodes as well, so it's totally possible to have a swarm where every node is a manager node, though, more than five manager nodes generally isn't thought of as a good idea. The premise here is that the more managers you have, the harder it is to get consensus, or the longer it takes to get consensus. Just the same way as 2 or 3 people deciding which restaurant to go to is way easier than, I don't know, 20 people.

Worker nodes on the other hand just accept tasks from managers and execute them.

Speaking of which, that leads us nicely to **services**. So, services are also a new concept introduced with swarm mode, meaning if you're not running in swarm mode, then you can't do services.

A service is a declarative way of running and scaling **tasks**. 

So, as an example, say you have an app that consists of a back-end store and a front-end web interface, you'd implement that as two services, one service for the back-end store and another for the web front end, only with services, we tell Docker what we want the app service to look like, and it's now up to Docker to make sure that happens. So let's say we wanted five instances of the container that was running the web front end, you'd tell Docker that when you define the service. This is an example command here.

```
docker service create --name web-fe replicas 5 ...
```

So it's telling Docker, go create a service called web-fe, and I want five instances of the container or task it's going to run. Docker is going to go away and spin up five tasks; think of tasks as containers for now, and it's going to spread them across all the work nodes in the cluster. 

Remember, managers are also workers, but here's the thing. It is going to make sure that there are always five of them running. If one of them dies, there's a reconciliation loop running in the background that'll say okay, I've got four tasks running for this service, wait up, I should have give, and it's going to start a new one. 

And that declarative model of expressing desired state and having Docker keep an eye on things, making sure that actual state always equals desired state, is both new and really powerful.

A **task** is the atomic unit of work assigned to a worker node. We as developers or sysadmins or whatever tell the manager about services, then the manager assigns the work of that service out to worker nodes as tasks. Now, right now tasks means containers. A little bit more so, they include metadata about how to initiate the container and some runtime stuff, but we can pretty much think of a task as being a container. 

We've got a swarm consisting of a bunch of manager nodes and worker nodes, we define services, declare them to the manager via the standard Docker API, albeit new endpoints in the API, the manager then splits the service into tasks and schedules those tasks against available nodes in the swarm. And then, in order to deploy complex apps consisting of multiple distributed independently scalable services, we've got stacks and distributed application bundles.

## Configuring Swarm Mode
