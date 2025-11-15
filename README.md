# KMU SDR ADALM Pluto firmware

This repository contains the modified ADALM Pluto firmware for the KMU SDR project.
See [analogdevicesinc/plutosdr-fw](https://github.com/analogdevicesinc/plutosdr-fw) for the default ADI firmware.

PlutoSDR Firmware for the [ADALM-PLUTO](https://wiki.analog.com/university/tools/pluto "PlutoSDR Wiki Page") Active Learning Module

Latest binary Release : 

Firmware License : 

[Instructions from the Wiki: Building the image](https://wiki.analog.com/university/tools/pluto/building_the_image)

* Build Instructions
```bash
 sudo apt-get install git build-essential fakeroot libncurses5-dev libssl-dev ccache
 sudo apt-get install dfu-util u-boot-tools device-tree-compiler libssl1.0-dev mtools
 sudo apt-get install bc python cpio zip unzip rsync file wget
 git clone --recursive https://github.com/analogdevicesinc/plutosdr-fw.git
 cd plutosdr-fw
 export VIVADO_SETTINGS=/opt/Xilinx/Vivado/2021.2/settings64.sh
 make

```
Vivado/Vitis와 Buildroot가 제공하는 AMD/Xilinx GCC 툴체인 간의 호환성 문제로 인해,
이 프로젝트는 Buildroot 외부 툴체인으로 전환되었다. : Linaro GCC 7.3-2018.05 7.3.1

https://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/arm-linux-gnueabihf/

해당 툴체인은 3가지로 빌드된다. : Buildroot, Linux, u-boot

## WorkFlow

### 1단계 : 전체 구조 생성

KMU-SDR은 독자적인 방식(Amaranth, HLS, Rust 등)으로 ADI 하드웨어를 제어하기 위한 프로젝트이다.
반면, 공식 ADI PlutoSDR 펌웨어는 다음과 같은 표준 구조를 따른다.

1. Build System: Makefile과 서브모듈(linux, hdl, u-boot, buildroot)을 조율
2. FGPA(HDL) : Xilinx Vivado( Verilog/VHDL + IP Integrator ) 사용
3. Linux Kernel : ADI가 수정한 리눅스 커널(adi-linux) 사용
4. User Space: 표준 리눅스 애플리케이션(C/C++, Python)과 libiio 라이브러리를 사용하여 하드웨어 제어

### 2단계 : 서므모듈 구성

1. 메인 펌웨어 저장소 Fork: 

2. 핵심 서브모듈 Fork :

- FPGA : https://github.com/analogdevicesinc/hdl (필수)

- 커널 : https://github.com/analogdevicesinc/linux

- 부트로더: https://github.com/analogdevicesinc/u-boot-xlnx

### 3단계 : Prerequisites

1. Vivado 설치

2. 리눅스 빌드 패키지 설치 or Docker 활용 예정


### 4단계 : FPGA 개발

- 1안: PlutoSDR FPGA 프로젝트 메인에 작업

- 2안: HLS 기반 프로젝트...

    system_top.xpr 생성과 system_top.bit와 system_top.xsa(하드웨어 핸드오프 파일) 생성 해야함
    사용하는 메모리 주소를 커널 모듈과 디바이스 트리 파일에 인식 시키는 것(추후 세부사항 정리할 것)

### 5단계 : SW 개발

1. Buildroot 구성
    - 새로운 프로그램(kmu_core.c)을 추가하려면 buildroot/package/ 안에 내 패키지를 등록하고 make menuconfig 에서 활성화 해야함

2. 드라이버 개발(linux/):
    - kmu-kmod와 같은 커널 모듈이 필요하면, linux/drivers/iio/... 경로에 직접 드라이버 코드를 추가하거나,
    buildroot의 package 기능을 이용해 외부 모듈(out-of-tree module)로 빌드하게 설정

3. 애플리케이션 제어:
    - 하드웨어 제어는 Rust 기반 Server Daemon과 libiio 기반 제어 2가지를 염두 중.

### 6단계 : 빌드

- 하드웨어 정의를 .bd로 할 수 잇지만 maia 프로젝트의 경우 maia-sdr.svd 로 자동생성
- 드라이버 커널 모듈 생성 시, mmap 기반 커스텀으로 가상 메모리를 활용하는 방법이 있었다.
- Rust 기반 Rest API 구현
- Makefile을 Custmomize 


### 빌드 산출물 List 
 * Main targets
 
     | File  | Comment |
     | ------------- | ------------- | 
     | pluto.frm | Main PlutoSDR firmware file used with the USB Mass Storage Device |
     | pluto.dfu | Main PlutoSDR firmware file used in DFU mode |
     | boot.frm  | First and Second Stage Bootloader (u-boot + fsbl + uEnv) used with the USB Mass Storage Device |
     | boot.dfu  | First and Second Stage Bootloader (u-boot + fsbl) used in DFU mode |
     | uboot-env.dfu  | u-boot default environment used in DFU mode |
     | plutosdr-fw-vX.XX.zip  | ZIP archive containg all of the files above |  
     | plutosdr-jtag-bootstrap-vX.XX.zip  | ZIP archive containg u-boot and Vivao TCL used for JATG bootstrapping |       
 
  * Other intermediate targets

     | File  | Comment |
     | ------------- | ------------- |
     | boot.bif | Boot Image Format file used to generate the Boot Image |
     | boot.bin | Final Boot Image |
     | pluto.frm.md5 | md5sum of the pluto.frm file |
     | pluto.itb | u-boot Flattened Image Tree |
     | rootfs.cpio.gz | The Root Filesystem archive |
     | sdk | Vivado/XSDK Build folder including  the FSBL |
     | system_top.bit | FPGA Bitstream (from HDF) |
     | system_top.hdf | FPGA Hardware Description  File exported by Vivado |
     | u-boot.elf | u-boot ELF Binary |
     | uboot-env.bin | u-boot default environment in binary format created form uboot-env.txt |
     | uboot-env.txt | u-boot default environment in human readable text format |
     | zImage | Compressed Linux Kernel Image |
     | zynq-pluto-sdr.dtb | Device Tree Blob for Rev.A |
     | zynq-pluto-sdr-revb.dtb | Device Tree Blob for Rev.B|     
     | zynq-pluto-sdr-revc.dtb | Device Tree Blob for Rev.C|
 

