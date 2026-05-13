<table class="sphinxhide" width="100%">
 <tr width="100%">
    <td align="center"><img src="https://raw.githubusercontent.com/Xilinx/Image-Collateral/main/xilinx-logo.png" width="30%"/><h1>AXI 패키징 및 Linux 드라이버 튜토리얼로 사용자 지정 IP 래핑</h1>
    </td>
 </tr>
</table>

<a id="baremetal-driver-development"></a>
# 베어메탈 드라이버 개발

튜토리얼의 이 섹션에서는 1-Wire 코어 IP용 베어메탈(독립형) 드라이버를 작성하는 프로세스를 다룹니다. 이 프로세스는 1-Wire 코어 IP를 대상으로 하며 드라이버가 IP마다 다르므로 모든 종류의 IP에 적용할 수 있는 예를 제시하기는 어렵습니다. 즉, 높은 수준의 프로세스는 대부분의 IP에 적용 가능해야 합니다.

<a id="outline"></a>
## 개요

1. [소개](#소개)
2. [1-Wire 코어 베어메탈 드라이버 개발](#the-1-wire-core-baremetal-drivers-development)
   1. [비트스트림 및 Vitis 프로젝트 생성](#creating-a-bitstream-and-a-vitis-project)
   2. [XSCT를 사용하여 IP 테스트](#test-the-ip-using-xsct)
   3. [Vitis 드라이버 개발](#developing-drivers-with-Vitis)
   4. [베어메탈 애플리케이션 테스트](#test-a-baremetal-application)

<a id="introduction"></a>
## 소개

베어메탈 드라이버는 운영 체제 없이 마이크로 컨트롤러 또는 SoC(시스템 온 칩)의 하드웨어와 직접 인터페이스하도록 설계된 하위 수준 소프트웨어 구성 요소입니다. 이는 임베디드 시스템에서 작업하는 개발자를 위한 필수 도구 역할을 하며, 하드웨어 장치를 조작하고, 레지스터에 액세스하고, 상위 수준 소프트웨어 추상화 계층을 통해서는 달성할 수 없는 기타 중요한 작업을 수행할 수 있는 수단을 제공합니다. 이러한 유형의 드라이버는 시뮬레이션만으로는 검증하기 어려울 수 있는 복잡한 지적 재산(IP)을 테스트하고 디버깅할 때 특히 유용합니다.

베어메탈 드라이버의 주요 목적은 엔지니어가 실제 상황에서 하드웨어를 제어하고 테스트할 수 있는 효율적이고 간소화된 방법을 제공하는 것입니다. 운영 체제와 관련된 오버헤드를 제거함으로써 개발자는 기본 하드웨어에 더 직접적으로 액세스할 수 있으므로 더 큰 유연성, 더 빠른 프로토타입 제작 및 시스템 동작에 대한 더 심층적인 통찰력을 얻을 수 있습니다. 이는 궁극적으로 더 빠른 개발 주기와 시스템 문제를 신속하게 식별하고 해결할 수 있는 능력으로 이어집니다.

Linux 기반 개발 환경으로 전환하기 전에 베어메탈 드라이버를 사용하여 설계를 검증하는 것이 중요합니다. 이 단계는 엔지니어에게 전체 운영 체제의 복잡성 없이 임베디드 시스템에서 하드웨어 구성 요소의 기본 기능을 확인할 수 있는 기회를 제공합니다. 모든 하드웨어 요소와 하위 시스템이 베어메탈 환경에서 예상대로 작동하도록 하면 개발 프로세스 초기에 잠재적인 문제를 감지하고 해결할 수 있으므로 나중에 하드웨어를 보다 포괄적인 Linux 기반 프레임워크에 통합할 때 통합 문제를 줄일 수 있습니다.

Linux 개발 환경에서 하드웨어와의 상호 작용은 일반적으로 특정 하위 수준 상호 작용을 가리거나 모호하게 할 수 있는 커널 드라이버를 통해 촉진됩니다. 그러나 엔지니어는 베어메탈 드라이버를 사용하여 먼저 설계를 테스트함으로써 직면하는 문제가 하드웨어와 관련된 것이며 전체 운영 체제에 존재하는 추가 소프트웨어 계층으로 인해 발생하는 것이 아닌지 확인할 수 있습니다. 하드웨어에 대한 이러한 철저한 검증을 통해 Linux 기반 아키텍처로 보다 원활하게 전환할 수 있는 길을 열어 보다 복잡한 애플리케이션을 자신있게 구축할 수 있는 견고한 기반을 보장합니다.

IP가 포함된 베어메탈 드라이버를 제공하면 고객이 AMD Vitis&trade; 통합 소프트웨어 플랫폼을 사용하여 설계를 검증하거나 애플리케이션을 더 빠르게 개발할 수 있습니다. 이러한 드라이버는 IP를 고객 시스템에 원활하게 통합할 뿐만 아니라 하드웨어 및 소프트웨어 구성 요소 모두에 대한 개발 및 디버깅 프로세스를 단순화합니다. 이는 궁극적으로 보다 신속한 프로토타입 제작, 출시 시간 단축, 최종 사용자를 위한 전반적인 향상된 경험으로 이어집니다.

Vitis 통합 소프트웨어 플랫폼은 개발자에게 AMD 하드웨어 기능의 잠재력을 최대한 활용하는 애플리케이션 제작을 위한 포괄적인 환경을 제공하도록 특별히 설계되었습니다. 베어메탈 드라이버를 IP 패키지에 통합함으로써 고객은 성능을 최적화하고 호환성 문제를 해결하며 AMD 기반 플랫폼에 애플리케이션을 배포할 때 원활한 경험을 보장하는 데 필요한 모든 것을 갖추게 됩니다.

<a id="the-1-wire-core-baremetal-drivers-development"></a>
## 1-Wire 코어 베어메탈 드라이버 개발

IP 드라이버를 개발하는 과정은 IP의 성격과 밀접한 관련이 있습니다. 1-Wire 코어 IP는 1-Wire 프로토콜을 구현하기 위해 몇 가지 특정 명령 세트를 실행하는 간단한 컨트롤러입니다. 각 명령어에는 상태 신호 레지스터에 알려진 특정 예상 값이 있습니다. 이러한 이유로 1-Wire 버스 마스터 신호용 드라이버를 작성하는 것은 그리 어렵지 않으며 IP 자체를 디버깅하는 데 도움이 됩니다. 지원되는 1-Wire 버스 마스터 신호는 다음과 같습니다.

+ 재설정/존재 신호
+ 비트(0 또는 1) 신호 쓰기
+ 비트(0 또는 1) 신호를 읽습니다.

순서는 항상 다음과 같습니다.

1. IP 상태 레지스터에서 `READY` 신호를 기다립니다.
2. 명령어 레지스터에 명령어를 씁니다.
3. `GO` 신호를 쓰고 제어 레지스터에서 _CONTROL RESET_ 신호를 삭제합니다.
4. IP 상태 레지스터에서 `DONE` 신호를 기다립니다.
5. 필요한 경우 데이터를 검색합니다.
6. 제어 레지스터에서 `GO` 신호를 삭제합니다.

1-Wire Core IP에 대한 자세한 내용은 [AXI 1-Wire Host](https://github.com/Xilinx/axi_1wire_host-design/blob/main/README.md) 문서를 참조하십시오.

이제 이전에 패키징한 1-Wire 코어를 통합하기 위한 하드웨어 설계 생성 프로세스를 진행하고, 비트스트림을 생성하고, 베어메탈 드라이버를 개발 및 테스트하기 위한 Vitis 프로젝트를 생성합니다. 또한 IP 코어를 디버깅하는 방법에 대한 통찰력도 얻을 수 있습니다.

<a id="creating-a-bitstream-and-a-vitis-project"></a>
### 비트스트림 및 Vitis 프로젝트 생성

1. 하드웨어 설계를 생성합니다.

   1. 새로운 Vivado 프로젝트를 생성하거나 이전에 생성된 프로젝트를 사용하세요. 이후 버전인 `<working_directory>/myproject/myproject.xpr`를 사용하게 됩니다.
   2. 새 블록 디자인을 생성하고 이름을 `mydesign`로 지정합니다.
   3. 블록 다이어그램에서 **IP 추가** ![소스 추가](./images/addSources.png)를 클릭하고 "1wire"를 검색한 다음 **AXI 1-Wire 버스 호스트**를 선택합니다.
   4. 다시 **IP 추가** ![소스 추가](./images/addSources.png)를 클릭하고 "MPSoC"를 검색한 다음 **Zynq UltraScale+ MPSoC**를 선택합니다.
   5. **블록 자동화 실행** 및 **보드 사전 설정 적용**을 클릭합니다.
   6. **연결 자동화 실행**을 클릭하고 1-Wire 코어의 AXI 슬레이브 포트를 Zynq UltraScale+ MPSoC AXI 마스터 HPM0 포트에 연결하는 기본값을 유지합니다.
   7. **연결 자동화 실행**을 다시 클릭합니다.
   8. AXI 1-Wire 버스 호스트의 _w1_bus_ 포트를 마우스 오른쪽 버튼으로 클릭하고 **외부 만들기**를 선택합니다.
   9. AXI 1-Wire 버스 호스트의 _w1_irq_ 포트를 Zynq UltraScale+ MPSoC의 `pl_ps_irq` 포트에 연결합니다.

   > **참고:** 기본적으로 Zynq UltraScale+ MPSoC의 _pl_clk0_ 클록은 99.999MHz이므로 1-Wire 코어(100)의 기본 클록 분배기 매개변수를 수정할 필요가 없습니다. 다른 시계를 사용하는 경우 시계 분배기의 값을 수정하십시오. 그렇게 하려면 1-Wire 코어를 더블 클릭하여 편집한다.

2. 비트스트림을 생성합니다.

   1. 소스 창에서 **Design Sources &rarr; mydesign.bd**를 마우스 오른쪽 버튼으로 클릭하고 **Create HDL Wrapper...**를 클릭합니다. **Vivado가 래퍼 및 자동 업데이트를 관리하도록 합니다**를 선택합니다.
   2. 소스 창에서 **Design Sources &rarr; mydesign_wrapper.v**를 마우스 오른쪽 버튼으로 클릭하고 **상위로 설정**을 클릭합니다.
   3. 소스 창에서 **소스 추가** ![소스 추가](./images/addSources.png)를 클릭하세요.
      1. **제약조건 추가 또는 생성**을 선택하고 **다음 >**을 클릭합니다.
      2. **파일 생성**을 선택하고 새 제약 조건 파일을 생성한 후 이름을 `myConstraint`로 지정합니다.
      3. 마침을 클릭합니다.
   4. 제약 조건 파일을 편집하고 KD240에 대해 다음을 추가합니다.

      ```verilog
      set_property PACKAGE_PIN H13 [get_ports w1_bus_0]
      set_property IOSTANDARD LVCMOS33 [get_ports w1_bus_0]
      set_property PULLUP true [get_ports w1_bus_0]
      ```

   5. **비트스트림 생성**을 클릭합니다.
   6. 구현된 디자인을 열어서 보실 수 있습니다.
   7. **파일 &rarr; &rarr; 내보내기 하드웨어 내보내기...**를 클릭합니다.
      1. 새 창의 첫 페이지에서 **다음 >**을 클릭하세요.
      2. **비트스트림 포함**을 선택하고 **다음 >**을 클릭합니다.
      3. 파일 이름과 위치를 기록해 두십시오. `<working_directory>/myproject/mydesign_wrapper.xsa`여야 하며 **Finish**를 클릭합니다.

3. Vitis 플랫폼을 생성합니다.

   1. 일반적으로 ```source /opt/Xilinx/Vitis/2024.2/settings64.sh```. Then, launch the Vitis IDE by entering ```vitis```와 유사한 명령을 사용하여 Vitis 설치를 소싱합니다.
   2. Vitis Unified IDE에서 **Set Workspace**를 선택합니다. 새 작업공간 `<working_directory>/myworkspace/`를 생성하고 새로 생성된 작업공간 디렉터리로 이동한 후 **열기**를 클릭합니다.
   3. 임베디드 개발에서 **플랫폼 구성 요소 생성**을 클릭합니다.
      1. 이름 및 위치:
         + 구성요소 이름: _1-wire_plat_
         + 구성 요소 위치: _<working_directory>/myworkspace/_ (기본 위치)
      2. 흐름:
         + **하드웨어 디자인**을 선택하세요.
         + **찾아보기**를 클릭하여 하드웨어 디자인(XSA) 파일을 선택하고 **`<working_directory>/myproject/mydesign_wrapper.xsa`**를 선택합니다.
      3. OS 및 프로세서:
         + 운영 체제: _독립형_
         + 프로세서: _psu_cortexa53_0_
         + **부팅 아티팩트 생성**을 선택한 상태로 유지합니다.
         + **PMU 펌웨어 생성**을 선택합니다.
         + FSBL을 생성할 대상 프로세서: _psu_cortexa53_0_을 선택합니다.
         ![플랫폼 구성요소 생성](./images/createPlatformComponent.png)
   4. (선택 사항) KD240에 대한 STDIN 및 STDOUT을 수정합니다. 다른 플랫폼의 경우 이 단계를 수행할 필요가 없을 수도 있습니다. 이 단계는 KD240이 UART에 출력을 표시하는 데 필요합니다.
      1. MYWORKSPACE에서 _1-wire_plat &rarr; 설정 &rarr; vitis-comp.json_을 엽니다.
      2. _1-wire_plat &rarr; psu_cortexa53_0 &rarr; 독립형_psu_cortexa53_0 &rarr; 보드 지원 패키지 &rarr; 독립형_으로 이동합니다.
      3. 다음과 같이 구성을 수정합니다.
         + 독립형_stdin: `psu_uart_1`
         + 독립형_stdout: `psu_uart_1`
      4. _1-wire_plat &rarr; psu_cortexa53_0 &rarr; zynqmp_fsbl &rarr; 보드 지원 패키지 &rarr; 독립형_으로 이동합니다.
      5. 다음과 같이 구성을 수정합니다.
         + 독립형_stdin: `psu_uart_1`
         + 독립형_stdout: `psu_uart_1`
      6. _1-wire_plat &rarr; psu_pmu_0 &rarr; zynqmp_pmufw &rarr; 보드 지원 패키지 &rarr; 독립형_으로 이동합니다.
      7. 다음과 같이 구성을 수정합니다.
         + 독립형_stdin: `psu_uart_1`
         + 독립형_stdout: `psu_uart_1`
   5. 플랫폼을 구축하세요.
      1. _Flow_ 창에서 *1-wire_plat* 구성 요소를 선택하고 **Build**를 클릭합니다.

<a id="test-the-ip-using-xsct"></a>
### XSCT를 사용하여 IP 테스트

이 튜토리얼의 첫 부분에서 언급했듯이 1-Wire 프로토콜과 AXI 프로토콜을 동시에 시뮬레이션하는 것은 복잡하기 때문에 1-Wire 코어를 시뮬레이션하는 것은 어렵습니다. 이제 AXI 레지스터를 직접 읽고 쓰는 방식으로 IP를 테스트합니다. 이렇게 하면 AXI 트랜잭션에 대해 생각할 필요가 없습니다.

1. 모든 것을 KD240에 연결하십시오:
   + USB Micro-B 케이블을 컴퓨터에서 KD240의 JTAG/UART에 연결합니다.
   + 1-Wire 장치를 KD240에 연결합니다. 여기서는 DS18b20 온도 센서를 사용하겠습니다.
   + 옵션: 추가 디버깅을 위해 신호를 보려면 오실로스코프를 1-Wire 소자 데이터 와이어에 연결합니다.
   + KD240에 전원 공급 장치를 연결합니다.
2. Xilinx Software Command Tool(XSCT)을 실행하고 보드를 프로그래밍합니다.
   1. 일반적으로 ```source /opt/Xilinx/Vitis/2024.2/settings64.sh```. Then, launch the XSCT by entering ```xsct```와 유사한 명령을 사용하여 Vitis 설치를 소싱합니다.
   2. xsct 콘솔에 ```connect```를 입력하여 KD240에 연결합니다.
   3. 다음을 입력하여 부팅 모드를 JTAG로 변경합니다.  

      ```bash
      targets -set -filter {name =~ "PSU"}
      mwr 0xffca0010 0x0
      mwr 0xff5e0200 0x0100
      rst -system
      ```

   4. 비트스트림을 KD240에 프로그래밍합니다.

      ```bash
      targets -set -filter {name =~ "PSU"}
      fpga <working_directory>/myworkspace/1-wire_plat/export/1-wire_plat/hw/sdt/mydesign_wrapper.bit
      mwr 0xffca0038 0x1FF
      ```

   5. PMU 프로그래밍:

      ```bash
      targets -set -filter {name =~ "MicroBlaze PMU"}
      dow <working_directory>/myworkspace/1-wire_plat/export/1-wire_plat/sw/boot/pmufw.elf
      con
      ```

   6. FSBL 프로그래밍:

      ```bash
      targets -set -filter {name =~ "Cortex-A53 #0"}
      rst -processor -clear-registers
      dow <working_directory>/myworkspace/1-wire_plat/export/1-wire_plat/sw/boot/fsbl.elf
      con
      after 10000
      stop
      ```

3. IP를 테스트합니다.
   1. 레지스터를 읽어 장치가 제대로 프로그래밍되었는지 확인합니다.
      > **참고:** Vivado 프로젝트로 돌아가서 1-Wire AXI IP의 기본 주소를 찾을 수 있습니다. 당신에게는 _0xA0000000_입니다.

      ```bash
      mrd -force 0xA0000018
         0x76000001
      mrd -force 0xA000001C
         0x10EE4453
      ```

   2. 초기화 시퀀스를 실행합니다.

      ```bash
      # Reset the 1-Wire IP
      mwr -force 0xA0000004 0x80000000
      # Read the signal register
      mrd -force 0xA000000C
         # 0x00000010 indicate the IP is in the ready state
      # Write the initialization command
      mwr -force 0xA0000000 0x00000800
      # Write the Go signal
      mwr -force 0xA0000004 0x00000001
      # Read the signal register
      mrd -force 0xA000000C
         # 0x00000001 indicate that the IP is done and no error were detected
      ```

      완료 신호로 감지된 오류 없음은 존재 오류가 없어 IP가 버스에서 장치를 감지했음을 나타냅니다. 계속해서 IP를 수동으로 테스트할 수 있지만 튜토리얼은 여기서 중지됩니다.  

      자신의 IP를 디버깅할 때 오실로스코프를 사용하거나 ILA(통합 로직 분석기) 또는 기타 도구를 추가하여 설계의 신호를 모니터링하고 예상대로 작동하지 않는 것이 무엇인지 확인할 수 있습니다. 설계 과정에서, 심지어 패키징하기 전이라도 가능한 한 빨리 코어 디버깅을 시작하는 것이 좋습니다.

<a id="developing-drivers-with-vitis"></a>
### Vitis로 드라이버 개발하기

여기서는 테스트 및 개발을 수행하기 전에 압축된 IP에 드라이버를 통합하여 속임수를 썼습니다. 작동하는 것으로 알고 있는 IP와 기존 드라이버로 시작하기 때문에 처음부터 모든 것을 패키징할 수 있었습니다. 현실적으로 최종 결과를 얻기 전에 몇 번의 반복을 수행해야 할 수도 있습니다. 다음 단계에서는 IP와 함께 패키지된 기존 드라이버가 없는 것처럼 계속합니다.

1. Vitis 작업 공간으로 돌아가서 플랫폼을 탐색하세요.
   + _1-wire_plat &rarr; 출력 &rarr; 1-wire_plat &rarr; sw &rarr; standalone_psu_cortexa53_0 &rarr; include_ 아래의 `xparameters.h` 파일을 살펴보세요.  

      Vitis의 `xparameter.h` 파일은 컴파일 프로세스 중에 AMD 도구에 의해 생성되는 기본 주소, 인터럽트 ID 및 장치 ID와 같은 다양한 하드웨어 관련 매개변수의 정의가 포함된 헤더 파일입니다. 이 파일을 사용하면 임베디드 시스템의 장치 및 주변 장치에 쉽게 액세스하고 구성할 수 있습니다.  

      `1wire`를 검색하면 1-Wire IP의 기본 주소에 대한 매개변수를 찾을 수 있습니다.
   + _1-wire_plat &rarr; 출력 &rarr; 1-wire_plat &rarr; hw &rarr; sdt &rarr; 드라이버 &rarr; axi_1wire_host_v1_0_을 살펴보세요.  
   IP에 패키징한 1-Wire 드라이버를 찾을 수 있습니다.
2. Hello World 애플리케이션을 생성합니다.  

   첫 번째 단계는 Hello World 애플리케이션을 생성하고 대상에서 실행하여 UART와 플랫폼이 올바르게 구성되었는지 확인하는 것입니다.
   1. 시작 탭으로 돌아가서 **예**를 클릭합니다.
   2. 예제 탭의 **임베디드 소프트웨어 예제**에서 **Hello World**를 선택합니다.
   3. Hello World 탭에서 **템플릿에서 애플리케이션 구성 요소 생성**을 클릭합니다.
   4. 애플리케이션을 생성합니다:
      1. 이름 및 위치:
         + 부품명 : `hello_world`
         + 구성 요소 위치: `<working_directory>/myworkspace`
      2. 하드웨어:
         + **1-wire_plat** 플랫폼을 선택합니다.
      3. 도메인:
         + **standalone_psu_cortexa53_0** 도메인을 선택합니다.
   5. Hello World 애플리케이션을 빌드합니다.
      + **Flow** 아래에서 **Build**를 클릭하고 **hello_world**가 선택된 구성요소인지 확인하세요.
   6. 애플리케이션 실행
      1. 이전에 XSCT에서 수행한 단계를 다시 실행하여 부트 모드를 JTAG로 구성하고 비트스트림, PMU 및 FSBL을 프로그래밍합니다.
      2. 응용 프로그램을 로드합니다:

         ```bash
         dow <working_directory/myworkspace/hello_world/build/hello_world.elf>
         con
         ```

         직렬 터미널에 _Hello World 애플리케이션을 성공적으로 실행했습니다_라는 메시지가 표시되어야 합니다.
3. 드라이버를 생성하고 개발합니다.

   애플리케이션을 사용하여 드라이버를 개발하면 IP를 다시 압축하고, 비트스트림을 재생성하고, 플랫폼을 재구축하는 대신 여기에서 직접 변경할 수 있으므로 프로세스가 단순화됩니다. 여기서는 애플리케이션 수준에서 작업합니다. 베어메탈 세계에서는 Linux 세계의 경우와 달리 애플리케이션 수준과 플랫폼 수준 사이에 실제로 차이가 없습니다. 그렇기 때문에 Linux에 존재하는 다양한 계층을 고려할 필요가 없으므로 베어메탈 세계에서 드라이버 개발을 시작하는 것이 항상 권장됩니다.  

   여기에서는 Hello World 애플리케이션으로 시작하고 드라이버 기능을 추가합니다.  
   Vivado IP Packager가 자동으로 생성한 헤더 파일에는 두 가지 기능이 정의되어 있습니다. 이는 1-Wire IP 레지스터를 읽고 쓰는 데 사용됩니다.

      ```C
      #define AXI_1WIRE_HOST_mWriteReg(BaseAddress, RegOffset, Data) \
         Xil_Out32((BaseAddress) + (RegOffset), (u32)(Data))
      #define AXI_1WIRE_HOST_mReadReg(BaseAddress, RegOffset) \
         Xil_In32((BaseAddress) + (RegOffset))
      ```

   1. 소스 및 헤더 드라이버 파일을 생성합니다.
      1. **WorkSpace &rarr; hello_world &rarr; Sources &rarr; src**를 마우스 오른쪽 버튼으로 클릭하고 **새 파일**을 선택합니다.
      2. 이름을 `w1_driver_dev.c`로 지정합니다.
         + 다음을 포함합니다:

            ```C
               #include "axi_1wire_host.h"
               #include "w1_driver_dev.h
            ```

      3. 두 번째 항목을 만들고 이름을 `w1_driver_dev.h`로 지정합니다.
         + 파일에 다음을 추가합니다.

            ```C
               #ifndef W1_DRIVER_DEV_H
               #define W1_DRIVER_DEV_H
               #include "xil_types.h"
               #include "xstatus.h"
               #endif
             ```

   2. 초기화 시퀀스를 실행하는 함수를 만듭니다.
      1. `w1_driver_dev.h`에서 ```#endif``` 문 앞에 초기화 함수를 선언합니다.

         ```C
         u8 w1_resetbus(u32 baseaddr);
         ```

      2. `w1_driver_dev.c`에서 초기화 함수를 정의합니다.

         ```C
         u8 w1_resetbus(u32 baseaddr) {
            u8 val = 0;

            /* Reset 1-wire Axi IP */
            AXI_1WIRE_HOST_mWriteReg(baseaddr, 0x4, 0x80000000);

            /* Wait for READY signal to be 1 to ensure 1-wire IP is ready */
            while((AXI_1WIRE_HOST_mReadReg(baseaddr, 0xC) & 0x00000010) == 0){}

            /* Write Initialization command in instruction register */
            AXI_1WIRE_HOST_mWriteReg(baseaddr, 0x0, 0x0800);

            /* Write Go signal and clear control reset signal in control register */
            AXI_1WIRE_HOST_mWriteReg(baseaddr, 0x4, 0x00000001);

            /* Wait for done signal to be 1 */
            while((AXI_1WIRE_HOST_mReadReg(baseaddr, 0xC) & 0x00000001) == 0){}

            /* Retrieve MSB bit in status register to get failure bit */
            if ((AXI_1WIRE_HOST_mReadReg(baseaddr, 0xC) & 0x80000000) != 0)
               val = 1;

            /* Clear Go signal in register 1 */
            AXI_1WIRE_HOST_mWriteReg(baseaddr, 0x4, 0x00000000);

            return val;
         }
         ```

         기본적으로 XSCT에서 수동으로 수행한 작업을 실행하는 함수를 작성하고 있습니다.
      3. 초기화 기능을 테스트합니다.
         1. hello world 파일(`helloworld.c`)에 드라이버 테스트 헤더 파일과 xparameter를 포함합니다.

            ```C
               #include "w1_driver_dev.h"
               #include "xparameter
            ```

         2. hello world 애플리케이션에서 초기화 함수를 호출합니다.

            ```C
               w1_resetbus(XPAR_AXI_1WIRE_HOST_0_BASEADDR);
            ```

         3. 애플리케이션 실행
            1. 이전에 XSCT에서 수행한 단계를 다시 실행하여 부트 모드를 JTAG로 구성하고 비트스트림, PMU, FSBL 및 hello world 애플리케이션을 프로그래밍합니다.
            2. 레지스터를 읽어 예상된 값이 있는지 확인합니다.

               ```bash
               mrd -force 0xA0000000
                  # 0x00000800
               mrd -force 0xA0000004
                  # 0x00000000
               mrd -force 0xA0000008
                  # 0x00000000
               mrd -force 0xA000000C
                  # 0x00000010
               mrd -force 0xA0000010
                  # 0x00000000
               mrd -force 0xA0000014
                  # 0x00000001
               mrd -force 0xA0000018
                  # 0x76000001
               mrd -force 0xA000001C
                  # 0x10EE4453
               ```

      4. 이제 기능을 테스트하고 검증했으므로 IP와 함께 패키징하려는 드라이버에 해당 기능을 추가할 수 있습니다.
   3. 드라이버로 구현하려는 모든 기능에 대해 동일한 과정을 반복하십시오. 드라이버의 모든 기능을 IP에 통합한 후에는 다시 패키징할 수 있습니다. _검토 및 패키지_ 탭으로 이동하여 **IP 재패키지**를 클릭하세요.  
   변경 사항을 반영하고 비트스트림을 재생성하려면 Vivado 프로젝트를 업데이트해야 합니다. 비트스트림이 생성되면 Vitis 플랫폼의 하드웨어 업데이트를 내보낼 수 있습니다.

<a id="test-a-baremetal-application"></a>
### 베어메탈 애플리케이션 테스트

DS18b20 온도 센서를 사용하여 온도를 모니터링하는 애플리케이션을 생성하게 됩니다. 원래 IP를 적절한 드라이버와 함께 패키지했으므로 이전에 생성한 것과 동일한 Vitis 프로젝트를 사용하게 됩니다.

1. 빈 애플리케이션을 생성합니다.

   Vitis의 시작 탭으로 돌아가서 임베디드 개발 아래에서 **임베디드 애플리케이션 생성...**을 선택하고 **빈 임베디드 애플리케이션 생성**을 선택합니다.
   1. 이름 및 위치:
      + 부품명 : `1-wire_app`
      + 구성 요소 위치: `<working_directory>/myworkspace`
   2. 하드웨어: **1-wire_plat** 플랫폼을 선택합니다.
   3. 도메인: **standalone_psu_cortexa53_0** 도메인을 선택합니다.
   4. 소스 파일: 아직 추가할 파일이 없습니다.
2. 소스를 애플리케이션으로 가져옵니다.
   1. _myworkspace_에서 **1-wire_app/Sources/src**를 마우스 오른쪽 버튼으로 클릭합니다.
   2. **&rarr; 파일 가져오기...**를 선택합니다.
   3. _<working_directory>/reference_files/application_으로 이동하여 **application_bm.c** 및 **application_bm.h**를 선택합니다.
   4. 두 파일을 모두 검토합니다. 보시다시피 IP에서 개발 및 패키지된 드라이버 기능을 활용하여 센서에서 온도를 구성하고 읽는 애플리케이션을 볼 수 있습니다.
3. 애플리케이션을 시작하기 위한 최상위 소스 파일을 생성합니다.
   1. _myworkspace_에서 **1-wire_app/Sources/src**를 마우스 오른쪽 버튼으로 클릭합니다.
   2. **새 파일**을 선택합니다.
   3. 이름을 `main.c`로 지정합니다.
   4. `main.c`에 다음을 추가합니다.

      ```C
      #include "application_bm.h"
      #include "axi_1wire_host.h"
      #include "xparameters.h"

      int main()
      {
         s8 t_high = 120;
         s8 t_low = -6;
         int resolution = 12;

         AXI_1WIRE_HOST_SelfTest(XPAR_AXI_1WIRE_HOST_0_BASEADDR);
         continuous_temperature_reading(t_high, t_low, resolution);
         return 0;
      }
      ```

4. 애플리케이션을 빌드합니다.
   1. _Flow_ 창에서 _1-wire_app_ 구성 요소가 선택되어 있는지 확인하세요.
   2. **빌드**를 클릭하고 애플리케이션이 오류 없이 빌드되었는지 확인합니다.
5. 애플리케이션을 실행합니다.
   1. XSCT의 단계를 반복하여 부트 모드를 JTAG로 구성하고 비트스트림, PMU 및 FSBL을 보드에 프로그래밍합니다.

      ```bash
      # Change bootmode
      targets -set -filter {name =~ "PSU"}
      mwr 0xffca0010 0x0
      mwr 0xff5e0200 0x0100
      rst -system
      # Load the bitstream
      targets -set -filter {name =~ "PSU"}
      fpga <working_directory>/myworkspace/1-wire_plat/export/1-wire_plat/hw/sdt/mydesign_wrapper.bit
      mwr 0xffca0038 0x1FF
      # Program the PMU
      targets -set -filter {name =~ "MicroBlaze PMU"}
      dow <working_directory>/myworkspace/1-wire_plat/export/1-wire_plat/sw/boot/pmufw.elf
      con
      # Program the FSBL
      targets -set -filter {name =~ "Cortex-A53 #0"}
      rst -processor -clear-registers
      dow <working_directory>/myworkspace/1-wire_plat/export/1-wire_plat/sw/boot/fsbl.elf
      con
      stop
      ```

   2. 애플리케이션을 로드합니다:

      ```bash
      dow <working_directory>/myworkspace/1-wire_app/build/1-wire_app.elf
      con
      ```

   3. 직렬 콘솔에 다음과 유사한 내용이 표시됩니다.

      ```
      ******************************
      * AXI 1-Wire Host Self Test
      * Reading IP ID and IP version
      * IP Subsystem vendor ID is 0x10EE
      * ID is 0x4453
      * IP version is 0.1
      ******************************

      Configuration done
      Temperature is: 19.3125
      ```

      물론 온도는 다를 수 있습니다.

   ---

   이제 온도를 표시하는 애플리케이션으로 테스트된 베어메탈 드라이버와 함께 패키지된 IP가 생겼습니다. 인터럽트는 베어메탈 플랫폼에서 처리하기 어렵고 사용되는 프로세서와 긴밀하게 연결되어 있으므로 아직 인터럽트를 통합하지 않았습니다. 인터럽트는 Linux에서 쉽게 관리되므로 다음 섹션에서 사용됩니다. 베어메탈 시스템에 인터럽트를 통합하는 것은 불가능하지 않지만 이는 이 튜토리얼의 범위를 벗어납니다.

   Vitis 통합 소프트웨어 플랫폼을 사용한 임베디드 디자인에 대한 자세한 내용은 _Vitis를 사용한 임베디드 디자인 개발 사용자 가이드_[(UG1701)](https://docs.amd.com/r/en-US/ug1701-vitis-accelerated-embedded)를 참조하세요.  
   Vitis 통합 소프트웨어 플랫폼을 사용한 임베디드 소프트웨어 개발에 대한 자세한 내용은 _Vitis 통합 소프트웨어 플랫폼 문서: 임베디드 소프트웨어 개발_ [(UG1400)](https://docs.amd.com/r/en-US/ug1400-vitis-embedded/Getting-Started-with-Vitis)을 참조하세요.  
   Vitis 통합 소프트웨어 플랫폼을 사용한 애플리케이션 가속 개발에 대한 자세한 내용은 _Vitis 통합 소프트웨어 플랫폼 문서: 애플리케이션 가속 개발_ [(UG1393)](https://docs.amd.com/r/en-US/ug1393-vitis-application-acceleration)을 참조하세요.

<p align="center"><b>다음 단계: <a href="./3_linux_driver.kr.md"> Linux 드라이버 개발</a></b></p>

---
<p class="sphinxhide" align="center"><sub>저작권 © 2025 Advanced Micro Devices, Inc.</sub></p>
<p class="sphinxhide" align="center"><sup><a href="https://www.amd.com/en/corporate/copyright">이용 약관</a></sup></p>
