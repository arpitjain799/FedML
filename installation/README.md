# Installing FedML

FedML supports Linux, MacOS, Windows, and Android.

## FedML Source Code Repository
[https://github.com/FedML-AI/FedML](https://github.com/FedML-AI/FedML)


## Installing with pip

```
pip install fedml
```

The default machine learning engine is `PyTorch`. FedML also supports `TensorFlow`, `Jax`, and `MXNet`.
You can install related engine as follows:
```
pip install "fedml[MPI]"
pip install "fedml[tensorflow]"
pip install "fedml[jax]"
pip install "fedml[mxnet]"
```
For MPI installation, it's used for local distributed simulation with MPI (https://mpi4py.readthedocs.io/en/stable/). On MacOS, the installation commands in conda environment is:
```
conda install mpi4py openmpi
```
About OpenMPI library installation for MPI, the reference is as follows: [https://docs.open-mpi.org/en/v5.0.x/installing-open-mpi/quickstart.html](https://docs.open-mpi.org/en/v5.0.x/installing-open-mpi/quickstart.html,)
For OpenMPI on MacOS, please review the following links:
[https://betterprogramming.pub/integrating-open-mpi-with-clion-on-apple-m1-76b7815c27f2](https://formulae.brew.sh/formula/open-mpi)
[https://formulae.brew.sh/formula/open-mpi](https://formulae.brew.sh/formula/open-mpi)

The above commands work properly in Linux environment.
For Windows/Mac OS (Intel)/Mac OS (M1), you may need to follow TensorFlow/Jax/MXNet official guidance to fix related installation issues.

## Installing FedML with Anaconda

```
conda create --name fedml-pip python=3.8
conda activate fedml-pip
conda install --name fedml-pip pip
pip install fedml
```
(Note: please use python 3.8 if you met any compatability issues. Currently, we support 3.7, 3.8, 3.9, 3.10, 3.11.
on MacOS and Python 3.11, if you meet any issues, please run the command:
brew install autoconf automake libffi libtool pkg-config,
then run the command to install fedml again: pip install fedml)

After installation, please use "pip list | grep fedml" to check whether `fedml` is installed.


## Installing FedML from Debugging and Editable Mode
```
cd python
pip install -e ./
```

## Installing FedML from Source
```
git clone https://github.com/FedML-AI/FedML.git && \
cd ./FedML/python && \
python setup.py install
```

If you want to run examples with tensforflow, jax or mxnet, you need to install optional dependencies:
```
git clone https://github.com/FedML-AI/FedML.git && \
cd ./FedML/python && \
python install -e '.[tensorflow]'
python install -e '.[jax]'
python install -e '.[mxnet]'
```
(Notes: Tensorflow example located in tf_mqtt_s3_fedavg_mnist_lr_example directory, Jax example location in jax_haiku_mqtt_s3_fedavg_mnist_lr_example directory)

If you need to install from a specific commit (normally used for the debugging/development phase), please follow commands below:
```
git clone https://github.com/FedML-AI/FedML.git && \
cd ./FedML && git checkout e798061d62560b03e049d514e7cc8f1a753fde6b && \
cd python && \
python setup.py install
```
Please change the above commit id to your own (you can find it at [https://github.com/FedML-AI/FedML/commits/master](https://github.com/FedML-AI/FedML/commits/master))


## Running FedML in Docker (Recommended)
FedML Docker Hub: [https://hub.docker.com/repository/docker/fedml/fedml](https://hub.docker.com/repository/docker/fedml/fedml)

We recommend using FedML in the Docker environment as it circumvents complex and tedious installation debugging. Currently, we maintain docker images for two settings:

- For Linux servers with x86_64 architecture

Please refer to the following commands and remember to change `WORKSPACE` to your own.

**(1) Pull the Docker image and prepare the docker environment**
```
FEDML_DOCKER_IMAGE=fedml/fedml:latest-torch1.13.1-cuda11.6-cudnn8-devel
docker pull $FEDML_DOCKER_IMAGE

# if you want to use GPUs in your host OS, please follow this link: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker
sudo apt-get update
sudo apt-get install -y nvidia-docker2
sudo systemctl restart docker
sudo chmod 777 /var/run/docker.sock
```

**(2) Run Docker with interactive mode**

***On GPUs:***
```
FEDML_DOCKER_IMAGE=fedml/fedml:latest-torch1.13.1-cuda11.6-cudnn8-devel
WORKSPACE=/home/fedml/fedml_source

docker run -t -i -v $WORKSPACE:$WORKSPACE --shm-size=64g --ulimit nofile=65535 --ulimit memlock=-1 --privileged \
--gpus all \
--network=host \
--env WORKSPACE=$WORKSPACE \
$FEDML_DOCKER_IMAGE \
/bin/bash
```

***On CPUs:***
```
FEDML_DOCKER_IMAGE=fedml/fedml:latest-torch1.13.1-cuda11.6-cudnn8-devel
WORKSPACE=/home/fedml/fedml_source

docker run -t -i -v $WORKSPACE:$WORKSPACE --shm-size=64g --ulimit nofile=65535 --ulimit memlock=-1 --privileged \
--network=host \
--env WORKSPACE=$WORKSPACE \
$FEDML_DOCKER_IMAGE \
/bin/bash
```

You should now see a prompt that looks something like:
```
fedml@ChaoyangHe-GPU-RTX2080Tix4:/$ 
fedml@ChaoyangHe-GPU-RTX2080Tix4:/$ cd $WORKSPACE
fedml@ChaoyangHe-GPU-RTX2080Tix4:/home/fedml/fedml_source$
```
If you want to debug in Docker container, please follow these commands
```
cd python
# You need sudo permission to install your debugging package in editable mode 
(-e means link the package to the original location, basically meaning any changes to the original package would reflect directly in your environment)
sudo pip install -e ./
```

**(3) Run the interpreter in PyCharm or Visual Studio using Docker environment**

- PyCharm

[https://www.jetbrains.com/help/pycharm/using-docker-as-a-remote-interpreter.html#summary](https://www.jetbrains.com/help/pycharm/using-docker-as-a-remote-interpreter.html#summary)

- Visual Studio

[https://code.visualstudio.com/docs/remote/containers](https://www.jetbrains.com/help/pycharm/using-docker-as-a-remote-interpreter.html#summary)

(4) Other useful commands
```
# docker rm $(docker ps -aq)
docker container kill $(docker ps -q)
```


## Running FedML on Kubernetes

This tutorial will guide you to deploy your fedml client and server to target Kubernetes pods running on GPU/CPU physical nodes.

The entire workflow is as follows:
1. In the file ./install_on_k8s/fedml-edge-client-server/deployment-client.yml, modify the variable ACCOUNT_ID to your desired value
2. Deploy the fedml client:  ```kubectl apply -f ./install_on_k8s/fedml-edge-client-server/deployment-client.yml```
3. In the file ./install_on_k8s/fedml-edge-client-server/deployment-server.yml, modify the variable ACCOUNT_ID to your desired value
4. Deploy the fedml server:  ```kubectl apply -f ./install_on_k8s/fedml-edge-client-server/deployment-server.yml```
5. Login the FedML MLOps platform (https://open.fedml.ai), the above deployed client and server will be found in the edge devices

If you want to scale up or scal down the pods to your desired count, you may run the following command:

```kubectl scale -n $YourNameSpace --replicas=$YourDesiredPodsCount deployment/fedml-client-deployment```

```kubectl scale -n $YourNameSpace --replicas=$YourDesiredPodsCount deployment/fedml-server-deployment```

# Q&A

1. Q: How to scale up or scale down?  
   A: Use the following commands:

```kubectl scale -n $YourNameSpace --replicas=$YourDesiredPodsCount deployment/fedml-client-deployment```

```kubectl scale -n $YourNameSpace --replicas=$YourDesiredPodsCount deployment/fedml-server-deployment```

2. Q: FedML Client send online status to FedML Server via which protocol?  
   A: Via MQTT


3. Q: FedML Client send model, gradient parameters to FedML Server via which protocol?  
   A: Use S3 protocol to store and exchange models and use MQTT to exchange messages between FedML Client and Server


4. Q: Why do we need AWS S3?  
   A: Use S3 protocol to store and exchange models.

## Guidance for Windows Users

Please follow instructions at [Windows Installation](./install/windows.md)

## Guidance for Raspberry Pi Users

Please follow instructions at [Raspberry Pi Installation](./install/rpi.md)

## Guidance for NVIDIA Jetson Devices

Please follow instructions at [NVIDIA Jetson Device Installation](./install/jetson.md)

## Testing if the installation succeeded
If the installation is successful, you will not see any issue when run `import fedml`.
```shell
(mnn37) chaoyanghe@Chaoyangs-MBP FedML-refactor % python
Python 3.7.7 (default, Mar 26 2020, 10:32:53) 
[Clang 4.0.1 (tags/RELEASE_401/final)] :: Anaconda, Inc. on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import fedml
>>> 

```

## Installing FedML Android SDK/APP
Please follow the instructions at `https://github.com/FedML-AI/FedML/java/README.md`

## Troubleshooting
If you met any issues during installation or have additional installation requirements, please post issues at [https://github.com/FedML-AI/FedML/issues](https://github.com/FedML-AI/FedML/issues)