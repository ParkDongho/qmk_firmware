# 자동 수정

자동 수정 기능은 습관, 순서 또는 사용자 오류로 인해 잘못 입력되는 단어들을 자동으로 수정하여 오타를 줄이는 데 도움을 줍니다.

## 어떻게 작동하나요? {#how-does-it-work}

이 기능은 최근에 입력된 키 프레스의 작은 버퍼를 유지합니다. 각 키 입력 시 버퍼가 인식된 오타로 끝나는지 확인하고, 그렇다면 자동으로 올바른 키 입력을 보냅니다.

버퍼에서 오타를 효율적으로 확인하는 것이 까다롭습니다. 오타를 저장하거나 검색하는 데 너무 많은 메모리나 시간을 소비하고 싶지 않습니다. 좋은 해결책은 오타를 트라이(trie) 데이터 구조로 표현하는 것입니다. 트라이는 각 노드가 문자이고, 단어는 리프(leaf) 중 하나로 가는 경로를 따라 형성되는 트리 데이터 구조입니다.

![An example trie](https://i.imgur.com/HL5DP8H.png)

버퍼가 오타로 끝나는지 확인하기 때문에, 트라이는 역순으로 저장됩니다. 트라이는 마지막 문자에서 시작하여, 그 다음 문자, 그 다음 문자 순으로 조회되며, 문자가 일치하지 않거나 리프에 도달할 때까지 진행됩니다. 리프에 도달하면 오타가 발견된 것입니다.

## 자동 수정 활성화 방법 {#how-do-i-enable-autocorrection}

`rules.mk` 파일에 다음을 추가하십시오:

```make
AUTOCORRECT_ENABLE = yes
```

또한 자동 수정 라이브러리가 필요합니다. 작은 샘플 라이브러리가 기본적으로 포함되어 있어 바로 사용할 수 있지만, 사용자 정의 라이브러리를 제공할 수도 있습니다.

기본적으로 자동 수정은 비활성화되어 있습니다. 이를 활성화하려면 `AC_TOGG` 키코드를 사용해야 합니다. 상태는 지속적인 메모리에 저장되므로, 다시 활성화할 필요는 없습니다.

## 자동 수정 라이브러리 사용자 정의 {#customizing-autocorrect-library}

커스텀 라이브러리를 제공하려면 수정 목록이 포함된 텍스트 파일을 생성해야 합니다. 예를 들어:

```text
:thier        -> their
fitler        -> filter
lenght        -> length
ouput         -> output
widht         -> width
```

구문은 `오타  ->  수정`입니다. 오타와 수정은 대소문자를 구분하지 않으며, 오타와 수정 앞뒤의 공백은 무시됩니다. 오타는 a–z 문자만 사용해야 하며, 특수 문자 : 는 단어 구분을 나타냅니다. 수정은 비유니코드 문자를 포함할 수 있습니다.

그런 다음, 다음 명령을 실행하십시오:

```sh
qmk generate-autocorrect-data autocorrect_dictionary.txt
```

이 명령은 파일을 처리하여 `autocorrect_data.h` 파일을 생성하며, 해당 폴더에 트라이 라이브러리를 포함합니다. 키보드와 키맵을 지정할 수도 있으며(예: `-kb planck/rev6 -km jackhumbert`), 해당 폴더에 파일을 배치할 것입니다. 하지만 파일이 키맵 폴더 또는 사용자 폴더에 있는 한 자동으로 인식될 것입니다.

이 파일은 다음과 같이 보일 것입니다:

```c
// :thier        -> their
// fitler        -> filter
// lenght        -> length
// ouput         -> output
// widht         -> width

#define AUTOCORRECT_MIN_LENGTH 5  // "ouput"
#define AUTOCORRECT_MAX_LENGTH 6  // ":thier"

#define DICTIONARY_SIZE 74

static const uint8_t autocorrect_data[DICTIONARY_SIZE] PROGMEM = {85, 7, 0, 23, 35, 0, 0, 8, 0, 76, 16, 0, 15, 25, 0, 0,
    11, 23, 44, 0, 130, 101, 105, 114, 0, 23, 12, 9, 0, 131, 108, 116, 101, 114, 0, 75, 42, 0, 24, 64, 0, 0, 71, 49, 0,
    10, 56, 0, 0, 12, 26, 0, 129, 116, 104, 0, 17, 8, 15, 0, 129, 116, 104, 0, 19, 24, 18, 0, 130, 116, 112, 117, 116,
    0};
```

### 잘못된 트리거 방지 {#avoiding-false-triggers}

기본적으로 오타는 단어 내에서 검색되어, maxFitlerOuput과 같은 긴 식별자 내에서 오타를 찾습니다. 이는 유용하지만, 오타가 올바르게 철자된 단어의 부분 문자열일 경우 자동 수정이 잘못 트리거되는 결과를 초래합니다. 예를 들어, thier -> their 항목이 있는 경우 "wealthier"나 "filthier"와 같은 단어에서 잘못 트리거될 수 있습니다.

해결책은 오타 전후에 단어 구분자 : 를 설정하여 일치 여부를 제한하는 것입니다. : 는 공백, 마침표, 쉼표, 밑줄, 숫자 및 대부분의 비알파벳 문자를 나타냅니다.

| 텍스트           | thier | :thier | thier: | :thier: |
| ---------------- | :---: | :----: | :----: | :-----: |
| see `thier` 오타 | 일치  |  일치  |  일치  |  일치   |
| it’s `thiers`    | 일치  |  일치  | 불일치 | 불일치  |
| wealthier words  | 일치  | 불일치 |  일치  | 불일치  |

:thier:는 thier이 전체 단어일 때만 일치합니다.

`qmk generate-autocorrect-data` 명령은 올바르게 철자된 단어의 부분 문자열로서 잘못 트리거될 수 있는 항목을 확인하려고 시도할 수 있습니다. 이 명령은 english_words Python 패키지의 25K 영어 단어 사전을 사용하여 각 오타를 검색합니다(패키지가 설치되어 있는 경우). (설치하려면 `python3 -m pip install english_words` 명령을 실행하십시오.)

::: tip
불행히도 현재 이 기능은 영어 단어에만 제한됩니다.
:::

## 자동 수정 무시하기

때때로 오타를 실제로 입력하고 싶을 수 있습니다(예: autocorrect_dict.txt를 편집할 때). 이 경우 자동 수정이 되지 않도록 하는 몇 가지 방법이 있습니다:

1. 오타를 입력하기 시작합니다.
2. 마지막 글자를 입력하기 전에 Ctrl 또는 Alt 키를 누르고 놓습니다.
3. 나머지 글자를 입력합니다.

이 방법은 자동 수정 구현이 단축키를 이해하지 못하므로, Shift 이외의 수정 키가 눌려질 때마다 자동 수정이 초기화되기 때문입니다.

또한 `AC_TOGG` 키코드를 사용하여 자동 수정 기능의 켜짐/꺼짐 상태를 전환할 수 있습니다.

### 키코드 {#keycodes}

| 키코드                  | 별칭      | 설명                                |
| ----------------------- | --------- | ----------------------------------- |
| `QK_AUTOCORRECT_ON`     | `AC_ON`   | 자동 수정 기능을 켭니다.            |
| `QK_AUTOCORRECT_OFF`    | `AC_OFF`  | 자동 수정 기능을 끕니다.            |
| `QK_AUTOCORRECT_TOGGLE` | `AC_TOGG` | 자동 수정 기능의 상태를 전환합니다. |

## 사용자 콜백 함수

### 자동 수정 처리

콜백 함수 `bool process_autocorrect_user(uint16_t *keycode, keyrecord_t *record, uint8_t *typo_buffer_size, uint8_t *mods)`를 사용하여 들어오는 키코드를 사용자 정의하고 예외를 처리할 수 있습니다. 이 함수를 사용하여 자동 수정 엔진에 전달되기 전에 입력을 정리할 수 있습니다.

::: tip
입력을 정리하는 것은 필수입니다. 자동 수정은 오타를 8비트 [기본 키코드](../keycodes_basic)로만 매칭하기 때문입니다. 유효한 수정 키 또는 16비트 키코드가 단어 입력의 일부로 전달되면 오타 문자 감지에 실패합니다. 예를 들어 [Mod-Tap](../mod_tap) 키 `LCTL_T(KC_A)`는 16비트이며 8비트 `KC_A`로 마스킹해야 합니다.
:::

기본 사용자 콜백 함수는 `quantum/process_keycode/process_autocorrect.c`에 있습니다. 이 함수는 QMK의 특수 기능 및 퀀텀 키코드에 대한 대부분의 사용 사례를 다루며, Shift 이외의 수정 키로 자동 수정을 무시

하는 것도 포함합니다. `process_autocorrect_user` 함수는 `keymap.c`(또는 코드 파일) 내에 사용자의 복사본이 덮어쓸 수 있도록 `weak`로 정의됩니다.

#### 자동 수정 처리 예시

`QMKBEST`라는 사용자 정의 키코드를 무시해야 하고, `QMKLAYER`라는 다른 사용자 정의 키코드가 자동 수정을 무시해야 하는 경우, 이를 소스 코드의 `process_autocorrect_user` `switch` 문 하단에 추가할 수 있습니다:

```c
bool process_autocorrect_user(uint16_t *keycode, keyrecord_t *record, uint8_t *typo_buffer_size, uint8_t *mods) {
    // 참조를 위해 quantum_keycodes.h에서 이 매칭 범위를 참조하십시오.
    switch (*keycode) {
        // 이 키코드를 처리에서 제외합니다.
        case KC_LSFT:
        case KC_RSFT:
        case KC_CAPS:
        case QK_TO ... QK_ONE_SHOT_LAYER_MAX:
        case QK_LAYER_TAP_TOGGLE ... QK_LAYER_MOD_MAX:
        case QK_ONE_SHOT_MOD ... QK_ONE_SHOT_MOD_MAX:
            return false;

        // 시프트 키의 기본 키코드를 마스킹합니다.
        case QK_LSFT ... QK_LSFT + 255:
        case QK_RSFT ... QK_RSFT + 255:
            if (*keycode >= QK_LSFT && *keycode <= (QK_LSFT + 255)) {
                *mods |= MOD_LSFT;
            } else {
                *mods |= MOD_RSFT;
            }
            *keycode &= 0xFF; // 기본 키코드를 가져옵니다.
            return true;
#ifndef NO_ACTION_TAPPING
        // 탭-홀드 키가 홀드될 때 제외하고, 탭될 때 기본 키코드로 마스킹합니다.
        case QK_LAYER_TAP ... QK_LAYER_TAP_MAX:
# ifdef NO_ACTION_LAYER
            // 레이어가 비활성화되었지만 액션 탭핑이 여전히 활성화된 경우, 레이어 탭을 제외합니다.
            return false;
# endif
        case QK_MOD_TAP ... QK_MOD_TAP_MAX:
            // 수정 키가 활성화되지 않은 경우 홀드를 제외합니다.
            if (!record->tap.count) {
                return false;
            }
            *keycode &= 0xFF;
            break;
#else
        case QK_MOD_TAP ... QK_MOD_TAP_MAX:
        case QK_LAYER_TAP ... QK_LAYER_TAP_MAX:
            // 비활성화된 경우 제외합니다.
            return false;
#endif
        // 스왑 핸드 키가 홀드될 때 제외하고, 탭될 때 기본 키코드로 마스킹합니다.
        case QK_SWAP_HANDS ... QK_SWAP_HANDS_MAX:
#ifdef SWAP_HANDS_ENABLE
            if (*keycode >= 0x56F0 || !record->tap.count) {
                return false;
            }
            *keycode &= 0xFF;
            break;
#else
            // 비활성화된 경우 제외합니다.
            return false;
#endif
        // 사용자 정의 키코드 처리
        case QMKBEST:
            return false;
        case QMKLAYER:
            *typo_buffer_size = 0;
            return false;
    }

    // Shift 이외의 수정 키가 활성화된 동안 자동 수정을 비활성화합니다.
    if ((*mods & ~MOD_MASK_SHIFT) != 0) {
        *typo_buffer_size = 0;
        return false;
    }

    return true;
}
```

::: tip
이 콜백 함수에서 `return false`는 해당 키코드에 대한 자동 수정 처리를 건너뜁니다. `*typo_buffer_size = 0`을 추가하면 현재 버퍼에 저장된 문자를 취소하면서 자동 수정 버퍼를 초기화합니다.
:::

### 자동 수정 적용

또한 `apply_autocorrect(uint8_t backspaces, const char *str, char *typo, char *correct)`를 사용하여 자동 수정에 추가 처리를 추가하거나 기능을 완전히 대체할 수 있습니다. 이 함수는 단어를 대체하기 위해 필요한 백스페이스 수와 대체 문자열(부분 단어, 전체 단어가 아님), 오타 및 수정된 문자열(전체 단어)을 전달합니다.

::: tip
코드가 작동하는 방식(단어 개념 없음, 단지 문자열 흐름) 때문에 `typo`와 `correct` 문자열은 예상과 다를 수 있습니다. 예를 들어 `wordtpyo` 및 `wordtypo` 대신 예상되는 `tpyo` 및 `typo`가 아닌 문자열을 얻을 수 있습니다.
:::

#### 자동 수정 적용 예시

다음 예제는 오타가 자동 수정될 때 소리를 재생하고 자동 수정을 실행합니다:

```c
#ifdef AUDIO_ENABLE
float autocorrect_song[][2] = SONG(TERMINAL_SOUND);
#endif

bool apply_autocorrect(uint8_t backspaces, const char *str, char *typo, char *correct) {
#ifdef AUDIO_ENABLE
    PLAY_SONG(autocorrect_song);
#endif
    for (uint8_t i = 0; i < backspaces; ++i) {
        tap_code(KC_BSPC);
    }
    send_string_P(str);
    return false;
}
```

::: tip
이 콜백 함수에서 `return false`는 자동 수정의 일반 처리를 중지하며, "잘못된" 문자를 제거하고 새 문자를 입력하는 처리를 수동으로 처리해야 합니다.
:::

::: warning
***중요***: `str`은 자동 수정을 위한 `PROGMEM` 데이터의 포인터입니다. `return false`를 사용하고 문자열을 보내려는 경우 `send_string_P`를 사용해야 하며 `send_string` 또는 `SEND_STRING`을 사용하지 않아야 합니다.
:::

자동 수정 이벤트를 감지하고 표시하지만 내부 코드에서 자동 수정을 실행하도록 `apply_autocorrect`를 사용할 수도 있습니다. 이 경우 `return true`를 사용합니다:

```c
bool apply_autocorrect(uint8_t backspaces, const char *str, char *typo, char *correct) {
#ifdef OLED_ENABLE
    oled_write_P(PSTR("Auto-corrected"), false);
#endif
#ifdef CONSOLE_ENABLE
    printf("'%s' was corrected to '%s'\n", typo, correct);
#endif
    return true;
}
```

### 자동 수정 상태

자동 수정을 조작하기 위한 추가 사용자 콜백 함수:

| 함수                       | 설명                                            |
| -------------------------- | ----------------------------------------------- |
| `autocorrect_enable()`     | 자동 수정을 켭니다.                             |
| `autocorrect_disable()`    | 자동 수정을 끕니다.                             |
| `autocorrect_toggle()`     | 자동 수정 기능을 전환합니다.                    |
| `autocorrect_is_enabled()` | 자동 수정이 현재 켜져 있는지 여부를 반환합니다. |

## 부록: 트라이 이진 데이터 형식 {#appendix}

이 섹션은 트라이가 autocorrect_data에서 이진 데이터로 직렬화되는 방식을 자세히 설명합니다. 이 자동 수정 구현을 사용하는 데는 이 부분을 이해할 필요는 없습니다. 그러나 구현을 수정하거나 작동 방식에 관심이 있는 경우를 위해 기록을 남겨두었습니다.

여기서 한 일은 상당히 임의적이지만, 해독하기 쉽고 작업을 완료하는 데 충분합니다.

### 인코딩 {#encoding}

모든 자동 수정 데이터는 단일 평면 배열 autocorrect_data에 저장됩니다. 각 트라이 노드는 이 배열의 바이트 오프셋과 연결되며, 해당 노드에 대한 데이터는 루트에서 오프셋 0으로 인코딩됩니다. 세 가지 종류의 노드가 있습니다. 노드의 첫 번째 바이트의 상위 두 비트는 다음과 같은 종류를 나타냅니다:

* 00 ⇒ 체인 노드: 단일 자식이 있는 트라이 노드.
* 01 ⇒ 분기 노드: 여러 자식이 있는 트라이 노드.
* 10 ⇒ 리프 노드: 오타에 해당하며 수정 데이터를 저장하는 리프.

![An example trie](https://i.imgur.com/HL5DP8H.png)

**분기 노드**. 각 분기는 키코드(KC_A–KC_Z)에 대한 한 바이트와 자식 노드로의 링크로 인코딩됩니다. 노드 간의 링크는 배열의 시작을 기준으로 한 16비트 바이트 오프셋이며, 작은 엔디언 순서로 직렬화됩니다.

모든 분기는 이와 같은 방식으로 직렬화되며, 마지막에 0 바이트로 종료됩니다. 위에서 설명한 대로, 노드는 첫 번째 키코드와 64를 비트 OR 연산하여 첫 번째 바이트의 상위 두 비트를 01로 설정하여 분기로 식별됩니다. 위 그림의 루트 노드는 다음과 같이 직렬화됩니다:

```
+-------+-------+-------+-------+-------+-------+-------+
| R|64  |    node 2     |   T   |    node 3     |   0   |
+-------+-------+-------+-------+-------+-------+-------+
```

**체인 노드**. 트라이는

 단일 자식 노드의 긴 체인을 가지는 경향이 있습니다. 예를 들어, fitler에서 f-i-t-l 체인과 같이 긴 체인을 가지고 있습니다. 따라서 공간을 절약하기 위해 체인 노드는 분기 노드와 다른 형식으로 인코딩됩니다. 체인은 루트에 가장 가까운 노드에서 시작하여 키코드 문자열로 인코딩되며, 0 바이트로 종료됩니다. 체인의 마지막 노드의 자식은 즉시 인코딩됩니다. 해당 자식은 분기 노드일 수도 있고 리프일 수도 있습니다.

위의 그림에서 f-i-t-l 체인은 다음과 같이 인코딩됩니다:

```
+-------+-------+-------+-------+-------+
|   L   |   T   |   I   |   F   |   0   |
+-------+-------+-------+-------+-------+
```

이 체인을 분기 노드와 같은 형식으로 인코딩하려면 각 노드마다 16비트 노드 링크를 인코딩해야 하므로, 이 예제에서 8바이트가 더 소모됩니다. 전체 트라이에서 이 비용이 쌓입니다. 편리하게도 체인의 중간 지점을 가리키고 이전과 동일한 방식으로 바이트를 해석할 수 있습니다. 예를 들어, l 대신 i에서 시작하면 하위 체인은 동일한 형식을 가집니다.

**리프 노드**. 리프 노드는 특정 오타에 해당하며 오타를 수정하는 데이터를 저장합니다. 리프는 백스페이스를 누를 횟수를 나타내는 한 바이트로 시작하며, 교체 텍스트의 null로 종료된 ASCII 문자열이 뒤따릅니다. 아이디어는 지정된 횟수만큼 백스페이스를 누른 후 이 문자열을 `send_string_P` 함수에 전달하는 것입니다. fitler의 경우 백스페이스를 3번 눌러야 합니다(오타를 마지막 'r'이 눌릴 때 감지하기 때문). 리프 노드를 식별하기 위해 백스페이스 수에 128을 비트 OR 연산하여 상위 두 비트를 10으로 설정합니다:

```
+-------+-------+-------+-------+-------+-------+
| 3|128 |  'l'  |  't'  |  'e'  |  'r'  |   0   |
+-------+-------+-------+-------+-------+-------+
```

### 디코딩 {#decoding}

이 형식은 의도적으로 단순한 논리로 디코딩할 수 있습니다. 16비트 변수 state는 트라이의 현재 위치를 나타내며, 루트 노드에서 시작하기 위해 0으로 초기화됩니다. 그런 다음 각 키코드에 대해 state의 바이트에서 상위 두 비트를 테스트하여 노드 종류를 식별합니다.

* 00 ⇒ **체인 노드**: 노드의 바이트가 키코드와 일치하면 다음 바이트로 이동하기 위해 state를 1 증가시킵니다. 다음 바이트가 0이면 다시 증가하여 다음 노드로 이동합니다.
* 01 ⇒ **분기 노드**: 분기에서 키코드와 일치하는 분기를 찾아 해당 노드 링크를 따릅니다.
* 10 ⇒ **리프 노드**: 오타가 발견되었습니다! 첫 번째 바이트를 읽어 백스페이스 횟수를 확인한 다음, 후속 바이트를 `send_string_P` 함수에 전달하여 수정을 입력합니다.

## 크레딧

[getreuer](https://github.com/getreuer)가 원래 [여기](https://getreuer.info/posts/keyboards/autocorrection/#how-does-it-work)에서 구현한 것에 대한 크레딧이 주어집니다. 또한 코드를 PROGMEM으로 변환하고 추가 개선을 한 [filterpaper](https://github.com/filterpaper)에게도 감사의 인사를 전합니다.
