# 외부 QMK 사용자 공간

QMK 펌웨어는 이제 사용자 키맵을 일반 QMK 펌웨어 리포지토리 외부에 저장하는 것을 공식적으로 지원합니다. 이를 통해 사용자는 QMK 펌웨어를 포크, 수정 및 유지 관리할 필요 없이 자신의 키맵을 관리할 수 있습니다.

외부 사용자 공간은 주요 QMK 펌웨어 리포지토리의 구조를 반영하지만, 빌드하려는 키맵만 포함합니다. 여전히 `keyboards/<my keyboard>/keymaps/<my keymap>`을 사용하여 키맵을 저장하거나 이전과 같이 `layouts/<my layout>/<my keymap>` 시스템을 사용할 수 있습니다. 단, QMK 펌웨어 외부에 저장됩니다.

빌드 시스템은 전통적인 QMK 펌웨어 [사용자 공간 기능](feature_userspace)을 사용하는 경우 `users/<my keymap>`의 사용을 여전히 허용합니다. 이제 외부에서도 동일한 위치에서 지원됩니다.

또한 GitHub Actions를 사용하여 키맵을 빌드하는 것을 첫 번째 클래스 지원으로 제공하여, 외부 사용자 공간 리포지토리에 변경 사항을 푸시할 때마다 키맵을 자동으로 컴파일할 수 있습니다.

::: warning
외부 사용자 공간은 새로운 기능으로, 문제가 있을 수 있습니다. `qmk` 명령과의 더 긴밀한 통합은 시간이 지남에 따라 이루어질 것입니다.
:::

::: tip
기존의 keymap.json 및 GitHub 기반 펌웨어 빌드 지침은 [여기](newbs_building_firmware_workflow)에서 찾을 수 있습니다. 이 문서는 그 지침을 대체하지만, 여전히 올바르게 작동해야 합니다.
:::

## QMK 로컬 설정

로컬 머신에서 빌드하려면 QMK를 로컬로 설정해야 합니다. 이는 한 번만 수행하면 되며, [초보자 설정 가이드](newbs)에 문서화되어 있습니다.

::: warning
외부 사용자 공간 정의를 조작하는 QMK CLI 명령을 사용하려면 QMK 펌웨어의 사본도 필요합니다.
:::

::: warning
로컬에서 빌드하는 것은 GitHub Actions가 완료되기를 기다리는 것보다 훨씬 빠릅니다.
:::

## 외부 사용자 공간 리포지토리 설정 (GitHub에서 포크)

기본적인 외부 사용자 공간 리포지토리는 [여기](https://github.com/qmk/qmk_userspace)에서 찾을 수 있습니다. 키맵을 GitHub에 유지하려면(강력히 권장됨!), 리포지토리를 포크하고 이를 기반으로 사용할 수 있습니다:

![Userspace Fork](https://i.imgur.com/hcegguh.png)

포크를 진행하면 계정에 복사되며, 로컬 머신으로 클론한 다음 키맵을 추가할 수 있습니다:

![Userspace Clone](https://i.imgur.com/CWYmsk8.png)

```sh
cd $HOME
git clone https://github.com/{myusername}/qmk_userspace.git
qmk config user.overlay_dir="$(realpath qmk_userspace)"
```

## 외부 사용자 공간 설정 (로컬에만 저장)

GitHub를 사용하지 않고 모든 것을 로컬로 유지하려는 경우, 기본 외부 사용자 공간의 사본을 로컬로 클론할 수 있습니다:

```sh
cd $HOME
git clone https://github.com/qmk/qmk_userspace.git
qmk config user.overlay_dir="$(realpath qmk_userspace)"
```

## 키맵 추가

_이 지침은 이미 QMK를 로컬로 설정하고, 머신에 QMK 펌웨어 리포지토리가 있는 상태를 전제로 합니다._

외부 사용자 공간 내의 키맵은 주요 QMK 리포지토리와 동일한 방식으로 정의됩니다. `qmk new-keymap` 명령을 사용하여 새 키맵을 생성하거나 `keyboards` 디렉토리에 새 디렉토리를 수동으로 생성할 수 있습니다.

또는 `layouts` 디렉토리를 사용하여 키맵을 저장할 수 있으며, 주요 QMK 리포지토리와 동일한 레이아웃 시스템을 사용할 수 있습니다. 이 경우 키맵 파일을 `layouts/<layout name>/<keymap name>/keymap.*` 경로에 저장해야 하며, `layout name`은 QMK의 기존 레이아웃과 일치해야 합니다(예: `tkl_ansi`).

새 키맵을 생성한 후, 키맵 빌드는 일반적인 QMK 사용과 일치합니다:

```sh
qmk compile -kb <keyboard> -km <keymap>
```

::: warning
외부 사용자 공간 리포지토리를 클론할 때 `qmk config user.overlay_dir=...` 명령을 실행해야 올바르게 작동합니다.
:::

## 외부 사용자 공간 빌드 대상에 키맵 추가

키맵을 생성한 후, GitHub Actions를 사용하여 펌웨어를 빌드하려면 외부 사용자 공간 빌드 대상에 추가해야 합니다. 이는 `qmk userspace-add` 명령을 사용하여 수행됩니다:

```sh
# 키보드/키맵 조합의 경우:
qmk userspace-add -kb <keyboard> -km <keymap>
# 또는, json 기반 키맵의 경우(독립적으로 유지되는 경우):
qmk userspace-add <relative/path/to/my/keymap.json>
```

이 명령은 외부 사용자 공간 디렉토리의 루트에 있는 `qmk.json` 파일을 업데이트합니다. 키맵을 저장하는 데 git 리포지토리를 사용하는 경우, 이제 자신의 포크에 커밋하고 푸시할 좋은 기회입니다.

## 외부 사용자 공간 빌드 대상 컴파일

키맵을 외부 사용자 공간 빌드 대상에 추가한 후, `qmk userspace-compile` 명령을 사용하여 한 번에 모두 컴파일할 수 있습니다:

```sh
qmk userspace-compile
```

외부 사용자 공간 빌드 대상에 추가된 모든 펌웨어 빌드가 컴파일되며, 생성된 펌웨어 파일은 외부 사용자 공간 디렉토리의 루트에 배치됩니다.

## GitHub Actions 사용

GitHub Actions를 사용하여 외부 사용자 공간 리포지토리에 변경 사항을 푸시할 때마다 키맵을 자동으로 빌드할 수 있습니다. 빌드 대상 목록을 설정한 경우, 이는 GitHub 리포지토리 설정에서 워크플로우를 활성화하는 것만큼 간단합니다:

![Repo Settings](https://i.imgur.com/EVkxOt1.png)

어떤 푸시도 구성된 모든 빌드를 컴파일하며, 완료되면 새로 생성된 펌웨어 파일이 포함된 새 릴리스를 GitHub에서 생성하여 이를 다운로드하고 키보드에 플래시할 수 있습니다:

![Releases](https://i.imgur.com/zmwOL5P.png)