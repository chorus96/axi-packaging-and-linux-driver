<table class="sphinxhide" width="100%">
 <tr width="100%">
    <td align="center"><img src="https://raw.githubusercontent.com/Xilinx/Image-Collateral/main/xilinx-logo.png" width="30%"/><h1>AXI 패키징 및 Linux 드라이버 튜토리얼로 사용자 지정 IP 래핑</h1>
    </td>
 </tr>
</table>

<a id="axi-ip-packaging"></a>
# AXI IP 패키징

튜토리얼의 이 섹션에서는 AMD Vivado&trade; Design Suite를 사용하여 AXI IP를 패키징하는 필수 단계를 다룹니다.

<a id="outline"></a>
## 개요

1. [소개](#소개)
    1. [AXI 프로토콜](#the-axi-protocol)
    2. [IP의 기본](#the-basics-of-an-ip)
2. [Vivado를 사용하여 사용자 정의 IP 패키징](#using-vivado-to-package-your-custom-ip)
    1. [Vivado IP Packager를 사용하여 IP 프로젝트 생성](#1-creating-an-ip-project-with-the-vivado-ip-packager)
    2. [IP HDL 편집](#2-editing-the-ip-hdl)
    3. [IP 패키지 편집](#3-editing-the-ip-package)
3. [IP 암호화](#encrypting-your-ip)

<a id="introduction"></a>
## 소개

<a id="the-axi-protocol"></a>
### AXI 프로토콜

IP(지적 재산) 코어용 AXI(Advanced eXtensible Interface) 프로토콜은 AXI 마스터와 AXI 슬레이브 간의 정보 교환 방식을 표준화하기 위해 온칩 통신 버스 프로토콜을 정의했습니다. IP는 의도한 애플리케이션에 따라 슬레이브가 될 수도 있고 마스터가 될 수도 있습니다. 프로세서에는 AXI 인터페이스도 있을 수 있습니다. 따라서 IP와 프로세서는 AXI 프로토콜을 통해 정보를 교환할 수 있습니다. AXI는 Arm AMBA(Advanced Microcontroller Bus Architecture)의 일부이며 시스템 온 칩 회로의 블록에 대한 온칩 상호 연결 사양을 정의합니다.

AMD-Xilinx는 2010년에 AMBA 4.0 릴리스를 기반으로 한 설계 도구에 이 인터페이스를 도입했습니다. 그 이후로 AXI는 AMD-Xilinx 제품에 널리 채택되었습니다. 세 가지 AXI4 인터페이스가 정의되어 있습니다.

+ **AXI4**: 고성능 메모리 매핑 요구 사항에 적합합니다.
+ **AXI4-Lite**: 간단하고 처리량이 적은 메모리 매핑 통신용입니다.
+ **AXI4-Stream:** 고속 스트리밍 데이터용.

AXI 프로토콜의 이점은 다음과 같습니다.

+ **생산성**: AXI 인터페이스 표준화를 통해 개발자는 IP용 단일 프로토콜만 학습하면 됩니다.
+ **유연성**: 애플리케이션에 적합한 프로토콜 제공:

  + AXI4는 메모리 매핑 인터페이스용이며 단일 주소 단계만으로 최대 256 데이터 전송 주기의 높은 처리량 버스트를 허용합니다.
    + AXI4-Lite는 경량의 단일 트랜잭션 메모리 매핑 인터페이스입니다.
    + AXI4-Stream은 주소 단계에 대한 요구 사항을 완전히 제거하고 무제한 데이터 버스트 크기를 허용합니다.
+ **가용성**: 업계 표준으로 전환하면 Vivado IP 카탈로그뿐만 아니라 전 세계 Arm 파트너 커뮤니티에도 액세스할 수 있습니다.

AXI 프로토콜은 각각 고유한 핸드셰이킹 신호를 갖는 5개의 독립 채널을 설명합니다.

+ 주소 채널 읽기
+ 주소 채널 쓰기
+ 데이터 채널 읽기
+ 데이터 채널 쓰기
+ 응답 채널 쓰기

다음 그림은 AXI4 읽기 트랜잭션을 보여줍니다.  
![읽기의 채널 아키텍처](./images/readChannelArch.png)

다음 그림은 AXI4 쓰기 트랜잭션을 보여줍니다.
![쓰기 채널 아키텍처](./images/writeChannelArch.png)

AXI 프로토콜에 대한 자세한 내용은 *AXI 참조 가이드* [(UG1037)](https://docs.amd.com/v/u/en-US/ug1037-vivado-axi-reference-guide)를 참조하세요.

<a id="the-basics-of-an-ip"></a>
### IP의 기본

IP는 SoC(시스템 온 칩) 하드웨어 설계, 특히 빌딩 블록 설계의 핵심 구성 요소입니다. 설계 빌딩 블록은 더 큰 아키텍처에 통합될 수 있는 SoC의 사전 설계되고 재사용 가능한 부분을 나타내며, 개발 프로세스를 간소화하고 비용을 최소화하며 출시 기간을 단축합니다. 이러한 빌딩 블록은 기능 장치, 논리 회로 및 맞춤형 프로세서 코어와 같은 요소를 포함하여 SoC 설계를 뒷받침하는 지적 재산을 형성합니다.

IP 빌딩 블록을 사용하면 입증된 설계 구성 요소를 재사용하여 모든 새 프로젝트에 대해 "바퀴를 재발명"할 필요가 없으므로 기업은 혁신에 집중할 수 있습니다. 사전 검증된 IP 설계 빌딩 블록을 사용함으로써 SoC 설계자는 설계 위험을 줄이고 최종 제품의 신뢰성을 향상시켜 업계 내 혁신을 지속적으로 가속화하는 이점을 누릴 수 있습니다. 이러한 디자인 빌딩 블록을 보호하는 것은 기업이 경쟁 우위를 유지하고 해당 분야에 대한 기여가 적절하게 인정되고 보상되도록 하는 데 필수적입니다.

기업은 설계 빌딩 블록을 다른 기업에 라이센스하여 특정 목적과 사전 정의된 조건에 따라 기술을 활용할 수 있습니다. 이 라이선싱 시스템은 혁신 공유와 경쟁 우위 보호를 모두 가능하게 하여 지적 재산 관리에 대한 균형 잡힌 접근 방식을 제공합니다.

SoC(시스템 온 칩) 설계에서 IP 설계 빌딩 블록을 활용하는 중요한 측면은 서로 다른 IP와 시스템 버스 간의 원활한 통신을 보장하는 것입니다. AMD FPGA 및 SoC의 맥락에서 AXI는 다양한 IP 간의 상호 연결을 용이하게 하는 업계 표준 프로토콜 역할을 합니다. Vivado Design Suite는 AXI 래퍼를 사용하여 IP 코어 간의 호환성과 원활한 통신을 달성함으로써 사용자 친화적인 접근 방식을 제공합니다.

AXI 래퍼는 기본적으로 IP 코어를 캡슐화하고 SoC 설계의 시스템 버스 및 다른 IP 코어와 통신할 수 있는 AXI 호환 인터페이스를 제공합니다. Vivado를 사용하면 설계자는 AXI 래퍼의 생성 또는 수정을 가능하게 하는 IP 통합기와 같은 내장 도구를 활용하여 일관되고 효율적인 IP 코어 통합을 보장할 수 있습니다. 표준 IP 블록에 대한 기존 AXI 래퍼 중에서 선택하거나 IP의 고유한 요구 사항에 맞는 사용자 정의 AXI 래퍼를 생성할 수 있습니다.

Vivado의 IP 패키저는 캡슐화된 코어와 함께 보충 파일을 제공하여 SoC 프로젝트에서 설계 빌딩 블록의 유용성을 더욱 향상시킵니다. 시뮬레이션 파일, 드라이버 파일, 제약 조건 파일 등을 포함하는 이러한 추가 파일은 설계자에게 개발 프로세스의 통합 문제를 완화하는 포괄적인 패키지를 제공합니다. 예를 들어, 시뮬레이션 파일을 사용하면 설계자는 통합 전에 IP 성능을 검증하고 개선할 수 있으며, 드라이버 파일에는 시스템과의 적절한 기능 및 인터페이스를 보장하는 필수 소프트웨어 구성 요소가 포함되어 있습니다. 반면에 제약 조건 파일을 사용하면 설계자는 핀 할당, 전압 레벨, 클록 요구 사항과 같은 필수 매개 변수를 지정하고 관리할 수 있습니다.

IP 코어에 대한 자세한 내용은 다음을 참조하세요.

+ *Vivado Design Suite 사용자 가이드: IP를 사용한 설계* [(UG896)](https://docs.amd.com/r/en-US/ug896-vivado-ip)
+ *Vivado Design Suite 사용자 가이드: IP 통합자를 사용한 IP 하위 시스템 설계* [(UG994)](https://docs.amd.com/r/en-US/ug994-vivado-ip-subsystems)
+ *Vivado Design Suite 사용자 가이드: 맞춤형 IP 생성 및 패키징* [(UG1118)](https://docs.amd.com/r/en-US/ug1118-vivado-creating-packaging-custom-ip)

<a id="using-vivado-to-package-your-custom-ip"></a>
## Vivado를 사용하여 사용자 정의 IP 패키징

이 튜토리얼에서는 1-Wire 코어를 사용하여 맞춤형 하드웨어 코어를 AXI IP에 패키징하는 방법을 보여줍니다. 사용되는 하드웨어 설명 언어는 Verilog이지만 VHDL과 같은 다른 언어도 사용할 수 있습니다. 튜토리얼에 필요한 모든 파일은 [reference_files](./reference_files/) 폴더에 제공됩니다. 구조는 다음과 같습니다.

```
AXI-packaging-and-Linux-driver/reference_files
├── application
|   ├── application_bm.c
|   └── application_bm.h
├── baremetal_driver
|   ├── data
|   |   ├── axi_1wire_host.mdd
|   |   ├── axi_1wire_host.tcl
|   |   └── axi_1wire_host.yaml
|   └── src
|       ├── axi_1wire_host_selftest.c
|       ├── axi_1wire_host.c
|       ├── axi_1wire_host.h
|       ├── CMakeLists.txt
|       └── Makefile
├── constraints
|   └── axi_1wire_host.xdc
├── hdl
|   ├── axi_1wire_host_slave_lite_v1_2_S00_AXI.v
│   ├── axi_1wire_host.v
│   ├── clk_div.v
│   ├── jcnt.v
│   ├── sr.v
│   └── w1_master.v
└──  linux_driver
    ├── amd_axi_w1.c
    ├── w1chardev.c
    └── xlnxw1-app.c
```

튜토리얼 전체에서 *AXI-packaging-and-Linux-driver/* 폴더는 *<working_directory>*로 참조됩니다.
> **참고:** *clk_div.v*, *jcnt.v* 및 *sr.v*는 암호화된 파일입니다. 이 코어는 메인 1-Wire 코어 내의 서브코어일 뿐이므로 튜토리얼에서는 구현 방법을 아는 것이 필요하지 않습니다.

<a id="1-creating-an-ip-project-with-the-vivado-ip-packager"></a>
### 1. Vivado IP Packager를 사용하여 IP 프로젝트 생성

AXI 래퍼로 래핑할 준비가 된 기존 하드웨어가 있고 1-Wire 호스트를 사용하여 프로세스를 진행한다고 가정합니다.

1. 일반적으로 ```source /opt/Xilinx/Vivado/2024.2/settings64.sh```. Then, to launch the Vivado GUI, enter ```vivado```와 유사한 명령을 사용하여 Vivado 설치를 소싱합니다.

2. 다음 설정을 사용하여 새 프로젝트를 생성합니다.
    + 프로젝트 이름
        + 프로젝트 이름: *myproject*
        + 프로젝트 위치: *<working_directory>*
        + 프로젝트 하위 디렉터리 생성: *선택*
    + 프로젝트 유형
        + 프로젝트 유형: *RTL 프로젝트*
        + 현재는 출처를 지정하지 마세요: *체크 해제*
        + 프로젝트는 확장 가능한 Vitis 플랫폼입니다: *선택 해제*
    + 소스 추가
        + *<working_directory>/reference_files/hdl/jcnt.v*
        + *<working_directory>/reference_files/hdl/sr.v*
        + *<working_directory>/reference_files/hdl/w1_master.v*
        + RTL 포함 파일을 스캔하고 프로젝트에 추가: *선택됨*
        + 소스를 프로젝트에 복사: *체크*
        + 대상 언어: *Verilog*
        + 시뮬레이터 언어: *혼합*
    + 기본 부분
        + 보드: *Kria KD240 드라이브 스타터 키트 SOM*
            + 동반자 카드 연결 추가:
                + K24 SOM(SOM240_1)의 커넥터 1: *드라이브 스타터 키트 캐리어(SOM240_1)*
                + K24 SOM(SOM40_2)의 커넥터 2: *드라이브 스타터 키트 캐리어(SOM40_2)*
    > **참고:** 앞서 언급한 것처럼 *clk_div.v*, *jcnt.v* 및 *sr.v*는 암호화된 파일입니다. 높은 수준의 기능에서 각 모듈이 구현하는 내용은 다음과 같습니다.
    > + *clk_div.v*: 입력 클럭의 주파수를 줄이기 위해 클럭 분배기를 구현합니다.
    > + *jcnt.v*: 비동기 재설정으로 Johnson 카운터를 구현합니다.
    > + *sr.v*: 비동기식 재설정을 사용하여 순환 시프트 레지스터를 구현합니다.

    이제 하드웨어 코어를 패키징할 준비가 되었습니다.

    Vivado IDE에 대한 자세한 내용은 다음을 참조하세요.

    + *Vivado Design Suite 사용자 가이드: Vivado IDE 사용* [(UG893)](https://docs.amd.com/r/en-US/ug893-vivado-ide)
    + *Vivado Design Suite 사용자 가이드: 디자인 흐름 개요* [(UG892)](https://docs.amd.com/r/en-US/ug892-vivado-design-flows-overview)

3. Vivado에서 *Tools* &rarr; **Create and Package New IP...**로 이동합니다.
    1. 새 창에서 **다음 >**을 클릭합니다.
    2. **새 AXI4 주변 장치 만들기**를 선택하고 **다음 >**을 클릭합니다.
    3. *주변 장치 세부 정보* 창에 다음을 입력합니다.
        + 이름: *axi_1wire_host*
        + 버전: *0.1*
        + 표시 이름: *AXI 1-Wire 버스 호스트*
        + 설명: *IP 패키징을 시연하기 위한 1-Wire AXI IP*
        + IP 위치: *<working_directory>/myIP*
        + 기존 덮어쓰기: *선택 해제됨*

        **다음 >**을 클릭합니다.
    4. *인터페이스 추가* 창에서 다음을 사용합니다.
        + 인터럽트 지원 활성화: *선택 해제*
        + 1개의 인터페이스(*S00_AXI*)로 기본 설정을 유지합니다.
        + 이름: *S00_AXI*
        + 인터페이스 유형: *라이트*
        + 인터페이스 모드: *슬레이브*
        + 데이터 폭(비트): *32*
        + 레지스터 수: *8*

        다음 스크린샷과 유사하게 표시됩니다.  
        ![인터페이스 추가](./images/addInterface.png)
        > **참고:** 여기서는 슬레이브 모드에서 AXI4-Lite 인터페이스를 사용하여 AXI4 IP를 생성합니다. 슬레이브 또는 마스터 모드의 AXI4 또는 AXI4-Stream 인터페이스 프로세스는 유사합니다.  
        > IP에 필요한 레지스터 수를 파악하려면 저장해야 하는 데이터가 무엇인지 생각하고, IP ​​코어에 보내야 하는 명령이 있는 경우 등록할 신호를 중단하세요. AXI 레지스터는 IP 코어와 기타 AXI 호환 구성 요소 간의 통신 방법을 제공합니다. IP 코어의 모든 신호를 AXI 레지스터에 등록할 필요는 없으며 일부 신호는 IP의 입력 및/또는 출력으로 설정할 수 있습니다.

        **다음 >**을 클릭합니다.

    5. 마지막 창에서 **Finish**를 클릭하기 전에 **Edit IP**를 선택합니다.

<a id="2-editing-the-ip-hdl"></a>
### 2. IP HDL 편집

1. Vivado IP Packager는 두 개의 파일을 생성합니다:
    + *axi_1wire_host_slave_lite_v0_1_S00_AXI.v*: AXI 버스 인터페이스 S00_AXI용 AXI 로직 및 레지스터입니다. 이 Verilog 파일은 AXI 레지스터와 Vivado IP Packager로 생성한 AXI 인터페이스용 레지스터를 읽고 쓰는 로직을 구현하는 데 사용됩니다. 일반적으로 여기에서 IP 코어와 IP에 필요한 기타 논리를 인스턴스화합니다.
    + *axi_1wire_host.v*: AXI IP 최상위 래퍼. 이 Verilog 파일은 각 AXI 인터페이스를 인스턴스화하는 데 사용됩니다. 여기서는 1-Wire IP에 하나의 AXI 인터페이스를 사용하고 있습니다.

2. IP 프로젝트에 소스 파일을 추가합니다.
    1. **소스 추가** ![소스 추가](./images/addSources.png)를 클릭합니다.
    2. **디자인 소스 추가 또는 만들기**를 선택하고 **다음 >**을 클릭합니다.
    3. **파일 추가**를 선택하고 *<working_directory>/myproject/myproject.srcs/sources_1/imports/hdl/*로 이동합니다.
    4. 세 개의 파일(*jcnt.v*, *sr.v* 및 *w1_master.v*)을 모두 선택하고 **확인**을 클릭합니다.
    5. 다시 **파일 추가**를 선택하고 *<working_directory>/reference_files/hdl*로 이동한 후 **clk_div.v**를 선택하고 ***확인**을 클릭합니다.
    6. **RTL 포함 파일을 스캔하여 프로젝트에 추가**를 선택 취소하고 **IP 디렉토리에 소스 복사**를 선택된 상태로 유지합니다.
        > **참고**  
        > *clk_div*는 1-Wire 코어의 일부는 아니지만 코어를 구동하기 위해 더 느린 클록을 생성하는 데 사용됩니다. 코어 클럭이 AMD 클럭킹 마법사 IP 범위에 있는 경우 AMD 클럭킹 마법사 IP를 사용할 수 있지만 1-Wire 프로토콜에는 제공 가능한 것보다 느린 클럭이 필요하므로 사용자 지정 클럭을 사용하고 있습니다.  

3. AXI 로직 및 레지스터 래퍼 *axi_1wire_host_slave_lite_v0_1_S00_AXI.v*를 편집합니다.
    > *<working_directory>/reference_files/hdl/axi_1wire_host_slave_lite_v1_2_S00_AXI.v*가 참조로 제공됩니다.
    1. 모듈 인스턴스화
        + 345행의 *// 여기에 사용자 로직 추가* 아래에서 클록 분할기를 인스턴스화합니다.

            ```verilog
            CLK_DIVIDER #(.DIVIDER(CLK_DIV_VAL_TO_1MHz)) CLK_DIVIDER(
                .areset(S_AXI_ARESETN),
                .clk_in(S_AXI_ACLK),
                .clk_out(clk_1MHz)
            );
            ```

        + 클록 분배기 다음에 1-Wire 코어를 인스턴스화합니다.

            ```verilog
            W1_MASTER W1_MASTER(
                .clk_1MHz(clk_1MHz),
                .areset(S_AXI_ARESETN),
                .ctrl_reset(ctrl_reset),
                .go(go),
                .command(command),
                .tx_data(tx_data),
                .from_dq(from_dq),
                .dq_ctrl(dq_ctrl_master),
                .dq_out(dq_out_master),
                .done(done),
                .ready(ready),
                .reg_wr(reg_wr),
                .failure(failure),
                .data_out(rx_data)
            );
            ```

        + 1-Wire 버스는 입력/출력 버스이므로 입력/출력 버스를 제어하려면 IOBUF가 필요합니다. 1-Wire 코어 다음에 인스턴스화합니다.

            ```verilog
            IOBUF IOBUF1(
                .O(from_dq), 
                .IO(w1_bus), 
                .I((dq_out_gpio & master_gpio_sel) | (dq_out_master & !master_gpio_sel)), 
                .T((dq_ctrl_gpio & master_gpio_sel) | (dq_ctrl_master & !master_gpio_sel))
            );
            ```

    2. 포트와 매개변수를 추가합니다.
        + *// 사용자 아래에 클럭 분배기 매개변수를 추가하여 7행의 여기에* 매개변수를 추가합니다.

            ```verilog
            parameter integer CLK_DIV_VAL_TO_1MHz = 100,
            ```

        + *// 사용자 아래에 인터럽트 및 1-Wire 버스 신호를 추가하여 라인 18의 여기에 포트를 추가합니다.

            ```verilog
            inout w1_bus,
		    output reg w1_irq,
            ```

    3. 사용자 신호를 추가합니다.
        + 라인 84의 AXI 레지스터와 로직에 1선 코어를 연결하는 와이어를 추가합니다.

            ```verilog
            // User signals
            wire 		clk_1MHz;
            wire 		ctrl_reset;
            wire 		go;
            wire        ready_irq_en;
            wire        done_irq_en;
            wire [3:0] 	command;
            wire [7:0]	tx_data;
            
            wire		done;
            wire        ready;
            wire		reg_wr;
            wire		failure;
            wire [7:0]	rx_data;

            wire		from_dq;
            wire        dq_ctrl_master;	// 1 to read, 0 to write to the 1 wire bus
            wire 		dq_out_master;
            wire		dq_ctrl_gpio;	// 1 to read, 0 to write to the 1 wire bus
            wire		dq_out_gpio;
            wire		master_gpio_sel;	// 0 to use the 1-wire master, 1 to use the gpio to drive the 1 wire bus
            ```

    4. AXI 레지스터 및 프로토콜 로직을 검토합니다.
        + 128~135행에서 AXI 레지스터가 선언됩니다.
            <details>
                <summary>라인 128 ~ 135</summary>

                reg [C_S_AXI_DATA_WIDTH-1:0] slv_reg0;
                reg [C_S_AXI_DATA_WIDTH-1:0] slv_reg1;
                reg [C_S_AXI_DATA_WIDTH-1:0] slv_reg2;
                reg [C_S_AXI_DATA_WIDTH-1:0] slv_reg3;
                reg [C_S_AXI_DATA_WIDTH-1:0] slv_reg4;
                reg [C_S_AXI_DATA_WIDTH-1:0] slv_reg5;
                reg [C_S_AXI_DATA_WIDTH-1:0] slv_reg6;
                reg [C_S_AXI_DATA_WIDTH-1:0] slv_reg7;
                
            </details>
        + 154~218행과 318~363행에서는 AXI 상태 기계에 대한 쓰기 및 읽기 상태 기계가 구현됩니다. 이는 AXI 프로토콜을 구현하는 데 사용됩니다. AXI 프로토콜은 표준이고 수정으로 인해 예기치 않은 AXI 트랜잭션 동작이 발생할 수 있으므로 일반적으로 코드의 이 부분을 편집하지 않습니다. 튜토리얼을 계속하기 전에 시간을 내어 논리를 검토하고 AXI 프로토콜 구현을 더 잘 이해하십시오.
            <details>
                <summary>라인 154 ~ 218 - 상태 머신 쓰기</summary>

                항상 @(posedge S_AXI_ACLK)
                    시작하다                                 
                        if (S_AXI_ARESETN == 1'b0)                                 
                        시작하다                                 
                            axi_awready <= 0;                                 
                            axi_wready <= 0;                                 
                            axi_bvalid <= 0;                                 
                            axi_bresp <= 0;                                 
                            axi_awaddr <= 0;                                 
                            state_write <= 유휴;                                 
                        끝                                 
                        그렇지 않으면                                  
                        시작하다                                 
                            케이스(상태_쓰기)                                 
                            유휴:                                      
                                시작하다                                 
                                if(S_AXI_ARESETN == 1'b1)                                  
                                    시작하다                                 
                                    axi_awready <= 1'b1;                                 
                                    axi_wready <= 1'b1;                                 
                                    state_write <= Waddr;                                 
                                    끝                                 
                                그렇지 않으면 state_write <= state_write;                                 
                                끝                                 
                            Waddr: //이 상태에서 슬레이브는 해당 제어 신호 및 첫 번째 데이터 패킷과 함께 주소를 수신할 준비가 되어 있습니다. 유효한 응답도 이 상태에서 처리됩니다.                                 
                                시작하다                                 
                                if (S_AXI_AWVALID && S_AXI_AWREADY)                                 
                                    시작하다                                 
                                        axi_awaddr <= S_AXI_AWADDR;                                 
                                        if(S_AXI_WVALID)                                  
                                        시작하다                                   
                                            axi_awready <= 1'b1;                                 
                                            state_write <= Waddr;                                 
                                            axi_bvalid <= 1'b1;                                 
                                        끝                                 
                                        그렇지 않으면                                  
                                        시작하다                                 
                                            axi_awready <= 1'b0;                                 
                                            state_write <= Wdata;                                 
                                            if (S_AXI_BREADY && axi_bvalid) axi_bvalid <= 1'b0;                                 
                                        끝                                 
                                    끝                                 
                                그렇지 않으면                                  
                                    시작하다                                 
                                        상태_쓰기 <= 상태_쓰기;                                 
                                        if (S_AXI_BREADY && axi_bvalid) axi_bvalid <= 1'b0;                                 
                                    끝                                 
                                끝                                 
                            Wdata: //이 상태에서 슬레이브는 전송 횟수가 버스트 길이와 같아질 때까지 데이터 패킷을 수신할 준비가 되어 있습니다.                                 
                                시작하다                                 
                                만약 (S_AXI_WVALID)                                 
                                    시작하다                                 
                                    state_write <= Waddr;                                 
                                    axi_bvalid <= 1'b1;                                 
                                    axi_awready <= 1'b1;                                 
                                    끝                                 
                                    그렇지 않으면                                  
                                    시작하다                                 
                                    상태_쓰기 <= 상태_쓰기;                                 
                                    if (S_AXI_BREADY && axi_bvalid) axi_bvalid <= 1'b0;                                 
                                    끝                                              
                                끝                                 
                            엔드케이스                                 
                            끝                                 
                        끝

            </details>
            <details>
                <summary>라인 318 ~ 363 - 상태 머신 읽기</summary>

                항상 @(posedge S_AXI_ACLK)
                    시작하다                                       
                        if (S_AXI_ARESETN == 1'b0)                                       
                            시작하다                                       
                            //리셋하는 동안 초기값을 모두 0으로 지정                                       
                            axi_arready <= 1'b0;                                       
                            axi_rvalid <= 1'b0;                                       
                            axi_rresp <= 1'b0;                                       
                            state_read <= 유휴;                                       
                            끝                                       
                        그렇지 않으면                                       
                            시작하다                                       
                            케이스(상태_읽기)                                       
                                유휴: //재설정이 완료되었으며 읽기/쓰기 트랜잭션을 수신할 준비가 되었음을 나타내는 초기 상태                                       
                                시작하다                                                
                                    if (S_AXI_ARESETN == 1'b1)                                        
                                    시작하다                                       
                                        state_read <= Raddr;                                       
                                        axi_arready <= 1'b1;                                       
                                    끝                                       
                                    else state_read <= state_read;                                       
                                끝                                       
                                Raddr: //이 상태에서 슬레이브는 해당 제어 신호와 함께 주소를 수신할 준비가 되었습니다.                                       
                                시작하다                                       
                                    if (S_AXI_ARVALID && S_AXI_ARREADY)                                       
                                    시작하다                                       
                                        state_read <= Rdata;                                       
                                        axi_araddr <= S_AXI_ARADDR;                                       
                                        axi_rvalid <= 1'b1;                                       
                                        axi_arready <= 1'b0;                                       
                                    끝                                       
                                    else state_read <= state_read;                                       
                                끝                                       
                                Rdata: //이 상태에서 슬레이브는 전송 횟수가 버스트 길이와 같아질 때까지 데이터 패킷을 보낼 준비가 되어 있습니다.                                       
                                시작하다                                           
                                    if (S_AXI_RVALID && S_AXI_RREADY)                                       
                                    시작하다                                       
                                        axi_rvalid <= 1'b0;                                       
                                        axi_arready <= 1'b1;                                       
                                        state_read <= Raddr;                                       
                                    끝                                       
                                    else state_read <= state_read;                                       
                                끝                                       
                            엔드케이스                                       
                            끝                                       
                        끝

            </details>

        + 365번 라인에서는 읽기 트랜잭션을 위한 로직이 구현됩니다. 필요한 경우 사용자 로직을 추가할 수 있습니다. 이 코어의 경우 아무것도 필요하지 않지만, 예를 들어 AXI 마스터와 코어에서 동시 읽기를 방지해야 하는 경우 여기에 필요한 로직을 구현합니다.
            <details>
                <summary>Line 365 - 트랜잭션 로직 읽기</summary>

                S_AXI_RDATA = (axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] == 3'h0) 할당 ? slv_reg0 : (axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] == 3'h1) ? slv_reg1 : (axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] == 3'h2) ? slv_reg2 : (axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] == 3'h3) ? slv_reg3 : (axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] == 3'h4) ? slv_reg4 : (axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] == 3'h5) ? slv_reg5 : (axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] == 3'h6) ? slv_reg6 : (axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] == 3'h7) ? slv_reg7 : 0;
            </details>

        + 229행부터 315행까지 쓰기 트랜잭션에 대한 논리가 구현됩니다.
            <details>
                <summary>라인 229~315 - 트랜잭션 로직 쓰기</summary>

                항상 @( posedge S_AXI_ACLK )
                    시작하다
                    if ( S_AXI_ARESETN == 1'b0 )
                        시작하다
                        slv_reg0 <= 0;
                        slv_reg1 <= 0;
                        slv_reg2 <= 0;
                        slv_reg3 <= 0;
                        slv_reg4 <= 0;
                        slv_reg5 <= 0;
                        slv_reg6 <= 0;
                        slv_reg7 <= 0;
                        끝 
                    그렇지 않으면 시작하다
                        만약 (S_AXI_WVALID)
                        시작하다
                            사례( (S_AXI_AWVALID) ? S_AXI_AWADDR[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] : axi_awaddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] )
                            3시 0분:
                                for ( byte_index = 0; byte_index <= (C_S_AXI_DATA_WIDTH/8)-1; byte_index = byte_index+1 )
                                if ( S_AXI_WSTRB[byte_index] == 1 ) 시작
                                    // 각 바이트 활성화는 쓰기 스트로브에 따라 어설션됩니다. 
                                    // 슬레이브 레지스터 0
                                    slv_reg0[(byte_index*8) +: 8] <= S_AXI_WDATA[(byte_index*8) +: 8];
                                끝  
                            3'h1:
                                for ( byte_index = 0; byte_index <= (C_S_AXI_DATA_WIDTH/8)-1; byte_index = byte_index+1 )
                                if ( S_AXI_WSTRB[byte_index] == 1 ) 시작
                                    // 각 바이트 활성화는 쓰기 스트로브에 따라 어설션됩니다. 
                                    // 슬레이브 레지스터 1
                                    slv_reg1[(byte_index*8) +: 8] <= S_AXI_WDATA[(byte_index*8) +: 8];
                                끝  
                            3'h2:
                                for ( byte_index = 0; byte_index <= (C_S_AXI_DATA_WIDTH/8)-1; byte_index = byte_index+1 )
                                if ( S_AXI_WSTRB[byte_index] == 1 ) 시작
                                    // 각 바이트 활성화는 쓰기 스트로브에 따라 어설션됩니다. 
                                    // 슬레이브 레지스터 2
                                    slv_reg2[(byte_index*8) +: 8] <= S_AXI_WDATA[(byte_index*8) +: 8];
                                끝  
                            3'h3:
                                for ( byte_index = 0; byte_index <= (C_S_AXI_DATA_WIDTH/8)-1; byte_index = byte_index+1 )
                                if ( S_AXI_WSTRB[byte_index] == 1 ) 시작
                                    // 각 바이트 활성화는 쓰기 스트로브에 따라 어설션됩니다. 
                                    // 슬레이브 레지스터 3
                                    slv_reg3[(byte_index*8) +: 8] <= S_AXI_WDATA[(byte_index*8) +: 8];
                                끝  
                            3분 4초:
                                for ( byte_index = 0; byte_index <= (C_S_AXI_DATA_WIDTH/8)-1; byte_index = byte_index+1 )
                                if ( S_AXI_WSTRB[byte_index] == 1 ) 시작
                                    // 각 바이트 활성화는 쓰기 스트로브에 따라 어설션됩니다. 
                                    // 슬레이브 레지스터 4
                                    slv_reg4[(byte_index*8) +: 8] <= S_AXI_WDATA[(byte_index*8) +: 8];
                                끝  
                            3시 5분:
                                for ( byte_index = 0; byte_index <= (C_S_AXI_DATA_WIDTH/8)-1; byte_index = byte_index+1 )
                                if ( S_AXI_WSTRB[byte_index] == 1 ) 시작
                                    // 각 바이트 활성화는 쓰기 스트로브에 따라 어설션됩니다. 
                                    // 슬레이브 레지스터 5
                                    slv_reg5[(byte_index*8) +: 8] <= S_AXI_WDATA[(byte_index*8) +: 8];
                                끝  
                            3시 6분:
                                for ( byte_index = 0; byte_index <= (C_S_AXI_DATA_WIDTH/8)-1; byte_index = byte_index+1 )
                                if ( S_AXI_WSTRB[byte_index] == 1 ) 시작
                                    // 각 바이트 활성화는 쓰기 스트로브에 따라 어설션됩니다. 
                                    // 슬레이브 레지스터 6
                                    slv_reg6[(byte_index*8) +: 8] <= S_AXI_WDATA[(byte_index*8) +: 8];
                                끝  
                            3시 7분:
                                for ( byte_index = 0; byte_index <= (C_S_AXI_DATA_WIDTH/8)-1; byte_index = byte_index+1 )
                                if ( S_AXI_WSTRB[byte_index] == 1 ) 시작
                                    // 각 바이트 활성화는 쓰기 스트로브에 따라 어설션됩니다. 
                                    // 슬레이브 레지스터 7
                                    slv_reg7[(byte_index*8) +: 8] <= S_AXI_WDATA[(byte_index*8) +: 8];
                                끝  
                            기본값 : 시작
                                        slv_reg0 <= slv_reg0;
                                        slv_reg1 <= slv_reg1;
                                        slv_reg2 <= slv_reg2;
                                        slv_reg3 <= slv_reg3;
                                        slv_reg4 <= slv_reg4;
                                        slv_reg5 <= slv_reg5;
                                        slv_reg6 <= slv_reg6;
                                        slv_reg7 <= slv_reg7;
                                        끝
                            엔드케이스
                        끝
                    끝
                끝

        > 추가된 라인 수에 따라 라인 번호가 조금씩 다를 수 있으나 위에서 설명한 것과 유사해야 합니다. 다음 단계에서는 파일을 계속 수정하게 되며 줄 번호 매기기가 더 이상 일치하지 않게 됩니다.

    5. AXI 레지스터의 초기 상태를 수정합니다.
        + 검증 및 식별 목적으로 레지스터 6과 7은 IP 버전을 등록하고 IP 공급업체를 식별하는 데 사용됩니다. 이는 드라이버 호환성을 확인하는 데 사용됩니다. 필수는 아니지만 버전과 식별 레지스터를 보유하는 것이 좋습니다.
        + 239행에서 v00.1(16진수로 0x76000001)을 사용하여 레지스터 6을 시작합니다.

            ```verilog
            slv_reg6 <= 32'h76000001;
            ```

        + 라인 240에서 XILINX 서브시스템 공급업체 ID(0x10ee) 및 1-Wire 장치 식별자 "DS"(0x4453)를 사용하여 레지스터 7을 시작합니다.

            ```verilog
            slv_reg7 <= 32'h10ee4453;
            ```

    6. 라인 313 뒤, 즉 마스터가 마스터에 쓰는 로직 뒤에 코어가 AXI 레지스터에 쓰는 로직을 추가합니다. 레지스터에 쓰기 위해서는 마스터에 우선순위가 주어지며, 필요한 경우 다른 로직을 구현하고 싶을 수도 있습니다.

        ```verilog
        else if (reg_wr)
		  begin
			slv_reg3[31]	<= failure;
			slv_reg3[0]		<= done;
			slv_reg3[4]     <= ready;
			slv_reg4[7:0]	<= rx_data;
			slv_reg5[0]		<= from_dq;
		  end
        ```

    7. IP 코어가 AXI IP 레지스터에 액세스하고 클럭 분배기 인스턴스화 위의 라인 374 아래에 출력 인터럽트를 생성하는 로직을 추가합니다.

        ```verilog
        assign ctrl_reset 	= slv_reg1[31];
        assign go			= slv_reg1[0];
        
        // (done & done_irq_en) | (ready & ready_irq_en) will raise an irq
        always @ ( posedge S_AXI_ACLK )
        begin
        w1_irq       = (slv_reg2[0 ] && slv_reg3[0]) || (slv_reg2[4] && slv_reg3[4]);
        end

        assign master_gpio_sel	= slv_reg0[31];
        assign command			= slv_reg0[11:8];
        assign tx_data			= slv_reg0[7:0];
        assign dq_ctrl_gpio		= slv_reg0[23];
        assign dq_out_gpio      = slv_reg0[16];
        ```

        > **참고:** `<working_directory>/reference_files/hdl/axi_1wire_host_slave_lite_v1_2_S00_AXI.v`를 다시 참조하여 파일을 올바르게 편집했는지 확인하세요. 참조 파일은 버전 1.2용이므로 버전 관리가 다릅니다.

4. AXI 최상위 모듈 래퍼 `axi_1wire_host.v`를 편집합니다.
    > `<working_directory>/reference_files/hdl/axi_1wire_host.v`는 참고용으로 제공됩니다.

    1. 50행에 추가된 매개변수와 72행에 신호를 포함하도록 인스턴스화된 모듈을 수정합니다. 결과 인스턴스화된 모듈은 다음과 같이 표시됩니다.

        ```verilog
        axi_1wire_host_slave_lite_v0_1_S00_AXI # ( 
            .C_S_AXI_DATA_WIDTH(C_S00_AXI_DATA_WIDTH),
            .C_S_AXI_ADDR_WIDTH(C_S00_AXI_ADDR_WIDTH),
            .CLK_DIV_VAL_TO_1MHz(CLK_DIV_VAL_TO_1MHz)
        ) axi_1wire_host_slave_lite_v0_1_S00_AXI_inst (
            .S_AXI_ACLK(s00_axi_aclk),
            .S_AXI_ARESETN(s00_axi_aresetn),
            .S_AXI_AWADDR(s00_axi_awaddr),
            .S_AXI_AWPROT(s00_axi_awprot),
            .S_AXI_AWVALID(s00_axi_awvalid),
            .S_AXI_AWREADY(s00_axi_awready),
            .S_AXI_WDATA(s00_axi_wdata),
            .S_AXI_WSTRB(s00_axi_wstrb),
            .S_AXI_WVALID(s00_axi_wvalid),
            .S_AXI_WREADY(s00_axi_wready),
            .S_AXI_BRESP(s00_axi_bresp),
            .S_AXI_BVALID(s00_axi_bvalid),
            .S_AXI_BREADY(s00_axi_bready),
            .S_AXI_ARADDR(s00_axi_araddr),
            .S_AXI_ARPROT(s00_axi_arprot),
            .S_AXI_ARVALID(s00_axi_arvalid),
            .S_AXI_ARREADY(s00_axi_arready),
            .S_AXI_RDATA(s00_axi_rdata),
            .S_AXI_RRESP(s00_axi_rresp),
            .S_AXI_RVALID(s00_axi_rvalid),
            .S_AXI_RREADY(s00_axi_rready),
            .w1_bus(w1_bus),
            .w1_irq(w1_irq)
	    );
        ```

    2. 7행에 클럭 분배기 매개변수를 추가합니다.

        ```
        parameter integer CLK_DIV_VAL_TO_1MHz = 100,
        ```

    3. 라인 18에 1-Wire 버스와 인터럽트 포트를 추가한다.

        ```
        inout w1_bus,
		output wire w1_irq,
        ```

    더 많은 인터페이스가 있고 인터페이스 간에 상호 작용이 있으면 더 많은 로직을 추가할 수 있습니다.

<a id="3-editing-the-ip-package"></a>
### 3. IP 패키지 편집

패키지 IP 창이 이미 열려 있어야 합니다. 그렇지 않은 경우 Vivado에서 *Design Sources &rarr; IP-XACT &rarr; component.xml*로 이동하세요.

1. 신분증:  
이 섹션은 귀하의 IP에 대한 기본 세부 정보를 제공하는 데 사용됩니다. 이 IP를 만든 사람, 높은 수준의 설명과 같은 필수 정보를 제공하고 IP를 분류합니다. 이렇게 하면 고객이 카탈로그에서 IP를 빠르게 식별할 수 있습니다.
    + 공급업체: 일반적으로 여기에 회사나 이름을 입력하거나 IP 공급업체를 식별하는 데 적절하다고 생각되는 모든 항목을 입력합니다. *AMD*를 사용하세요.
    + 라이브러리: 해당 IP가 속한 라이브러리를 식별합니다.
    + 이름: IP의 이름입니다. *axi_1wire_host*를 유지하세요.
    + 버전: IP 버전. 기능 및 드라이버 호환성을 수정하는 사소한 변경 사항에 대해 부 버전을 늘립니다. 구현에 영향을 줄 수 있는 더 중요한 변경 사항이 있는 경우 주 버전을 높이세요. 이는 일반적인 지침이므로 다른 버전 관리를 따를 수도 있습니다.
    + 표시 이름: *AXI 1-Wire 버스 호스트*를 유지합니다.
    + 설명: IP 생성 시 입력한 값을 그대로 유지합니다. 보시다시피, IP를 생성할 때 입력한 IP 세부 정보를 수정하려는 경우 여기에서 수정할 수 있습니다.
    + 공급업체 표시 이름: *공급업체*와 동일할 수 있지만 두 필드 간의 차이점을 강조하기 위한 것입니다. *Advanced Micro Devices*라는 긴 이름을 사용하세요.
    + 회사 URL: IP 공급업체의 웹사이트로, 고객이 IP와 관련하여 연락해야 할 경우 연락처 정보를 찾는 데 유용합니다.
    + 카테고리: 목록에서 여러 카테고리를 선택하여 IP를 더 잘 분류할 수 있습니다. */AXI_Peripheral*, */Communication_&_Networking/Serial_Interfaces* 및 */Embedded_Processing/AXI_Peripheral/Low_Speed_Peripheral*을 선택합니다.
    ![식별](./images/identification.png)

2. 호환성:  
섹션 이름에서 알 수 있듯이 귀하의 IP와 호환되는 장치 제품군을 지정하십시오.
    + Vitis용 패키지: 선택하지 않은 상태로 둡니다. Vitis와 함께 사용할 IP를 패키징하려면 확인란을 선택하고 제어 프로토콜을 선택하세요.
    + IPI용 패키지: Vivado IP 통합자와 함께 IP를 사용하기를 원하므로 선택된 상태로 둡니다.
        + Freq_Hz 무시: 선택하지 않은 상태로 둡니다.
    + 제품군: 귀하의 IP와 호환되는 장치 제품군을 추가합니다. 지원되는 모든 제품군을 수동으로 선택하거나 정규식을 사용할 수 있습니다. 라이프사이클 지원을 선택할 수도 있습니다. 또한 도구가 가족 지원을 자동으로 파악하도록 하는 옵션도 있습니다. 이것이 레벨 베타를 위해 여기서 할 일입니다.
    + 시뮬레이터: 지원되는 시뮬레이터를 열거합니다. 여기서는 기본값을 유지합니다.
    ![호환성](./images/compatibility.png)

3. 파일 그룹:  
이 섹션은 귀하의 IP와 함께 제공될 다양한 유형의 파일을 통합하는 데 사용됩니다. 최소 요구 사항은 IP의 RTL 파일을 통합하는 것입니다. 드라이버 파일, 제약 조건, 시뮬레이션 파일, 테스트 벤치, 예제 프로젝트, 사용자 가이드 등과 같은 훨씬 더 많은 것을 통합할 수 있는 옵션이 있습니다.
    1. HDL 파일을 추가할 때 이전에 수행한 변경 사항을 반영하려면 **파일 그룹 마법사의 변경 사항 병합**을 클릭합니다. 모든 디렉터리를 확장하고 새 파일이 추가되었는지 확인합니다.
        ![새 파일이 포함된 파일 그룹](./images/filegroup_expanded.png)
    2. 드라이버 파일을 편집합니다. 보시다시피 IP Packager는 이미 소프트웨어 드라이버 아래에 베어메탈 드라이버를 생성했습니다.
        1. *<working_directory>/reference_files/baremetal_driver/src/Makefile*과 일치하도록 Makefile을 편집합니다. Vitis가 드라이버를 올바르게 사용하려면 이러한 변경이 필요합니다.
        2. 일치하도록 자체 테스트 드라이버 파일 *axi_1wire_host_selftest.c"를 편집합니다. *<working_directory>/reference_files/baremetal_driver/src/axi_1wire_host_selftest.c*
            + 여기서는 AXI 레지스터에 저장된 IP ID와 버전을 읽는 간단한 테스트를 수행하고 있습니다. 이는 IP가 설계에서 제대로 작동하는지 보장하지는 않지만 IP가 존재하고 드라이버에 적합한 버전이 로드되었는지 확인합니다.
        3. *<working_directory>/reference_files/baremetal_driver/src/axi_1wire_host.h*와 일치하도록 드라이버 헤더 파일 *axi_1wire_host.h*를 편집합니다.
            + 레지스터 오프셋은 여기에 정의됩니다.

                ```h
                #define AXI_1WIRE_HOST_INSTR_REG_OFFSET 0x0
                #define AXI_1WIRE_HOST_CTRL_REG_OFFSET 0x4
                #define AXI_1WIRE_HOST_IRQCTRL_REG_OFFSET 0x8
                #define AXI_1WIRE_HOST_STAT_REG_OFFSET 0xC
                #define AXI_1WIRE_HOST_RXDATA_REG_OFFSET 0x10
                #define AXI_1WIRE_HOST_GPIODATA_REG_OFFSET 0x14
                #define AXI_1WIRE_HOST_IPVER_REG_OFFSET 0x18
                #define AXI_1WIRE_HOST_IPID_REG_OFFSET 0x1C
                ```

            + 1-Wire 코어 명령어도 헤더 파일에 정의되어 있습니다.

                ```h
                #define AXI_1WIRE_HOST_INITPRES	0x0800
                #define AXI_1WIRE_HOST_READBIT	0x0C00
                #define AXI_1WIRE_HOST_WRITEBIT	0x0E00
                #define AXI_1WIRE_HOST_READBYTE	0x0D00
                #define AXI_1WIRE_HOST_WRITEBYTE	0x0F00
                #define AXI_1WIRE_HOST_RESET    0x80000000
                ```

            + AXI IP에 대한 일반적인 읽기 및 쓰기는 헤더 파일에 정의되어 있습니다. IP 생성시 자동으로 추가됩니다.

                ```h
                #define AXI_1WIRE_HOST_mWriteReg(BaseAddress, RegOffset, Data) \
  	                Xil_Out32((BaseAddress) + (RegOffset), (u32)(Data))
                #define AXI_1WIRE_HOST_mReadReg(BaseAddress, RegOffset) \
                    Xil_In32((BaseAddress) + (RegOffset))
                ```

            + 파일의 나머지 부분은 드라이버 기능을 정의합니다. 당신은 그것을 검토할 수 있습니다.
        4. *<working_directory>/reference_files/baremetal_driver/src/axi_1wire_host.c*와 일치하도록 드라이버 소스 파일 *axi_1wire_host.c*를 편집합니다.
        여기에서 드라이버 기능을 구현합니다.
        5. *axi_1wire_host.tcl* 및 *axi_1wire_host.mdd*를 엽니다. 1-Wire IP에는 변경이 필요하지 않습니다.
        6. 새로운 Vitis IDE와 호환되도록 드라이버용 YAML 파일을 가져옵니다.
            1. Tcl 콘솔을 사용하여 YAML 파일을 IP 드라이버 폴더에 복사합니다.

                ```
                cp 
                <working_directory>/reference_files/baremetal_driver/data/axi_1wire_host.yaml <working_directory>/myIP/axi_1wire_host_0_1/drivers/axi_1wire_host_v1_0/data/
                ```

            2. **소프트웨어 드라이버**를 마우스 오른쪽 버튼으로 클릭하고 **파일 추가...**를 선택합니다.
            3. **파일 추가**를 클릭합니다.
            4. **<working_directory>/myIP/axi_1wire_host_0_1/drivers/axi_1wire_host_v1_0/data/axi_1wire_host.yaml**을 선택합니다.
            5. **RTL 포함 파일을 스캔하여 프로젝트에 추가** 및 **소스를 IP 디렉터리에 복사**를 선택 취소된 상태로 둡니다.
        7. 새 Vitis IDE와 호환되도록 드라이버용 CMakeLists 파일을 가져옵니다.
            1. TCL 콘솔을 사용하여 CMakeLists 파일을 IP 드라이버 폴더에 복사합니다.

                ```
                cp <working_directory>/reference_files/baremetal_driver/src/CMakeLists.txt <working_directory>/myIP/axi_1wire_host_0_1/drivers/axi_1wire_host_v1_0/src/
                ```

            2. **소프트웨어 드라이버**를 마우스 오른쪽 버튼으로 클릭하고 **파일 추가...**를 선택합니다.
            3. **파일 추가**를 클릭합니다.
            4. **<working_directory>/myIP/axi_1wire_host_0_1/drivers/axi_1wire_host_v1_0/src/CMakeLists.txt**를 선택합니다.
            5. **RTL 포함 파일을 스캔하여 프로젝트에 추가* 및 *소스를 IP 디렉터리에 복사**를 선택하지 않은 상태로 둡니다.
    3. 패키지에 제약 조건을 추가합니다.

        1. 소스 창에서 **소스 추가** ![소스 추가](./images/addSources.png)를 클릭하세요. **제약조건 추가 또는 생성**을 선택하고 **다음 >**을 클릭합니다.
        2. **파일 추가**를 선택한 다음 **<working_directory>/reference_files/constraints/axi_1wore_host.xdc**를 선택합니다. **소스를 IP 디렉터리에 복사**를 선택하고 **마침**을 클릭합니다.
        3. 다시 **파일 그룹 마법사의 변경 사항 병합**을 클릭하고 제약 조건 파일이 이제 *Advanced/Verilog Synesis* 아래에 나타나는지 확인합니다.
        4. 제약 조건 파일은 *Verilog Synesis*로 분류되어서는 안 됩니다. *소스* 창에서 **Constraints &rarr; contrs_1 &rarr; axi_1wire_host.xdc**를 마우스 오른쪽 버튼으로 클릭하고 **사용된 위치 설정...**을 선택한 다음 **합성**을 선택 취소합니다.
        5. **파일 그룹 마법사의 변경 사항 병합**을 다시 클릭합니다. 새 파일 그룹 *Implementation*이 생성되었고 XDC 파일이 그곳으로 이동되었는지 확인합니다.
        6. 파일을 클릭하여 IP 파일 속성을 편집하고 처리 순서를 *늦게*로 변경합니다.

            ![제약조건 처리 순서](./images/processingOrder.png)

            제약 조건 파일을 열고 검토합니다. 도구가 IP 내에서 생성된 클럭의 주파수를 알 수 있도록 하는 두 개의 *create_generated_clock* 제약 조건과 두 번째 클럭이 입력 클럭과 물리적으로 배타적이라는 것을 도구에 알리는 하나의 제약 조건 *set_clock_groups*가 있습니다.
    4. IP 패키지에 시뮬레이션 파일을 추가합니다.  
        외부 인터페이스의 응답자 모델이나 IP를 시뮬레이션하고 검증하는 데 필요한 기타 콘텐츠와 같은 시뮬레이션 전용 파일을 패키지된 IP에 추가할 수도 있습니다.  
        외부 1-Wire 소자를 에뮬레이션하는 복잡성을 감안할 때 1-Wire 호스트 설계에는 응답기 모델이 제공되지 않습니다. 코어는 DS18B20 장치의 알려진 특정 시퀀스를 사용하여 AXI IP에 패키징되기 전에 제한된 시뮬레이션을 통해 검증되었습니다.  
        패키지 IP에 시뮬레이션 파일을 추가하려면 다음을 수행할 수 있습니다.
        1. 파일 그룹 창에서 **Verilog Simulation**을 마우스 오른쪽 버튼으로 클릭합니다.
        2. **파일 추가...**를 선택합니다.
        3. 파일을 추가하고 **소스를 IP 디렉터리에 복사**를 선택합니다.

4. 사용자 정의 매개변수:  
이 섹션은 IP 사용자 정의 가능 매개변수에 대한 정보를 제공하는 데 사용됩니다. 기본적으로 AXI 매개변수는 표시되며 편집할 수 없습니다. 사용자 정의 매개변수를 표시하도록 선택할 수 있으며 사용자가 매개변수 값을 수정하도록 허용하는 옵션도 있습니다.
    1. **사용자 지정 매개변수 마법사의 변경 사항 병합**을 클릭하여 이전에 수행한 변경 사항을 반영합니다.
    2. *숨겨진 매개변수* 아래에 새 매개변수가 추가되었습니다. AXI 래퍼를 편집할 때 추가된 클럭 구분선 값입니다.
    3. 매개변수를 클릭하고 IP 매개변수 속성 창에 다음 설명을 추가합니다. *S00_AXI_CLK 분배기 to 도달 1MHz 대상 주파수*.
    4. 마우스 오른쪽 버튼을 클릭하고 **매개변수 편집...**을 선택합니다.
        1. **사용자 정의 GUI에 표시**를 선택합니다.
        2. 디스플레이 이름을 *S00 AXI CLK DIVIDER*로 변경합니다. 이는 입력 AXI 클럭 주파수를 줄이는 매개변수의 목적을 더 잘 반영합니다.
        3. 다음 도구 설명을 추가합니다. *S00_AXI_CLK 분배기 값은 1MHz 클록을 생성합니다(예: 100MHz S00_AXI_CLK의 경우 분배기는 100입니다)*.
        4. **범위 지정**을 선택합니다.
        5. 유형으로 **정수 범위**를 선택합니다.
        6. 최소값은 1, 최대값은 255를 입력합니다.
        ![clockDividerEdit](./images/clkdivedit.png)

5. 포트 및 인터페이스:  
이 섹션은 IP 포트 및 인터페이스에 대한 일부 정보를 제공하는 데 사용됩니다. 기본적으로 AXI 인터페이스는 IP 생성 시 제공된 정보를 기반으로 AXI 프로토콜에 필요한 값으로 미리 채워져 있습니다. 여기에는 *S00_AXI*라는 하나의 AXI 인터페이스만 있습니다. 확장하면 모든 신호와 데이터 포트가 표시됩니다. AXI 인터페이스와 관련된 클럭 및 리셋 신호는 *클럭 및 리셋 신호* 인터페이스 그룹에서 찾을 수 있습니다.
    1. **사용자 지정 매개변수 마법사의 변경 사항 병합**을 클릭하여 이전에 수행한 변경 사항을 반영합니다.
    2. Vivado IP 패키징 도구는 인터럽트 신호를 자동으로 인식하고 *w1_irq* 인터페이스를 생성했습니다. 마우스 오른쪽 버튼을 클릭하여 인터페이스를 편집할 수 있습니다. 또한 이를 확장하고 인터럽트 신호를 마우스 오른쪽 버튼으로 클릭하여 포트를 편집할 수도 있습니다. 여기서는 기본값을 유지합니다.
    3. *w1_bus* 신호는 인터페이스의 일부가 아니므로 그대로 유지될 수 있습니다. IP 포트가 공통 인터페이스의 일부인 경우 인터페이스를 추가하고 포트를 여기에 연결하여 유용성을 높일 수 있습니다.
        > **참고**: *시계 인터페이스 'S00_AXI_CLK'에 FREQ_HZ 매개변수가 없습니다*라는 경고가 표시될 수 있습니다. IP에 AXI 인터페이스에 대한 특정 클록 주파수가 필요한 경우 AXI 클록 인터페이스(*클럭 및 재설정 신호 &rarr; S00_AXI_CLK*)를 마우스 오른쪽 버튼으로 클릭하고 *인터페이스 편집...*을 선택합니다. 그런 다음 *매개변수*로 이동하여 *사용자 설정 &rarr; FREQ_HZ* 필요*를 추가한 다음 *사용자 설정*에서 해당 값을 지정합니다. 여기서 귀하의 IP는 1MHz보다 높은 모든 주파수를 지원하므로 경고를 무시해도 됩니다.
    ![포트 및 인터페이스](./images/portsAndInterfaces.png)

6. 주소 지정 및 메모리:  
이 섹션은 IP의 메모리 맵과 주소 공간을 정의하는 데 사용됩니다. 이 도구는 AXI 인터페이스 주소 맵을 자동으로 추론합니다. 도구로 채워지지 않은 사용자 정의 주소 공간이 있는 경우 마우스 오른쪽 버튼을 클릭하고 *IP 주소 지정 및 메모리 마법사*를 시작합니다. 여기서는 수정할 사항이 없습니다.
    ![주소 지정 및 메모리](./images/adressingMemory.png)

7. 사용자 정의 GUI:  
이 섹션은 IP 통합자 흐름에서 IP를 인스턴스화하는 데 사용되는 IP GUI를 사용자 정의하는 데 사용됩니다. 여러 페이지를 생성하고 표시할 매개변수와 숨길 매개변수를 결정할 수 있습니다. 각 페이지에 매개변수가 표시되도록 결정할 수도 있습니다. 여기서는 기본 레이아웃을 유지합니다. S00 AXI CLK DIVIDER는 페이지가 두 개 이상인 경우 모든 페이지에 나타납니다. AXI 매개변수는 0페이지에만 나타납니다.
    ![사용자 정의 GUI](./images/CustomizationGUI.png)

8. 검토 및 패키지:  
    이 마지막 섹션은 패키지 IP의 모든 변경 사항을 병합하는 데 사용됩니다. 또한 귀하의 IP와 IP 위치에 대한 기본 정보 요약도 볼 수 있습니다. IP를 다시 패키지할 때마다 주 버전과 부 버전이 아닌 개정 번호가 증가합니다. 버전을 생성하려면 *식별* 섹션으로 돌아가야 합니다. IP를 패키징한 후에는 언제든지 *<working_directory>/myIP/edit_axi_1wire_host_v0_1.xpr* 프로젝트로 돌아와서 IP를 다시 편집하고 필요한 사항을 변경할 수 있습니다.

    코어가 제대로 테스트되었더라도 일단 포장되면 문제가 발생할 수 있습니다. 패키지된 IP를 테스트하기 전에는 어떤 제약 조건이 필요한지 알 수 없으므로 언제든지 IP를 패키징하고 이를 디자인에 통합하고 제약 조건을 파악한 후 여기로 돌아와 패키지에 추가할 수 있습니다. HDL, 드라이버 파일 및 기타 모든 항목에도 동일하게 적용됩니다.

    1. **IP가 수정되었습니다**를 클릭하여 도구가 패키지의 변경 사항을 통합했는지 확인합니다.
    2. **포장 설정 편집**을 클릭합니다.
        1. *패키징 후 &rarr; 자동 동작*에서 **IP 아카이브 생성**을 선택합니다. 이는 IP 공유를 단순화하고 IP의 다양한 버전을 보관하는 데 사용될 수 있습니다.
        2. *자동 동작 &rarr; IP 패키지 프로그램의 IP 편집*에서 **패키징 후 프로젝트 삭제**를 선택 취소합니다. 이는 다시 돌아와서 프로젝트에서 일부 테스트나 수정을 수행하려는 경우에 유용합니다.
    3. **패키지 IP**를 클릭합니다. 이제 귀하의 IP가 성공적으로 패키징되었습니다. 프로젝트를 계속 작업하려면 프로젝트를 닫거나 열어 둘 수 있습니다. 기존 IP 코어로 작업 중이므로 여기서 프로젝트를 닫습니다.

AMD Vivado™ IP 패키저에 대한 자세한 내용은 *Vivado Design Suite 사용자 가이드: 맞춤형 IP 생성 및 패키징* [(UG1118)](https://docs.amd.com/r/en-US/ug1118-vivado-creating-packaging-custom-ip)을 참조하세요.  

사용자 정의 IP를 생성하고 패키징하는 다른 옵션에 대한 자세한 내용은 *Vivado Design Suite 튜토리얼: 사용자 정의 IP 생성 및 패키징* [(UG1119)](https://docs.amd.com/r/en-US/ug1119-vivado-creating-packaging-ip-tutorial)을 참조하세요.

<a id="encrypting-your-ip"></a>
## IP 암호화

Vivado Design Suite는 비트스트림 생성까지 암호화를 지원합니다. 다양한 수준의 프로세스를 암호화할 수 있으며 HDL 파일 및/또는 설계 체크포인트를 암호화할 수 있습니다. 이 튜토리얼에서는 HDL 파일의 암호화만 살펴보겠습니다. IP를 암호화하면 귀하의 자산이 보호되고 복제될 수 없습니다.

이 튜토리얼에서 제공되는 3개의 파일은 암호화되어 있습니다(`<working_directory>/reference_files/hdl/clk_div.v`, `<working_directory>/reference_files/hdl/jcnt.v` 및 `<working_directory>/reference_files/hdl/sr.v`). 모든 HDL 파일은 암호화될 수 있지만 귀하가 파일에 할당한 권한은 Vivado가 귀하의 IP를 처리하는 방법에 영향을 미친다는 점에 유의하세요.

자체 암호화 키 파일을 사용하거나 `<Install_Dir>/Vivado/<version>/data/pubkey`에 있는 Vivado 도구와 함께 제공되는 공개 키 파일을 사용할 수 있습니다. 암호화 키 파일을 지정하지 않으면 도구는 암호화 키가 디자인 소스 파일에 있다고 가정합니다. 새 파일 확장자를 지정할 때 주의하세요. 그렇지 않으면 원본 확장자를 덮어쓰게 됩니다.
일반적으로 다음 Tcl 명령을 사용하여 파일을 암호화합니다.

```tcl
encrypt -key <keyfile> -ext <new_extension> -lang <verilog> myFile.v
```

+ `&lt;keyfile&gt;`: 암호화 키가 포함된 키 파일입니다. `-key &lt;keyfile&gt;`를 생략하면 도구는 디자인 소스 파일 내에서 키를 찾습니다.
+ `<new_extension>`: 새로운 확장 프로그램입니다. 점을 포함하는 것을 잊지 마세요(예: *.vp*). `-ext <new_extension>`를 포함하지 않으면 원본 파일을 덮어쓰게 됩니다.
+ *&lt;verilog&gt;*: 디자인 파일 HDL을 지정합니다. *VHDL* 또는 *verilog*를 사용할 수 있습니다.

IP 암호화에 대한 자세한 내용은 [Vivado의 IP 암호화 섹션](https://docs.amd.com/r/en-US/ug1118-vivado-creating-packaging-custom-ip/Encrypting-IP-in-Vivado)을 참조하세요.
<p align="center"><b>다음 단계: <a href="./2_baremetal_driver.kr.md"> 베어메탈 드라이버 개발</a></b></p>

---
<p class="sphinxhide" align="center"><sub>저작권 © 2024-2025 Advanced Micro Devices, Inc.</sub></p>
<p class="sphinxhide" align="center"><sup><a href="https://www.amd.com/en/corporate/copyright">이용 약관</a></sup></p>
