# <div align="left"><img src="img/rapids_logo.png" width="90px"/>&nbsp;cuDF - GPU DataFrames</div>

[![Build Status](http://18.191.94.64/buildStatus/icon?job=cudf-master)](http://18.191.94.64/job/cudf-master/)&nbsp;&nbsp;[![Documentation Status](https://readthedocs.org/projects/cudf/badge/?version=latest)](https://cudf.readthedocs.io/en/latest/)

The [RAPIDS](https://rapids.ai) cuDF library is a GPU DataFrame manipulation library based on Apache Arrow that accelerates loading, filtering, and manipulation of data for model training data preparation. The RAPIDS GPU DataFrame provides a pandas-like API that will be familiar to data scientists, so they can now build GPU-accelerated workflows more easily.

## Quick Start

Please see the [Demo Docker Repository](https://hub.docker.com/r/rapidsai/rapidsai/), choosing a tag based on the NVIDIA CUDA version you’re running. This provides a ready to run Docker container with example notebooks and data, showcasing how you can utilize cuDF.

## Install cuDF

### Conda

You can get a minimal conda installation with [Miniconda](https://conda.io/miniconda.html) or get the full installation with [Anaconda](https://www.anaconda.com/download).

You can install and update cuDF using the conda command:

```bash
conda install -c nvidia -c rapidsai -c numba -c conda-forge -c defaults cudf=0.3.0
```

Note: This conda installation only applies to Linux and Python versions 3.5/3.6.

You can create and activate a development environment using the conda commands:

```bash
# create the conda environment (assuming in base `cudf` directory)
$ conda env create --name cudf_dev --file conda/environments/dev_py35.yml
# activate the environment
$ source activate cudf_dev
# when not using default arrow version 0.10.0, run
$ conda install -c nvidia -c rapidsai -c numba -c conda-forge -c defaults pyarrow=$ARROW_VERSION
```

This installs the required `cmake`, `nvstrings`, `pyarrow` and other
dependencies into the `cudf_dev` conda environment and activates it.

### Pip

Support is coming soon, please use conda for the time being.

## Development Setup

The following instructions are tested on Linux Ubuntu 16.04 & 18.04, to enable
from source builds and development. Other operatings systems may be compatible,
but are not currently supported.

### Get libcudf Dependencies

Compiler requirements:

* `gcc`     version 5.4
* `nvcc`    version 9.2
* `cmake`   version 3.12

CUDA/GPU requirements:

* CUDA 9.2+
* NVIDIA driver 396.44+
* Pascal architecture or better

You can obtain CUDA from [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)

Since `cmake` will download and build Apache Arrow (version 0.7.1 or
0.8+) you may need to install Boost C++ (version 1.58+) before running
`cmake`:

```bash
# Install Boost C++ for Ubuntu 16.04/18.04
$ sudo apt-get install libboost-all-dev
```

or

```bash
# Install Boost C++ for Conda
$ conda install -c conda-forge boost
```

### Build from Source

To install cuDF from source, ensure the dependencies are met and follow the steps below:

1. Clone the repository
```bash
git clone --recurse-submodules https://github.com/rapidsai/cudf.git
cd cudf
```
2. Create the conda development environment `cudf` as detailed above
3. Build and install `libcudf`
```bash
$ cd /path/to/cudf/cpp                              # navigate to C/C++ CUDA source root directory
$ mkdir build                                       # make a build directory
$ cd build                                          # enter the build directory
$ cmake .. -DCMAKE_INSTALL_PREFIX=/install/path     # configure cmake ... use $CONDA_PREFIX if you're using Anaconda
$ make -j                                           # compile the libraries librmm.so, libcudf.so ... '-j' will start a parallel job using the number of physical cores available on your system
$ make install                                      # install the libraries librmm.so, libcudf.so to '/install/path'
```
To run tests (Optional):

```bash
$ make test
```

Build and install cffi bindings:
```bash
$ make python_cffi                                  # build CFFI bindings for librmm.so, libcudf.so
$ make install_python                               # install python bindings into site-packages
$ cd python && py.test -v                           # optional, run python tests on low-level python bindings
```

4. Build the `cudf` python package, in the `python` folder:
```bash
$ cd ../../python
$ python setup.py build_ext --inplace
```

To run Python tests (Optional):
```bash
$ py.test -v                                        # run python tests on cudf python bindings
```
5. Finally, install the Python package to your Python path:

```bash
$ python setup.py install                           # install cudf python bindings
```

## Automated Build in Docker Container

A Dockerfile is provided with a preconfigured conda environment for building and installing cuDF from source based off of the master branch.

### Prerequisites

* Install [nvidia-docker2](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)) for Docker + GPU support
* Verify NVIDIA driver is `396.44` or higher
* Ensure CUDA 9.2+ is installed

### Usage

From cudf project root run the following, to build with defaults:
```bash
$ docker build --tag cudf .
```
After the container is built run the container:
```bash
$ docker run --runtime=nvidia -it cudf bash
```
Activate the conda environment `cudf` to use the newly built cuDF and libcudf libraries:
```
root@3f689ba9c842:/# source activate cudf
(cudf) root@3f689ba9c842:/# python -c "import cudf"
(cudf) root@3f689ba9c842:/#
```

### Customizing the Build

Several build arguments are available to customize the build process of the
container. These are spcified by using the Docker [build-arg](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg)
flag. Below is a list of the available arguments and their purpose:

| Build Argument | Default Value | Other Value(s) | Purpose |
| --- | --- | --- | --- |
| `CUDA_VERSION` | 9.2 | 10.0 | set CUDA version |
| `LINUX_VERSION` | ubuntu16.04 | ubuntu18.04 | set Ubuntu version |
| `CC` & `CXX` | 5 | 7 | set gcc/g++ version; **NOTE:** gcc7 requires Ubuntu 18.04 |
| `CUDF_REPO` | This repo | Forks of cuDF | set git URL to use for `git clone` |
| `CUDF_BRANCH` | master | Any branch name | set git branch to checkout of `CUDF_REPO` |
| `NUMBA_VERSION` | 0.40.0 | Not supported | set numba version |
| `NUMPY_VERSION` | 1.14.3 | Not supported | set numpy version |
| `PANDAS_VERSION` | 0.20.3 | Not supported | set pandas version |
| `PYARROW_VERSION` | 0.10.0 | 0.8.0+ | set pyarrow version |
| `PYTHON_VERSION` | 3.5 | 3.6 | set python version |

---

## <div align="left"><img src="img/rapids_logo.png" width="265px"/></div> Open GPU Data Science

The RAPIDS suite of open source software libraries aim to enable execution of end-to-end data science and analytics pipelines entirely on GPUs. It relies on NVIDIA® CUDA® primitives for low-level compute optimization, but exposing that GPU parallelism and high-bandwidth memory speed through user-friendly Python interfaces.

<p align="center"><img src="img/rapids_arrow.png" width="80%"/></p>

### Apache Arrow on GPU

The GPU version of [Apache Arrow](https://arrow.apache.org/) is a common API that enables efficient interchange of tabular data between processes running on the GPU. End-to-end computation on the GPU avoids unnecessary copying and converting of data off the GPU, reducing compute time and cost for high-performance analytics common in artificial intelligence workloads. As the name implies, cuDF uses the Apache Arrow columnar data format on the GPU. Currently, a subset of the features in Apache Arrow are supported.
