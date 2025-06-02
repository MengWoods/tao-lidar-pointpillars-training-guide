# tao-lidar-pointpillars-training-guide (WIP)

The repository provides a step-by-step guide for training a PointPillars 3D object detector using the TAO Toolkit with the [KITTI dataset](https://www.cvlibs.net/datasets/kitti/).

To clone the repository:
```bash
# Clone the repository to $HOME path
cd ~
git clone git@github.com:MengWoods/tao-lidar-pointpillars-training-guide.git
# Clone the tao-toolkit submodule
cd tao-lidar-pointpillars-training-guide
git submodule update --init --recursive
```

**[PointPillars](https://arxiv.org/abs/1812.05784)** is a popular deep learning architecture designed for efficient 3D object detection from LiDAR point cloud data. By encoding sparse point clouds into dense pseudo-images, it enables fast and accurate detection of objects like vehicles and pedestrians, making it well-suited for autonomous driving tasks.

The **[TAO Toolkit](https://developer.nvidia.com/tao-toolkit)** (Train, Adapt, Optimize) is NVIDIA’s AI training framework that simplifies the process of building models. It provides pre-trained models, transfer learning workflows, and optimization tools, allowing developers to accelerate model development without needing deep AI expertise.

The original training guide is missing some important details, and parts of the information are outdated or incorrect. I created this repository based on my experience, along with a customized training Python notebook, to help others accelerate the training process and avoid the mistakes I encountered.

The repository covers the following key aspects:
- KITTI dataset preprocessing for the TAO training pipeline
- Neural network training, evaluation, and inference
- Deployment using ONNX and TensorRT model engines


## 1. Requirements

Recommended system configuration:

### 1.1 Hardware

- \>= 32 GB system RAM
- \>= 32 GB GPU RAM
- \>= 8-core CPU
- \>= 1 NVIDIA GPU
- \>= 100 GB SSD storage (TAO Toolkit container ~26 GB)

### 1.2 Software

- Ubuntu LTS: 20.04 or later
- Python: >3.7, <=3.10
- Python pip: >= 21.06
- [Miniconda](https://docs.anaconda.com/miniconda/install/#quick-command-line-install)
- [docker-ce](https://docs.docker.com/engine/install/ubuntu/): >19.03.5
- Docker API: 1.40
- [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html): >1.3.0-1
- NVIDIA container runtime: 3.4.0-1
- nvidia-docker2: 2.5.0-1
- NVIDIA driver: >= 535.xx
- [NVIDIA GPU Cloud CLI](https://org.ngc.nvidia.com/setup/installers/cli)
- NVIDIA CUDA (12.0+)
- TensorRT (8.6.3.1+)

You can also run the training session on an AWS EC2 instance, such as `g4dn.xlarge`. Note that multi-GPU configurations may not work as expected.

## 2. TAO Toolkit Installation

Install TAO launcher PyPI:

```bash
# Create virtual python environment for TAO in the repo root
python3 -m venv venv_tao
# Activate virtual environment
source venv_tao/bin/activate
# Install TAO launcher
pip3 install --upgrade nvidia-tao
# Check the version
tao info --verbose
# Export the variable to the used python version
export VIRTUALENVWRAPPER_PYTHON=~/tao-lidar-pointpillars-training-guide/venv_tao/bin/python3.x
```

Register account and sign in the Nvidia [NGC](https://catalog.ngc.nvidia.com/) page.
Top-right corner click user name, Setup, Generate API Key, Generate Personal Key, Select NGC Catalog in the Key Permissions (double check), Generate Personal Key, copy your API Key and save it secretly.

Login the docker registry:

```bash
docker login nvcr.io
```

## 2. Data Preprocessing

### 2.1 Download KITTI 3D Object Data

Download KITTI 3d object detection annotation data from the [website](https://www.cvlibs.net/datasets/kitti/eval_object.php?obj_benchmark=3d)
Download the options:
- Left color images of object data set (12GB)
- Velodyne point clouds (29B)
- Camera calibration matrices of object data set (16 MB)
- Training labels of object data set (5 MB)

Extract the download data into below structure:
```
│── ImageSets
│── training
│   ├──calib & velodyne & label_2 & image_2
│── testing
│   ├──calib & velodyne & image_2
```
