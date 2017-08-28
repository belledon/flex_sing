BootStrap: docker
From: nvidia/cuda:8.0-cudnn7-devel-ubuntu16.04

%environment
    echo "Adding NVIDIA PATHs to environment..."
    export LC_ALL=C
    export NV_DRIVER_PATH=/usr/local/NVIDIA-Linux-x86_64
    export LD_LIBRARY_PATH=$NV_DRIVER_PATH:$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64
    export PATH=$PATH:$NV_DRIVER_PATH
    export CUDA_PATH=/usr/local/cuda-8.0
    export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH
    export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}



%post
    echo "Hello from inside the container"
    # Set up some required environment defaults
    export LC_ALL=C
    export PATH=/bin:/sbin:/usr/bin:/usr/sbin:$PATH

    
    echo "Apt-getting packages"
    apt-get -y install build-essential
    apt-get update

    apt-get -y install locales
    locale-gen en_US.UTF-8

    apt-get -y install  git \
                        cmake \
                        g++ \
                        vim \
                        wget \
                        gdb \
                        libpng-dev \
                        freeglut3-dev \
                        # mesa-utils \

    echo "Manually installing glew packages"

    cd /

    wget http://mirrors.kernel.org/ubuntu/pool/main/g/glew/libglew1.10_1.10.0-3_amd64.deb
    dpkg -i libglew1.10_1.10.0-3_amd64.deb
    rm libglew1.10_1.10.0-3_amd64.deb

    wget http://mirrors.kernel.org/ubuntu/pool/main/g/glew/libglew-dev_1.10.0-3_amd64.deb
    dpkg -i libglew-dev_1.10.0-3_amd64.deb
    rm libglew-dev_1.10.0-3_amd64.deb


    apt-get clean

    cd /
    # wget http://us.download.nvidia.com/XFree86/Linux-x86_64/375.39/NVIDIA-Linux-x86_64-375.39.run
    # echo "Installing Driver (Needed for nccl)"

    NV_DRIVER_VERSION=375.20      # <---- EDIT: CHANGE THIS FOR YOUR SYSTEM
    NV_DRIVER_PATH=/usr/local/NVIDIA-Linux-x86_64

    if [ -d "$NV_DRIVER_PATH"]; then

        echo "NVIDIA driver already present"

    else

        NV_DRIVER_FILE=NVIDIA-Linux-x86_64-${NV_DRIVER_VERSION}.run

        working_dir=$(pwd)
        # download and run NIH NVIDIA driver installer
        wget http://us.download.nvidia.com/XFree86/Linux-x86_64/${NV_DRIVER_VERSION}/NVIDIA-Linux-x86_64-${NV_DRIVER_VERSION}.run

        echo "Unpacking NVIDIA driver into container..."
        cd /usr/local/
        sh ${working_dir}/${NV_DRIVER_FILE} -x
        rm ${working_dir}/${NV_DRIVER_FILE}    
        mv NVIDIA-Linux-x86_64-${NV_DRIVER_VERSION} NVIDIA-Linux-x86_64
        cd NVIDIA-Linux-x86_64/

        echo "Creating NVIDIA links"
        for n in *.$NV_DRIVER_VERSION; do
            ln -v -s $n ${n%.375.20}   # <---- EDIT: CHANGE THIS IF DRIVER VERSION
        done
        ln -v -s libnvidia-ml.so.$NV_DRIVER_VERSION libnvidia-ml.so.1
        ln -v -s libcuda.so.$NV_DRIVER_VERSION libcuda.so.1
        cd $working_dir
        
    fi
    
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64:$NV_DRIVER_PATH
    export PATH=$PATH:$NV_DRIVER_PATH
    

    if [ -d "/yaml-cpp"]; then

        echo "Found yaml-cpp"
    else

        echo "installing yaml-cpp"
        cd /
        git clone https://github.com/jbeder/yaml-cpp.git
        cd yaml-cpp && mkdir build && cd build
        cmake ..
        make
        make install
        make clean
    fi

    apt-get install -y --no-install-recommends python3-dev
    apt-get install -y --no-install-recommends python3-pip

    python3 -m pip install --upgrade pip
    python3 -m pip install -U setuptools
    python3 -m pip install h5py
    python3 -m pip install transforms3d
    python3 -m pip install Pillow
   
    

    mkdir /om || echo "/om exists"
    mkdir /cm || echo "/cm exists"  

    echo "Updating library paths"
    cd /etc/ld.so.conf.d
    echo "/usr/local/NVIDIA-Linux-x86_64" | cat > nvidia-driver.conf
    cd /
    ldconfig

%test
    echo $LD_LIBRARY_PATH
    nvidia-smi    
