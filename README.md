# buildpack-opencv

OpenCV 3 with extra modules for Heroku Rails apps

```
# 1. Install Numpy via Conda
heroku buildpacks:add --index 2 https://github.com/kennethreitz/conda-buildpack
echo "numpy=1.11.2" > "conda-requirements.txt"
git push heroku master

# 2. Start a one-off dyno with a shell prompt
heroku run bash

# 3. Download and install CMake
wget https://cmake.org/files/v3.7/cmake-3.7.1-Linux-x86_64.tar.gz
tar zxf cmake-3.7.1-Linux-x86_64.tar.gz && rm cmake-3.7.1-Linux-x86_64.tar.gz
mkdir -p .heroku/cmake
mv cmake-3.7.1-Linux-x86_64/bin .heroku/cmake
mv cmake-3.7.1-Linux-x86_64/share .heroku/cmake

# 4. Download and install OpenCV 3 with extra modules (CMake for some reason couldn't find the PYTHON2_INCLUDE_DIR and PYTHON2_LIBRARY dirs)
wget https://github.com/opencv/opencv/archive/3.2.0.zip
unzip 3.2.0.zip && rm 3.2.0.zip
wget https://github.com/opencv/opencv_contrib/archive/3.2.0.zip
unzip 3.2.0.zip && rm 3.2.0.zip
cd opencv-3.2.0
../.heroku/cmake/bin/cmake -DPYTHON2_INCLUDE_DIR=/app/.heroku/miniconda/include/python2.7 -DPYTHON2_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython2.7.so -DCMAKE_BUILD_TYPE=Release -DOPENCV_EXTRA_MODULES_PATH=/app/opencv_contrib-3.2.0/modules -DCMAKE_INSTALL_PREFIX=/app/.heroku/opencv .
make && make install

# 5. Compress binaries and manually upload it to your S3 bucket via transfer.sh (don't forget to make the file public)
cd ..
tar zcf opencv-with-contrib.env.tgz .heroku/opencv .heroku/cmake
curl --upload-file opencv-with-contrib.env.tgz https://transfer.sh/opencv-with-contrib.env.tgz

# 6. Add buildpack, the URL to the binaries and deploy
heroku buildpacks:add --index 2 https://github.com/razola/buildpack-opencv
heroku config:set OPEN_CV_S3_URL=http://s3.amazonaws.com/something1234/opencv-with-contrib.env.tgz
git push heroku master
```

Note that before "import cv2" you'll need to do:

```
import sys
sys.path.append("/app/.heroku/opencv/lib/python2.7/site-packages")
import cv2
```