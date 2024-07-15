# 오디오

키보드가 소리를 낼 수 있습니다! 여분의 핀이 있다면 간단한 스피커를 연결하여 소리를 내게 할 수 있습니다. 이 소리를 사용하여 레이어 전환, 수정 키, 특수 키를 나타내거나 단순히 재미있는 8비트 음악을 재생할 수 있습니다.

이 기능을 활성화하려면 `rules.mk`에 `AUDIO_ENABLE = yes`를 추가하세요.

## AVR 기반 보드

Atmega32U4 기반 보드에서는 최대 두 개의 동시 톤을 렌더링할 수 있습니다. 한 스피커는 타이머 3에 의해 구동되는 PORTC의 PWM 가능한 핀에 연결하고, 다른 스피커는 타이머 1에 의해 구동되는 PORTB의 PWM 핀 중 하나에 연결할 수 있습니다.

다음 핀은 `config.h`에서 오디오 출력으로 구성할 수 있습니다. 한 스피커를 설정하려면 다음 중 하나를 설정하세요:

* `#define AUDIO_PIN C4`
* `#define AUDIO_PIN C5`
* `#define AUDIO_PIN C6`
* `#define AUDIO_PIN B5`
* `#define AUDIO_PIN B6`
* `#define AUDIO_PIN B7`

그리고 *선택적으로*, 두 번째 스피커를 위해 다음 중 하나를 설정할 수 있습니다:

* `#define AUDIO_PIN_ALT B5`
* `#define AUDIO_PIN_ALT B6`
* `#define AUDIO_PIN_ALT B7`

### 배선

예를 들어 피에조 부저를 사용할 때, 각 스피커의 검은색 리드를 그라운드에 연결하고 빨간색 리드를 선택한 AUDIO_PIN에 연결합니다. 두 번째 스피커도 마찬가지로 AUDIO_PIN_ALT에 연결합니다.

## ARM 기반 보드

기술적 세부 사항에 대해서는 [오디오 드라이버](../drivers/audio)에 대한 설명을 참조하세요.

### DAC (기본)

대부분의 STM32 MCU에는 DAC 주변 장치가 있지만, STM32F1xx 시리즈는 예외입니다. 일반적으로 DAC 주변 장치는 A4 또는 A5 핀을 구동합니다. STM32 장치에서 DAC 기반 오디오 출력을 활성화하려면 `rules.mk`에 `AUDIO_DRIVER = dac_basic`을 추가하고 `config.h`에 다음 중 하나를 설정합니다:

`#define AUDIO_PIN A4` 또는 `#define AUDIO_PIN A5`

다른 DAC 채널은 선택적으로 두 번째 스피커에 사용할 수 있으며, 다음을 설정합니다:

`#define AUDIO_PIN_ALT A4` 또는 `#define AUDIO_PIN_ALT A5`

dac_basic 드라이버는 한 번에 하나의 톤만 재생할 수 있으며, 더 많은 톤을 동시에 재생하려면 dac_additive 드라이버를 사용해 보세요.

#### 배선

예를 들어 `AUDIO_PIN A4`와 `AUDIO_PIN_ALT A5`로 구성된 두 개의 피에조를 사용하려면: 빨간색 리드를 A4에, 검은색 리드를 그라운드에 연결합니다. 두 번째 스피커도 마찬가지로 A5는 빨간색, 그라운드는 검은색으로 연결합니다.

또 다른 방법으로, 하나의 피에조를 두 개의 DAC 핀으로 구동할 수도 있습니다. 빨간색 리드를 A4에, 검은색 리드를 A5에 연결하고 `#define AUDIO_PIN_ALT_AS_NEGATIVE`를 `config.h`에 추가합니다.

##### Proton-C 예제

Proton-C는 (옵션으로) 하나의 '내장' 피에조를 제공하며, 이는 A4+A5에 연결됩니다. 이 보드의 `config.h`는 다음과 같이 구성됩니다:

```c
#define AUDIO_PIN A5
#define AUDIO_PIN_ALT A4
#define AUDIO_PIN_ALT_AS_NEGATIVE
```

### DAC (additive)

