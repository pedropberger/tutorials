In this blog post, I will introduce some basic Docker concepts for Data Science and show you how to use some basic commands to create and run Docker containers. Docker is a tool that allows you to package your code and its dependencies into isolated, lightweight, and portable units called containers. Containers can run on any machine that has Docker installed, regardless of the operating system or the hardware configuration. This makes it easier to deploy, share, and reproduce your data science projects.

Some of the benefits of using Docker for Data Science are:

- You can ensure that your code runs in the same environment across different machines, avoiding the "it works on my machine" problem.
- You can avoid installing and managing multiple versions of libraries and packages on your local machine, reducing clutter and potential conflicts.
- You can easily share your code and data with others by pushing and pulling containers from online repositories like Docker Hub.
- You can leverage existing containers that have pre-installed tools and frameworks for data science, such as TensorFlow, PyTorch, Jupyter Notebook, etc.

# Key concepts

First of all, it's important to know some fundamental docker concepts. Let's break down the key concepts in the context of data science:

Image:
A Docker image is a lightweight, standalone, and executable software package that includes everything needed to run a piece of software, including the code, runtime, system tools, libraries, and settings. In the context of data science, an image might contain a specific version of Python, required libraries (like NumPy, pandas, scikit-learn, etc.), and other dependencies necessary for your data analysis or machine learning tasks. Docker images are created from a Dockerfile, which is a script-like file that specifies the configuration and steps to build the image.

Container:
A Docker container is an instance of a Docker image. Containers are isolated environments that encapsulate the application and its dependencies. They are lightweight, share the host OS kernel, and can be started, stopped, and moved between different environments with consistent behavior. In data science, you might use containers to encapsulate your data preprocessing, analysis scripts, or machine learning models. Containers help ensure that your code behaves the same way regardless of the environment it runs in.

Volume:
Docker volumes are mechanisms for persistently storing and sharing data between a Docker host and its containers. In the context of data science, volumes are particularly useful for managing datasets and model checkpoints. By using volumes, you can ensure that your data remains accessible and consistent across different runs of your containers. For example, you could mount a directory containing your dataset as a volume inside a container, so changes made to the data are reflected on both the host system and the container.

Clusters:
Docker clusters, often referred to as container orchestration, involve managing multiple containers across a cluster of machines. While clusters are more commonly associated with deploying applications in production, they can also be relevant in data science workflows. For instance, if you're dealing with distributed computing or running experiments on multiple nodes, tools like Kubernetes can help manage and scale your containers efficiently.

![image](https://github.com/pedropberger/tutorials/assets/98188778/fc43b066-923c-4d0b-8290-107c6870e16c)


In a data science context, here's how these concepts might work together:

Development and Experimentation:
- You create a Docker image containing the specific Python version and libraries you need for your data analysis or machine learning project.
- You develop and test your scripts within Docker containers created from this image. This ensures consistent behavior across different development environments.

Data Preprocessing:
- You use Docker volumes to mount your dataset into the container, ensuring that changes to the dataset are reflected both inside and outside the container.

Training Models:
- You use Docker containers to encapsulate your model training code and dependencies.
- You might deploy multiple containers in a cluster to distribute training tasks if the workload is large.

Model Deployment:
- You create a Docker image that includes your trained model and a lightweight API.
- This image is deployed in a production environment, and Docker ensures that the runtime environment remains consistent, reducing the chances of deployment-related issues.

By utilizing Docker's concepts of images, containers, volumes, and, in some cases, clusters, data scientists can create reproducible, isolated, and scalable environments for their data analysis and machine learning projects.

# Let's pratice

To get started with Docker, you need to install it on your machine. You can find the installation instructions for different operating systems here: https://docs.docker.com/get-docker/

Once you have installed Docker, you can use the following basic commands to create and run containers (in Shell or Windows CMD):

- `docker pull`: This command allows you to download an existing image from a repository. An image is a template that contains the code and the dependencies for a container. For example, if you want to download an image that has Python 3.8 and Jupyter Notebook installed, you can run:

```bash
docker pull jupyter/scipy-notebook:python-3.8
```

![image](https://github.com/pedropberger/tutorials/assets/98188778/e1be28fc-05a7-4c81-9a53-5061448e01f2)

Just wait and a brand new container with Python + Jupyter will be waiting for you.

- `docker run`: This command allows you to create and start a container from an image. You can also specify various options to customize the behavior of the container, such as port mapping, volume mounting, environment variables, etc. For example, if you want to create and run a container from the image you downloaded in the previous step, and access the Jupyter Notebook from your browser, you can run:

```bash
docker run -p 8888:8888 -v /path/to/your/data:/home/app/jupyter/scipy-notebook:python-3.8
```

What we did here: This command will map the port 8888 of the container to the port 8888 of your host machine, mount the directory `/path/to/your/data` from your host machine to the directory `/home/app/jupyter/` inside the container, and start the Jupyter Notebook server. You can then open your browser and go to `http://localhost:8888` to access the notebook.

- `docker ps`: This command allows you to list all the running containers on your machine. You can see information such as the container ID, the image name, the status, the ports, etc. For example:

```bash
docker ps
```
![image](https://github.com/pedropberger/tutorials/assets/98188778/37e12b3c-33f1-4b2b-b412-66f578e472a8)

The CONTAINER ID column shows the ID of your container (obvious, I know), but this is the value you can use to take down or make changes to your container. The IMAGE column contains the name along with the tag (yes, you can and should tag it) so you can control where your container will be reborn from. The rest helps you find your way around the space. Try different versions of docker ps for different results, such as `docker ps -a`, which will also bring up information about containers that aren't running.

Whenever you don't give your container a name, docker will assign one automatically, you can use this name to play with the container just like the ID.

- `docker stop`: This command allows you to stop a running container by specifying its ID or name. For example, if you want to stop the container you created in the previous step, you can run:

```bash
docker stop 97e2adde5894
```

or

```bash
docker stop recursing_blackburn
```

- `docker rm`: This command allows you to remove a stopped container by specifying its ID or name. For example, if you want to remove the container you stopped in the previous step, you can run:

```bash
docker rm 97e2adde5894
```

or

```bash
docker rm recursing_blackburn
```

Try running the command `docker ps` and `docker ps -a` after removal, you'll see that it's gone for good. Yes, you've deleted your beloved container forever.

But reviving it is easy, since its image still exists. To see the list of images, just use the command

```bash
docker image ls
```

and you'll see all the images you have on your machine. 

If you want to delete these images, just use the command ```docker rmi```  followed the image name.

# That's all folks

These are some of the basic Docker commands that you can use to create and run containers for data science. There are many more commands and options that you can explore in the official documentation: https://docs.docker.com/

I hope this help you to understand some basic Docker concepts for data science and how to use some basic commands to create and run containers. If you have any questions or feedback, please leave a comment below. Thank you for reading!


