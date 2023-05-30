# O-DU
O-DU Installation

* Download `odu/l2` and `odu/phy`
```bash
git clone -b f-release "https://gerrit.o-ran-sc.org/r/o-du/l2"
```
```bash
git clone "https://gerrit.o-ran-sc.org/r/o-du/phy"
```

* Install LFS in ubuntu 22
```bash
sudo apt-get update
sudo apt-get -y install git-lfs
git lfs install
```

* Clone FlexRAN
```bash
git clone https://github.com/intel/FlexRAN.git
```

* Download and Install oneAPI
```bash
sudo apt -y install bzip2
```
```bash
wget https://registrationcenter-download.intel.com/akdlm/irc_nas/18673/l_BaseKit_p_2022.2.0.262_offline.sh
```
```bash
sudo sh ./l_BaseKit_p_2022.2.0.262_offline.sh
```

* Setup ICX(oneAPI) environment variable
```bash
vim ~/.bashrc

source /opt/intel/oneapi/setvars.sh
export PATH=$PATH:/opt/intel/oneapi/compiler/2022.1.0/linux/bin
```
Open a new terminal

* Check the version of ICX
```bash
icx --version
```
*  Install ninja
```bash
apt-get install ninja-build meson
```
* Download DPDK
```bash
wget http://static.dpdk.org/rel/dpdk-20.11.3.tar.xz
```
```bash
tar -xf dpdk-20.11.3.tar.xz
```
* Add env variables
```bash
vim ~/.bashrc
```
```bash
export RTE_TARGET=x86_64-native-linuxapp-icx
export RTE_SDK=<DPDK_Path>/dpdk-stable-20.11.3
export WIRELESS_SDK_TOOLCHAIN=icx
export SDK_BUILD=build-avx512-icc
export PATH=$PATH:/opt/intel/oneapi/compiler/2022.0.1/linux/bin-llvm/
```

* Modify the meson_options.txt
```bash
cd <Intallation_DIR>/dpdk-stable-20.11.3
vim meson_options.txt
```
In meson_options.txt

Before:
```
option('machine', type: 'string', value: 'native',
        description: 'set the target machine type')
```
After:
```
option('machine', type: 'string', value: 'default',
        description: 'set the target machine type')
```

* Install packages
```bash
sudo apt install libmusdk mlx5 libssl-dev zlib1g-dev libfdt-dev mlx4 libnfb-dev libpcap-dev libsze2-dev libaio-dev libisal-dev
```

*  Build DPDK
```bash
meson build
cd build
ninja
ninja install
```
* Install Google Test
```bash
cd ~
wget https://github.com/google/googletest/archive/refs/tags/release-1.8.1.tar.gz
tar -xvf release-1.8.1.tar.gz
mv googletest-release-1.8.1/ gtest-1.7.0
```
* Build and Installation
```bash
export GTEST_DIR=~/gtest-1.7.0/googletest/
cd ${GTEST_DIR}
g++ -isystem ${GTEST_DIR}/include -I${GTEST_DIR} -pthread -c ${GTEST_DIR}/src/gtest-all.cc
```

```bash
ar -rv libgtest.a gtest-all.o
```

```bash
cd /root/gtest-1.7.0/googlemock/build-aux
```

```bash
cmake ${GTEST_DIR}
```
* make
```bash
make
```
If it creates some error make these changes
Open the file `/root/gtest-1.7.0/googletest/src/gtest-death-test.cc` in a text editor.

Locate the function bool testing::internal::StackGrowsDown() around line 1224.
In this function, you'll find the declaration of an uninitialized variable named dummy on line 1222.
```
int dummy;
```
To fix the error, initialize the variable dummy with a value.
```
int dummy = 0;
```
Then do `make`