dac_basic (사각파를 통해 소리를 재생하는 것) 외에 또 다른 옵션은 DAC를 사용하여 추가적 파형 합성을 수행하는 것입니다. 여러 개의 사전 정의된 파형 또는 샘플을 실시간으로 생성하는 구현을 제공합니다. 이 기능을 사용하려면 `rules.mk`에 `AUDIO_DRIVER = dac_additive`를 설정하고 `config.h`에 다음 중 하나를 설정합니다: `#define AUDIO_PIN A4` 또는 `#define AUDIO_PIN A5`.

사용되는 파형은 기본적으로 사인파이지만, 다른 파형을 선택할 수 있습니다. `config.h`에 다음 정의 중 하나를 추가합니다:

* `#define AUDIO_DAC_SAMPLE_WAVEFORM_SINE`
* `#define AUDIO_DAC_SAMPLE_WAVEFORM_TRIANGLE`
* `#define AUDIO_DAC_SAMPLE_WAVEFORM_TRAPEZOID`
* `#define AUDIO_DAC_SAMPLE_WAVEFORM_SQUARE`

DAC 유닛으로 자체 샘플 테이블을 생성하고 사용하는 것을 선택할 수도 있습니다. `uint16_t dac_value_generate(void)`을 키보드에서 구현합니다. 예제 구현은 keyboards/planck/keymaps/synth_sample 또는 keyboards/planck/keymaps/synth_wavetable를 참조하세요.

### PWM (소프트웨어)

DAC 핀이 사용 불가능하거나 MCU에 사용할 수 있는 DAC가 없을 경우 (STM32F1xx와 같이), PWM이 대안이 될 수 있습니다. 현재는 하나의 스피커/핀만 지원됩니다.

`rules.mk`에 `AUDIO_DRIVER = pwm_software`를 설정하고 `config.h`에 `#define AUDIO_PIN C13` (아무 핀이든 가능)으로 설정하여 선택한 핀이 타이머 콜백에서 소프트웨어로 토글되어 PWM 신호를 출력하도록 합니다.

#### 배선

일반적인 피에조 배선: 빨간색 리드는 선택한 AUDIO_PIN에, 검은색 리드는 그라운드에 연결합니다.

또는 하나의 피에조를 두 핀으로 구동할 수 있습니다. 예를 들어 `#define AUDIO_PIN B1`, `#define AUDIO_PIN_ALT B2`를 `config.h`에 설정하고 `#define AUDIO_PIN_ALT_AS_NEGATIVE`를 설정합니다. 빨간색 리드를 B1에, 검은색 리드를 B2에 연결합니다.

### PWM (하드웨어)

STM32F1xx는 PWM을 하드웨어에서 사용해야 하지만, 현재는 하나의 스피커/핀만 지원됩니다.

`rules.mk`에 `AUDIO_DRIVER = pwm_hardware`를 설정하고 `config.h`에 다음을 설정합니다:
`#define AUDIO_PIN A8`
`#define AUDIO_PWM_DRIVER PWMD1`
`#define AUDIO_PWM_CHANNEL 1`
(STM32F2 이상의 경우 `#define AUDIO_PWM_PAL_MODE 42`도 설정) 이 설정은 타이머 1을 사용하여 PA8 핀을 PWM 하드웨어를 통해 직접 구동합니다 (TIM1_CH1 = PA8). 다른 핀과 타이머에서 PWM 하드웨어를 사용하려면 STM32 데이터 시트를 참조하여 적절한 TIMx_CHy와 핀 대체 기능을 선택하세요.

## 톤 멀티플렉싱

대부분의 드라이버는 한 번에 하나의 톤만 렌더링할 수 있으며 (단 하나의 예외: ARM dac-additive), 톤 멀티플렉싱이라는 "워크어라운드 기능"이 존재합니다. 이 기능은 이름에서 알 수 있듯이, 활성 톤 세트를 주기적으로 순환하여 하나의 톤씩 하나의 스피커 또는 몇 개의 스피커를 통해 출력합니다.

이 기능을 활성화하고 시작 속도를 구성하려면 `config.h`에 다음 정의를 추가하세요:

```c
#define AUDIO_ENABLE_TONE_MULTIPLEXING
#define AUDIO_TONE_MULTIPLEXING_RATE_DEFAULT 10
```

