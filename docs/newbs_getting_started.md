# QMK 환경 설정

키맵을 빌드하기 전에 소프트웨어를 설치하고 빌드 환경을 설정해야 합니다. 이는 얼마나 많은 키보드의 펌웨어를 컴파일하더라도 한 번만 하면 됩니다.

## 1. 필수 조건

시작하려면 몇 가지 소프트웨어가 필요합니다.

* [텍스트 에디터](newbs_learn_more_resources#text-editor-resources)
  * 일반 텍스트 파일을 편집하고 저장할 수 있는 프로그램이 필요합니다. 많은 운영 체제에 기본적으로 제공되는 에디터는 일반 텍스트 파일을 저장하지 않으므로, 선택한 에디터가 일반 텍스트 파일을 저장할 수 있는지 확인해야 합니다.
* [Toolbox (선택 사항)](https://github.com/qmk/qmk_toolbox)
  * Windows 및 macOS용 그래픽 프로그램으로, 맞춤형 키보드를 프로그래밍하고 디버그할 수 있습니다.

::: tip
Linux/Unix 명령 줄을 사용해본 적이 없다면, 몇 가지 기본 개념과 명령어를 배우는 것이 좋습니다. [이 자료들](newbs_learn_more_resources#command-line-resources)은 QMK와 함께 작업할 수 있을 정도로 충분히 가르쳐 줄 것입니다.
:::

## 2. 빌드 환경 준비 {#set-up-your-environment}

QMK를 가능한 한 쉽게 설정할 수 있도록 노력했습니다. Linux 또는 Unix 환경만 준비한 다음, 나머지는 QMK가 설치하도록 합니다.

:::::tabs

==== Windows

QMK는 MSYS2, CLI 및 모든 필수 종속성을 포함하는 번들을 유지 관리합니다. 또한 올바른 환경으로 직접 부팅할 수 있는 편리한 `QMK MSYS` 터미널 바로가기를 제공합니다.

#### 필수 조건

[QMK MSYS](https://msys.qmk.fm/)를 설치해야 합니다. 최신 릴리스는 [여기](https://github.com/qmk/qmk_distro_msys/releases/latest)에서 사용할 수 있습니다.

:::: details 고급 사용자

::: danger
<b style="font-size:150%">이 과정은 초보자에게 권장되지 않습니다.</b>
:::

MSYS2를 수동으로 설치하려면 다음 섹션을 따르십시오.

#### 필수 조건

[MSYS2](https://www.msys2.org)를 설치해야 합니다. 설치가 완료되면 모든 열린 MSYS 터미널(보라색 아이콘)을 닫고 시작 메뉴에서 새로운 MinGW 64비트 터미널(파란색 아이콘)을 엽니다.

::: warning
**참고:** MinGW 64비트 터미널은 설치가 완료될 때 열리는 MSYS 터미널과 동일하지 않습니다. 프롬프트에 "MINGW64"가 보라색 텍스트로 표시되어야 하며, "MSYS"가 아닙니다. 차이점에 대한 자세한 내용은 [이 페이지](https://www.msys2.org/wiki/MSYS2-introduction/#subsystems)를 참조하십시오.
:::

#### 설치

다음 명령을 실행하여 QMK CLI를 설치합니다:

```sh
pacman --needed --noconfirm --disable-download-timeout -S git mingw-w64-x86_64-python-qmk
```

::::

==== macOS

QMK는 CLI 및 모든 필수 종속성을 자동으로 설치하는 Homebrew 탭과 공식을 유지 관리합니다.

#### 필수 조건

Homebrew를 설치해야 합니다. https://brew.sh의 지침을 따르십시오.

::: tip
Apple Silicon 머신을 사용하는 경우, GitHub 액션에 ARM 및 AVR 툴체인용 바이너리 패키지를 빌드하는 네이티브 러너가 없기 때문에 설치 과정이 상당히 오래 걸릴 수 있습니다.
:::

#### 설치

다음 명령을 실행하여 QMK CLI를 설치합니다:

```sh
brew install qmk/qmk/qmk
```

==== Linux/WSL

::: tip
**WSL 사용자 참고**: 기본적으로 설치 과정은 WSL 홈 디렉토리에 QMK 리포지토리를 클론하지만, 수동으로 클론한 경우 WSL 인스턴스 내에 위치하도록 해야 하며 Windows 파일 시스템(즉, `/mnt` 내)이 아니어야 합니다. 현재 접근 속도가 [매우 느립니다](https://github.com/microsoft/WSL/issues/4197).
:::

#### 필수 조건

Git 및 Python을 설치해야 합니다. 이미 둘 다 설치되어 있을 가능성이 높지만, 설치되지 않은 경우 다음 명령 중 하나를 실행하여 설치할 수 있습니다:

* Debian / Ubuntu / Devuan: `sudo apt install -y git python3-pip`
* Fedora / Red Hat / CentOS: `sudo yum -y install git python3-pip`
* Arch / Manjaro: `sudo pacman --needed --noconfirm -S git python-pip libffi`
* Void: `sudo xbps-install -y git python3-pip`
* Solus: `sudo eopkg -y install git python3`
* Sabayon: `sudo equo install dev-vcs/git dev-python/pip`
* Gentoo: `sudo emerge dev-vcs/git dev-python/pip`

#### 설치

다음 명령을 실행하여 QMK CLI를 설치합니다:

```sh
python3 -m pip install --user qmk
```

#### 커뮤니티 패키지

이 패키지는 커뮤니티 구성원이 유지 관리하므로 최신 상태가 아니거나 완전히 기능하지 않을 수 있습니다. 문제가 발생하면 해당 유지 관리자에게 보고하십시오.

Arch 기반 배포판에서는 공식 리포지토리에서 CLI를 설치할 수 있습니다(작성 당시 일부 종속성을 선택적으로 표시하는 패키지):

```sh
sudo pacman -S qmk
```

AUR에서 `qmk-git` 패키지를 시도할 수도 있습니다:

```sh
yay -S qmk-git
```

==== FreeBSD

#### 설치

다음 명령을 실행하여 QMK CLI용 FreeBSD 패키지를 설치합니다:

```sh
pkg install -g "py*-qmk"
```

참고: 설치가 완료되면 출력되는 지침을 따르십시오(`pkg info -Dg "py*-qmk"`를 사용하여 다시 볼 수 있습니다).

:::::

## 3. QMK 설정 실행 {#set-up-qmk}

::::tabs

=== Windows

QMK MSYS를 열고 다음 명령을 실행하십시오:

```sh
qmk setup
```

대부분의 경우 모든 프롬프트에 `y`로 답하는 것이 좋습니다.

=== macOS

터미널을 열고 다음 명령을 실행하십시오:

```sh
qmk setup
```

대부분의 경우 모든 프롬프트에 `y`로 답하는 것이 좋습니다.

=== Linux/WSL

선호하는 터미널 앱을 열고 다음 명령을 실행하십시오:

```sh
qmk setup
```

대부분의 경우 모든 프롬프트에 `y`로 답하는 것이 좋습니다.

::: info Debian, Ubuntu 및 파생 버전에 대한 참고 사항:
`bash: qmk: command not found`와 같은 오류가 발생할 수 있습니다.
이것은 Debian이 Bash 4.4 릴리스에서 `$HOME/.local/bin`을 PATH에서 제거한 [버그](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=839155) 때문입니다. 이 버그는 나중에 Debian과 Ubuntu에서 수정되었습니다.
안타깝게도 Ubuntu는 이 버그를 다시 도입했고 [아직 수정되지 않았습니다](https://bugs.launchpad.net/ubuntu/+source/bash/+bug/1588562).
다행히도 수정은 간단합니다. 사용자로서 다음 명령을 실행하십시오: `echo 'PATH="$HOME/.local/bin:$PATH"' >> $HOME/.bashrc && source $HOME/.bashrc`
:::

=== FreeBSD

선호하는 터미널 앱을 열고 다음 명령을 실행하십시오:

```sh
qmk setup
```

대부분의 경우 모든 프롬프트에 `y`로 답하는 것이 좋습니다.

::::

::: tip
QMK 홈 폴더는 `qmk setup -H <path>`로 설정 시 지정할 수 있으며, 이후 [CLI 구성](cli_configuration#single-key-example) 및 `user.qmk_home` 변수를 사용하여 수정할 수 있습니다. 사용 가능한 모든 옵션은 `qmk setup --help`를 실행하여 확인할 수 있습니다.
:::

::: tip
GitHub 사용 방법을 이미 알고 있다면, [이 지침을 따르고](getting_started_github) `qmk setup <github_username>/qmk_firmware`를 사용하여 개인 포크를 클론하는 것이 좋습니다. 그렇지 않다면 이 메시지를 무시해도 됩니다.
:::

## 4. 빌드 환경 테스트

QMK 빌드 환경을 설정했으므로, 키보드의 펌웨어를 빌드할 수 있습니다. 키보드의 기본 키맵을 빌드하여 시작해 보십시오. 다음 형식의 명령으로 이를 수행할 수 있어야 합니다:

```sh
qmk compile -kb <keyboard> -km default
```

예를 들어, Clueboard 66%용 펌웨어를 빌드하려면 다음 명령을 사용합니다:

```sh
qmk compile -kb clueboard/66/rev3 -km

 default
```

::: tip
키보드 옵션은 키보드 디렉토리에 대한 상대 경로입니다. 위의 예제는 `qmk_firmware/keyboards/clueboard/66/rev3`에 있습니다. 확실하지 않은 경우 `qmk list-keyboards`를 사용하여 지원되는 키보드 목록을 볼 수 있습니다.
:::

작업이 완료되면 다음과 유사한 출력이 나타납니다:

```
Linking: .build/clueboard_66_rev3_default.elf                                                       [OK]
Creating load file for flashing: .build/clueboard_66_rev3_default.hex                               [OK]
Copying clueboard_66_rev3_default.hex to qmk_firmware folder                                        [OK]
Checking file size of clueboard_66_rev3_default.hex                                                 [OK]
 * The firmware size is fine - 26356/28672 (2316 bytes free)
```

# 키맵 생성

이제 개인 키맵을 만들 준비가 되었습니다! [첫 펌웨어 빌드하기](newbs_building_firmware)로 이동하십시오.