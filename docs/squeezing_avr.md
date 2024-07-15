# AVR 최대한 활용하기

AVR은 자원이 매우 제한되어 있으며, QMK가 계속 성장함에 따라 이러한 제약에 맞추기 어려운 새로운 개발을 지원하기 위해 AVR 지원을 레거시 상태로 이동해야 할 수도 있습니다.

하지만 AVR의 제한된 플래시 크기에 맞게 펌웨어 크기를 줄여야 하는 경우 여러 가지 옵션이 있습니다.

## `rules.mk` 설정

가장 먼저 링크 시간 최적화를 활성화하는 것입니다. 이를 위해 `rules.mk`에 다음을 추가하세요:
```make
LTO_ENABLE = yes
```
이렇게 하면 최종 단계에서 시간이 더 걸리지만, 컴파일된 크기가 더 작아질 것입니다. 이 옵션은 Action Functions와 Action Macros도 비활성화하며, 이는 모두 더 이상 사용되지 않는 기능입니다. 대부분의 상황에서 가장 큰 절약 효과를 얻을 수 있습니다.

그 후, 불필요한 시스템을 비활성화하면 도움이 됩니다. 예를 들어:
```make
CONSOLE_ENABLE = no
COMMAND_ENABLE = no
MOUSEKEY_ENABLE = no
EXTRAKEY_ENABLE = no
```
이는 필요하지 않은 일부 기능을 비활성화합니다. 하지만 extrakeys는 미디어 키와 시스템 볼륨 제어와 같은 기능을 비활성화한다는 점을 유의하세요.