* build linked file
```bash
cd ${GTEST_DIR}
ln -s build-aux/libgtest_main.a libgtest_main.a
```
* Set the google test Path
```bash
nano ~/.bashrc
export DIR_ROOT_GTEST=~/gtest-1.7.0/googletest
export GTEST_ROOT=/home/oai/gtest-1.7.0/googletest

source ~/.bashrc
```
* Set up the environment
modifiy the env variables acc to env
```bash
sudo vim <Your_Directory>/phy/setupenv.sh
```
```
#!/bin/bash

export HOME=/home/four
export DIR_ROOT=$HOME
#set the L1 binary root DIR
export DIR_ROOT_L1_BIN=$DIR_ROOT/FlexRAN
#set the phy root DIR
export DIR_ROOT_PHY=$DIR_ROOT/phy
#set the DPDK root DIR
export DIR_ROOT_DPDK=/$DIR_ROOT/dpdk-stable-20.11.3/
#set the GTEST root DIR
export DIR_ROOT_GTEST=/root/gtest-1.7.0

export WIRELESS_SDK_TARGET_ISA=avx512
export DIR_WIRELESS_TEST_5G=$DIR_ROOT_L1_BIN/testcase
export DIR_WIRELESS_SDK=$DIR_ROOT_L1_BIN/sdk/build-avx512-icc
export DIR_WIRELESS_TABLE_5G=$DIR_ROOT_L1_BIN/l1/bin/nr5g/gnb/l1/table
#source /opt/intel/system_studio_2019/bin/iccvars.sh intel64 -platform linux
export XRAN_DIR=$DIR_ROOT_PHY/fhi_lib
export XRAN_LIB_SO=true
export RTE_TARGET=x86_64-native-linuxapp-icc
export RTE_SDK=$DIR_ROOT_DPDK
#export DESTDIR=$DIR_ROOT_DPDK

#Uncomment to run tests - it's commented to make builds faster.
export GTEST_ROOT=$DIR_ROOT_GTEST

export ORAN_5G_FAPI=true
export DIR_WIRELESS_WLS=$DIR_ROOT_PHY/wls_lib
export DEBUG_MODE=true
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DIR_WIRELESS_WLS:$XRAN_DIR/lib/build
export DIR_WIRELESS=$DIR_ROOT_L1_BIN
export DIR_WIRELESS_ORAN_5G_FAPI=$DIR_ROOT_PHY/fapi_5g
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DIR_ROOT_L1_BIN/libs/cpa/bin:/opt/intel/oneapi/mkl/2022.1.0/lib/intel64:/home/four/dpdk-stable-20.11.3/build/lib
```
* Run the script
```bash
source <Your_Directory>/phy/setupenv.sh
```

* Append the command into .bashrc file
```bash
vim ~/.bashrc
source <Your_Directory>/phy/setupenv.sh
```
Open a new terminal.
*  Install xRAN Library (FHI Library)
```bash
sudo apt-get install libnuma-dev
```
```bash
sudo ln -s /usr/bin/pkg-config /usr/bin/pkgconf
```
```bash
cd $XRAN_DIR
./build.sh xclean
```
```bash
cd $XRAN_DIR/lib
make
```
* Add a line in `/home/four/phy/fhi_lib/lib/src/xran_bfp_ref.cpp`
```bash
#include <limits>
```

* Modify `/home/four/phy/fhi_lib/app/Makefile`
and add `-lnuma` at line 286
```bash
$(PROJECT_BINARY): $(DIRLIST) echo_start_build $(GENERATE_DEPS) $(PRE_BUILD) $(CC_OBJTARGETS) $(CPP_OBJTARGETS) $(AS_OBJTARGETS)
        @$(MD) $(BUILDDIR)
        @echo "[LD] $@ "
        @$(LD) -o $@ $(CC_OBJTARGETS) $(CPP_OBJTARGETS) $(AS_OBJTARGETS) $(LD_FLAGS) -lnuma -Wl,-L $(BUILDDIR) -lrt -lpthread
```
* Install Octave in ubuntu
```bash
sudo apt install octave
```
* Generate IQ Samples of Sample app
```bash
cd $XRAN_DIR/app
octave gen_test.m
```
### Now Installation of xRAN Library (FHI Library) is completed 
