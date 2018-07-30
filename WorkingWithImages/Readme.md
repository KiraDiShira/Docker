- [index](https://github.com/KiraDiShira/Docker/blob/master/README.md#docker)

# Working with images

## Images: The Big Picture

An **image** is a ready-only template for creating application containers. 

Inside of it is all the code and supporting files to run an application.

Now, images are build-time constructs and containers are their run-time siblings. So think of a container as maybe a running image or, I suppose, vice versa as an image as stop container. 

All an image really is a bunch of files and a manifest. The files obviously include your app files, but they're also going to be all the OS and the library files and stuff that your app needs to run and ideally, just the ones it needs to run. Then the manifest is a JSON file explaining how it all fits together. 

An image is a bunch of **layers** that are stacked on top of each other. But it looks and feels like a unified file system. So, the stacking's important and we'll get to all the luscious details soon but for now, there's magic going on to hide all this layering and make it feel like a single flat image. 

Anyway, we take this and we start containers from it. So umpteen containers all sharing a single image. An it's insane for efficiency and container start time. Now, we store images in a registry, which can be cloud or on prem. We pull them down to our hosts using the docker image pull command. Once on our hosts, we can start containers from them.

The image is **read-only**, so, ah, how do the containers write stuff?

I mean, read-only and writing, I don't know about you but from my experience, they're usually not a great combo. Well, for each container, we create a thin writable layer and effectively lash it on top of the ready-only image. Then, any writes and updates the container needs to make get preformed here.

Now, because the writable layer pretty much is the container then it is a one to many relationship with the image here. One writable layer per container.

