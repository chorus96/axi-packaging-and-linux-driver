# `application_bm.h` 분석

## 개요

`application_bm.h`는 baremetal 온도 센서 예제 애플리케이션의 공개 함수 선언을 제공하는 헤더입니다. include guard로 중복 포함을 방지하고, 외부 코드가 `continuous_temperature_reading()`을 호출할 수 있도록 선언합니다.

## 블록 다이어그램

```mermaid
flowchart LR
    MAIN["main.c 또는 예제 진입점"]
    HDR["application_bm.h\nprototype"]
    APP["application_bm.c\ncontinuous_temperature_reading"]
    DRV["axi_1wire_host driver API"]

    MAIN --> HDR --> APP --> DRV
```

## 선언 내용

| 선언 | 설명 |
|---|---|
| `#ifndef APPLICATION_BM_H` / `#define APPLICATION_BM_H` | 헤더 중복 include를 막는 include guard입니다. |
| `void continuous_temperature_reading(s8, s8, int);` | 온도 상한, 하한, 해상도를 받아 센서를 설정하고 연속 온도 읽기를 수행하는 함수입니다. |

## 의존성 주의

- 프로토타입에서 `s8` 타입을 사용하므로 이 헤더를 include하는 파일은 보통 Xilinx 타입 정의(`xil_types.h`)가 먼저 필요합니다.
- 구현 파일인 `application_bm.c`는 `application_bm.h`, `xil_types.h`, `xil_printf.h`, `axi_1wire_host.h`, `xparameters.h`를 함께 사용합니다.
