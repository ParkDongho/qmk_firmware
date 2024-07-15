# 키보드 플래싱하기

이제 커스텀 펌웨어 파일을 빌드했으므로, 키보드를 플래싱할 준비가 되었습니다.

## 키보드를 DFU(부트로더) 모드로 전환하기

커스텀 펌웨어를 플래싱하려면 먼저 키보드를 특별한 플래싱 모드로 전환해야 합니다. 이 모드에서는 키보드를 사용할 수 없으며, 펌웨어가 작성되는 동안 키보드를 분리하거나 플래싱 과정을 중단하지 않도록 주의해야 합니다.

키보드마다 이 모드로 전환하는 방법이 다릅니다. 현재 PCB가 QMK, TMK 또는 PS2AVRGB(Bootmapper Client)를 실행 중이고 특정 지침을 받지 않았다면 다음 방법을 순서대로 시도해 보십시오:

* 두 쉬프트 키를 누른 상태에서 `Pause` 키를 누릅니다.
* 두 쉬프트 키를 누른 상태에서 `B` 키를 누릅니다.
* 키보드의 전원을 뽑고, 스페이스바와 `B` 키를 동시에 누른 상태에서 키보드를 연결하고 잠시 기다린 후 키를 놓습니다.
* 키보드의 전원을 뽑고, 상단 또는 하단 왼쪽 키(보통 Escape 또는 왼쪽 컨트롤)를 누른 상태에서 키보드를 연결합니다.
* 보통 PCB의 하단에 위치한 물리적인 `RESET` 버튼을 누릅니다.
* PCB에 `RESET`과 `GND`라고 표시된 헤더 핀을 찾아서 PCB를 연결하는 동안 이를 단락시킵니다.

