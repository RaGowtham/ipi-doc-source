title: How to Install MRAA Library
---

Eclipse Mraa (Libmraa) is a C/C++ library with bindings to Java, Python and JavaScript to interface with the IO, with a structured and sensors API where port names/numbering matches the board that you are on. Use of libmraa does not tie you to specific hardware with board detection done at runtime you can create portable code that will work across the supported platforms.

The intent is to make it easier for developers and sensor manufacturers to map their sensors & actuators on top of supported hardware and to allow control of low level communication protocol by high level languages & constructs.



**Note:** By default, MRAA library is already on PX30 Yocto image & also mentioned in the section of Ubuntu instruction page to guide you how to install 

<br>

Our MRAA libraries has been ready on [**Eclipse MRAA GitHub**]( https://github.com/eclipse/mraa) whic is supported on **LEC-PX30 & LEC-AL-AI with industrial Pi-SMARC** as the following screen.

![image-20191215174758939](HowToInstallMRAA.assets/image-20191215174758939.png)







## How to Build & Install MRAA on your boards:

#### Building mraa on Ubuntu:

1. Build dependencies is required:

   ```
   $ sudo apt-get install git build-essential swig3.0 python-dev nodejs-dev cmake libjson-c-dev python3-dev
   ```
   
   **Note: **On Ubuntu Bionic (18.04) you'll need to downgrade node.js (see [nodesource](https://github.com/nodesource/distributions) for some handy         install scripts) or patch SWIG. 
   
2. Basic build Steps:

   ```
   $ git clone https://github.com/eclipse/mraa.git
   $ cd mraa
   $ make build
   $ cd build
   $ cmake ..
   ```

   **Note:** If this goes wrong and you have all the dependencies installed, then please file an issue with the full output of `cmake ..` and `make` or however far you got.

3. After that you can install built files by running:

   ```
   sudo make install
   ```

   

#### Building mraa on Yocto:

Intel mraa recipe is here http://git.yoctoproject.org/cgit/cgit.cgi/meta-intel-iot-middleware/tree/recipes-devtools?h=daisy please add it to your building Yocto.

