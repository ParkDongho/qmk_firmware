# QMK 코드 이해하기

이 문서는 QMK 펌웨어가 어떻게 동작하는지에 대해 높은 수준에서 설명하고자 합니다. 기본적인 프로그래밍 개념을 이해하고 있다는 가정하에 작성되었으며, C 언어에 익숙하지 않더라도 필요한 부분에서만 설명합니다. 다음 문서들을 기본적으로 이해하고 있다고 가정합니다:

* [소개](getting_started_introduction)
* [키보드의 작동 원리](how_keyboards_work)
* [FAQ](faq_general)

## 시작

QMK를 다른 컴퓨터 프로그램과 다르지 않다고 생각할 수 있습니다. 시작하여 작업을 수행하지만 이 프로그램은 절대 끝나지 않습니다. 다른 C 프로그램처럼 진입점은 `main()` 함수입니다. QMK의 `main()` 함수는 [`quantum/main.c`](https://github.com/qmk/qmk_firmware/blob/0.15.13/quantum/main.c#L55)에 있습니다.

`main()` 함수를 살펴보면, 설정된 하드웨어(호스트에 대한 USB 포함)를 초기화하는 것으로 시작합니다. QMK의 가장 일반적인 플랫폼은 AVR 프로세서(atmega32u4 등)에서 실행되는 `lufa`입니다. 이 플랫폼을 위해 컴파일되면 [`platforms/avr/platform.c`](https://github.com/qmk/qmk_firmware/blob/0.15.13/platforms/avr/platform.c#L19)의 `platform_setup()`과 [`tmk_core/protocol/lufa/lufa.c`](https://github.com/qmk/qmk_firmware/blob/0.15.13/tmk_core/protocol/lufa/lufa.c#L1066)의 `protocol_setup()`을 호출합니다. 다른 플랫폼(`chibios`, `vusb` 등)으로 컴파일되면 다른 구현을 사용합니다. 처음에는 많은 기능처럼 보일 수 있지만, 대부분의 경우 코드가 `#define`에 의해 비활성화됩니다.

그런 다음 `main()` 함수는 [`while (true)`](https://github.com/qmk/qmk_firmware/blob/0.15.13/quantum/main.c#L63)로 프로그램의 핵심 부분을 시작합니다. 이것이 [메인 루프](#the-main-loop)입니다.

## 메인 루프

이 코드 섹션은 "메인 루프"라고 불리며, 끝에 도달하지 않고 영원히 동일한 명령 집합을 반복하는 역할을 합니다. 여기서 QMK는 키보드가 수행해야 할 모든 작업을 담당하는 함수로 분배됩니다.

메인 루프는 [`protocol_task()`](https://github.com/qmk/qmk_firmware/blob/0.15.13/quantum/main.c#L38)를 호출하며, 이는 [`quantum/keyboard.c`](https://github.com/qmk/qmk_firmware/blob/0.15.13/quantum/keyboard.c#L377)의 `keyboard_task()`를 호출합니다. 여기에서 모든 키보드 관련 기능이 분배되며, 매트릭스의 변화 감지와 상태 LED의 켜짐 및 꺼짐을 담당합니다.

`keyboard_task()` 내에서는 다음과 같은 작업을 처리하는 코드를 찾을 수 있습니다:

* [매트릭스 스캔](#matrix-scanning)
* 마우스 처리
* 키보드 상태 LED (Caps Lock, Num Lock, Scroll Lock)

#### 매트릭스 스캔

매트릭스 스캔은 키보드 펌웨어의 핵심 기능입니다. 현재 어떤 키가 눌려 있는지 감지하는 과정이며, 이 기능은 초당 여러 번 실행됩니다. 펌웨어의 CPU 시간의 99%가 매트릭스 스캔에 사용된다고 해도 과언이 아닙니다.

실제 매트릭스 감지에 대한 다양한 전략이 있지만, 이 문서의 범위를 벗어납니다. 매트릭스 스캔을 블랙 박스로 취급하고, 매트릭스의 현재 상태를 요청하여 다음과 같은 데이터 구조를 얻는 것으로 충분합니다:

```
{
    {0,0,0,0},
    {0,0,0,0},
    {0,0,0,0},
    {0,0,0,0},
    {0,0,0,0}
}
```

이 데이터 구조는 5행 4열의 숫자 패드에 대한 매트릭스를 직접적으로 나타냅니다. 키가 눌리면 해당 키의 매트릭스 내 위치가 `0` 대신 `1`로 반환됩니다.

매트릭스 스캔은 초당 여러 번 실행됩니다. 정확한 속도는 다르지만, 일반적으로 인지할 수 있는 지연을 피하기 위해 최소한 초당 10번 실행됩니다.

##### 매트릭스에서 물리적 레이아웃으로의 매핑

키보드의 각 스위치 상태를 알게 되면 이를 키코드로 매핑해야 합니다. QMK에서는 물리적 레이아웃 정의와 키코드 정의를 분리하기 위해 C 매크로를 사용합니다.

키보드 수준에서는 매트릭스를 물리적 키에 매핑하는 C 매크로(`LAYOUT()`)를 정의합니다. 때로는 매트릭스에 모든 위치에 스위치가 없는 경우도 있으며, 이 매크로를 사용하여 이를 `KC_NO`로 미리 채워 키맵 정의를 쉽게 할 수 있습니다. 다음은 숫자 패드에 대한 `LAYOUT()` 매크로의 예입니다:

```c
#define LAYOUT( \
    k00, k01, k02, k03, \
    k10, k11, k12, k13, \
    k20, k21, k22, \
    k30, k31, k32, k33, \
    k40,      k42 \
) { \
    { k00, k01,   k02, k03   }, \
    { k10, k11,   k12, k13   }, \
    { k20, k21,   k22, KC_NO }, \
    { k30, k31,   k32, k33   }, \
    { k40, KC_NO, k42, KC_NO } \
}
```

두 번째 블록이 매트릭스 스캔 배열과 일치하는 것을 볼 수 있습니까? 이 매크로가 매트릭스 스캔 배열을 키코드에 매핑하는 역할을 합니다. 17키 숫자 패드를 보면 더 큰 키로 인해 매트릭스에 스위치가 없는 3곳이 있습니다. 이를 `KC_NO`로 채워 키맵 정의를 더 쉽게 만들 수 있습니다.

이 매크로를 사용하여 예외적인 매트릭스 레이아웃을 처리할 수도 있습니다. 예를 들어 [Alice](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/keyboards/sneakbox/aliceclone/aliceclone.h#L24). 이를 설명하는 것은 이 문서의 범위를 벗어납니다.

##### 키코드 할당

키맵 수준에서는 앞서 정의한 `LAYOUT()` 매크로를 사용하여 키코드를 물리적 위치에서 매트릭스 위치로 매핑합니다. 다음과 같이 보입니다:

```c
const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
    [0] = LAYOUT(
        KC_NUM,  KC_PSLS, KC_PAST, KC_PMNS,
        KC_P7,   KC_P8,   KC_P9,   KC_PPLS,
        KC_P4,   KC_P5,   KC_P6,
        KC_P1,   KC_P2,   KC_P3,   KC_PENT,
        KC_P0,            KC_PDOT
    )
}
```

모든 인수가 이전 섹션의 `LAYOUT()` 매크로의 첫 번째 절반과 일치하는 것을 볼 수 있습니다. 이렇게 해서 키코드를 이전의 매트릭스 스캔과 매핑합니다.

##### 상태 변화 감지

위에서 설명한 매트릭스 스캔은 주어진 순간에 매트릭스의 상태를 알려줍니다. 하지만 컴퓨터는 현재 상태가 아닌 변경 사항만을 원합니다. QMK는 마지막 매트릭스 스캔의 결과를 저장하고 현재 매트릭스의 결과와 비교하여 키가 눌리거나 릴리스될 때를 감지합니다.

예를 들어 보겠습니다. 키보드 스캔 루프의 중간에 들어가서 이전 스캔이 다음과 같다고 가정합니다:

```
{
    {0,0,0,0},
    {0,0,0,0},
    {0,0,0,0},
    {0,0,0,0},
    {0,0,0,0}
}
```

현재 스캔이 완료되면 다음과 같습니다:

```
{
    {1,0,0,0},
    {0,0,0,0},
    {0,0,0,0},
    {0,0,0,0},
    {0,0,0,0}
}
```

키맵을 비교하면 눌린 키가 `KC_NUM`임을 알 수 있습니다. 여기서부터는 `process_record` 함수 집합

으로 분배됩니다.

<!-- FIXME: Magic happens between here and process_record -->

##### 프로세스 레코드

`process_record()` 함수는 겉보기에는 단순해 보이지만, QMK의 다양한 레벨에서 기능을 재정의할 수 있는 게이트웨이를 숨기고 있습니다. 이벤트 체인은 아래에 나열되어 있으며, 키보드/키맵 레벨 함수를 살펴봐야 할 때는 클루카드를 사용합니다. `rules.mk` 또는 다른 곳에서 설정된 옵션에 따라 아래 함수의 일부만 최종 펌웨어에 포함됩니다.

* [`void action_exec(keyevent_t event)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/action.c#L78-L140)
    * [`void pre_process_record_quantum(keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/quantum.c#L204)
      * [`bool pre_process_record_kb(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/27119fa77e8a1b95fff80718d3db4f3e32849298/quantum/quantum.c#L117)
        * [`bool pre_process_record_user(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/27119fa77e8a1b95fff80718d3db4f3e32849298/quantum/quantum.c#L121)
      * [`bool process_combo(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_combo.c#L521)
  * [`void process_record(keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/action.c#L254)
    * [`bool process_record_quantum(keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/quantum.c#L224)
      * [이 레코드를 키코드로 매핑](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/quantum.c#L225)
      * [`void velocikey_accelerate(void)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/velocikey.c#L27)
      * [`void update_wpm(uint16_t keycode)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/wpm.c#L109)
      * [`void preprocess_tap_dance(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_tap_dance.c#L118)
      * [`bool process_key_lock(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_key_lock.c#L64)
      * [`bool process_dynamic_macro(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_dynamic_macro.c#L160)
      * [`bool process_clicky(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_clicky.c#L84)
      * [`bool process_haptic(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_haptic.c#L87)
      * [`bool process_record_via(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/via.c#L160)
      * [`bool process_record_kb(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/keyboards/planck/ez/ez.c#L271)
        * [`bool process_record_user(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/keyboards/planck/keymaps/default/keymap.c#L183)
      * [`bool process_secure(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_secure.c#L23)
      * [`bool process_sequencer(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_sequencer.c#L19)
      * [`bool process_midi(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_midi.c#L75)
      * [`bool process_audio(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_audio.c#L18)
      * [`bool process_backlight(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_backlight.c#L25)
      * [`bool process_steno(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_steno.c#L159)
      * [`bool process_music(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_music.c#L103)
      * [`bool process_key_override(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/5a1b857dea45a17698f6baa7dd1b7a7ea907fb0a/quantum/process_keycode/process_key_override.c#L397)
      * [`bool process_tap_dance(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_tap_dance.c#L135)
      * [`bool process_caps_word(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_caps_word.c#L17)
      * [`bool process_unicode_common(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_unicode_common.c#L290)
        다음 중 하나를 호출:
          * [`bool process_unicode(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da

02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_unicode.c#L21)
          * [`bool process_unicodemap(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_unicodemap.c#L42)
          * [`bool process_ucis(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_ucis.c#L70)
      * [`bool process_leader(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_leader.c#L48)
      * [`bool process_auto_shift(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_auto_shift.c#L353)
      * [`bool process_dynamic_tapping_term(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_dynamic_tapping_term.c#L35)
      * [`bool process_space_cadet(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_space_cadet.c#L123)
      * [`bool process_magic(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_magic.c#L40)
      * [`bool process_grave_esc(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_grave_esc.c#L23)
      * [`bool process_rgb(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_rgb.c#L53)
      * [`bool process_joystick(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_joystick.c#L9)
      * [`bool process_programmable_button(uint16_t keycode, keyrecord_t *record)`](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/process_keycode/process_programmable_button.c#L21)
      * [Quantum 전용 키코드 식별 및 처리](https://github.com/qmk/qmk_firmware/blob/325da02e57fe7374e77b82cb00360ba45167e25c/quantum/quantum.c#L343)

이 이벤트 체인 동안 언제든지 함수(예: `process_record_kb()`)가 `return false`를 호출하여 모든 추가 처리를 중지할 수 있습니다.

이 호출 후, 추가 정리를 처리하는 `post_process_record()`가 호출됩니다.

* [`void post_process_record(keyrecord_t *record)`]()
  * [`void post_process_record_quantum(keyrecord_t *record)`]()
    * [이 레코드를 키코드로 매핑]()
    * [`void post_process_clicky(uint16_t keycode, keyrecord_t *record)`]()
    * [`void post_process_record_kb(uint16_t keycode, keyrecord_t *record)`]()
      * [`void post_process_record_user(uint16_t keycode, keyrecord_t *record)`]()

<!--
#### 마우스 처리

FIXME: 작성 필요

#### 시리얼 링크

FIXME: 작성 필요

#### 키보드 상태 LED (Caps Lock, Num Lock, Scroll Lock)

FIXME: 작성 필요

-->