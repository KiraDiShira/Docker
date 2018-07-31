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

# Images in Detail

We said an image is a bunch of layers. We read the manifest file, we stack them up, then we throw in some magic to make it all look unified. 

To get an image we have got to pull it. The easiest way to do that is with docker image pull. 

Example, pulling redis:

<img src="https://github.com/KiraDiShira/Docker/blob/master/WorkingWithImages/Images/wwi1.PNG" />

Each of these pull completes here is a **layer**.

Now then, there's actually no such thing as an image, at least not as a single blob or object. You see, an image is actually a bunch of independent layers that are very loosely connected by a manifest file, which I know is at odds actually with the way always talk about them as a single blob. But they're not. 

<img src="https://github.com/KiraDiShira/Docker/blob/master/WorkingWithImages/Images/wwi2.PNG" />

The **manifest file**, which we sometimes call a config file, describes the image, like its ID and tags and when it was created, but importantly it also includes a list of the layers that get stacked and how to stack them.

Now these layers are unaware of the bigger image. Just a bunch a files that are what they are, an independent layer. They have no clue that they're going to be stacked up and made into something bigger. One layer in image has no references to other layers. All of that's done in the manifest.

And I think we get an idea of this actually with the pull we just did. We're not pulling the image, so to speak, as a single object with a single progress bar. It's obvious that we're pulling individual layers and it looks like six of them?

Behind the scenes, the pull command's generating an API request to the docker registry API on a registry somewhere (Docker hub). 

<img src="https://github.com/KiraDiShira/Docker/blob/master/WorkingWithImages/Images/wwi3.PNG" />

Pulling an image is actually a two step process. Get the manifest, pull the layers.

Now the manifest's a bit interesting, right? We've got this push towards **multi architecture support** and **content addressable storage**. One's about supporting a single image across multiple architectures, maybe X86 and ARM and PowerPC or whatever, and the other's about using cryptography to guarantee we get the images we asked for.

Anyway, step one's getting the manifest. Now, the client first looks for something called a **fat manifest**, and that's like a manifest of manifests, right? Basically it's a list of architectures supported and a manifest for each of those. So, if your architecture's on the list, it points you to the image manifest that you need.

So, I suppose, getting the manifest is a bit of a two step process these days. Get the fat manifest first, then get the image manifest for your architecture. 

Once we've got that, we pass it for the list of layers and we download them from the registry's blob store which is what we saw here with the pull. 

Now then, we mentioned content addressable storage. So back in the day, Docker didn't hash its images and there really wasn't an easy way to know if the image we got was the image we asked for. Well, that's obviously not good, especially when most of us are pulling our images across the internet. So, problem.

Well, solution, and thanks a ton to the community and OCI, but in Docker 1.10 we got content addressable storage, which at the high level says, "okay, every layer has got "a bunch of files and stuff inside. "So now I tell you what, "let's grab a hash of all that content "and use it for the image ID." That way we can say, "Okay, registry, "give me the image with this particular hash, please." We download it, run the hash again, and see if the two match.

<img src="https://github.com/KiraDiShira/Docker/blob/master/WorkingWithImages/Images/wwi4.PNG" />

And in one fell swoop we have got a way more secure storage model.

<img src="https://github.com/KiraDiShira/Docker/blob/master/WorkingWithImages/Images/wwi5.PNG" />

So, you're seeing the image manifest here that we're referencing stuff via a hash. These are the layer IDs. The syntax is just algorithm sha256 in this case colon and then the hash and hex.

Okay, we've got out layers, they're linked and stacked thanks to a manifest, and behind the scenes we've got some crypto goodness giving us immutability.

Now then, behind the scenes, there's a **storage driver** pulling together all this layering. For us it is the aufs driver, oh, and most of our images and container stuff is going to exist somewhere here in valid docker. On Windows, this is going to be c program dated docker and your storage drive is always going to be Windows filter, though, those could be famous last words. 

Here on Linux, right, there are a ton of storage options. Aufs is the oldest driver and you still see it everywhere.

Layering pretty much works like this: There's a base layer at the very bottom. I tend to think of this as like the OS layer. It's the one that's got all the files and OS that build a basic OS, like ubuntu or something. In fact, let's assume it is ubuntu, right?

Then, on top of the base layer, we stack more layers. Things like your app code and the like, right. Copy in some app files as another layer, I don't know, maybe add another layer of updates later, and the storage driver waves its magic wand and

<img src="https://github.com/KiraDiShira/Docker/blob/master/WorkingWithImages/Images/wwi6.PNG" />

aba-cadabra and the layers disappear and we're staring at what looks like a single unified file system, fabulous.

<img src="https://github.com/KiraDiShira/Docker/blob/master/WorkingWithImages/Images/wwi7.PNG" />

Each of these layers is represented in a directory in the file system under var/lib/docker and then your storage driver name, for us it's aufs and then diff. Windows, that's going to default to c program data docker Windows filter.

<img src="https://github.com/KiraDiShira/Docker/blob/master/WorkingWithImages/Images/wwi8.PNG" />

Okay, six directories, and funnily enough, six layers. Simple right? Mmm, maybe not so much. You see, since we got this fancy new content addressable storage model, there's no easy way to match the layer IDs with their respective directories on the host. And it's definitely annoying.

The base layer of an image has all of the OS files and directories. Then these other layers are going to have like the app code and the config files and stuff.

If we go `docker history redis`: history of the image and how its layers were built. 

<img src="https://github.com/KiraDiShira/Docker/blob/master/WorkingWithImages/Images/wwi9.PNG" />

Then any other line that's got a non zero value at the end here will have created a new layer. Anything that is zero, well, that probably added something to the image's config JSON.


So, a bunch of layers with files and stuff in them. Stacked on top of each other and unified by a storage driver according to instructions in a manifest. Then, we start containers from them. Multiple per image if we want.
