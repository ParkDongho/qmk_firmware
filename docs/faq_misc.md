# 기타 FAQ

## 키보드를 어떻게 테스트하나요? {#testing}

키보드를 테스트하는 것은 일반적으로 매우 간단합니다. 모든 키를 눌러서 기대한 키가 출력되는지 확인하세요. [QMK Configurator](https://config.qmk.fm/#/test/)의 테스트 모드를 사용하여 키보드를 확인할 수 있습니다. QMK를 실행하지 않는 키보드도 가능합니다.

## 안전 고려 사항

키보드를 "벽돌"로 만들어 펌웨어를 다시 쓸 수 없게 만들고 싶지 않으실 겁니다. 다음은 무엇이 (그리고 무엇이 아닌지) 너무 위험하지 않은지 보여주는 몇 가지 매개변수입니다.

- 키보드 맵에 QK_BOOT가 포함되어 있지 않으면, PCB의 리셋 버튼을 눌러야 DFU 모드에 진입할 수 있으며, 이를 위해서는 바닥을 풀어야 합니다.
- tmk_core / common 파일을 건드리면 키보드가 작동하지 않을 수 있습니다.
- 너무 큰 .hex 파일은 문제를 일으킬 수 있습니다. `make dfu`는 블록을 지우고, 크기를 테스트합니다(잘못된 순서!). 이로 인해 키보드를 플래시하지 못하고 DFU 모드에 남게 됩니다.
  - 예를 들어 Planck의 최대 .hex 파일 크기는 7000h (28672 decimal)입니다.

```
Linking: .build/planck_rev4_cbbrowne.elf                                                            [OK]
Creating load file for Flash: .build/planck_rev4_cbbrowne.hex                                       [OK]

Size after:
   text    data     bss     dec     hex filename
      0   22396       0   22396    577c planck_rev4_cbbrowne.hex
```

  - 위 파일의 크기는 22396/577ch으로, 28672/7000h보다 작습니다.
  - 적절한 대체 .hex 파일이 있다면 다시 시도할 수 있습니다.
  - 키보드의 Makefile에서 지정할 수 있는 몇 가지 옵션은 추가 메모리를 소비합니다. BOOTMAGIC_ENABLE, MOUSEKEY_ENABLE, EXTRAKEY_ENABLE, CONSOLE_ENABLE 등을 주의하세요.
- DFU 도구는 부트로더를 덮어쓰지 않으므로(추가 옵션을 사용하지 않는 한) 큰 위험은 없습니다.
- EEPROM은 약 100000(10만) 회 쓰기 사이클을 가집니다. 펌웨어를 반복적으로 계속해서 덮어쓰지 마세요. 그렇게 하면 결국 EEPROM이 소모됩니다.

## NKRO가 작동하지 않아요

먼저 **Makefile**에서 `NKRO_ENABLE` 빌드 옵션이 활성화되어 있어야 합니다.

**NKRO**가 여전히 작동하지 않으면 `Magic` **N** 명령(기본적으로 `LShift+RShift+N`)을 시도하세요. 이 명령을 사용하여 **NKRO**와 **6KRO** 모드를 일시적으로 전환할 수 있습니다. 일부 상황에서는 **NKRO**가 작동하지 않을 수 있으며, 이때 **6KRO** 모드로 전환해야 합니다. 특히 BIOS에 있을 때 그렇습니다.

## TrackPoint는 리셋 회로가 필요합니다 (PS/2 마우스 지원)

리셋 회로가 없으면 하드웨어 초기화가 제대로 이루어지지 않아 일관성 없는 결과가 발생할 수 있습니다. TPM754의 회로 다이어그램을 참조하세요:

- https://geekhack.org/index.php?topic=50176.msg1127447#msg1127447
- https://www.mikrocontroller.net/attachment/52583/tpm754.pdf

## 매트릭스의 16번째 열 이상을 읽을 수 없어요

`read_cols()` 함수에서 `1<<16` 대신 `1UL<<16`을 사용하세요. C에서는 `1`이 [int] 타입을 의미하며, AVR의 경우 [16 비트]입니다. 따라서 15 이상 왼쪽으로 시프트할 수 없습니다. `1<<16`을 계산하면 0이 되므로, 이를 해결하려면 [unsigned long] 타입을 사용해야 합니다.

https://deskthority.net/workshop-f7/rebuilding-and-redesigning-a-classic-thinkpad-keyboard-t6181-60.html#p146279

## 특별한 추가 키가 작동하지 않아요 (시스템, 오디오 제어 키)

QMK에서 이를 사용하려면 `rules.mk`에 `EXTRAKEY_ENABLE`을 정의해야 합니다.

```
EXTRAKEY_ENABLE = yes          # 오디오 제어 및 시스템 제어
```

## 절전 모드에서 깨우기가 작동하지 않아요

Windows에서 **장치 관리자**의 **전원 관리** 속성 탭에서 `이 장치를 사용하여 컴퓨터를 깨울 수 있음` 설정을 확인하세요. 또한 BIOS 설정도 확인하세요. 절전 중에 키를 누르면 호스트가 깨어나야 합니다.

## Arduino를 사용 중인가요?

**Arduino 핀 명명법은 실제 칩과 다릅니다.** 예를 들어, Arduino 핀 `D0`는 `PD0`가 아닙니다. 회로를 직접 회로도와 확인하세요.

- https://arduino.cc/en/uploads/Main/arduino-leonardo-schematic_3b.pdf
- https://arduino.cc/en/uploads/Main/arduino-micro-schematic.pdf

Arduino Leonardo와 Micro는 **ATMega32U4**를 사용하며 TMK에 사용할 수 있지만, Arduino 부트로더는 문제가 될 수 있습니다.

## JTAG 활성화

기본적으로 키보드가 시작되면 JTAG 디버깅 인터페이스가 비활성화됩니다. JTAG 가능 MCU는 출고 시 `JTAGEN` 퓨즈가 설정되어 있으며, 이 퓨즈는 보드가 스위치 매트릭스, LED 등으로 사용하고 있을 수 있는 MCU의 특정 핀을 차지합니다.

JTAG를 계속 활성화하려면 `config.h`에 다음을 추가하세요:

```c
#define NO_JTAG_DISABLE
```

## USB 3 호환성

USB 3.x 포트에서 USB 2.0 포트로 전환하면 몇 가지 문제가 해결될 수 있습니다.

## Mac 호환성

### OS X 10.11과 허브

참고: https://geekhack.org/index.php?topic=14290.msg1884034#msg1884034

## BIOS(UEFI) 설정/재개(절전 및 깨우기)/전원 주기 문제

일부 사용자는 BIOS에서 또는 재개(전원 주기) 후에 키보드가 작동을 멈춘다고 보고했습니다.

현재로서는 근본 원인이 명확하지 않지만, 일부 빌드 옵션이 관련이 있는 것으로 보입니다. Makefile에서 `CONSOLE_ENABLE`, `NKRO_ENABLE`, `SLEEP_LED_ENABLE` 등의 옵션을 비활성화해 보세요.

추가 정보:

- https://github.com/tmk/tmk_keyboard/issues/266
- https://geekhack.org/index.php?topic=41989.msg1967778#msg1967778