오디오 코어는 `keymap.c`에서 톤 멀티플렉싱 속도를 가져오고 설정하고 변경할 수 있는 인터페이스 함수를 제공합니다.

## 노래

다음은 추가 구성 없이 자동으로 활성화되는 몇 가지 다른 사운드입니다:

```
STARTUP_SONG // 키보드 시작 시 재생 (audio.c)
GOODBYE_SONG // QK_BOOT 키를 누를 때 재생 (quantum.c)
AG_NORM_SONG // AG_NORM을 누를 때 재생 (quantum.c)
AG_SWAP_SONG // AG_SWAP을 누를 때 재생 (quantum.c)
CG_NORM_SONG // CG_NORM을 누를 때 재생 (quantum.c)
CG_SWAP_SONG // CG_SWAP을 누를 때 재생 (quantum.c)
MUSIC_ON_SONG // 음악 모드가 활성화될 때 재생 (process_music.c)
MUSIC_OFF_SONG // 음악 모드가 비활성화될 때 재생 (process_music.c)
CHROMATIC_SONG // 크로마틱 음악 모드가 선택될 때 재생 (process_music.c)
GUITAR_SONG // 기타 음악 모드가 선택될 때 재생 (process_music.c)
VIOLIN_SONG // 바이올린 음악 모드가 선택될 때 재생 (process_music.c)
MAJOR_SONG // 메이저 음악 모드가 선택될 때 재생 (process_music.c)
```

기본 노래를 재정의하려면 `config.h`에서 다음과 같이 설정할 수 있습니다:

```c
#ifdef AUDIO_ENABLE


# define STARTUP_SONG SONG(STARTUP_SOUND)
#endif
```

