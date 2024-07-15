# QMK 커리큘럼

이 페이지는 기본 개념을 먼저 소개하고 필요한 개념을 이해하도록 안내하여 QMK를 숙달할 수 있도록 도와줍니다.

## 초급 주제

이 섹션의 문서를 읽는 것만으로도 많은 도움이 됩니다. [튜토리얼](newbs)을 읽고 나면 기본적인 키맵을 생성하고, 컴파일하고, 키보드에 플래시할 수 있어야 합니다. 나머지 문서들은 이러한 기본 지식을 보충해줄 것입니다.

* **QMK 도구 사용법 배우기**
    * [튜토리얼](newbs)
    * [CLI](cli)
    * [GIT](newbs_git_best_practices)
* **키맵에 대해 배우기**
    * [레이어](feature_layers)
    * [키코드](keycodes)
        * 사용할 수 있는 키코드의 전체 목록입니다. 일부는 중급 또는 고급 주제에서 배운 지식이 필요할 수 있습니다.
* **IDE 구성하기** - 선택 사항
    * [Eclipse](other_eclipse)
    * [VS Code](other_vscode)

## 중급 주제

이 주제들은 QMK가 지원하는 일부 기능을 다룹니다. 모든 문서를 읽을 필요는 없지만, 고급 주제 섹션의 일부 문서는 중급 주제를 건너뛰면 이해하기 어려울 수 있습니다.

* **기능 구성하는 방법 배우기**
    * [오디오](features/audio)
    * 조명
        * [백라이트](features/backlight)
        * [LED 매트릭스](features/led_matrix)
        * [RGB 조명](features/rgblight)
        * [RGB 매트릭스](features/rgb_matrix)
    * [탭-홀드 구성](tap_hold)
    * [AVR 공간 절약하기](squeezing_avr)
* **키맵에 대해 더 배우기**
    * [키맵](keymap)
    * [커스텀 함수 및 키코드](custom_quantum_functions)
    * 매크로
        * [동적 매크로](features/dynamic_macros)
        * [컴파일된 매크로](feature_macros)
    * [탭 댄스](features/tap_dance)
    * [콤보](features/combo)
    * [사용자 공간](feature_userspace)
    * [키 오버라이드](features/key_overrides)

## 고급 주제

이후의 모든 내용은 많은 기초 지식을 요구합니다. 고급 기능을 사용하여 키맵을 만들 수 있을 뿐만 아니라 `config.h`와 `rules.mk`를 사용하여 키보드 옵션을 구성할 수 있어야 합니다.

* **QMK 내에서 키보드 유지 관리하기**
    * [키보드 손으로 배선하기](hand_wire)
    * [키보드 가이드라인](hardware_keyboard_guidelines)
    * [info.json 참고자료](reference_info_json)
    * [디바운스 API](feature_debounce_type)
* **고급 기능**
    * [유니코드](features/unicode)
    * [API](api_overview)
    * [부트매직 라이트](features/bootmagic)
* **하드웨어**
    * [키보드 작동 방식](how_keyboards_work)
    * [키보드 매트릭스 작동 방식](how_a_matrix_works)
    * [분할 키보드](features/split_keyboard)
    * [속기](features/stenography)
    * [포인팅 디바이스](features/pointing_device)
* **코어 개발**
    * [코딩 규칙](coding_conventions_c)
    * [호환 가능한 마이크로컨트롤러](compatible_microcontrollers)
    * [커스텀 매트릭스](custom_matrix)
    * [QMK 이해하기](understanding_qmk)
* **CLI 개발**
    * [코딩 규칙](coding_conventions_python)
    * [CLI 개발 개요](cli_development)


