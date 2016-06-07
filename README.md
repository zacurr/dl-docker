## All-in-one Docker image for Deep Learning
Here are Dockerfiles to get you up and running with a fully functional deep learning machine. It contains all the popular deep learning frameworks with CPU and GPU support (CUDA and cuDNN included). The CPU version should work on Linux, Windows and OS X. The GPU version will, however, only work on Linux machines. This is because on Windows and OS X, Docker runs inside a Virtual Box VM which does not have access to the GPU on the host.

If you are not familiar with Docker, but would still like an all-in-one solution, start here: [What is Docker?](#what-is-docker)

## Specs
This is what you get out of the box when you create a container with the provided image/Dockerfile:
* Ubuntu 14.04
* [CUDA 7.5](https://developer.nvidia.com/cuda-toolkit) (GPU version only)
* [cuDNN v4](https://developer.nvidia.com/cudnn) (GPU version only)
* [Tensorflow](https://www.tensorflow.org/)
* [Caffe](http://caffe.berkeleyvision.org/)
* [Theano](http://deeplearning.net/software/theano/)
* [Keras](http://keras.io/)
* [Lasagne](http://lasagne.readthedocs.io/en/latest/)
* [Torch](http://torch.ch/) (includes nn, cutorch, cunn and cuDNN bindings)
* [iPython/Jupyter Notebook](http://jupyter.org/) (including iTorch Torch kernel)
* [Numpy](http://www.numpy.org/), [SciPy](https://www.scipy.org/), [Pandas](http://pandas.pydata.org/), [Scikit Learn](http://scikit-learn.org/), [Matplotlib](http://matplotlib.org/)
* A few common libraries used for deep learning

## Setup
### Prerequisites
1. Install Docker following the installation guide for your platform: [https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)
2. **GPU Version Only**: Install Nvidia drivers on your machine either from [Nvidia](http://www.nvidia.com/Download/index.aspx?lang=en-us) directly or follow the instructions [here](https://github.com/saiprashanths/dl-setup#nvidia-drivers). Note that you **don't** have to install CUDA and cuDNN on your host machine. These are included in the Docker container.
3. **GPU Version Only**: Install nvidia-docker. Follow the instructions here: [https://github.com/NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker). This will install a replacement for the docker client. It takes care of setting up the Nvidia host driver environment inside the Docker containers and a few other things.
4. Build the Docker image. Note that this will take an hour or two depending on your machine since it compiles a few libraries from scratch.

	git clone https://github.com/saiprashanths/dl-docker.git
	cd dl-docker
	
**CPU Version**
	docker build -t dl-docker:cpu github.com/saiprashanths/dl-docker.git -f Dockerfile.cpu .

**GPU Version**
	docker build -t dl-docker:gpu github.com/saiprashanths/dl-docker.git -f Dockerfile.gpu .
  
## Running the Docker image as a Container

This will spin up a Docker container for you with all the frameworks installed. 

**CPU Version**
	docker run -it -p 8888:8888 6006:6006 -v /sharedfolder:/root/sharedfolder dl-docker:cpu bash
		
**GPU Version**
	nvidia-docker run -it -p 8888:8888 6006:6006 dl-docker:gpu bash
Note the use of `nvidia-docker` rather than just `docker`

| Parameter      | Explanation |
|----------------|-------------|
|`-it`             | This creates an interactive terminal you can use to iteract with your container |
|`-p 8888:8888`    | This exposes the ports inside the container so they can be accessed from the host. The format is `-p <host-port>:<container-port>`. The default iPython Notebook runs on port 8888 and Tensorboard on 6006 |
|`-v /sharedfolder:/root/sharedfolder/` | This shares the folder `/sharedfolder` on your host machine to `/root/sharedfolder/` inside your container. Any data written to this folder by the container will be persistent. You can modify this to anything of the format `-v /local/shared/folder:/shared/folder/in/container/`. See [Docker container persistence](#Docker-container-persistence)
|`dl-docker:cpu`   | This the image that you want to run. The format is `image:tag`. In our case, we use the tag `gpu` or `cpu` to spin up the appropriate image |
|`bash`       | This provides the default command when the container is started. Even if this was not provided, bash is the default command and just starts a Bash session. You can modify this to be whatever you'd like to be executed when your container starts. For example, you can execute `docker run -it -p 8888:8888 6006:6006 dl-docker:cpu jupyter notebook`. This will execute a script that executes the command `jupyter notebook` and starts your Jupyter Notebook for you when the container starts

## Some common scenarios

### Notebook
The image comes pre-installed with iPython and iTorch Notebooks, and you can use this to work with the deep learning frameworks. If you spin up the docker container with docker-run -p <host-port>:<container-port> (as shown above in the [instructions](#running-the-Docker-image-as-a-Container)), you will have access to these ports on your host and can access them at http://127.0.0.1:<host-port>. The default iPython notebook uses port 8888 and Tensorboard uses port 6006. Since we expose both these ports when we run the container, we can access them both from the host.

However, you still need to start the Notebook inside the container to be able to access it from the host. You can either do this from the container terminal by executing `jupyter notebook` or you can pass this command in directly while spinning up your container using the `docker run -it -p 8888:8888 6006:6006 dl-docker:cpu jupyter notebook` CLI. The Jupyter Notebook has both Python (for TensorFlow, Caffe, Theano, Keras, Lasagne) and iTorch (for Torch) kernels.

### Data Sharing
See [Docker container persistence](#Docker-container-persistence). Consider this: You have a script that you've written on your host machine. You want to run this in the container and get the output data (say, a trained model) back into your host. The way to do this is using a [Shared Volumne](Docker container persistence). By passing in the `-v /sharedfolder/:/root/sharedfolder` to the CLI, we are sharing the folder between the host and the container, with persistence. You could copy your script into `/sharedfolder` folder on the host, execute your script from inside the container (located at `/root/sharedfolder`) and write the results data back to the same folder. This data will be accessible even after you kill the container.

## What is Docker?
[Docker](https://www.docker.com/what-docker) itself has a great answer to this question.

Docker is based on the idea that one can package code along with its dependencies into a self-contained unit. In this case, we start with a base Ubuntu 14.04 image, a bare minimum OS. When we build our initial Docker image using _docker build_, we install all the deep learning frameworks and its dependencies on the base, as defined by the _Dockerfile_. This gives us an image which has all the packages we need installed in it. We can now spin up as many instances of this image as we like, using the _docker run_ command. Each instance is called a _container_. Each of these containers can be thought of as a fully functional and isolated OS with all the deep learning libraries installed in it. 

## Why do I need a Docker?
Installing all the deep learning frameworks correctly is an exercise in dependency hell. Unfortunately, given the current state of DL development and research, it is almost impossible to rely on just one framework. This Docker is intended to provide a solution for this use case.

If you would rather install all the frameworks yourself manually, take a look at this guide: [Setting up a deep learning machine from scratch](https://github.com/saiprashanths/dl-setup)

### Do I really need an all-in-one container?
No. The provided all-in-one solution is useful if you have dependencies on multiple frameworks (say, load a pre-trained Caffe model, finetune it, convert it to Tensorflow and continue developing there) or if you just want to play around with the various frameworks.

The Docker philosophy is to build a container for each logical task/framework. If we followed this, we should have one container for each of the deep learning frameworks. This minimizes clashes between frameworks and is easier to maintain as things evolve. In fact, if you only intend to use one of the frameworks, or at least only one framework at a time, follow this approach. You can find Dockerfiles for individual frameworks here:
* [Tensorflow Docker](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/tools/docker)
* [Caffe Docker](https://github.com/BVLC/caffe/tree/master/docker)
* [Theano Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-theano)
* [Keras Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-keras/cuda_v7.5)
* [Lasagne Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-lasagne/cuda_v7.5)
* [Torch Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-torch-plus)

## FAQs

### Performance
Running the DL frameworks as Docker containers should have no performance impact during runtime. Spinning up a Docker container itself is very fast and should take only a couple of seconds or less

### Docker container persistence
Keep in mind that the changes made inside Docker container are not persistent. Lets say you spun up a Docker container, added and deleted a few files and then kill the container. The next time you spin up a container using the same image, all your previous changes will be lost and you will be presented with a fresh instance of the image. This is great, because if you mess up your container, you can always kill it and start afresh again. It's bad if you don't know/understand this and forget to save your work before killing the container. There are a couple of ways to work around this:

1. **Commit**: If you make changes to the image itself (say, install a few new libraries), you can commit the changes and settings into a new image. Note that this will create a new image, which will take a few GBs space on your disk. In your next session, you can create a container from this new image. For details on commit, see [Docker's documentaion](https://docs.docker.com/engine/reference/commandline/commit/).

2. **Shared volume**: If you don't make changes to the image itself, but only create data (say, train a new Caffe model), then commiting the image each time is an overkill. In this case, it is easier to persist the data changes to a folder on your host OS using shared volumes. Simple put, the way this works is you share a folder from your host into the container. Any changes made to the contents of this folder from inside the container will persist, even after the container is killed. For more details, see Docker's docs on [Managing data in containers](https://docs.docker.com/engine/reference/commandline/commit/)
 
### What operating systems are supported?
Docker is supported on all the OSes mentioned here: [Install Docker Engine](https://docs.docker.com/engine/installation/). The CPU version (Dockerfile.cpu) will run on all the above operating systems. However, the GPU version will only run on Linux OS. This is because Docker runs inside a virtual machine on Windows and OS X. Virtual machines don't have direct access to the GPU on the host. Unless PCI passthrough is implemented for these hosts, GPU support isn't available on non-Linux OSes.