위의 방법을 모두 시도했으나 실패한 경우, 보드의 메인 칩에 `STM32` 또는 `RP2-B1`이라고 적혀 있다면 약간 더 복잡할 수 있습니다. 일반적으로 [디스코드](https://discord.gg/Uq7gcHh)에서 도움을 요청하는 것이 좋습니다. 보드의 사진을 미리 준비해 두면 도움이 될 것입니다!

그 외에는 QMK Toolbox에서 다음과 유사한 노란색 메시지를 볼 수 있어야 합니다:

```
*** DFU device connected: Atmel Corp. ATmega32U4 (03EB:2FF4:0000)
```

또한 이 부트로더 장치는 장치 관리자, 시스템 정보.app 또는 `lsusb`에서도 확인할 수 있습니다.

## QMK Toolbox로 키보드 플래싱하기

키보드를 플래싱하는 가장 간단한 방법은 [QMK Toolbox](https://github.com/qmk/qmk_toolbox/releases)를 사용하는 것입니다.

그러나 Toolbox는 현재 Windows와 macOS에서만 사용할 수 있습니다. Linux를 사용 중이거나 명령줄에서 펌웨어를 플래싱하려는 경우, [명령줄에서 키보드 플래싱](#flash-your-keyboard-from-the-command-line) 섹션으로 이동하십시오.

::: tip
QMK Toolbox는 [RP2040 장치](flashing#raspberry-pi-rp2040-uf2) 플래싱에는 필요하지 않습니다.
:::

### QMK Toolbox에 파일 로드하기

먼저 QMK Toolbox 애플리케이션을 엽니다. Finder 또는 Explorer에서 펌웨어 파일을 찾습니다. 키보드 펌웨어는 `.hex` 또는 `.bin` 형식일 수 있습니다. QMK는 적절한 형식을 `qmk_firmware` 디렉토리의 루트에 복사하려고 합니다.

Windows 또는 macOS에서는 현재 폴더를 Explorer 또는 Finder에서 쉽게 열 수 있는 명령이 있습니다.

::::tabs

=== Windows

```
start .
```

=== macOS

```
open .
```

::::

펌웨어 파일은 항상 다음과 같은 이름 형식을 따릅니다:

```
<keyboard_name>_<keymap_name>.{bin,hex}
```

예를 들어, `planck/rev5`의 `default` 키맵은 다음과 같은 파일 이름을 가집니다:

```
planck_rev5_default.hex
```

펌웨어 파일을 찾으면 QMK Toolbox의 "Local file" 상자로 드래그하거나 "Open"을 클릭하여 펌웨어 파일이 저장된 위치로 이동합니다.

### 키보드 플래싱하기

QMK Toolbox에서 `Flash` 버튼을 클릭합니다. 다음과 유사한 출력이 표시됩니다:

```
*** DFU device connected: Atmel Corp. ATmega32U4 (03EB:2FF4:0000)
*** Attempting to flash, please don't remove device
>>> dfu-programmer.exe atmega32u4 erase --force
    Erasing flash...  Success
    Checking memory from 0x0 to 0x6FFF...  Empty.
>>> dfu-programmer.exe atmega32u4 flash "D:\Git\qmk_firmware\gh60_satan_default.hex"
    Checking memory from 0x0 to 0x3F7F...  Empty.
    0%                            100%  Programming 0x3F80 bytes...
    [>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]  Success
    0%                            100%  Reading 0x7000 bytes...
    [>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]  Success
    Validating...  Success
    0x3F80 bytes written into 0x7000 bytes memory (56.70%).
>>> dfu-programmer.exe atmega32u4 reset
    
*** DFU device disconnected: Atmel Corp: ATmega32U4 (03EB:2FF4:0000)
```

## 명령줄에서 키보드 플래싱하기

이전보다 훨씬 간단해졌습니다. 펌웨어를 컴파일하고 플래싱할 준비가 되면 터미널 창을 열고 플래시 명령을 실행합니다:

```sh
qmk flash
```

빌드 환경 구성 시 CLI에서 키보드/키맵 이름을 설정하지 않았거나 여러 개의 키보드가 있는 경우, 키보드와 키맵을 지정할 수 있습니다:

```sh
qmk flash -kb <my_keyboard> -km <my_keymap>
```

이 명령은 키보드의 구성을 확인한 다음, 지정된 부트로더를 기반으로 플래싱을 시도합니다. 따라서 키보드가 사용하는 부트로더를 알 필요가 없습니다. 명령을 실행하고 명령이 알아서 처리하도록 하십시오.

그러나 이는 키보드에 의해 부트로더가 설정된 경우에만 작동합니다. 이 정보가 구성되지 않았거나 지원되는 타겟이 없는 보드를 사용하는 경우 다음과 같은 오류가 발생할 수 있습니다:

```
WARNING: This board's bootloader is not specified or is not supported by the ":flash" target at this time.
```

이 경우, 부트로더를 지정해야 합니다. 자세한 내용은 [펌웨어 플래싱](flashing) 가이드를 참조하십시오.

::: warning
`qmk flash`에서 부트로더가 감지되지 않는 경우, 일반적인 문제를 해결하는 방법에 대한 제안을 얻으려면 `qmk doctor`를 실행해 보십시오.
:::

## 테스트하기

축하합니다! 커스텀 펌웨어가 키보드에 프로그래밍되었으며, 이제 테스트할 준비가 되었습니다!

운이 좋다면 모든 것이 완벽하게 작동할 것입니다. 그렇지 않다면 무엇이 문제인지 알아내는 데 도움이 되는 단계가 있습니다. 키보드를 테스트하는 것은 일반적으로 매우 간단합니다. 모든 키를 눌러보고 예상한 키가 전송되는지 확인하십시오. QMK가 실행되지 않더라도 [QMK 설정 도구](https://config.qmk.fm/#/test/)의 테스트 모드를 사용하여 키보드를 확인할 수 있습니다.

여전히 작동하지 않나요? 자세한 정보를 얻으려면 FAQ 항목을 살펴보거나 [디스코드에서 문의](https://discord.gg/Uq7gcHh)하십시오.