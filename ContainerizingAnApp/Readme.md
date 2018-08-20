- [index](https://github.com/KiraDiShira/Docker/blob/master/README.md#docker)

# Containerizing an App

## The Big Picture

We've got our code. So running it means running it in a container. But how do we do that?

<img src="https://github.com/KiraDiShira/Docker/blob/master/ContainerizingAnApp/Images/caa1.PNG" />

Our app code. To get it into an image, we create a **Dockerfile** (this is a list of instructions on how to build an image with our code inside). We use the docker `image build` command to actually build the image. And out pops an image!

## Containerizing an App

It's normal to put your Dockerfile in the root folder of your app.

Now, all a Dockerfile is, is a list of instructions on how to build an image with an app inside. This is going to document the app. So, describe it to the rest of the team, and to new starts, and also to the ops guys.

The example I'm using here is a Linux app, right? So we need some form of a Linux base image. I'm a fan of Alpine these days, so I'm going to start it from that.

It's convention that **instructions** in a Dockerfile are capitalized. In fact, all this file really is is a list of key value pairs. Almost. You always start a Dockerfile with a `FROM` instruction. This is going to be the base image that we add our app on top of.

```
FROM alpine
```

Next up, I'm going to list myself as the maintainer to show you how to use labels. 

```
LABEL mainteiner="alarosa@hotmail.com"
```

The app that we're containerizing happens to be a node app, so we need to install `node` and `npm`, the node package manager. 

```
RUN apk add --update nodejs nodejs-npm
```

The from instruction up here pulls the alpine image and uses that as the base layer for the image that we're building. The label that I'm using here, all that does is adds a bit of metadata to the image config. But then `RUN` here creates a new layer. We're adding some software to the image, so we get a new layer.

Next, we want to copy in our source code, so everything from the same directory as our Dockerfile here, and we'll copy that into `/src` image. 

```
COPY . /src
```

We're adding more content, so we get another layer. 

Well, we're set the working directory to `/src`.

```
WORKDIR /src
```

We won't need a layer for that, it's just metadata.

Next up, we want to install our dependencies. 

```
RUN npm install
```

It's the RUN instruction again, and like before up here, it is running commands to install and build stuff. It's adding content. So we get another layer. 

This particular app listens on port 8080, so we'll expose that.

```
EXPOSE 8080
```

Just metadata again. 

And then last but not least we need to run an app. 

```
ENTRYPOINT ["node","./app.js"]
```

And this is relative to WORKDIR up here, okay? Anyway, these here don't really create layers. Just metadata in the config. 

<img src="https://github.com/KiraDiShira/Docker/blob/master/ContainerizingAnApp/Images/caa2.PNG" />

We'll go, docker image build. 

```
docker image build -t psweb .
```

So we're done. That means that if we do `docker image ls`, there it is.

Let's run a container from it. 

```
docker container run -d --name web1 -p 8080:8080 psweb
```

## Digging Deeper
So, we just containerized our web server. Pretty sweet, yeah? We had our code, threw in a Dockerfile, mixed in a bit of docker image build magic, and out popped an image. Well, let's do away with some of the mystery. First up, there's absolutely nothing magical about a Dockerfile. It's just a text file with a bunch of instructions. All right, it's got a few quirks like you need to name it with a capital D, and it's Dockerfile, all one word. But inside, it's just a bunch of really simple text instructions. Then, docker image build comes along, reads it, in order, one line at a time, starting from the top. Proper simple. But as well as that, it's going to be read by the ops team, by your fellow developers, by new hires. Pretty much everyone is going to look to this to understand your app. Anyway, each line in the file is an instruction. And you don't have to, but it is convention to write them as uppercase. Just the instruction name, though. The contents of the instruction you can totally write lowercase. Anyway, every Dockerfile starts with a FROM instruction. This is what starts the ball rolling. It pulls down the base image that everything else will build on top of. And following our best practices, if we can use an official image and a small one, you know what, we probably should. Well, we've gone Alpine, and because we've not put a tag on, we'll get the latest. Now, if you're doing this in production, right, you really want to get specific with tags. But you know what? The FROM instruction will go away and pull whatever image you specify from Docker Hub. Cool. Well, that is layer one. Next, we're adding a label to set the maintainer. And like I said before, honestly, I am not going to be maintaining this. I mean, it'll always be around for you if you're following around and the likes, but I have zero intentions of patching it and keeping it up to date. I'm really just showing you this so you can see how to use labels which are great for adding custom metadata. RUN here, and here, are actually how we execute commands and stuff inside the image. We're running some package installs here, and installing dependencies here. So, another couple of layers there, right? Well, COPY here is how we pull files and the likes into the image, and you probably guessed it. More content means more layers. And then we've got WORKDIR expose an entry point for adding metadata. Beautiful. But there's still a few gaps, I think. First up, I think I mentioned the build context at least a couple of times. Well, all it is is the location of your source code. If all is right, it's been the working directory. So, I've got my shell sitting in a folder called psweb here, and that has got all of my source code in it. That way, when I run the docker image build, I can just say dot at the end to use the current directory. If my shell was somewhere else in the file system, that's still cool, right? I just have to spell out exactly where the build context was. So, yeah. Build context, that's where your code is. And you can nest stuff, as well, right? Because it gets read recursively. So subfolders are fair game. Which you need to be careful about, actually. Because when you do the build, everything in your build context get sent to the Daemon. And if you've got a ton of stuff in subfiles that you don't need, you're going to waste resources, especially if your Daemon's across the network somewhere. So, yeah. Only have the code that you need in your build context. And feel free to throw your Dockerfile in there, as well. We did. But when you type docker image build, whatever's in there gets sent to the Daemon and processes part of the build. Now, for us, the client and Daemon are on the same host, but it is totally possible to have clients talking to remote Daemons over the network. Which brings me to probably our last thing, actually. Your build context can absolutely be a remote Git repo. In fact, let's do that. So, we've got nothing on this host right now. No images, and then, obviously, no containers. And then over here, this is the URL of the server that I am on. And we can see, yeah. There's definitely nothing listening on 8080 here. So, we've got a clean slate. Well, if we run another docker image build, and again, you can just go docker build if you want. Old habits die hard for some, I know that. We'll tag it the same again, but this time, instead of using dot to use the current working directory as the build context, I'm going to point it to a Git repo. This one here. And away that goes. Exactly the same as before, right? Only this time, the Daemon is pulling the build context across the wire from GitHub. But the process is just the same. And you know what? Sorry, I lied. That's not the last thing. Let's take a look at this output. So, sending context to Docker daemon, well, we just talked about that. Then the first instruction, one of eight, actually, FROM Alpine, pulls the latest Alpine image. And that there is its ID. In fact, yeah. Computer? (computer beeps) Stack each image layer in temporary containers as I'm going through the explanation, please? (Computer Voice) Acknowledge Ok, so we got the base image at the bottom. Next up, we have the maintainer label. To do this, Docker spins up a new container, and it churns out a layer. Once the layer's created, it gets rid of that intermediate container. Poof, gone. Next up, we're installing node and npm. I think we see this as doing a bunch of stuff, fetching and installing and all that goodness. But again, it is doing it inside a temporary container up here. Anyway. It spits out a layer here, and it loses the container. After that, we're copying in the app. Hmm. Huh. Yeah. Looks like maybe we don't need a container for that. Huh. Okay, actually, I'm not sure about that. But look, we get the layer. Then we're setting the working directory. Again, spits out a layer, shoots the container. Then we run npm install. That runs inside a container. Does all of this. And, as usual, spits out a layer and shoots the container. And the same for exposing ENTRYPOINT. Spin up that intermediate container, produce the layers, and shoot the containers. That's the process. Now, then. Only the layers that contain actual data are kept. So, even though it looks like these entry points and expose instructions create layers, they really don't. At least, not ones that we keep. So if we do this ... Okay. First up, we can see that only four of these layers actually contain any data. Magic. But see these IDs over here? See how they match what we built in the picture here, commands and all? More magic. But really, we only need these ones. The others are just metadata. So if we inspect the image, like this, okay. Only four real layers. And no, the IDs don't match, annoyingly. Go back to the previous module if you want to know why. But, yeah, you know what? That is the greasy, oily detail of building an image. Next up, the exciting world of multi-stage builds. This stuff is the future.
