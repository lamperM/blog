# Googletest 移植 Arm64 

## 交叉编译

Download source code
```sh
git clone https://github.com/google/googletest.git -b v1.14.0
```


编写适用于Arm64平台的规则，指定交叉编译器的位置。此文件放在项目的根目录下。
```cmake
set(CMAKE_CROSSCOMPILING TRUE)

set(CMAKE_FIND_ROOT_PATH ~/tools/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/)

# Cross compiler
SET(CMAKE_C_COMPILER   aarch64-none-linux-gnu-gcc)
SET(CMAKE_CXX_COMPILER aarch64-none-linux-gnu-g++)
set(CMAKE_LIBRARY_ARCHITECTURE aarch64-none-linux-gnu)

# Search for programs in the build host directories
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)

# Libraries and headers in the target directories
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

编译，这里没有编译Googlemock
```sh
#!/bin/bash

rm -rf build
mkdir build && cd build

cmake .. -DCMAKE_TOOLCHAIN_FILE=../toolchain_arm64.cmake -DGTEST_HAS_PTHREAD=0 -DBUILD_GMOCK=OFF

# check file: lib/libgtest_main.a and lib/libgtest.a
```