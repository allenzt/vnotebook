# iotedge docker 编译命令

## 1. 克隆工程
git clone --recurse-submodules 'https://github.com/Azure/iotedge' azure_iotedge.git
cd azure_iotedge.git && git reset --hard 1.0.9.3

## 2. 使用docker命令编译

### 2.1 运行docker

```sh
docker run --rm --user root -e USER=root --name debian8.11-gnueabi-iotedged -v /home/dengzt/repo/azure_iotedge.git:/project -it azureiotedge/gcc-linaro-7.3.1-2018.05-x86_64_arm-linux-gnueabihf:debian_8.11-1 /bin/bash
```
### 2.2 安装交叉编译工具链

```sh
dpkg --add-architecture armel
apt update
apt install wget -y
wget https://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/arm-linux-gnueabi/gcc-linaro-7.3.1-2018.05-x86_64_arm-linux-gnueabi.tar.xz
rm -rf /toolchain
tar -Jxv -f gcc-linaro-7.3.1-2018.05-x86_64_arm-linux-gnueabi.tar.xz
mv gcc-linaro-7.3.1-2018.05-x86_64_arm-linux-gnueabi toolchain
rm -rf gcc-linaro-7.3.1-2018.05-x86_64_arm-linux-gnueabi.tar.xz
```

### 2.2  安装所需软件

```sh
apt-get download openssl:armel libssl1.0.0:armel libssl-dev:armel curl:armel libcurl4-openssl-dev:armel
dpkg -X openssl_1.0.1t-1+deb8u12_armel.deb download-install
dpkg -X libssl1.0.0_1.0.1t-1+deb8u12_armel.deb download-install
dpkg -X libssl-dev_1.0.1t-1+deb8u12_armel.deb download-install
dpkg -X curl_7.38.0-4+deb8u16_armel.deb download-install
dpkg -X libcurl4-openssl-dev_7.38.0-4+deb8u16_armel.deb download-install
cp -fpR /download-install/etc/ssl/* /download-install/usr/ && rm -rf /download-install/etc
cp /download-install/usr/include/arm-linux-gnueabi/openssl/opensslconf.h /download-install/usr/include/openssl/ && rm -rf /download-install/usr/include/arm-linux-gnueabi
rm -rf /download-install/usr/lib/arm-linux-gnueabi/libcurl.so
rm -rf /download-install/usr/lib/ssl
mv /download-install/usr/lib/arm-linux-gnueabi/* /download-install/usr/lib/ && rm -rf /download-install/usr/lib/arm-linux-gnueabi/
cp -fpR /download-install/* /toolchain/arm-linux-gnueabi/libc/ && rm -rf /download-install

#cmake
wget https://codeload.github.com/Kitware/CMake/tar.gz/v3.11.4 -O CMake-3.11.4.tar.gz
tar -zxv -f CMake-3.11.4.tar.gz
cd CMake-3.11.4
./bootstrap
make
make install
```

### 2.3 安装rust

```sh
#指定rust交叉编译工具链
mkdir -p ~/.cargo
echo '[target.armv7-unknown-linux-gnueabi]' > ~/.cargo/config
echo 'linker = "arm-linux-gnueabi-gcc"' >> ~/.cargo/config

#配置rust国内源
echo "export RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup" >> ~/.bashrc
source ~/.bashrc

#下载安装rust
curl -sSLf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env
```

### 2.4 设置环境变量

```sh
export CARGO_TARGET_ARMV7_UNKNOWN_LINUX_GNUEABI_LINKER=arm-linux-gnueabi-gcc
export CARGO_TARGET_ARMV7_UNKNOWN_LINUX_GNUEABI_RUNNER=qemu-arm
export CC_armv7_unknown_linux_gnueabi=arm-linux-gnueabi-gcc
export CXX_armv7_unknown_linux_gnueabi=arm-linux-gnueabi-g++
export OpenSSLDir=/toolchain/arm-linux-gnueabi/libc/usr
export OPENSSL_DIR=/toolchain/arm-linux-gnueabi/libc/usr
export OPENSSL_LIB_DIR=/toolchain/arm-linux-gnueabi/libc/usr/lib
export OPENSSL_INCLUDE_DIR=/toolchain/arm-linux-gnueabi/libc/usr/include
export PATH=/root/.cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/toolchain/bin:/toolchain/arm-linux-gnueabi/bin
export SYSROOT=/toolchain/arm-linux-gnueabi/libc
```
### 2.5 编译hsm

```sh
mkdir -p /project/edgelet/target/hsm && cd /project/edgelet/target/hsm 

cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED=On -Drun_unittests=Off -Duse_default_uuid=On -Duse_emulator=Off -Duse_http=Off -DCPACK_GENERATOR=DEB -DCPACK_PACKAGE_VERSION=1.0.9.3 -DCPACK_DEBIAN_PACKAGE_RELEASE=1 -DOPENSSL_DEPENDS_SPEC=libssl1.0.0 -DCPACK_DEBIAN_PACKAGE_ARCHITECTURE=armel -DCPACK_RPM_PACKAGE_ARCHITECTURE=armv7hl -DCMAKE_SYSTEM_NAME=Linux -DCMAKE_SYSTEM_VERSION=1 -DCMAKE_SYSROOT=/toolchain/arm-linux-gnueabi/libc -DCMAKE_C_COMPILER=/toolchain/bin/arm-linux-gnueabi-gcc -DCMAKE_CXX_COMPILER=/toolchain/bin/arm-linux-gnueabi-g++ /project/edgelet/hsm-sys/azure-iot-hsm-c/

make -j package
```

### 2.6 编译iotedged

```sh
cd /project/edgelet

rustup target add armv7-unknown-linux-gnueabi

make deb8 VERSION='1.0.9.3' REVISION=1 CARGOFLAGS='--target armv7-unknown-linux-gnueabi' TARGET=target/armv7-unknown-linux-gnueabi/release DPKGFLAGS='-b -us -uc -i --host-arch armel'
```