이것만으로도 펌웨어 크기를 줄이는 데 충분하지 않다면, 다음과 같은 추가 기능을 비활성화할 수 있습니다:
```make
SPACE_CADET_ENABLE = no
GRAVE_ESC_ENABLE = no 
MAGIC_ENABLE = no
```
이러한 기능은 기본적으로 활성화되어 있지만, 필요하지 않을 수 있습니다. 확인해 보세요. [Magic Keycodes](keycodes_magic)는 가장 크며 NKRO 전환, GUI 및 ALT/CTRL 전환 등을 제어합니다. 이를 비활성화하면 이러한 기능이 비활성화됩니다. 관련 기능을 비활성화하려면 [Magic Functions](#magic-functions)를 참조하세요.

`sprintf` 또는 `snprintf` 함수를 사용하는 경우 약 ~400바이트를 절약할 수 있는 옵션을 활성화할 수 있습니다.
```make
AVR_USE_MINIMAL_PRINTF = yes
```

이 옵션을 활성화하면 AVR의 libc에서 더 작은 구현을 펌웨어에 포함합니다. 이는 [완전한 기능을 제공하지 않습니다](https://www.nongnu.org/avr-libc/user-manual/group__avr__stdio.html). 예를 들어, 0 패딩 및 필드 너비 지정자는 지원되지 않습니다. 따라서 다음과 같이 `sprintf` 또는 `snprintf`를 사용하는 경우:
```c
sprintf(wpm_str, "%03d", get_current_wpm());
snprintf(keylog_str, sizeof(keylog_str), "%dx%d, k%2d : %c");
```

표준 구현이 여전히 필요합니다.

## `config.h` 설정

이 모든 것을 수행했지만 RGB, 오디오, OLED 등을 비활성화하지 않으려는 경우, 도움이 될 수 있는 추가 옵션이 있습니다.

우선 Lock Key 지원입니다. Cherry MX Lock 스위치가 있다면, 이 작업을 하지 않는 것이 좋습니다. 하지만 그렇지 않은 경우, `config.h`에 다음을 추가하세요:
```c
#undef LOCKING_SUPPORT_ENABLE
#undef LOCKING_RESYNC_ENABLE
```
Oneshots를 사용하지 않는 경우, 다음을 `config.h`에 추가하여 기능을 비활성화할 수 있습니다:
```c
#define NO_ACTION_ONESHOT
```
Tapping 키(모드 탭, 레이어 탭 등)를 사용하지 않는 경우에도 동일하게 할 수 있습니다:
```c
#define NO_ACTION_TAPPING
```
## 오디오 설정

오디오 기능을 사용하는 경우, 기본적으로 음악 모드 기능도 포함됩니다. 이 기능은 매트릭스 위치를 노트로 변환합니다. 정말 멋진 기능이지만, 대부분의 경우 사용하지 않을 것입니다. 이를 비활성화하려면 `config.h`에 다음을 추가하세요:
```c
#define NO_MUSIC_MODE
```
그리고 `rules.mk`에 다음을 추가하세요:
```make
MUSIC_ENABLE = no
```

## 레이어

펌웨어 크기를 줄이기 위한 레이어 옵션도 있습니다. 이러한 설정은 모두 `config.h`에 적용됩니다.

펌웨어가 사용하는 레이어 수를 제한할 수 있습니다. 최대 8개의 레이어를 사용하는 경우:
```c
#define LAYER_STATE_8BIT
```
또는 최대 16개의 레이어가 필요한 경우:
```c
#define LAYER_STATE_16BIT
```
레이어를 전혀 사용하지 않는 경우, 해당 기능을 완전히 제거할 수도 있습니다:
```c
#define NO_ACTION_LAYER
```

## 매직 함수

매직 키코드를 사용자 정의하기 위한 두 개의 `__attribute__ ((weak))` 플레이스홀더 함수가 있습니다. 백슬래시를 백스페이스로 교환하는 등 키코드를 교환하는 기능을 사용하지 않는다면, `keymap.c` 또는 사용자 공간 코드에 다음을 추가하세요:
```c
#ifndef MAGIC_ENABLE
uint16_t keycode_config(uint16_t keycode) {
    return keycode;
}
#endif
```
또한, Control을 GUI로 교환하는 등 매직 키코드를 사용하여 수정자를 교환하지 않는 경우, `keymap.c` 또는 사용자 공간 코드에 다음을 추가하세요:
```c
#ifndef MAGIC_ENABLE
uint8_t mod_config(uint8_t mod) {
    return mod;
}
#endif
```
두 함수 모두 플레이스홀더 함수를 간단한 반환문으로 덮어써서 펌웨어 크기를 줄입니다.

## OLED 조정

여기서 많은 공간을 절약할 수 있는 한 가지 방법은 `sprintf` 또는 `snprintf`를 사용하지 않는 것입니다. 이 함수 호출은 펌웨어 공간에서 약 1.5kB를 차지하며, 이를 다시 작성할 수 있습니다. 예를 들어, WPM은 이를 많이 사용합니다.

다음을:
```c
    // OLD CODE
    char wpm_str[4] = {0};
    sprintf(wpm_str, "WPM: %03d", get_current_wpm());
    oled_write(wpm_str, ' '), false);
```
다음으로 변환할 수 있습니다:
```c
    // NEW CODE
    oled_write_P(PSTR("WPM: "), false);
    oled_write(get_u8_str(get_current_wpm(), ' '), false);
```
이는 `WPM:   5`를 출력합니다. 또는:
```c
    // NEW CODE
    oled_write_P(PSTR("WPM: "), false);
    oled_write(get_u8_str(get_current_wpm(), '0'), false);
```
이는 `WPM: 005`를 출력합니다.

## RGB 설정

보드에서 RGB를 사용하는 경우, RGB Light(언더글로우)와 RGB Matrix(각 키 RGB) 모두 다른 애니메이션을 활성화하려면 정의가 필요합니다. 일부 키보드는 기본적으로 많은 애니메이션을 활성화하므로, 사용하지 않는 특정 애니메이션을 비활성화하면 약간의 공간을 확보할 수 있습니다. RGB Light의 경우 `config.h`에서 이를 비활성화할 수 있습니다:
```c
#undef RGBLIGHT_ANIMATIONS
#undef RGBLIGHT_EFFECT_BREATHING
#undef RGBLIGHT_EFFECT_RAINBOW_MOOD
#undef RGBLIGHT_EFFECT_RAINBOW_SWIRL
#undef RGBLIGHT_EFFECT_SNAKE
#undef RGBLIGHT_EFFECT_KNIGHT
#undef RGBLIGHT_EFFECT_CHRISTMAS
#undef RGBLIGHT_EFFECT_STATIC_GRADIENT
#undef RGBLIGHT_EFFECT_RGB_TEST
#undef RGBLIGHT_EFFECT_ALTERNATING
#undef RGBLIGHT_EFFECT_TWINKLE
```

RGB Matrix의 경우, 이를 명시적으로 활성화해야 합니다. 키보드에서 활성화된 항목을 비활성화하려면 키맵의 `config.h`에 다음 중 하나 이상을 추가하세요:
```c
#undef ENABLE_RGB_MATRIX_ALPHAS_MODS
#undef ENABLE_RGB_MATRIX_GRADIENT_UP_DOWN
#undef ENABLE_RGB_MATRIX_GRADIENT_LEFT_RIGHT
#undef ENABLE_RGB_MATRIX_BREATHING
#undef ENABLE_RGB_MATRIX_BAND_SAT
#undef ENABLE_RGB_MATRIX_BAND_VAL
#undef ENABLE_RGB_MATRIX_BAND_PINWHEEL_SAT
#undef ENABLE_RGB_MATRIX_BAND_PINWHEEL_VAL
#undef ENABLE_RGB_MATRIX_BAND_SPIRAL_SAT
#undef ENABLE_RGB_MATRIX_BAND_SPIRAL_VAL
#undef ENABLE_RGB_MATRIX_CYCLE_ALL
#undef ENABLE_RGB_MATRIX_CYCLE_LEFT_RIGHT
#undef ENABLE_RGB_MATRIX_CYCLE_UP_DOWN
#undef ENABLE_RGB_MATRIX_RAINBOW_MOVING_CHEVRON
#undef ENABLE_RGB_MATRIX_CYCLE_OUT_IN
#undef ENABLE_RGB_MATRIX_CYCLE_OUT_IN_DUAL
#undef ENABLE_RGB_MATRIX_CYCLE_PINWHEEL
#undef ENABLE_RGB_MATRIX_CYCLE_SPIRAL
#undef ENABLE_RGB_MATRIX_DUAL_BEACON
#undef ENABLE_RGB_MATRIX_RAINBOW_BEACON
#undef ENABLE_RGB_MATRIX_RAINBOW_PINWHEELS
#undef ENABLE_RGB_MATRIX_FLOWER_BLOOMING
#undef ENABLE_RGB_MATRIX_RAINDROPS
#undef ENABLE_RGB_MATRIX_JELLYBEAN_RAINDROPS
#undef ENABLE_RGB_MATRIX_HUE_BREATHING
#undef ENABLE_RGB_MATRIX_HUE_PENDULUM
#undef ENABLE_RGB_MATRIX_HUE_WAVE
#undef ENABLE_RGB_MATRIX_PIXEL_FRACTAL
#undef ENABLE_RGB_MATRIX_PIXEL_FLOW
#undef ENABLE_RGB_MATRIX_PIXEL_RAIN

#undef ENABLE_RGB_MATRIX_TYPING_HEATMAP
#undef ENABLE_RGB_MATRIX_DIGITAL_RAIN



#undef ENABLE_RGB_MATRIX_SOLID_REACTIVE_SIMPLE
#undef ENABLE_RGB_MATRIX_SOLID_REACTIVE
#undef ENABLE_RGB_MATRIX_SOLID_REACTIVE_WIDE
#undef ENABLE_RGB_MATRIX_SOLID_REACTIVE_MULTIWIDE
#undef ENABLE_RGB_MATRIX_SOLID_REACTIVE_CROSS
#undef ENABLE_RGB_MATRIX_SOLID_REACTIVE_MULTICROSS
#undef ENABLE_RGB_MATRIX_SOLID_REACTIVE_NEXUS
#undef ENABLE_RGB_MATRIX_SOLID_REACTIVE_MULTINEXUS
#undef ENABLE_RGB_MATRIX_SPLASH
#undef ENABLE_RGB_MATRIX_MULTISPLASH
#undef ENABLE_RGB_MATRIX_SOLID_SPLASH
#undef ENABLE_RGB_MATRIX_SOLID_MULTISPLASH
```

# 최종 생각

이 모든 작업을 수행했음에도 불구하고 펌웨어가 여전히 너무 크다면, ARM으로 전환하는 것을 고려해야 할 때입니다. 다음은 ARM 컨트롤러가 있는 Pro Micro 교체품의 예입니다:
* [Bonsai C](https://github.com/customMK/Bonsai-C) (오픈 소스, DIY/PCBA)
* [STeMCell](https://github.com/megamind4089/STeMCell) (오픈 소스, DIY/PCBA)
* [Adafruit KB2040](https://learn.adafruit.com/adafruit-kb2040)
* [SparkFun Pro Micro - RP2040](https://www.sparkfun.com/products/18288)
* [Blok](https://boardsource.xyz/store/628b95b494dfa308a6581622)
* [Elite-Pi](https://keeb.io/products/elite-pi-usb-c-pro-micro-replacement-rp2040)
* [0xCB Helios](https://keeb.supply/products/0xcb-helios) ([오픈 소스](https://github.com/0xCB-dev/0xCB-Helios), DIY/PCBA/Shop)
* [Liatris](https://splitkb.com/products/liatris)
* [Imera](https://splitkb.com/products/imera)
* [Michi](https://github.com/ci-bus/michi-promicro-rp2040)
* [Proton C](https://qmk.fm/proton-c/) (품절)

다른 비-Pro Micro 호환 보드도 있습니다. 가장 인기 있는 것은:
* [WeAct Blackpill F411](https://www.aliexpress.com/item/1005001456186625.html) (~$6 USD)