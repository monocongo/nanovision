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
Next we'll source the `~/.bashrc` file to put the above into play:
```
$ source ~/.bashrc
```

###### Python virtula environment
Now we'll create and activate a Python virtual environment:
```
$ mkvirtualenv nanoviz -p python3
$ workon nanoviz
```
We'll add packages into the virtual environment we'll need, beginning with [numpy]():
```
$ pip install numpy
```
Add the official Jetson Nano TensorFlow package:

```
$ pip install --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v42 tensorflow-gpu==1.13.1+nv19.3
```

Install SciPy and Keras:
```
$ pip install scipy
$ pip install keras
```

#### Build the Jetson Inference engine
###### Clone down the jetson-inference git repo:
```
$ mkdir ~/git 
$ cd git
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
```
$ cd ~/git 
$ git clone https://github.com/mdegans/nano_build_opencv.git
$ cd nano_build_opencv
$ ./build_opencv.sh 
```
