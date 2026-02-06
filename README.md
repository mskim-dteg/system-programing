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

# Linux Application 운영 환경
## Linux System 메모리 맵
## User Space Application 메모리 맵
# 32bit system과 64bit system의 차이
# 메모리 관련한 여러가지 주의사항
## Register <-> L1/L2/L3 cache <-> SDRAM 구조 이해
## DMA에 대한 이해
## Alignment
## Memory management
### c++ smartpointer
