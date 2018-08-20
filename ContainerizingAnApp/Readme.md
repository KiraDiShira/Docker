- [index](https://github.com/KiraDiShira/Docker/blob/master/README.md#docker)

# Containerizing an App

## The Big Picture

We've got our code. So running it means running it in a container. But how do we do that?

<img src="https://github.com/KiraDiShira/Docker/blob/master/ContainerizingAnApp/Images/caa1.PNG" />

Our app code. To get it into an image, we create a **Dockerfile** (this is a list of instructions on how to build an image with our code inside). We use the docker `image build` command to actually build the image. And out pops an image!
