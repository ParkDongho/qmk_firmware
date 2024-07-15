# 디버깅 FAQ

이 페이지는 키보드를 문제 해결할 때 사람들이 자주 묻는 다양한 질문을 다룹니다.

## 디버깅 {#debugging}

키보드의 `rules.mk` 파일에 `CONSOLE_ENABLE = yes`가 있으면 키보드가 디버그 정보를 출력합니다. 기본적으로 출력은 매우 제한적이지만, 디버그 모드를 켜서 디버그 출력을 늘릴 수 있습니다. 키맵에서 `DB_TOGG` 키코드를 사용하거나 [Command](features/command) 기능을 사용하여 디버그 모드를 활성화할 수 있으며, 또는 다음 코드를 키맵에 추가할 수 있습니다.

```c
void keyboard_post_init_user(void) {
  // 원하는 동작으로 값을 사용자 정의
  debug_enable=true;
  debug_matrix=true;
  //debug_keyboard=true;
  //debug_mouse=true;
}
```

## 디버깅 도구

키보드를 디버그하는 데 사용할 수 있는 다양한 도구가 있습니다.

### QMK Toolbox를 사용한 디버깅

호환되는 플랫폼의 경우 [QMK Toolbox](https://github.com/qmk/qmk_toolbox)를 사용하여 키보드의 디버그 메시지를 표시할 수 있습니다.

### QMK CLI를 사용한 디버깅

터미널 기반 솔루션을 선호하십니까? [QMK CLI 콘솔 명령](cli_commands#qmk-console)을 사용하여 키보드의 디버그 메시지를 표시할 수 있습니다.

### hid_listen을 사용한 디버깅

독립형 솔루션을 원하십니까? PJRC에서 제공하는 [hid_listen](https://www.pjrc.com/teensy/hid_listen.html)을 사용하여 디버그 메시지를 표시할 수 있습니다. Windows, Linux, MacOS용으로 미리 빌드된 바이너리가 제공됩니다.

## 사용자 정의 디버그 메시지 보내기 {#debug-api}

가끔 [사용자 정의 코드](custom_quantum_functions)에서 디버그 메시지를 출력하는 것이 유용할 때가 있습니다. 이는 매우 간단합니다. 먼저 파일 상단에 `print.h`를 포함하십시오:

```c
#include "print.h"
```

그런 다음 몇 가지 다른 출력 함수를 사용할 수 있습니다:

* `print("string")`: 간단한 문자열을 출력합니다.
* `uprintf("%s string", var)`: 형식화된 문자열을 출력합니다.
* `dprint("string")`: 간단한 문자열을 출력하지만 디버그 모드가 활성화된 경우에만 출력합니다.
* `dprintf("%s string", var)`: 형식화된 문자열을 출력하지만 디버그 모드가 활성화된 경우에만 출력합니다.

## 디버깅 예제

아래는 실제 디버깅 예제 모음입니다. 추가 정보는 [QMK 디버깅/문제 해결](faq_debug)을 참조하십시오.

### 이 키 입력이 어느 매트릭스 위치에 있습니까?

포팅할 때나 PCB 문제를 진단할 때, 키 입력이 올바르게 스캔되는지 확인하는 것이 유용할 수 있습니다. 이 시나리오에 대한 로깅을 활성화하려면 다음 코드를 키맵의 `keymap.c`에 추가하십시오.

```c
bool process_record_user(uint16_t keycode, keyrecord_t *record) {
  // 콘솔이 활성화된 경우, 누른 각 키의 매트릭스 위치와 상태를 출력합니다.
#ifdef CONSOLE_ENABLE
    uprintf("KL: kc: 0x%04X, col: %2u, row: %2u, pressed: %u, time: %5u, int: %u, count: %u\n", keycode, record->event.key.col, record->event.key.row, record->event.pressed, record->event.time, record->tap.interrupted, record->tap.count);
#endif 
  return true;
}
```

예제 출력
```
Waiting for device:.......
Listening:
KL: kc: 169, col: 0, row: 0, pressed: 1, time: 15505, int: 0, count: 0
KL: kc: 169, col: 0, row: 0, pressed: 0, time: 15510, int: 0, count: 0
KL: kc: 174, col: 1, row: 0, pressed: 1, time: 15703, int: 0, count: 0
KL: kc: 174, col: 1, row: 0, pressed: 0, time: 15843, int: 0, count: 0
KL: kc: 172, col: 2, row: 0, pressed: 1, time: 16303, int: 0, count: 0
KL: kc: 172, col: 2, row: 0, pressed: 0, time: 16411, int: 0, count: 0
```

### 키 입력을 스캔하는 데 얼마나 걸렸습니까?

성능 문제를 테스트할 때, 스위치 매트릭스가 스캔되는 빈도를 아는 것이 유용할 수 있습니다. 이 시나리오에 대한 로깅을 활성화하려면 다음 코드를 키맵의 `config.h`에 추가하십시오.

```c
#define DEBUG_MATRIX_SCAN_RATE
```

예제 출력
```
  > matrix scan frequency: 315
  > matrix scan frequency: 313
  > matrix scan frequency: 316
  > matrix scan frequency: 316
  > matrix scan frequency: 316
  > matrix scan frequency: 316
```

## `hid_listen`이 장치를 인식하지 못함

디바이스의 디버그 콘솔이 준비되지 않은 경우 다음과 같은 메시지가 표시됩니다:

```
Waiting for device:.........
```

장치가 연결되면 *hid_listen*이 이를 찾으면 다음 메시지가 표시됩니다:

```
Waiting for new device:.........................
Listening:
```

이 'Listening:' 메시지를 받을 수 없는 경우 [Makefile]에 `CONSOLE_ENABLE=yes`로 빌드해보십시오.

Linux와 같은 OS에서 장치에 액세스하려면 권한이 필요할 수 있습니다. `sudo hid_listen`을 시도해보십시오.

많은 Linux 배포판에서 root로 hid_listen을 실행할 필요 없이 다음 내용의 파일을 `/etc/udev/rules.d/70-hid-listen.rules`에 생성하여 이를 피할 수 있습니다:

```
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="abcd", ATTRS{idProduct}=="def1", TAG+="uaccess", RUN{builtin}+="uaccess"
```

abcd와 def1을 키보드의 공급업체 및 제품 ID로 대체하십시오. 문자는 소문자여야 합니다. `RUN{builtin}+="uaccess"` 부분은 이전 배포판에서만 필요합니다.

## 콘솔에 메시지가 표시되지 않음

확인 사항:
- *hid_listen*이 장치를 찾습니다. 위를 참조하십시오.
- **Magic**+d를 눌러 디버그를 활성화합니다. [Magic Commands](https://github.com/tmk/tmk_keyboard#magic-commands)를 참조하십시오.
- `debug_enable=true`로 설정합니다. [디버깅](#debugging)을 참조하십시오.
- `print` 함수를 사용하는 것을 시도해보십시오. **common/print.h**를 참조하십시오.
- 콘솔 기능이 있는 다른 장치의 연결을 해제하십시오. [Issue #97](https://github.com/tmk/tmk_keyboard/issues/97)를 참조하십시오.
- 모든 문자열이 개행 문자(`\n`)로 끝나는지 확인하십시오. QMK Toolbox는 콘솔 출력을 한 줄 단위로 인쇄합니다.