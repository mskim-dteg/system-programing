# NPU를 포함한 SOC 분석
EN683 구조
![Interconnect architecture of EN683X](en683_interconnect.jpg)
* CPU
  + Core 0~3 : 32KB I/D-cache
  + L2 Cache : 512KB
* NPU
  + SRAM : 2MB

[QCS6490](https://www.qualcomm.com/internet-of-things/products/q6-series/qcs6490) 구조
![QCS6490 Architecture](qcs6490_architecture.jpg)
* CPU
  + Silver Cores (x4)
      - L1 cache : 32KB I/D
      - L2 cache : 128KB
  + Gold Cores (x3), Prime Core (x1)
      - L1 cache : 32KB I/D
      - L2 cache : 128KB
  + L3 cache : 2MiB
* HTP(NPU, hexagon dsp)
  + 1MB L2 cache
  + vTCM : 2MB
![Hexagon Tensor Processor](qualcomm_htp.jpg)

Video Pipe Line

# 32bit system과 64bit system의 차이
* c/c++ data type 크기
  |   |
  |---|---

# Linux Application 운영 환경
## [linux virtual address space layout](https://www.google.com/search?q=linux+virtual+address+space+layout)

32bit architecture

![32bit linux memory layout](https://i.sstatic.net/5PUMH.jpg)

64bit architecture
![64bit linux memory layout](https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Linux_Virtual_Memory_Layout_64bit.svg/960px-Linux_Virtual_Memory_Layout_64bit.svg.png)

* "physical memory map" vs "virtual memory map"
  + physical memory map은 SOC 마다 다름
  + physical memory 구성 요소
    - SDRAM
    - PCIe
    - peripheral registers (memory mapped io)
  + linux kernel은 MMU를 사용하여 상위 virtual memory space에 physical memory를 전부 mapping (재배치)
  + 최근의 SOC에서 모든 HW resource를 memory라고 해도 과언이 아님
    - [linux kernel memory barriers](https://www.kernel.org/doc/Documentation/memory-barriers.txt)
* "kernel memory space" vs "user memory space"
  + kernel memory mapping은 1개만 존재
  + user space memory mapping은 process마다 1개씩 생성
    ![physical vs virtual](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d6/Virtual_address_space_and_physical_address_space_relationship.png/773px-Virtual_address_space_and_physical_address_space_relationship.png)
    - process간 memory protection
    - thread는 동일한 memory mapping 공유
    - kernel thread는 kernel memory mapping 사용
    - process 전환시 user space memory mapping만 바꿈
    - user space memory mapping은 physical address뿐만 아니라 file도 mapping
  + kernel api(open,close,read,write,ioctl,poll....) call시 memory mapping 전환 불필요
    - interrupt 처리 시에도 memory mapping 전환 불필요

## User Space Application 메모리 맵
https://wxdublin.gitbooks.io/deep-into-linux-and-beyond/content/address_space.html
![linux user space memory layout](https://wxdublin.gitbooks.io/deep-into-linux-and-beyond/content/linuxFlexibleAddressSpaceLayout.png)
* process memory map 확인 방법
  ```shell
  cat /proc/[pid]/maps
  ```
  + 자신의 memory map
    ```shell
    /proc/self/maps
    ```
* ELF file format : https://stackoverflow.com/questions/14361248/whats-the-difference-of-section-and-segment-in-elf-file-format
  ![elf file layout](https://i.sstatic.net/RMV0g.png)
  + elf file 정보 확인 방법 : [readelf](https://man7.org/linux/man-pages/man1/readelf.1.html)
    - pre-linked shared library 확인 방법
      ```shell
      readelf -d [executable file or shared library]
      ```
* [demand paging](https://en.wikipedia.org/wiki/Demand_paging)
  + Text segment, ro data, 및 ro memory mapping segment는 실제 사용할 때만 메모리에 적재한다. 메모리 사용량의 증가에 따라서 최근에 접근하지 않은 page는 메모리에서 제거하고 필요시 다시 적재
  + 항상 메모리에 적재해야할 경우(realtime procssing) [mlock](https://man7.org/linux/man-pages/man2/mlock.2.html) api 사용
  + [mmap](https://man7.org/linux/man-pages/man2/mmap.2.html) : file을 virtual memory space에 mapping
    ```c
    #include <sys/mman.h>
    void *mmap(size_t length, void *addr, size_t length, int prot, int flags, int fd, off_t offset);
    int munmap(size_t length, void *addr, size_t length);
    ```
# 메모리 관련한 여러가지 주의사항
## Register <-> L1/L2/L3 cache <-> SDRAM 구조 이해
## DMA에 대한 이해
## Alignment
## Memory management
### c++ smartpointer
