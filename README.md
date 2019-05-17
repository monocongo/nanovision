# nanovision
Configuration of a NVIDIA Jetson Nano with deep learning based computer vision monitoring software.

#### Install required system packages
```
$ sudo apt-get update
$ sudo apt-get install git cmake
$ sudo apt-get install libatlas-base-dev gfortran
$ sudo apt-get install libhdf5-serial-dev hdf5-tools
$ sudo apt-get install python3-dev
```

#### Configure Python development environment
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

Add packages into the virtual environment we'll need, beginning with [NumPy](https://www.numpy.org/):
```
$ pip install numpy
```
Add the official Jetson Nano TensorFlow package:

```
$ pip install --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v42 tensorflow-gpu==1.13.1+nv19.3
```

Install [SciPy](https://www.scipy.org/) and [Keras](https://keras.io/):
```
$ pip install scipy
$ pip install keras
```

#### Build the Jetson Inference engine
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

#### Build OpenCV
###### (Option #1) Build from source
```
$ cd ~/git
$ git clone https://github.com/opencv/opencv.git
$ git clone https://github.com/opencv/opencv_contrib.git
$ cd opencv
$ mkdir build
$ cd build
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_CUDA=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUBLAS=1 \
-D WITH_GSTREAMER=ON \
-D WITH_LIBV4L=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules \
-D BUILD_EXAMPLES=OFF ..
$ make -j4
$ sudo make install
$ sudo ldconfig
```

###### (Option #2) Use build script from mdegans 
```
$ cd ~/git 
$ git clone https://github.com/mdegans/nano_build_opencv.git
$ cd nano_build_opencv
$ ./build_opencv.sh 
```
