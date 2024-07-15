# 키맵 FAQ

이 페이지는 키맵에 대해 자주 묻는 질문들을 다룹니다. 먼저 [Keymap Overview](keymap)를 읽어보세요.

## 사용할 수 있는 키코드는 무엇인가요?

사용 가능한 키코드에 대한 색인은 [Keycodes](keycodes)를 참조하세요. 가능할 때마다 더 자세한 문서로 링크되어 있습니다.

키코드는 실제로 [quantum/keycode.h](https://github.com/qmk/qmk_firmware/blob/master/quantum/keycode.h)에서 정의되어 있습니다.

## 기본 키코드는 무엇인가요?

전 세계적으로 사용되는 표준 키보드 레이아웃은 세 가지가 있습니다 - ANSI, ISO, JIS. 북미는 주로 ANSI를 사용하고, 유럽과 아프리카는 주로 ISO를 사용하며, 일본은 JIS를 사용합니다. 언급되지 않은 지역은 일반적으로 ANSI 또는 ISO를 사용합니다. 이러한 레이아웃에 해당하는 키코드는 다음과 같습니다:

<!-- Source for this image: https://www.keyboard-layout-editor.com/#/gists/bf431647d1001cff5eff20ae55621e9a -->
![Keyboard Layout Image](https://i.imgur.com/5wsh5wM.png)

## 복잡한 키코드에 대해 사용자 정의 이름을 만들 수 있나요?

때로는 가독성을 위해 일부 키코드에 대해 사용자 정의 이름을 정의하는 것이 유용합니다. 사람들은 종종 `#define`을 사용하여 사용자 정의 이름을 정의합니다. 예를 들어:

```c
#define FN_CAPS LT(_FL, KC_CAPS)
#define ALT_TAB LALT(KC_TAB)
```

이렇게 하면 `FN_CAPS`와 `ALT_TAB`을 키맵에서 사용할 수 있어 읽기 쉽게 유지할 수 있습니다.

## 키맵을 플래시했을 때 업데이트되지 않아요

이는 주로 VIA와 관련이 있으며, VIA가 키맵을 처리하는 방식과 관련이 있습니다.

처음 실행 시, VIA 펌웨어의 코드는 플래시 메모리에서 EEPROM으로 키맵을 복사하여, VIA 앱에서 런타임 시 다시 쓸 수 있도록 합니다. 이 시점부터 QMK는 플래시 대신 EEPROM에 저장된 키맵을 사용하므로 `keymap.c`의 업데이트가 반영되지 않습니다.

이 문제를 해결하려면 EEPROM을 지우면 됩니다. 여러 가지 방법이 있습니다:

* 보드의 상단 왼쪽/ESC 키를 누른 상태로 보드를 연결하면, 보드가 부트로더 모드로 진입합니다. 그런 다음 보드를 뽑았다가 다시 꽂습니다.
* 키맵에서 접근할 수 있는 경우 `QK_CLEAR_EEPROM`/`EE_CLR` 키코드를 누릅니다.
* 보드를 부트로더 모드로 전환하고 "Clear EEPROM" 버튼을 누릅니다. 이는 모든 부트로더에서 사용할 수 없으며, 이후에 보드를 다시 플래시해야 할 수도 있습니다.

## 일부 키가 바뀌거나 작동하지 않아요

QMK에는 키보드의 동작을 즉석에서 변경할 수 있는 몇 가지 기능이 있습니다. 여기에는 Ctrl/Caps 전환, GUI 비활성화, Alt/GUI 전환, Backspace/Backslash 전환, 모든 키 비활성화 등과 같은 행동 수정이 포함됩니다.

위의 EEPROM 지우기 방법을 참조하여 해당 키를 정상 작동 상태로 되돌리세요. 그래도 해결되지 않으면 다음을 참조하세요:

* [Magic Keycodes](keycodes_magic)
* [Command](features/command)

## 메뉴 키가 작동하지 않아요

대부분의 현대 키보드에서 `KC_RGUI`와 `KC_RCTL` 사이에 있는 키는 실제로 `KC_APP`입니다. 이는 HID 사양에 이미 "Menu"라는 키가 있었기 때문에, Microsoft가 새로운 키를 만들고 "Application"이라고 부르기로 했기 때문입니다.

## 전원 키가 작동하지 않아요

약간 혼란스러운 점은 QMK에는 두 가지 "전원" 키코드가 있다는 것입니다: Keyboard/Keypad HID 사용 페이지의 `KC_KB_POWER`와 Consumer 페이지의 `KC_SYSTEM_POWER` (또는 `KC_PWR`).

전자는 macOS에서만 인식되며, 후자인 `KC_SLEP` 및 `KC_WAKE`는 모든 주요 운영 체제에서 지원되므로 이 키들을 사용하는 것이 좋습니다. Windows에서는 이러한 키가 즉시 적용되지만, macOS에서는 대화 상자가 나타날 때까지 키를 눌러야 합니다.

## One Shot Modifier

One Shot Shift는 개인적으로 'the' 문제를 해결합니다. 자주 'the' 또는 'THe'를 잘못 입력했었는데, One Shot Shift가 이를 완화해줍니다.
https://github.com/tmk/tmk_keyboard/issues/67

## Modifier/Layer가 고정됩니다

Modifier 키 또는 레이어가 고정될 수 있습니다. Modifier 키와 레이어 동작을 올바르게 구성하지 않으면 발생할 수 있습니다. Modifier 키와 레이어 동작에는 릴리스 이벤트 시 Modifier 키의 등록을 해제하거나 이전 레이어로 돌아가기 위해 대상 레이어의 동일한 위치에 `KC_TRNS`를 배치해야 합니다.

* https://github.com/tmk/tmk_core/blob/master/doc/keymap.md#31-momentary-switching
* https://geekhack.org/index.php?topic=57008.msg1492604#msg1492604
* https://github.com/tmk/tmk_keyboard/issues/248

## 기계식 잠금 스위치 지원

이 기능은 [Alps 기계식 잠금 스위치](https://deskthority.net/wiki/Alps_SKCL_Lock)와 같은 *기계식 잠금 스위치*를 위한 것입니다. 이를 활성화하려면 `config.h`에 다음을 추가하세요:

```c
#define LOCKING_SUPPORT_ENABLE
#define LOCKING_RESYNC_ENABLE
```

이 기능을 활성화한 후에는 키맵에서 `KC_LCAP`, `KC_LNUM`, `KC_LSCR` 키코드를 대신 사용하세요.

구형 빈티지 기계식 키보드에는 때때로 잠금 스위치가 있지만, 현대 키보드에는 없습니다. ***대부분의 경우 이 기능이 필요 없으며, `KC_CAPS`, `KC_NUM`, `KC_SCRL` 키코드를 사용하면 됩니다.***

## ASCII 이외의 특수 문자를 입력하는 방법 (예: Cédille 'Ç')

[Unicode](features/unicode) 기능을 참조하세요.

## macOS의 `Fn` 키

대부분의 Fn 키와 달리 Apple 키보드의 Fn 키는 실제로 자체 키코드를 가지고 있습니다. Apple 키보드는 실제로 5KRO만 지원하기 때문에 기본 6KRO HID 보고서의 여섯 번째 키코드를 차지합니다.

QMK에서 이 키를 보내는 것은 기술적으로 가능합니다. 하지만 이를 위해 보고서 형식을 수정하여 Fn 키의 상태를 추가해야 합니다.
더 나쁜 것은, 키보드의 VID와 PID가 실제 Apple 키보드와 일치하지 않으면 인식되지 않는다는 점입니다. 이 기능을 공식적으로 지원하면 발생할 수 있는 법적 문제로 인해 QMK에서 이 기능을 지원할 가능성은 낮습니다.

자세한 정보는 [이 이슈](https://github.com/qmk/qmk_firmware/issues/2179)를 참조하세요.

## Mac OSX에서 지원되는 키는 무엇인가요?

OSX에서 어떤 키코드가 지원되는지 알 수 있는 소스 코드는 다음과 같습니다.

`usb_2_adb_keymap` 배열은 Keyboard/Keypad 페이지 사용법을 ADB 스캔 코드(OSX 내부 키코드)로 매핑합니다.

https://opensource.apple.com/source/IOHIDFamily/IOHIDFamily-606.1.7/IOHIDFamily/Cosmo_USB2ADB.c

그리고 `IOHIDConsumer::dispatchConsumerEvent`는 Consumer 페이지 사용법을 처리합니다.

https://opensource.apple.com/source/IOHIDFamily/IOHIDFamily-606.1.7/IOHIDFamily/IOHIDConsumer.cpp

## Mac OSX에서 JIS 키

일본 JIS 키보드의 특정 키인 `無変換(Muhenkan)`, `変換(Henkan)`, `ひらがな(hiragana)`는 OSX에서 인식되지 않습니다. **Seil**을 사용하여 이러한 키를 활성화할 수 있습니다. 다음 옵션을 시도해 보세요.

* PC 키보드에서 NFER 키 활성화
* PC 키보드에서 XFER 키 활성화
* PC 키보드에서 KATAKANA 키 활성화

https://pqrs.org/osx/karabiner/seil.html

## Karabiner와 함께 RN-42 Bluetooth가 작동하지 않아요

Mac OSX의 키맵핑 도구인 Karabiner는 기본적으로 RN-42 모듈의 입력을 무시합니다. 이 옵션을 활성화하여 Karabiner가 키보드와 함께 작동하도록 해야 합니다.
https://github.com/tekezo/Karabiner/issues/403#issuecomment-102559237

이 문제에 대한 자세한 내용은 다음을 참조하세요.
https://github.com/tmk/tmk_keyboard/issues/213
https://github.com/tekezo/K

arabiner/issues/403

## Esc와 `<code>&#96;</code>`를 단일 키로 사용하기

[Grave Escape](features/grave_esc) 기능을 참조하세요.

## Mac OSX에서 Eject 키

`KC_EJCT` 키코드는 OSX에서 작동합니다. https://github.com/tmk/tmk_keyboard/issues/250
Windows 10에서는 이 코드가 무시되는 것 같고, Linux/Xorg에서는 인식되지만 기본적으로 매핑되지 않습니다.

Apple의 실제 키보드에서 Eject 키의 키코드가 무엇인지 정확히는 알 수 없습니다. HHKB는 Mac 모드에서 Eject 키(`Fn+F`)로 `F20`을 사용하지만, 이는 아마도 Apple Eject 키코드와 다를 것입니다.

## "실제" 및 "약한" 수정자는 무엇인가요?

실제 수정자는 실제/물리적 수정자 키의 상태를 나타내며, 약한 수정자는 실제 수정자 키의 내부 상태에 영향을 미치지 않아야 하는 "가상" 또는 임시 수정자의 상태를 나타냅니다.

실제 및 약한 수정자 상태는 키보드 보고서가 전송될 때 OR 연산이 수행되므로, 동일한 실제 수정자가 여전히 눌려 있는 동안 약한 수정자를 해제하면 보고서가 변경되지 않습니다:

 1. **물리적 왼쪽 Shift를 누르고 있음:** 실제 수정자에 왼쪽 Shift가 포함되어 있으며, 최종 상태는 왼쪽 Shift입니다.
 2. **약한 왼쪽 Shift 추가:** 약한 수정자에 왼쪽 Shift가 포함되며, 최종 상태는 왼쪽 Shift입니다.
 3. **약한 왼쪽 Shift 제거:** 약한 수정자에 아무 것도 포함되지 않으며, 최종 상태는 왼쪽 Shift입니다.
 4. **물리적 왼쪽 Shift 해제:** 실제 수정자에 아무 것도 포함되지 않으며, 최종 상태는 아무 것도 아닙니다.