#docker build -t pycuvslam:v02 -f dockerfiles/Dockerfiles .
#docker tag pycuvslam:v02 registry.gitlab.com/yuvalb29/pycuvslam:v02
#docker push registry.gitlab.com/yuvalb29/pycuvslam:v02
FROM nvcr.io/nvidia/cuda:12.8.0-runtime-ubuntu22.04

# Install system dependencies
RUN apt-get update \
 && DEBIAN_FRONTEND="noninteractive" apt-get install -y --no-install-recommends --allow-change-held-packages \
 build-essential \
 cmake \
 git \
 libgtk-3-dev \
 libpython3.10 \
 libxkbcommon-x11-0 \
 python3-pip \
 python3-dev \
 unzip \
 vulkan-tools \
 wget \
 pkg-config \
 libusb-1.0-0-dev \
 libssl-dev \
 libglfw3-dev \
 libgl1-mesa-dev \
 libglu1-mesa-dev \
 libatlas-base-dev \
 libsuitesparse-dev \
 libeigen3-dev \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Upgrade pip
RUN pip3 install --upgrade pip

# Install matplotlib and opencv-python early (before other dependencies)
RUN pip3 install --no-cache-dir \
 matplotlib \
 opencv-python \
 opencv-contrib-python

# Clone and build librealsense following the README instructions
RUN git clone https://github.com/IntelRealSense/librealsense.git /opt/librealsense \
 && cd /opt/librealsense \
 && mkdir build && cd build \
 && cmake ../ \
      -DCMAKE_BUILD_TYPE=Release \
      -DFORCE_RSUSB_BACKEND=ON \
      -DBUILD_EXAMPLES=true \
      -DBUILD_PYTHON_BINDINGS=true \
      -DPYTHON_EXECUTABLE=$(which python3) \
 && make -j$(nproc) \
 && make install

# Copy udev rules for RealSense cameras
RUN mkdir -p /etc/udev/rules.d/ && cp /opt/librealsense/config/99-realsense-libusb.rules /etc/udev/rules.d/

# Set Python path for pyrealsense2 (following README instructions)
ENV PYTHONPATH=/opt/librealsense/build/Release

# Install Git LFS
RUN apt-get update && apt-get install -y git-lfs

# Initialize Git LFS
RUN git lfs install

# Clone the pycuvslam repository
RUN git clone https://github.com/NVlabs/PyCuVSLAM.git /pycuvslam

# Set working directory for pycuvslam
WORKDIR /pycuvslam

# Install Python dependencies for pycuvslam
COPY examples/requirements.txt /tmp/requirements.txt
RUN pip3 install -r /tmp/requirements.txt

# Install additional GUI dependencies for rerun-sdk and OpenCV
RUN apt-get update \
 && DEBIAN_FRONTEND="noninteractive" apt-get install -y --no-install-recommends \
 libx11-6 \
 libxext6 \
 libxrender1 \
 libxtst6 \
 libxi6 \
 libgl1-mesa-glx \
 libglib2.0-0 \
 libsm6 \
 libxrandr2 \
 libxss1 \
 libxcb1 \
 libxcomposite1 \
 libxcursor1 \
 libxdamage1 \
 libxfixes3 \
 libxinerama1 \
 libxrandr2 \
 libxrender1 \
 libxtst6 \
 libnss3 \
 libcups2 \
 libdbus-1-3 \
 libatk1.0-0 \
 libgtk-3-0 \
 libgdk-pixbuf2.0-0 \
 libpangocairo-1.0-0 \
 libcairo2 \
 libpango-1.0-0 \
 libatspi2.0-0 \
 libdrm2 \
 libgbm1 \
 libasound2 \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Test that pyrealsense2 can be imported
RUN python3 -c "import pyrealsense2 as rs; print('pyrealsense2 imported successfully')"

# Test that rerun-sdk can be imported
RUN python3 -c "import rerun as rr; print('rerun-sdk imported successfully')"

# Test that matplotlib and cv2 can be imported
RUN python3 -c "import matplotlib; print('matplotlib imported successfully')"
RUN python3 -c "import cv2; print('opencv-python (cv2) imported successfully'); print(f'OpenCV version: {cv2.__version__}')"

# Install pycuvslam package in editable mode if bin/x86_64 exists
RUN if [ -d './bin/x86_64' ]; then \
      echo 'Installing pycuvslam package in editable mode for x86 during build...'; \
      pip3 install -e ./bin/x86_64; \
      echo 'pycuvslam package installed successfully'; \
    else \
      echo 'Warning: /pycuvslam/bin/x86_64 not found. Skipping pycuvslam installation during build.'; \
    fi

# Set a simple, non-interactive CMD (PyCharm will override this)
CMD ["/bin/bash"]

# Startup commands are now handled directly in run_docker_x86.sh