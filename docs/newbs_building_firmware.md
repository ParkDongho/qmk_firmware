# 첫 펌웨어 빌드하기

이제 빌드 환경을 설정했으므로, 커스텀 펌웨어를 빌드할 준비가 되었습니다. 이 섹션에서는 파일 관리자, 텍스트 에디터, 터미널 창을 번갈아 가며 사용할 것입니다. 키보드 펌웨어 작업이 완료되고 만족할 때까지 이 세 가지 프로그램을 모두 열어 두십시오.

## 빌드 환경 기본값 설정 (선택 사항)

빌드 환경의 기본값을 설정하여 QMK 작업을 덜 번거롭게 할 수 있습니다. 지금 해보겠습니다!

QMK를 처음 사용하는 대부분의 사람들은 하나의 키보드만 가지고 있습니다. `qmk config` 명령을 사용하여 이 키보드를 기본값으로 설정할 수 있습니다. 예를 들어, 기본 키보드를 `clueboard/66/rev4`로 설정하려면 다음과 같이 합니다:

```sh
qmk config user.keyboard=clueboard/66/rev4
```

::: tip
키보드 옵션은 키보드 디렉토리에 대한 상대 경로입니다. 위의 예제는 `qmk_firmware/keyboards/clueboard/66/rev4`에 있습니다. 확실하지 않은 경우 `qmk list-keyboards`를 사용하여 지원되는 키보드 목록을 볼 수 있습니다.
:::

기본 키맵 이름도 설정할 수 있습니다. 대부분의 사람들은 이전 단계에서 키맵 이름처럼 GitHub 사용자 이름을 사용합니다:

```sh
qmk config user.keymap=<github_username>
```

## 새로운 키맵 만들기

자신만의 키맵을 만들려면 `default` 키맵을 복사하여 사용합니다. 이전 단계에서 빌드 환경을 구성한 경우 QMK CLI를 사용하여 쉽게 할 수 있습니다:

```sh
qmk new-keymap
```

환경을 구성하지 않았거나 여러 키보드가 있는 경우, 키보드 이름을 지정할 수 있습니다:

```sh
qmk new-keymap -kb <keyboard_name>
```

해당 명령의 출력을 확인하면 다음과 같은 내용이 표시됩니다:

```
Ψ Created a new keymap called <github_username> in: /home/me/qmk_firmware/keyboards/clueboard/66/rev3/keymaps/<github_username>.
```

이 위치가 새 `keymap.c` 파일의 위치입니다.

## 즐겨 사용하는 텍스트 에디터에서 `keymap.c` 열기

텍스트 에디터에서 `keymap.c` 파일을 엽니다. 이 파일 내부에는 키보드 동작을 제어하는 구조가 있습니다. `keymap.c` 상단에는 키맵을 쉽게 읽을 수 있도록 하는 몇 가지 정의와 열거형(enum)이 있을 수 있습니다. 그 아래에는 다음과 같은 줄이 있습니다:

```c
const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
```

이 줄은 레이어 목록이 시작되는 위치를 나타냅니다. 그 아래에는 `LAYOUT`을 포함한 줄이 있으며, 이는 레이어의 시작을 나타냅니다. 그 줄 아래에는 특정 레이어를 구성하는 키 목록이 있습니다.

::: warning
키맵 파일을 편집할 때 쉼표를 추가하거나 제거하지 않도록 주의하십시오. 그렇게 하면 펌웨어가 컴파일되지 않으며, 추가되거나 누락된 쉼표를 찾기가 어려울 수 있습니다.
:::

## 원하는 대로 레이아웃 사용자 정의

이 단계를 완료하는 방법은 전적으로 사용자에게 달려 있습니다. 불편했던 한 가지를 변경하거나 모든 것을 완전히 재구성할 수 있습니다. 모든 레이어가 필요하지 않으면 제거할 수 있으며, 최대 32개의 레이어를 추가할 수 있습니다. QMK에는 많은 기능이 있으며, "QMK 사용하기" 섹션에서 전체 목록을 확인할 수 있습니다. 시작하기 위해 다음은 사용하기 쉬운 몇 가지 기능입니다:

* [기본 키코드](keycodes_basic)
* [퀀텀 키코드](quantum_keycodes)
* [Grave/Escape](features/grave_esc)
* [마우스 키](features/mouse_keys)

::: tip
키맵 작동 방식을 익히는 동안 각 변경 사항을 작게 유지하십시오. 큰 변경 사항은 발생하는 문제를 디버그하기 어렵게 만듭니다.
:::

## 펌웨어 빌드하기 {#build-your-firmware}

키맵에 대한 변경 사항을 완료하면 펌웨어를 빌드해야 합니다. 이를 위해 터미널 창으로 돌아가서 컴파일 명령을 실행합니다:

```sh
qmk compile
```

환경에 대한 기본값을 설정하지 않았거나 여러 개의 키보드를 사용하는 경우, 키보드 및/또는 키맵을 지정할 수 있습니다:

```sh
qmk compile -kb <keyboard> -km <keymap>
```

컴파일하는 동안 많은 출력이 화면에 나타나며, 어떤 파일이 컴파일되고 있는지 알려줍니다. 다음과 유사한 출력으로 끝나야 합니다:

```
Linking: .build/planck_rev5_default.elf                                                             [OK]
Creating load file for flashing: .build/planck_rev5_default.hex                                     [OK]
Copying planck_rev5_default.hex to qmk_firmware folder                                              [OK]
Checking file size of planck_rev5_default.hex                                                       [OK]
 * The firmware size is fine - 27312/28672 (95%, 1360 bytes free)
```

## 펌웨어 플래싱하기

키보드에 새 펌웨어를 쓰는 방법을 배우려면 [펌웨어 플래싱](newbs_flashing)으로 이동하십시오.