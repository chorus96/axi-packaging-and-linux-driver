<table class="sphinxhide" width="100%">
 <tr width="100%">
    <td align="center"><img src="https://raw.githubusercontent.com/Xilinx/Image-Collateral/main/xilinx-logo.png" width="30%"/><h1>AXI 패키징 및 Linux 드라이버 튜토리얼로 사용자 지정 IP 래핑</h1>
    </td>
 </tr>
</table>

<a id="linux-driver-development"></a>
# 리눅스 드라이버 개발

튜토리얼의 이 섹션에서는 1-Wire 코어 IP용 Linux 드라이버를 작성하는 프로세스를 다룹니다. 이 프로세스는 1-Wire 코어 IP를 대상으로 합니다. 드라이버는 IP마다 다르므로 모든 종류의 IP에 적용할 수 있는 예를 제시하기는 어렵습니다. 즉, 높은 수준의 프로세스는 대부분의 IP에 적용 가능해야 합니다.

<a id="outline"></a>
## 개요

1. [소개](#소개)
2. [1-Wire Core Linux 드라이버 개발](#the-1-wire-core-linux-drivers-development)
   1. [문자 장치 드라이버](#character-device-driver)
   2. [1-Wire 서브시스템 드라이버](#1-wire-subsystem-driver)

<a id="introduction"></a>
## 소개

Linux 드라이버 개발은 임베디드 시스템 설계의 중요한 측면입니다. AMD PetaLinux 도구는 맞춤형 임베디드 Linux 드라이버 생성 프로세스를 간소화합니다. 이러한 맥락에서 커널 드라이버는 Linux 운영 체제와 IP(지적 재산), 주변 장치 및 기타 중요한 시스템 요소를 포함한 기본 하드웨어 구성 요소 간의 직접 통신을 가능하게 하는 데 필수적인 역할을 합니다.

Linux 드라이버를 개발하면 엔지니어는 복잡한 애플리케이션을 실행하기 위한 견고한 기반을 구축하는 동시에 하드웨어 리소스와 시스템 성능을 효율적으로 관리할 수 있습니다. PetaLinux 개발 환경을 사용하여 개발자는 특정 하드웨어 구성에 대한 드라이버를 만들고 잠재적인 통합 문제를 식별 및 해결하며 Linux 운영 체제에서 하드웨어의 원활한 작동을 보장할 수 있습니다.

Linux 드라이버 개발을 시작하기 전에 앞서 설명한 대로 베어메탈 드라이버를 사용하여 기본 하드웨어를 검증하는 것이 중요합니다. 이 단계는 엔지니어에게 전체 운영 체제에서 발생하는 복잡성 없이 하드웨어 장치의 기본 기능에 대한 귀중한 통찰력을 제공합니다. 하드웨어 구성 요소가 베어메탈 드라이버를 통해 철저히 테스트되고 검증되면 개발자는 자신 있게 더 높은 수준의 추상화 및 통합을 제공하는 Linux 드라이버를 만들 수 있습니다.

PetaLinux를 사용한 Linux 드라이버 개발은 하드웨어와 운영 체제 간의 상호 작용을 촉진할 뿐만 아니라 맞춤형 IP 통합 프로세스를 간소화하고 시스템 설계 프로세스 중에 발생할 수 있는 모든 소프트웨어 충돌을 해결합니다. 또한 이를 통해 개발자는 대상 하드웨어를 기반으로 임베디드 Linux 배포판을 구성, 디버깅 및 최적화할 때 최대의 유연성을 유지할 수 있습니다.

PetaLinux와 함께 작동하도록 특별히 맞춤화된 Linux 드라이버를 제공함으로써 고객이 다양한 임베디드 플랫폼에서 애플리케이션을 신속하게 개발하고 배포할 수 있도록 지원합니다. 이러한 드라이버는 통합 프로세스를 크게 단순화하여 시스템 설계자가 IP를 프로젝트에 원활하게 통합하고 개발 및 디버깅 프로세스를 가속화할 수 있도록 합니다. 궁극적으로 이는 프로토타입 제작 속도를 높이고 출시 기간을 단축하며 최종 사용자 경험을 전반적으로 향상시킵니다.

<a id="the-1-wire-core-linux-drivers-development"></a>
## 1-Wire 코어 Linux 드라이버 개발

Linux 프레임워크의 특정 드라이버는 특정 통신 프로토콜을 준수하는 장치를 관리하도록 설계된 특정 하위 시스템 내에 존재합니다. 그러한 서브시스템 중 하나가 1-Wire 프로토콜을 사용하는 장치에 적합한 1-Wire Linux 서브시스템입니다. 1-Wire 서브시스템은 [1-Wire 서브시스템 디렉토리](https://github.com/torvalds/linux/tree/master/drivers/w1)에서 찾을 수 있는 광범위한 마스터 및 슬레이브 장치 목록으로 구성됩니다. 또한 이 서브시스템에는 공식 1-Wire 드라이버를 생성하기 위한 지침을 간략하게 설명하고 필요한 최소 기능을 식별하는 [1-Wire 프레임워크](https://github.com/torvalds/linux/blob/master/include/linux/w1.h)가 있습니다.

공식 1-Wire 드라이버 자격을 얻으려면 드라이버는 핵심 기능 세트의 일부로 최소한 터치 비트 및 버스 재설정 기능을 구현해야 합니다. 공식 드라이버의 예로는 AMD 1-Wire IP용으로 특별히 개발된 [AMD 1-Wire 드라이버](https://github.com/torvalds/linux/blob/master/drivers/w1/masters/amd_axi_w1.c)가 있습니다. 이 구성 요소는 Linux 기반 시스템에서 IP의 효과적인 통합 및 활용을 촉진합니다.

이 튜토리얼에서는 공식 1-Wire 드라이버 개발 기술을 너무 구체적으로 다루지는 않을 것이다. 대신 읽기 및 쓰기 작업을 수행하는 다양한 장치를 수용하는 드라이버 개발에 대한 보다 다양한 접근 방식인 문자 장치 프레임워크를 사용하는 데 중점을 둘 것입니다. 이 프레임워크는 주로 AXI 레지스터 읽기 및 쓰기와 관련된 AXI IP 및 해당 드라이버에 특히 적합합니다. Linux 시스템은 특히 인터럽트 처리에 능숙하여 베어메탈 시스템에 비해 몇 가지 장점을 제공합니다. 인터럽트 기반 메커니즘을 활용함으로써 Linux의 드라이버는 효율성을 크게 높이고 리소스 활용도를 향상시켜 전체 시스템 성능과 응답성을 최적화할 수 있습니다.

Linux 드라이버 개발에 문자 장치 프레임워크를 사용하면 다양한 장치에서 원활하게 작동하는 강력하고 확장 가능하며 유연한 드라이버를 구축할 수 있습니다. 이러한 광범위한 호환성은 궁극적으로 드라이버 개발 프로세스뿐만 아니라 IP 및 하드웨어 구성 요소를 고급 고성능 Linux 기반 임베디드 시스템에 통합하는 작업도 단순화합니다.

<a id="character-device-driver"></a>
### 문자 장치 드라이버

1. PetaLinux 프로젝트를 생성합니다.
   1. 일반적으로 ```source <petalinux_install>/settings.sh```와 유사한 명령을 사용하여 PetaLinux 설치를 소싱합니다.
   2. 작업 디렉토리 ```cd <working_directory>```로 이동합니다.
   3. 프로젝트 생성: ```petalinux-create -t project -n 1wire --template zynqMP```
   4. 프로젝트 디렉토리 ```cd 1wire```로 이동합니다.
   5. PetaLinux 프로젝트의 하드웨어 구성을 하드웨어 ```petalinux-config --get-hw-description=<working_directory>/myproject/mydesign_wrapper.xsa```로 업데이트하세요.
   6. PetaLinux 시스템 구성 창에서 *하위 시스템 하드웨어 설정 &rarr; 직렬 설정*으로 이동하여 STDIN 및 STDOUT 설정을 변경합니다.
      + *PMUFW 직렬 stdin/stdout*: psu_uart_1
      + *FSBL 직렬 stdin/stdout*: psu_uart_1
      + *TF-A 직렬 stdin/stdout*: psu_uart_1
      + *U-boot/Linux 직렬 stdin/stdout*: psu_uart_1
   7. 종료하고 새 구성을 저장합니다.
2. 드라이버 모듈을 생성하고 구현합니다.
   1. 모듈 생성:```petalinux-create -t modules -n xlnxw1 --enable```
   2. `<working_directory>/1wire/project-spec/meta-user/recipes-modules/xlnxw1` 아래에 생성된 모듈의 내용을 살펴보세요.
   3. 이제 드라이버 소스 파일을 편집합니다.
      1. `<working_directory>/1wire/project-spec/meta-user/recipes-modules/xlnxw1/files/xlnxw1.c`를 엽니다.
      2. `<working_directory>/reference_files/linux_driver/w1chardev.c`의 내용을 소스 파일에 복사합니다.
   4. 드라이버의 내용을 학습할 수 있습니다.
      + AXI 레지스터를 읽고 쓰는 기본 기능:
         <details>
         <summary>읽기 및 쓰기 레지스터</summary>

         ```C
            static inline void xlnxw1_write_register(u8 reg_offset, u32 val);
            static inline u32 xlnxw1_read_register(u8 reg_offset);
         ```

         </details>
      + 이러한 기능은 1-Wire 기능을 구현하는 데 사용됩니다.
         <details>
         <summary>1-와이어 기능</summary>

         ```C
            static long xlnxw1_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
            {
               switch (cmd)
               {
               case XLNX_IOCTL_RESET_BUS:
                  ...
               case XLNX_IOCTL_READ_BIT:
                  ...
               case XLNX_IOCTL_WRITE_BIT:
                  ...
               case XLNX_IOCTL_READ_BYTE:
                  ...
               case XLNX_IOCTL_WRITE_BYTE:
                  ...
               }
            };
         ```

         </details>
      + ```xlnxw1_open```: 장치가 사용 중인지 확인하고 사용 횟수를 증가시키며 모듈이 사용되는 동안 로드되는지 확인합니다.
      + ```xlnxw1_release```: 장치의 사용 횟수를 줄이고 모듈 참조를 해제합니다.
      + ```xlnxw1_irq```: 인터럽트가 트리거되면 IRQ 활성화 레지스터를 지우고 대기 대기열을 깨웁니다.
      + ```xlnxw1_probe```: 드라이버 리소스(메모리, IRQ 등)를 초기화하고 장치 관련 정보를 저장합니다.
      + ```xlnxw1_remove```: 드라이버 리소스(메모리, IRQ 등)를 해제하고 장치 데이터 구조의 할당을 해제합니다.
      + ```xlnxw1_of_match```: 드라이버를 장치 트리 노드와 일치시키는 데 사용되는 호환 장치 문자열이 포함된 테이블입니다.
      + ```xlnxw1_driver```: 플랫폼 드라이버, 이름, 프로브, 제거 기능 및 호환성 테이블을 정의합니다.
      + ```xlnxw1_init```: 메이저 번호로 캐릭터 디바이스를 등록하고, 디바이스 클래스와 디바이스를 생성하고, 플랫폼 드라이버를 등록합니다.
      + ```xlnxw1_exit```: 장치를 파괴하고, 장치 클래스를 등록 취소하고, 캐릭터 장치를 등록 취소하고, 플랫폼 드라이버를 등록 취소합니다.
3. 1선 애플리케이션을 생성하고 구현합니다.
   1. 애플리케이션 생성: ```petalinux-create -t apps -n xlnxw1-app --enable```
   2. 튜토리얼에서 제공하는 애플리케이션인 ```cp <working_directory>/reference_files/linux_driver/xlnxw1-app.c <working_directory>/1wire/project-spec/meta-user/recipes-apps/xlnxw1-app/files/xlnxw1-app.c```로 애플리케이션을 덮어씁니다.
   3. `xlnxw1-app.c`의 내용을 살펴볼 수 있습니다. 애플리케이션은 1-Wire 온도 센서의 온도를 프로브하고 이를 표시합니다.
4. 1-Wire 드라이버와 애플리케이션을 구축하고 테스트한다.
   1. 프로젝트 빌드: ```petalinux-build```
   2. 모든 것을 연결하세요:
      + 1-Wire 온도 센서를 KD240에 연결합니다.
      + USB JTAG/UART 케이블을 KD240에서 컴퓨터로 연결하세요
      + KD240의 네트워크 이더넷 케이블을 네트워크 스위치에 연결합니다
      + KD240에 SD 카드가 삽입되어 있지 않은지 확인하십시오.
      + KD240에 전원 공급 장치를 연결합니다
   3. TFTP를 사용하여 프로젝트를 로드합니다.
      1. ```ifconfig```를 사용하여 컴퓨터 IP 주소를 검색하세요.
      2. KD240에서 ```ZynqMP>``` 프롬프트가 표시되면 다음을 수행하여 비트스트림을 로드하고 PetaLinux를 부팅합니다.

         ```bash
            ZynqMP> setenv serverip <your computer IP>
            ZynqMP> tftpboot 0x02000000 system.bit
            ZynqMP> fpga load 0 0x02000000 $filesize
            ZynqMP> pxe get
            ZynqMP> pxe load
         ```

      3. 커널 메시지를 보면 다음이 표시됩니다.

         ```bash
            [    8.660297] Registration successful, 1-wire device's major number is 237.
            [    8.667246] 1-wire class registration successful.
            [    8.676375] 1-wire device created successfully.
            [    8.681842] xlnxw1 a0000000.axi_1wire_host: xlnxw1 mapped to 0x81ea0000, irq=53
         ```

         타임스탬프, 주요 번호, 주소 및 irq는 세션에 따라 다를 수 있습니다.
   4. 로그인하려면 사용자 이름 `Petalinux`를 사용하고 비밀번호를 설정하세요.
   5. ```sudo xlnxw1-app```를 입력하여 애플리케이션을 실행하면 장치가 열려 있고 온도가 인쇄된다는 메시지가 표시됩니다.

<a id="1-wire-subsystem-driver"></a>
### 1선 서브시스템 드라이버

앞서 언급한 것처럼 1-Wire 장치 제품군에는 특정 드라이버 서브시스템이 있습니다. AMD 1-Wire IP는 기본 Linux 커널로 업스트림된 고유한 특정 1-Wire 드라이버를 사용하여 개발되었습니다. 드라이버 세부 정보를 살펴보지 않고 PetaLinux에서 활성화하여 주변 장치로 테스트하게 됩니다.

1. 1-Wire 서브시스템을 사용하도록 PetaLinux 프로젝트를 수정합니다.
   1. 이전에 생성된 캐릭터 드라이버를 1-Wire 드라이버 ```cp <working_directory>/reference_files/linux_driver/amd_axi_w1.c <working_directory>/1wire/project-spec/meta-user/recipes-modules/xlnxw1/files/xlnxw1.c```로 교체합니다.
   2. 1선 서브시스템 및 슬레이브 장치를 활성화합니다.
      1.```petalinux-config -c kernel```
      2. *장치 드라이버*로 이동하여 *댈러스의 1선 지원*을 찾으세요. 1-Wire 서브시스템을 포함하려면 **Y**를 누르십시오.
      3. **Dallas의 1선 지원**을 선택하고 **커넥터를 통한 사용자 공간 통신**을 포함해야 합니다.
      4. **1-wire 슬레이브**를 선택하고 원하는 경우 모든 슬레이브를 포함하거나 장치만 포함합니다.
      5. 종료하고 구성을 저장합니다.
   3. lm 센서 패키지 그룹 활성화
      1.```petalinux-config -c rootfs```
      2. *PetaLinux 패키지 그룹 &rarr; packagegroup-lmsensors*로 이동하고 **Y**를 눌러 *packagegroup-lmsensors*를 포함합니다.
      3. 종료하고 구성을 저장합니다.
2. 1-Wire 프로젝트를 구축하고 테스트합니다.
   1. 프로젝트 재구축: ```petalinux-build```
   2. TFTP를 사용하여 프로젝트를 로드합니다.
      1. ```ifconfig```를 사용하여 컴퓨터 IP 주소를 검색하세요.
      2. KD240에서는 ```ZynqMP>``` 프롬프트가 표시됩니다. 비트스트림을 로드하고 PetaLinux를 부팅하려면 다음을 수행하십시오.

         ```bash
            ZynqMP> setenv serverip <your computer IP>
            ZynqMP> tftpboot 0x02000000 system.bit
            ZynqMP> fpga load 0 0x02000000 $filesize
            ZynqMP> pxe get
            ZynqMP> pxe load
         ```

      3. 커널 메시지를 보면 다음이 표시됩니다.

         ```bash
            [    7.906413] w1_master_driver w1_bus_master1: Attaching one wire slave 28.000000040711 crc 2d
         ```

         타임스탬프와 장치 세부 정보는 KD240에 연결한 장치에 따라 다릅니다.

      4. 로그인하려면 사용자 이름 `petalinux`를 사용하고 비밀번호를 설정하세요.
      5. ```sensors```를 입력하면 온도 센서에서 읽은 온도나 1선 장치에서 캡처한 모든 정보를 볼 수 있습니다.

 ---

   이제 통합, 패키지 IP용 드라이버가 생겼습니다. 이 드라이버는 IP에 내장된 인터럽트 기능을 효율적으로 활용합니다. Linux 드라이버를 사용하여 인터럽트를 구현하는 것이 필수는 아니지만 불필요한 CPU 리소스 소비를 방지하려면 그렇게 하는 것이 좋습니다.

   PetaLinux 도구에 대한 자세한 내용은 *PetaLinux 도구 설명서: 참조 가이드* [(UG1144)](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide)를 참조하세요.

---
<p class="sphinxhide" align="center"><sub>저작권 © 2025 Advanced Micro Devices, Inc.</sub></p>
<p class="sphinxhide" align="center"><sup><a href="https://www.amd.com/en/corporate/copyright">이용 약관</a></sup></p>