모든 사운드의 전체 목록은 [quantum/audio/song_list.h](https://github.com/qmk/qmk_firmware/blob/master/quantum/audio/song_list.h)에서 확인할 수 있습니다. 이 목록에 자신만의 노래를 추가하세요! 사용할 수 있는 모든 음표는 [quantum/audio/musical_notes.h](https://github.com/qmk/qmk_firmware/blob/master/quantum/audio/musical_notes.h)에서 확인할 수 있습니다.

추가로, (저작권이 있는 노래와 같은) 자신의 노래 목록을 유지하고 싶고, 리포지토리에 추가되지 않도록 하려면 `user_song_list.h` 파일을 생성하여 keymap 폴더 (또는 사용자 폴더)에 놓으세요. 이 파일은 자동으로 포함됩니다.

특정 시간에 사용자 정의 소리를 재생하려면 파일 상단 근처에 다음과 같이 노래를 정의할 수 있습니다:

```c
float my_song[][2] = SONG(QWERTY_SOUND);
```

그리고 다음과 같이 노래를 재생하세요:

```c
PLAY_SONG(my_song);
```

또는 다음과 같이 반복 재생할 수도 있습니다:

```c
PLAY_LOOP(my_song);
```

오디오 기능을 사용하지 않을 때 문제가 발생하지 않도록 `#ifdef AUDIO_ENABLE` / `#endif`로 모든 오디오 기능을 감싸는 것이 좋습니다.

사용 가능한 오디오 키코드는 다음과 같습니다:

| 키               | 별칭      | 설명                        |
| ----------------- | --------- | ---------------------------- |
| `QK_AUDIO_ON`     | `AU_ON`   | 오디오 기능 켜기            |
| `QK_AUDIO_OFF`    | `AU_OFF`  | 오디오 기능 끄기            |
| `QK_AUDIO_TOGGLE` | `AU_TOGG` | 오디오 상태 전환            |

::: 경고
이 키코드는 모든 오디오 기능을 켜고 끕니다. 오디오 기능을 끄면 오디오 피드백, 오디오 클릭, 음악 모드 등이 완전히 비활성화됩니다.
:::

## 오디오 구성

| 설정                              | 기본값           | 설명                                                      |
| --------------------------------- | ----------------- | ---------------------------------------------------------- |
| `AUDIO_PIN`                       | *정의되지 않음*   | 스피커가 연결된 핀을 설정합니다.                           |
| `AUDIO_PIN_ALT`                   | *정의되지 않음*   | 두 번째 스피커 또는 하나의 스피커에 연결된 두 번째 핀을 설정합니다. |
| `AUDIO_PIN_ALT_AS_NEGATIVE`       | *정의되지 않음*   | 하나의 스피커가 두 핀에 연결된 것을 지원합니다.           |
| `AUDIO_INIT_DELAY`                | *정의되지 않음*   | USB 시작 문제를 고려하여 시작 노래 동안 지연을 활성화합니다. |
| `AUDIO_ENABLE_TONE_MULTIPLEXING`  | *정의되지 않음*   | 동시에 여러 톤을 생성하기 위해 시간 분할/멀티플렉싱을 활성화합니다. |
| `AUDIO_POWER_CONTROL_PIN`         | *정의되지 않음*   | 스피커에 전원을 공급하거나 차단하는 전원 제어 코드를 활성화합니다 (예: PAM8302 앰프). |
| `AUDIO_POWER_CONTROL_PIN_ON_STATE` | `1`               | 오디오가 "켜진" 상태일 때 오디오 전원 제어 핀의 상태입니다. - `1`은 고전압, `0`은 저전압. |
| `STARTUP_SONG`                    | `STARTUP_SOUND`   | 키보드가 시작될 때 재생되는 노래 (audio.c)                  |
| `GOODBYE_SONG`                    | `GOODBYE_SOUND`   | QK_BOOT 키를 누를 때 재생되는 노래 (quantum.c)             |
| `AG_NORM_SONG`                    | `AG_NORM_SOUND`   | AG_NORM을 누를 때 재생되는 노래 (process_magic.c)          |
| `AG_SWAP_SONG`                    | `AG_SWAP_SOUND`   | AG_SWAP을 누를 때 재생되는 노래 (process_magic.c)          |
| `CG_NORM_SONG`                    | `AG_NORM_SOUND`   | CG_NORM을 누를 때 재생되는 노래 (process_magic.c)          |
| `CG_SWAP_SONG`                    | `AG_SWAP_SOUND`   | CG_SWAP을 누를 때 재생되는 노래 (process_magic.c)          |
| `MUSIC_ON_SONG`                   | `MUSIC_ON_SOUND`  | 음악 모드가 활성화될 때 재생되는 노래 (process_music.c)     |
| `MUSIC_OFF_SONG`                  | `MUSIC_OFF_SOUND` | 음악 모드가 비활성화될 때 재생되는 노래 (process_music.c)   |
| `MIDI_ON_SONG`                    | `MUSIC_ON_SOUND`  | MIDI 모드가 활성화될 때 재생되는 노래 (process_music.c)     |
| `MIDI_OFF_SONG`                   | `MUSIC_OFF_SOUND` | MIDI 모드가 비활성화될 때 재생되는 노래 (process_music.c)   |
| `CHROMATIC_SONG`                  | `CHROMATIC_SOUND` | 크로마틱 음악 모드가 선택될 때 재생되는 노래 (process_music.c) |
| `GUITAR_SONG`                     | `GUITAR_SOUND`    | 기타 음악 모드가 선택될 때 재생되는 노래 (process_music.c) |
| `VIOLIN_SONG`                     | `VIOLIN_SOUND`    | 바이올린 음악 모드가 선택될 때 재생되는 노래 (process_music.c) |
| `MAJOR_SONG`                      | `MAJOR_SOUND`     | 메이저 음악 모드가 선택될 때 재생되는 노래 (process_music.c) |
| `DEFAULT_LAYER_SONGS`             | *정의되지 않음*   | [`set_single_persistent_default_layer(layer)`](../ref_functions#setting-the-persistent-default-layer) (quantum.c)을 사용하여 기본 레이어를 전환할 때 재생되는 노래 |
| `SENDSTRING_BELL`                 | *정의되지 않음*   | "enter" ("\a") 문자가 전송될 때 벨 소리가 재생됩니다 (send_string.c) |

## 템포

SONG이 재생되는 속도는 설정된 템포에 따라 결정되며, 템포는 분당 비트 수(beats-per-minute)로 측정됩니다. 음표 길이는 이에 상대적으로 정의됩니다. 초기/기본 템포는 120bpm으로 설정되어 있지만, `config.c`에서 `TEMPO_DEFAULT`를 설정하여 변경할 수 있습니다. 또한 사용자/keymap 코드 내에서 템포를 수정하기 위한 함수가 제공됩니다:

```c
void audio_set_tempo(uint8_t tempo);
void audio_increase_tempo(uint8_t tempo_change);
void audio_decrease_tempo(uint8_t tempo_change);
```

## ARM 오디오 볼륨

ARM 장치의 경우, DAC 샘플 값을 조정할 수 있습니다. 보드가 너무 시끄럽거나 동료들에게 방해가 될 경우, `config.h`에 `AUDIO_DAC_SAMPLE_MAX`를 설정하여 최대값을 설정할 수 있습니다:

```c
#define AUDIO_DAC_SAMPLE_MAX 4095U
```

DAC는 일반적으로 12비트 모드로 작동하므로 볼륨 100% = 4095U입니다.

참고: 이는 오직 WAVEFORM_SQUARE에만 적용됩니다. 다른 파형은 하드코딩된/미리 계산된 샘플 버퍼를 사용하기 때문입니다.

## 음성

"오디오 효과"라고도 하며, 이를 설정하려면 `config.h`에 `#define AUDIO_VOICES`를 설정하고, 기본 효과를 선택하려면 `#define AUDIO_VOICE_DEFAULT something`를 설정하세요.
자세한 내용은 quantum/audio/voices.h 및 .c를 참조하세요.

사용 가능한 키코드는 다음과 같습니다:

| 키                       | 별칭      | 설명                        |
| ------------------------- | --------- | ---------------------------- |
| `QK_AUDIO_VOICE_NEXT`     | `AU_NEXT` | 오디오 음성을 순환합니다.   |
| `QK_AUDIO_VOICE_PREVIOUS` | `AU_PREV` | 오디오 음성을 역순으로 순환합니다. |

## 음악 모드

음악 모드는 열을 반음계로, 행을 옥타브로 매핑합니다. 이는 직선 배열 키보드에 가장 적합하지만, 다른 키보드에서도 작동할 수 있습니다. `0xFF`보다 작은 모든 키코드는 차단되므로 노트를 재생하는 동안 타이핑하지 않습니다. 특수 키/수정 키가 있는 경우, 이러한 키는 여전히 작동합니다. 이를 해결하기 위해 KC_NO로 된 다른 레이어로 전환하거나 (또는 후) 음악 모드를 활성화할 수 있습니다.

녹음 기능은 메모리 문제로 인해 실험적입니다. 이상한 행동을 경험하면 키보드를 뽑았다가 다시 꽂으면 문제를 해결할 수 있습니다.

사용 가능한 키코드는 다음과

 같습니다:

| 키                  | 별칭      | 설명                        |
| -------------------- | --------- | ---------------------------- |
| `QK_MUSIC_ON`        | `MU_ON`   | 음악 모드를 켭니다.         |
| `QK_MUSIC_OFF`       | `MU_OFF`  | 음악 모드를 끕니다.         |
| `QK_MUSIC_TOGGLE`    | `MU_TOGG` | 음악 모드를 전환합니다.     |
| `QK_MUSIC_MODE_NEXT` | `MU_NEXT` | 음악 모드를 순환합니다.     |

사용 가능한 모드는 다음과 같습니다:

* `CHROMATIC_MODE` - 반음계, 행이 옥타브를 변경합니다.
* `GUITAR_MODE` - 반음계, 행이 줄을 변경합니다 (+5 st).
* `VIOLIN_MODE` - 반음계, 행이 줄을 변경합니다 (+7 st).
* `MAJOR_MODE` - 장조 음계

음악 모드에서는 다음 키코드가 다르게 작동하며, 통과되지 않습니다:

* `LCTL` - 녹음 시작
* `LALT` - 녹음/재생 중지
* `LGUI` - 녹음 재생
* `KC_UP` - 재생 속도 증가
* `KC_DOWN` - 재생 속도 감소

음높이 표준(`PITCH_STANDARD_A`)은 기본적으로 440.0f로 설정되어 있습니다. 이를 변경하려면 `config.h`에 다음과 같이 추가하세요:

```c
#define PITCH_STANDARD_A 432.0f
```

음악 모드를 완전히 비활성화할 수도 있습니다. 이는 컨트롤러의 공간이 부족할 경우 유용합니다. 비활성화하려면 `config.h`에 다음을 추가하세요:

```c
#define NO_MUSIC_MODE
```

### 음악 마스크

기본적으로 `MUSIC_MASK`는 `keycode < 0xFF`로 설정되어 있어, `0xFF`보다 작은 키코드는 노트로 변환되며, 아무것도 출력되지 않습니다. 이를 변경하려면 `config.h`에 다음과 같이 정의하세요:

```c
#define MUSIC_MASK keycode != KC_NO
```

이는 모든 키코드를 캡처합니다. 이 경우 음악 모드에 갇히게 되므로 주의하세요!

키코드를 처리할지를 제어하는 더 고급 방법으로는 `<keyboard>.c`의 `music_mask_kb(keycode)`와 `keymap.c`의 `music_mask_user(keycode)`를 사용할 수 있습니다:

```c
bool music_mask_user(uint16_t keycode) {
    switch (keycode) {
        case RAISE:
        case LOWER:
            return false;
        default:
            return true;
    }
}
```

거짓을 반환하는 항목은 마스크의 일부가 아니며, 항상 처리됩니다.

### 음악 맵

기본적으로 음악 모드는 키의 음계를 결정하기 위해 열과 행을 사용합니다. 직사각형 매트릭스를 사용하는 키보드 레이아웃에서는 괜찮지만, 복잡한 매트릭스를 사용하는 키보드(예: Planck Rev6 또는 많은 스플릿 키보드)에서는 경험이 왜곡될 수 있습니다.

하지만 음악 맵 옵션을 사용하면 음악 모드의 스케일을 재매핑하여 레이아웃에 맞추고 더 자연스럽게 만들 수 있습니다.

이 기능을 활성화하려면 `config.h`에 `#define MUSIC_MAP`를 추가하고, 키보드의 `c` 파일이나 `keymap.c`에 `uint8_t music_map`를 추가합니다.

```c
const uint8_t music_map[MATRIX_ROWS][MATRIX_COLS] = LAYOUT_ortho_4x12(
    36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47,
    24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35,
    12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23,
     0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11
);
```

사용하는 `LAYOUT` 매크로를 사용하여 이를 설정합니다. 올바른 키 위치에 매핑합니다. 키보드 레이아웃의 왼쪽 아래에서 시작하여 오른쪽으로 이동한 다음 위로 이동합니다. 모든 항목을 채울 때까지 행렬을 완성합니다.

Planck 키보드를 구현하는 예제는 [Planck Keyboard](https://github.com/qmk/qmk_firmware/blob/e9ace1487887c1f8b4a7e8e6d87c322988bec9ce/keyboards/planck/planck.c#L24-L29)를 참조하세요.

## 오디오 클릭

이 기능은 키를 누를 때마다 클릭 소리를 추가하여 키보드의 클릭 소리를 시뮬레이트합니다. 빠르게 타이핑해도 소리가 긴 노트처럼 들리지 않도록 각 키 누름마다 소리가 약간 다릅니다.

사용 가능한 키코드는 다음과 같습니다:

| 키                      | 별칭      | 설명                        |
| ------------------------ | --------- | ---------------------------- |
| `QK_AUDIO_CLICKY_TOGGLE` | `CK_TOGG` | 오디오 클릭 모드를 전환합니다. |
| `QK_AUDIO_CLICKY_ON`     | `CK_ON`   | 오디오 클릭 모드를 켭니다.  |
| `QK_AUDIO_CLICKY_OFF`    | `CK_OFF`  | 오디오 클릭 모드를 끕니다.  |
| `QK_AUDIO_CLICKY_UP`     | `CK_UP`   | 클릭 소리의 주파수를 증가시킵니다. |
| `QK_AUDIO_CLICKY_DOWN`   | `CK_DOWN` | 클릭 소리의 주파수를 감소시킵니다. |
| `QK_AUDIO_CLICKY_RESET`  | `CK_RST`  | 주파수를 기본값으로 재설정합니다. |

이 기능은 기본적으로 비활성화되어 있습니다. 이를 활성화하려면 `config.h`에 다음을 추가하세요:

```c
#define AUDIO_CLICKY
```

기본, 최소 및 최대 주파수, 단계 및 내장된 무작위성을 구성하려면 다음 값을 정의하세요:

| 옵션                         | 기본값          | 설명                                                      |
| ---------------------------- | --------------- | --------------------------------------------------------- |
| `AUDIO_CLICKY_FREQ_DEFAULT`  | 440.0f          | 클릭 소리의 기본/시작 오디오 주파수를 설정합니다.         |
| `AUDIO_CLICKY_FREQ_MIN`      | 65.0f           | 가장 낮은 주파수를 설정합니다 (60f 이하에서는 버그가 발생할 수 있습니다). |
| `AUDIO_CLICKY_FREQ_MAX`      | 1500.0f         | 가장 높은 주파수를 설정합니다. 너무 높으면 동료들이 공격할 수 있습니다. |
| `AUDIO_CLICKY_FREQ_FACTOR`   | 1.18921f        | UP/DOWN 키 코드의 단계를 설정합니다. 이는 곱셈 계수입니다. 기본값은 주파수를 음악적 소음(작은 3도)으로 증가/감소시킵니다. |
| `AUDIO_CLICKY_FREQ_RANDOMNESS` | 0.05f         | 클릭 소리의 무작위성을 설정합니다. 이 값을 `0f`로 설정하면 각 클릭 소리가 동일하게 들리며, `1.0f`로 설정하면 90년대 컴퓨터 화면 스크롤/타이핑 효과처럼 들립니다. |
| `AUDIO_CLICKY_DELAY_DURATION` | 1              | 지연 기간을 정수로 설정하며, 1은 템포의 1/16 또는 64분 음표를 나타냅니다 (구현 세부 사항은 `quantum/audio/musical_notes.h`를 참조하세요). 주 클릭 효과는 이 기간 동안 지연됩니다. 클릭 소리가 큰 스위치에 대해 보정하기 위해 이 값을 6-12 사이로 조정하는 것이 좋습니다. |

## MIDI 기능

[MIDI](midi)를 참조하세요.

## 오디오 키코드

| 키                       | 별칭      | 설명                        |
| ------------------------- | --------- | ---------------------------- |
| `QK_AUDIO_ON`             | `AU_ON`   | 오디오 기능 켜기            |
| `QK_AUDIO_OFF`            | `AU_OFF`  | 오디오 기능 끄기            |
| `QK_AUDIO_TOGGLE`         | `AU_TOGG` | 오디오 상태 전환            |
| `QK_AUDIO_CLICKY_TOGGLE`  | `CK_TOGG` | 오디오 클릭 모드 전환       |
| `QK_AUDIO_CLICKY_ON`      | `CK_ON`   | 오디오 클릭 모드 켜기       |
| `QK_AUDIO_CLICKY_OFF`     | `CK_OFF`  | 오디오 클릭 모드 끄기       |
| `QK_AUDIO_CLICKY_UP`      | `CK_UP

`   | 클릭 소리 주파수 증가       |
| `QK_AUDIO_CLICKY_DOWN`    | `CK_DOWN` | 클릭 소리 주파수 감소       |
| `QK_AUDIO_CLICKY_RESET`   | `CK_RST`  | 주파수를 기본값으로 재설정  |
| `QK_MUSIC_ON`             | `MU_ON`   | 음악 모드 켜기              |
| `QK_MUSIC_OFF`            | `MU_OFF`  | 음악 모드 끄기              |
| `QK_MUSIC_TOGGLE`         | `MU_TOGG` | 음악 모드 전환              |
| `QK_MUSIC_MODE_NEXT`      | `MU_NEXT` | 음악 모드 순환              |
| `QK_AUDIO_VOICE_NEXT`     | `AU_NEXT` | 오디오 음성 순환            |
| `QK_AUDIO_VOICE_PREVIOUS` | `AU_PREV` | 오디오 음성을 역순으로 순환 |

이제 QMK 오디오 기능을 사용하여 키보드에서 다양한 소리를 즐길 수 있습니다!