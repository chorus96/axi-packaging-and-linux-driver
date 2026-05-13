<table class="sphinxhide" width="100%">
 <tr width="100%">
    <td align="center"><img src="https://raw.githubusercontent.com/Xilinx/Image-Collateral/main/xilinx-logo.png" width="30%"/><h1>AXI 패키징 및 Linux 드라이버 튜토리얼로 사용자 지정 IP 래핑</h1>
    </td>
 </tr>
</table>

# 소개

이 튜토리얼에서는 다양한 프로젝트를 위해 AMD Vivado&trade;와 함께 사용할 수 있는 AXI 래퍼로 자신의 지적 재산(IP) 코어를 패키징하는 필수 단계를 다룹니다. 또한 해당 IP에 대한 Linux 드라이버를 개발하고 통합하는 단계도 다룹니다.  
이 튜토리얼에서는 IP를 패키징하고 Linux 드라이버를 개발하는 방법의 예로 AXI 1-Wire 호스트 설계를 사용합니다. 참조 파일은 다음에서 공개적으로 제공됩니다.

* 패키지형 1-Wire 호스트 IP 설계: [AMD AXI 1-Wire 호스트 설계](https://github.com/Xilinx/axi_1wire_host-design)
* 1-Wire 하위 시스템 호환 Linux 드라이버 [AMD AXI 1-Wire 드라이버](https://github.com/Xilinx/linux-xlnx/blob/master/drivers/w1/masters/amd_axi_w1.c)
* 1-Wire HDL, 제약 조건 및 드라이버 파일: [AXI 패키징 및 Linux 드라이버 튜토리얼 참조 파일](./reference_files/)

[AMD Kria&trade; KD240 드라이브 스타터 키트](https://www.amd.com/en/products/system-on-modules/kria/k24/kd240-drives-starter-kit.html)는 1-Wire 인터페이스가 탑재되어 있어 이 튜토리얼을 지원하는 것을 목표로 하지만 처리 시스템(PS)(예: Arm 프로세서 또는 [AMD MicroBlaze™](https://www.amd.com/en/products/software/adaptive-socs-and-fpgas/microblaze.html) 또는 [AMD MicroBlaze™ V](https://www.amd.com/en/products/software/adaptive-socs-and-fpgas/microblaze-v.html)과 같은 소프트 프로세서)이 있는 다른 AMD 플랫폼은 사용.

> **중요**  
> 전용 1-Wire 인터페이스가 장착되지 않은 플랫폼을 사용하는 경우 1-Wire 장치 제조업체에서 명시한 외부 풀업 저항기를 사용하십시오.

## 시작하기 전에

필요한 도구:

* Linux가 설치된 호스트 머신(*PetaLinux 도구 설명서: 참조 가이드*([UG1144](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Installation-Requirements)) 참조(지원되는 OS의 경우)
* AMD Vivado&trade; 디자인 스위트 2024.2
* AMD Vitis&trade; 통합 소프트웨어 플랫폼 2024.2
* AMD PetaLinux 도구 2024.2
* Linux 시스템에 구성된 TFTP 서버([PetaLinux용 TFTP 서버 설정](https://www.instructables.com/Setting-Up-TFTP-Server-for-PetaLinux/) 참조)

### 튜토리얼 참조 파일에 액세스하기

터미널에 `git clone https://github.com/Xilinx/axi-packaging-and-linux-driver.git` 명령을 입력합니다.

## 튜토리얼 개요

1. [AXI IP 패키징](./1_axi_packaging.kr.md): Vivado Design Suite를 사용하여 AXI IP를 적절하게 패키징하는 단계를 살펴봅니다.
2. [베어메탈 드라이버 개발](./2_baremetal_driver.kr.md): Vitis 통합 소프트웨어 플랫폼을 사용하여 패키지된 AXI IP용 베어메탈 드라이버를 개발하는 과정을 안내합니다.
3. [Linux 드라이버 설계 및 배포](./3_linux_driver.kr.md): AMD PetaLinux 도구를 사용하여 패키지된 AXI IP용 일반 Linux 드라이버를 개발하는 방법과 이를 배포하는 방법을 보여줍니다.

<p align="center"><b>다음 단계 시작: <a href="./1_axi_packaging.kr.md">AXI IP 패키징</a></b></p>


<p class="sphinxhide" align="center"><sub>저작권 © 2025 Advanced Micro Devices, Inc.</sub></p>
<p class="sphinxhide" align="center"><sup><a href="https://www.amd.com/en/corporate/copyright">이용 약관</a></sup></p>
