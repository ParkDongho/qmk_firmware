# QMK Configurator 아키텍처

이 페이지는 QMK Configurator의 웹 아키텍처에 대해 높은 수준에서 설명합니다. QMK Configurator 코드 자체의 아키텍처에 관심이 있다면 [qmk_configurator](https://github.com/qmk/qmk_configurator) 저장소를 시작점으로 삼으십시오.

# 개요

![QMK Configurator Architecture Diagram](configurator_diagram.svg)

# 상세 설명

QMK Configurator는 [단일 페이지 애플리케이션](https://en.wikipedia.org/wiki/Single-page_application)으로, 사용자가 QMK 호환 키보드용 커스텀 키맵을 생성할 수 있도록 합니다. 사용자는 키맵의 JSON 표현을 내보내고, 펌웨어 바이너리를 컴파일하여 [QMK Toolbox](https://github.com/qmk/qmk_toolbox)와 같은 도구를 사용하여 키보드에 플래시할 수 있습니다.

Configurator는 키보드 메타데이터 저장소에서 키보드에 대한 메타데이터를 가져오고, QMK API에 컴파일 요청을 제출합니다. 이러한 컴파일 요청의 결과는 S3 호환 데이터 저장소인 [Digital Ocean Spaces](https://www.digitalocean.com/products/spaces/)에 제공됩니다.

## Configurator 프런트엔드

주소: <https://config.qmk.fm>

[Configurator 프런트엔드](https://config.qmk.fm)는 정적 파일 세트로 컴파일되어 Github Pages에서 제공됩니다. 이 작업은 [qmk_configurator `master`](https://github.com/qmk/qmk_configurator) 브랜치에 커밋이 푸시될 때마다 수행됩니다. 이러한 작업의 상태는 [qmk_configurator actions 탭](https://github.com/qmk/qmk_configurator/actions/workflows/build.yml)에서 확인할 수 있습니다.

## 키보드 메타데이터

주소: <https://keyboards.qmk.fm>

키보드 메타데이터는 [qmk_firmware](https://github.com/qmk/qmk_firmware)의 키보드가 변경될 때마다 생성됩니다. 생성된 JSON 파일은 Spaces에 업로드되어 Configurator가 각 키보드에 대한 UI를 생성하는 데 사용됩니다. 이 작업의 상태는 [qmk_firmware actions 탭](https://github.com/qmk/qmk_firmware/actions/workflows/api.yml)에서 확인할 수 있습니다. QMK 협력자인 경우 `workflow_dispatch` 이벤트 트리거를 사용하여 이 작업을 수동으로 실행할 수 있습니다.

## QMK API

주소: <http://api.qmk.fm>

QMK API는 컴파일을 위해 `keymap.json` 파일을 수락합니다. 이 파일들은 `qmk compile` 및 `qmk flash`와 직접 사용할 수 있는 동일한 파일입니다. `keymap.json`이 제출되면 브라우저는 주기적으로 (최소 2초마다) 작업 상태를 폴링하여 완료될 때까지 기다립니다. 최종 상태 JSON은 키맵의 소스 및 바이너리 다운로드에 대한 포인터를 포함합니다.

QMK API는 항상 소스 및 바이너리 다운로드를 GPL 준수를 위해 나란히 표시합니다.

오류가 아닌 상태 응답에는 3가지가 있습니다.

1. 컴파일 작업 대기 중
2. 컴파일 작업 실행 중
3. 컴파일 작업 완료

### 컴파일 작업 대기 중

이 상태는 작업이 아직 [QMK Compiler](#qmk-compiler) 노드에서 처리되지 않았음을 나타냅니다. Configurator는 이 상태를 "오븐을 기다리는 중"으로 표시합니다.

### 컴파일 작업 실행 중

이 상태는 작업이 컴파일되기 시작했음을 나타냅니다. Configurator는 이 상태를 "베이킹 중"으로 표시합니다.

### 컴파일 작업 완료

이 상태는 작업이 완료되었음을 나타냅니다. 상태 JSON에 소스 및 바이너리 다운로드 키가 있습니다.

## Redis/RQ

QMK API는 RQ를 사용하여 작업을 사용할 수 있는 [QMK Compiler](#qmk-compiler) 노드에 배포합니다. `keymap.json`이 수신되면 RQ 큐에 넣어지고, `qmk_compiler` 노드가 이를 처리합니다.

## QMK Compiler

[QMK Compiler](https://github.com/qmk/qmk_compiler)는 `keymap.json`의 컴파일을 실제로 수행합니다. 요청된 `qmk_firmware` 브랜치를 체크아웃하고 `qmk compile keymap.json`을 실행한 다음 결과 소스 및 바이너리를 Digital Ocean Spaces에 업로드합니다.

사용자가 소스/바이너리를 다운로드할 때 API는 인증된 Spaces 다운로드 URL로 리디렉션합니다.