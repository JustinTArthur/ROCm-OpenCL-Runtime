# ROCm OpenCL™ Compatible Runtime 

Developer preview Version 2 of the new 

* OpenCL 1.2 compatible language runtime and compiler
* OpenCL 2.0 compatible kernel language support with OpenCL 1.2 compatible runtime
* Supports offline ahead of time compilation today; during the Beta phase we will add in-process/in-memory compilation.


## GETTING REPO

Repo is a git wrapper that manages a collection of git repositories. Install this tool and add it to the command search PATH:

    curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
    chmod a+x ~/bin/repo

## GETTING THE SOURCE CODE

Main OpenCL™ Compatible Components:

* https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime
* https://github.com/RadeonOpenCompute/ROCm-Device-Libs 
* https://github.com/RadeonOpenCompute/ROCm-OpenCL-Driver 
* https://github.com/RadeonOpenCompute/llvm 
* https://github.com/RadeonOpenCompute/clang
* https://github.com/RadeonOpenCompute/lld 
* https://github.com/KhronosGroup/OpenCL-ICD-Loader

Download the git projects with the following commands:

    ~/bin/repo init -u https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime.git -b amd-master -m opencl.xml
    ~/bin/repo sync
    
## INSTALL ROCm

Follow the instructions at https://rocm.github.io/install.html to install ROCm.

## BUILDING

Use out-of-source CMake build and create separate directory to run CMake.

The following build steps are performed:

    mkdir -p build && cd build
    cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..
    make
    
## RUNNING clinfo

    LLVM_BIN=./compiler/llvm/bin LD_LIBRARY_PATH=./lib clinfo

OpenCL™ is registered Trademark of Apple  
