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

 So, from within the directory of our source code and our Dockerfile, right? We'll go, docker image build. Docker build on its own works as well, right? Well, we'll tag this one as psweb, and then the dot here says use the current directory as the context where the code is, yeah? And away that goes. Now then, to keep the momentum going, I'm going to bend a bit of space-time. Right. That's finished. Okay? And we'll get under the covers a bit on the next lesson. But the image is built, and it's tagged, so we're done. That means that if we do docker image ls, there it is. Psweb, and only just built. Tell you what, let's run a container from it. Well, that'll be docker container run. We'll run this one detached, right, so it doesn't steal our terminal. And you know what, it's just a web app, right? So it can tick away in the background. We'll call it, whatever, web1. It runs on port 8080, so I'm going to map that to 8080 on the host, as well. It's host container, like this. And we're going to run it from that psweb image that we just built. So, I guess moment of truth. Okay, we're good. That's a container ID, which means, if we open a web browser here. This is the server the container's running on, yeah? And it was 8080. And there we are! That's our app. So, from source code to running web server in, whatever that was. Probably less than a minute, if you take out my waffling. But I think pretty flipping cool, yeah? Now, if you're on Windows, or Raspberry Pi, or whatever's your thing, right, the commands in the process are the same. I mean, sure, you're not going to run a Win32 app on Raspberry Pi. But assuming you've got your app on the right architecture, the process is the same. Code, Dockerfile, docker image build. Simples, right? Have some of that. Well, that's cool and all, but let's take a look at what went on under the hood.
