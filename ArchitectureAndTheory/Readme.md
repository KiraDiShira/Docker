# Architecture and theory

## Architecture Big Picture

All a container really is, is this ring fenced area of an operating system, with some limits on how much system resources it can use, and that's it. 

<img src="https://github.com/KiraDiShira/Docker/blob/master/ArchitectureAndTheory/Images/aat1.PNG" />

Now, to build them, we'll leverage a bunch of low-level kernel stuff, in particular we use namespaces and control groups.It's actually the Docker engine, that makes all of this really easy, and that's our two main topics. The kernel building blocks, and the Docker engine.

## Kernel Internals

**Namespaces** are about isolation, and **Control Groups** are about grouping objects and setting limits, so Namespaces first.

These letters take an operating system, and carve it into multiple, isolated, virtual operating systems. It's a bit like hypervisors in virtual machines.

In the container world, we use **Namespaces** to take a single operating system with all of its resources, which tend to be high-level constructs like fire systems and process trees and users, and we carve all of that up into multiple virtual operating systems, called containers. 

Well, each container gets its own virtual or containerized root file system, it's own processed tree, it's own zero interface, it's own root user, the Full Monty. 

In the container world, each container looks, smells, and feels exactly like a regular OS, only it's not.

Now we've got these Namespaces, and Docker container is basically an organized collection of them.

<img src="https://github.com/KiraDiShira/Docker/blob/master/ArchitectureAndTheory/Images/aat2.PNG" />

To realistically have containers, you know, something that you'd run in production, we need something to polish the consumption of system resources. In the Linux world, this is **control groups**, and if you call, you called it C-groups. In Windows, it's Job Objects. But, credit to the Microsoft folks. They seem to be playing nice and generally calling them Control Groups as well. 

The idea is to group processes, and then impose limits.

Control Groups that are, say, okay, Container A over here is only going to get this amount of CPU, this amount of memory, and this amount of disk I/O, and then, Container B, this amount, and these as well.

With these two technologies, Namespace and Control Groups, we have got a realistic shot at workable containers in a union file system, or some way of combining a bunch of read-only file systems or blocked devices, lashing away to the layer on top, and presenting to the system as a unified view. 

<img src="https://github.com/KiraDiShira/Docker/blob/master/ArchitectureAndTheory/Images/aat3.PNG" />

## The Docker Engine

It exposes an API, it interfaces with all the kernel magic, and out pop containers. 

<img src="https://github.com/KiraDiShira/Docker/blob/master/ArchitectureAndTheory/Images/aat4.PNG" />

<img src="https://github.com/KiraDiShira/Docker/blob/master/ArchitectureAndTheory/Images/aat5.PNG" />
