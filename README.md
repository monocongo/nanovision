# nanovision
Configuration of a NVIDIA Jetson Nano with deep learning based computer vision monitoring software.

### Install required system packages
```
$ sudo apt-get update
$ sudo apt-get install git cmake
$ sudo apt-get install libatlas-base-dev gfortran
$ sudo apt-get install libhdf5-serial-dev hdf5-tools
$ sudo apt-get install python3-dev
$ sudo apt-get install libssl-dev
$ sudo apt-get install libffi-dev
```

### Add swap space
The Jetson Nano comes with 4GB of RAM. To supplement this for cases where this is insufficient we'll add swap space to allow for moving pages of memory out of RAM, as described [here](https://linuxize.com/post/how-to-add-swap-space-on-ubuntu-18-04/).
```
$ sudo fallocate -l 6G /swapfile
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile
```

To make the swap permanent we'll add the following to the end of the `/etc/fstab` file:

```
/swapfile swap swap defaults 0 0
```

### Configure Python development environment
###### Install pip, virtualenv, and virtualenvwrapper
```
$ wget https://bootstrap.pypa.io/get-pip.py
$ sudo python3 get-pip.py
$ rm get-pip.py
$ sudo pip install virtualenv virtualenvwrapper
```
###### Update .bashrc

To facilitate using Python virtual environments we'll add the below to the end of our existing `~/.bashrc` file:
```
# virtualenv and virtualenvwrapper
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source /usr/local/bin/virtualenvwrapper.sh
```

Source the `~/.bashrc` file to put the above environment variables and virtual environment settings into effect:
```
$ source ~/.bashrc
```

###### Python virtual environment

Create and activate a Python virtual environment:
```
$ mkvirtualenv nano -p python3
$ workon nano
```

To avoid errors/warnings about the current user not having correct permissions we change the owner of the `.cache` directory. For example the errors we're avoiding:

```
  WARNING: Building wheel for numpy failed: [Errno 13] Permission denied: '/home/james/.cache/pip/wheels/8c'
```

Use `sudo` to recursively change the owner of the directory to the current user:

```
$ sudo chown -R james ~/.cache
```

Install packages into the virtual environment we'll need, beginning with [NumPy](https://www.numpy.org/):
```
$ pip install numpy
```
Install the official [Jetson Nano TensorFlow](https://docs.nvidia.com/deeplearning/frameworks/install-tf-xavier/index.html) package:

```
$ pip install --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v42 tensorflow-gpu==1.13.1+nv19.3
```

Install [SciPy](https://www.scipy.org/) and [Keras](https://keras.io/):
```
$ pip install scipy
$ pip install keras
```

### Build the Jetson Inference engine
###### Clone down the jetson-inference git repo:
```
$ mkdir ~/git 
$ cd ~/git
$ git clone https://github.com/dusty-nv/jetson-inference
$ cd jetson-inference
$ git submodule update --init
```

###### Configure the build using cmake
```
$ mkdir build
$ cd build
$ cmake ..
```
###### Compile and install the Jetson Inference engine
```
$ make
$ sudo make install
```

### Build OpenCV
###### Build from source
Use the script for OpenCV (v4.0.0) build/installation provided by NVIDIA.
```
$ mkdir ~/opencv_install
$ cd ~/opencv_install
$ git clone https://github.com/AastaNV/JEP.git
$ cd JEP/script
$ chmod +x install_opencv4.0.0_Nano.sh
$ sudo ./install_opencv4.0.0_Nano.sh ~/opencv_install
```
###### Add Python bindings
Add the OpenCV Python bindings into the virtual environment by creating a link to the OpenCV Python module within the virtual environment's `site-packages` directory. For example if the OpenCV module was built and installed as `/usr/local/python/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so` and the virtual environment is named `nano` then we'll create the link like so:

```
$ ln -s /usr/local/python/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so  ~/.virtualenvs/cvtest/lib/python3.6/site-packages/cv2.so
```
Rather than doing this for the individual virtual environments we can create a link to the OpenCV Python bindings within `/usr/lib/python3.6` which will be included in the `PYTHONPATH` of all virtual environments:
```
$ sudo ln -s /usr/local/python/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so /usr/lib/python3.6/cv2.cpython-36m-aarch64-linux-gnu.so
```

At this point we should see the OpenCV bindings with the correct version available to the Python virtual environment:
```
$ workon nano
(nano) $ python
Python 3.6.7 (default, Oct 22 2018, 11:32:17) 
[GCC 8.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import cv2
>>> cv2.__version__
'4.0.0'
```
### deep_monitor
In order to perform object detection and facial recognition we'll install and run [**deep_monitor**](https://github.com/monocongo/deep_monitor) which is a Python application that utilizes deep learning models, OpenCV, and a PostGRES database.

The **deep_monitor** application utilizes a PyTorch implmentation of the RetinaNet object detection model provided by NVIDIA, and as such it is required to run within a Docker container. In order to leverage the NVIDIA GPU with CUDA it's further required to run the Docker container via the NVIDIA Docker wrapper, nvidia-docker.
Docker should be pre-installed on the Jetson Nano. To verify this we'll find it's installed location:
```
$ which docker
/usr/bin/docker
```
If no location is reported by the command above then we need to install the Docker package following the instructions [here].