# simple-vtk-app

## intellisense for vtk
https://learn.microsoft.com/en-us/shows/one-dev-minute/configure-c-intellisense-in-visual-studio-code

c/c++ configurations (JSON)

add this include path:
    "/home/xiao/vtk/**"


## Run VTK with nvidia container toolkit

```shell
xhost +local:

# to match the share lib path

sudo docker run --pid=host --rm --runtime=nvidia -it --gpus all -v $HOME:/home/xiao -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY -e XAUTHORITY -e NVIDIA_DRIVER_CAPABILITIES=all nvidia/opengl:1.2-glvnd-runtime-ubuntu22.04 /bin/bash

# inside the docker 
apt-get update
sudo apt-get install mesa-utils libglu1-mesa-dev freeglut3-dev mesa-common-dev

```

## Run OpenGL XServer with nvidia container toolkit
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -

curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list


sudo apt-get update \
    && sudo apt-get install -y nvidia-container-toolkit-base

nvidia-ctk --version

sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

grep "  name:" /etc/cdi/nvidia.yaml 

  <!-- name: "0"
  name: all
 -->


/usr/share/containers/oci/hooks.d/oci-nvidia-hook.json

sudo nvidia-ctk runtime configure --runtime=docker

sudo systemctl restart docker

sudo docker run --rm --runtime=nvidia --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi

xhost +local:

sudo docker run --pid=host --rm --runtime=nvidia -it --gpus all -v $HOME/cuda-samples/Samples:/Samples -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY -e XAUTHORITY -e NVIDIA_DRIVER_CAPABILITIES=all nvidia/cuda:12.2.0-base-ubuntu22.04 /bin/bash

apt-get update
## install cuda samples locally

cuda_12.2.1_535.86.10_linux.run

sh cuda_12.2.1_535.86.10_linux.run --silent --samples --samplespath=~/cudaprojects

Key: NVIDIA-settings -> NVIDIA(performance mode)

===========
= Summary =
===========


To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-12.2/bin
***WARNING: Incomplete installation! This installation did not install the CUDA Driver. A driver of version at least 535.00 is required for CUDA 12.2 functionality to work.
To install the driver using this installer, run the following command, replacing <CudaInstaller> with the name of this run file:
    sudo <CudaInstaller>.run --silent --driver

Logfile is /var/log/cuda-installer.log
apt install \
build-essential \
cmake \
cmake-curses-gui \
mesa-common-dev \
mesa-utils \
freeglut3-dev \
ninja-build


### delete conflict repository references  list
grep -l "nvidia.github.io" /etc/apt/sources.list.d/* | grep -vE "/nvidia-container-toolkit.list\$"
sudo rm /etc/apt/sources.list.d/nvidia-docker.list


## NVIDIA Driver Containers

```shell
cat /etc/nvidia-container-runtime/config.toml

disable-require = falsesudo docker run --gpus all nvidia/cuda:11.0-base nvidia-smi
#swarm-resource = "DOCKER_RESOURCE_GPU"
#accept-nvidia-visible-devices-envvar-when-unprivileged = true
#accept-nvidia-visible-devices-as-volume-mounts = false

[nvidia-container-cli]
#root = "/run/nvidia/driver"
#path = "/usr/bin/nvidia-container-cli"
environment = []
#debug = "/var/log/nvidia-container-toolkit.log"
#ldcache = "/etc/ld.so.cache"
load-kmods = true
#no-cgroups = false
#user = "root:video"
ldconfig = "@/sbin/ldconfig.real"

[nvidia-container-runtime]
#debug = "/var/log/nvidia-container-runtime.log"
log-level = "info"

# Specify the runtimes to consider. This list is processed in order and the PATH
# searched for matching executables unless the entry is an absolute path.
runtimes = [
    "docker-runc",
    "runc",
]

mode = "auto"

    [nvidia-container-runtime.modes.csv]

    mount-spec-path = "/etc/nvidia-container-runtime/host-files-for-container.d"


sudo sed -i 's/^#root/root/' /etc/nvidia-container-runtime/config.toml

sudo docker run --name nvidia-driver -d --privileged --pid=host \
  -v /run/nvidia:/run/nvidia:shared \
  -v /var/log:/var/log \
  --restart=unless-stopped \
  nvidia/driver:450.80.02-ubuntu18.04

sudo docker run --gpus all nvidia/cuda:11.0-base nvidia-smi

```

## Build

```shell
mkdir -p build

cd build

# use custom build vtk, not sdk



cmake -DVTK_DIR:PATH=/home/xiao/vtk/build ..

lscpu | egrep 'Model name|Socket|Thread|NUMA|CPU\(s\)'

make -j 16

./SimpleVtkApp /home/xiao/vtk/source/ThirdParty/vtkm/vtkvtkm/vtk-m/data/data/uniform/venn250.vtk
```