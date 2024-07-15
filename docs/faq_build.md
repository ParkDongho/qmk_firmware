# 자주 묻는 빌드 질문

이 페이지에서는 QMK 빌드와 관련된 자주 묻는 질문들을 다룹니다. 아직 읽지 않으셨다면 [빌드 환경 설정](newbs_getting_started) 및 [Make 지침](getting_started_make_guide) 가이드를 읽어보세요.

## 리눅스에서 프로그래밍이 안 돼요
디바이스를 작동하려면 적절한 권한이 필요합니다. 리눅스 사용자는 아래의 `udev` 규칙에 대한 지침을 참조하세요. `udev`에 문제가 있는 경우, `sudo` 명령을 사용하는 것이 대안이 될 수 있습니다. `sudo` 명령에 익숙하지 않은 경우, `man sudo` 명령어를 참고하거나 [이 웹페이지](https://linux.die.net/man/8/sudo)를 참조하세요.

예를 들어, 컨트롤러가 ATMega32u4인 경우 다음과 같이 사용할 수 있습니다:

```sh
$ sudo dfu-programmer atmega32u4 erase --force
$ sudo dfu-programmer atmega32u4 flash your.hex
$ sudo dfu-programmer atmega32u4 reset
```

또는 다음과 같이 할 수도 있습니다:

```sh
$ sudo make <keyboard>:<keymap>:flash
```

`make`를 `sudo`로 실행하는 것은 일반적으로 ***좋지 않은*** 방법이므로 가능한 경우 위의 다른 방법을 사용하세요.

### 리눅스 `udev` 규칙 {#linux-udev-rules}

리눅스에서는 부트로더 디바이스와 통신하려면 적절한 권한이 필요합니다. 펌웨어를 플래시할 때 `sudo`를 사용할 수도 있지만(권장하지 않음), [이 파일](https://github.com/qmk/qmk_firmware/tree/master/util/udev/50-qmk.rules)을 `/etc/udev/rules.d/`에 넣을 수 있습니다.

파일을 추가한 후 다음 명령어를 실행하세요:

```sh
sudo udevadm control --reload-rules
sudo udevadm trigger
```

**참고:** ModemManager의 이전 버전(< 1.12)에서는 필터링이 엄격 모드에서만 작동합니다. 다음 명령어를 실행하여 해당 설정을 업데이트할 수 있습니다:

```sh
printf '[Service]\nExecStart=\nExecStart=/usr/sbin/ModemManager --filter-policy=default' | sudo tee /etc/systemd/system/ModemManager.service.d/policy.conf
sudo systemctl daemon-reload
sudo systemctl restart ModemManager
```

### 부트로더 모드에서 시리얼 디바이스가 감지되지 않아요
커널에 디바이스에 적합한 지원이 있는지 확인하세요. USB ACM을 사용하는 디바이스(Pro Micro(Atmega32u4) 등)의 경우 `CONFIG_USB_ACM=y`가 포함되어 있는지 확인하세요. 다른 디바이스는 `USB_SERIAL` 및 그 하위 옵션이 필요할 수 있습니다.

## DFU 부트로더에 대한 알 수 없는 디바이스

윈도우에서 키보드를 플래시할 때 발생하는 문제는 잘못된 드라이버가 설치되었거나 드라이버가 아예 설치되지 않았을 때가 가장 많습니다.

QMK 설치 스크립트를 다시 실행하거나(`./util/qmk_install.sh`를 MSYS2 또는 WSL에서 실행) QMK Toolbox를 다시 설치하면 문제를 해결할 수 있습니다. 그렇지 않으면 [`qmk_driver_installer`](https://github.com/qmk/qmk_driver_installer) 패키지를 수동으로 다운로드하고 실행해 보세요.

그래도 문제가 해결되지 않으면 Zadig을 다운로드하고 실행해야 할 수 있습니다. 자세한 내용은 [Zadig을 사용한 부트로더 드라이버 설치](driver_installation_zadig)를 참조하세요.

## USB VID와 PID
`config.h`를 편집하여 원하는 ID를 사용할 수 있습니다. 일반적으로 사용되지 않는 ID를 사용하는 것은 실제로 아무런 문제가 없지만, 다른 제품과 충돌할 가능성은 거의 없습니다.

대부분의 QMK 보드에서는 `0xFEED`를 공급업체 ID로 사용합니다. 다른 키보드를 확인하여 고유한 제품 ID를 선택하는 것이 좋습니다.

또한 이 페이지도 참고하세요:
https://github.com/tmk/tmk_keyboard/issues/150

개인 사용을 위해서는 정말로 고유한 VID:PID를 구입할 수 있습니다.
- https://www.obdev.at/products/vusb/license.html
- https://www.mcselec.com/index.php?page=shop.product_details&flypage=shop.flypage&product_id=92&option=com_phpshop&Itemid=1

### 키보드를 플래시했는데 아무것도 안 돼요/키 입력이 등록되지 않아요 - 또한 ARM 기반이에요 (예: Planck rev6, Clueboard 60, HS60v2 등) (2019년 2월)
ARM 기반 칩에서 EEPROM이 작동하는 방식으로 인해 저장된 설정이 더 이상 유효하지 않을 수 있습니다. 이로 인해 기본 레이어가 영향을 받으며, 특정 상황에서는 키보드를 사용할 수 없게 만들 수도 있습니다. EEPROM을 재설정하면 이 문제를 해결할 수 있습니다.

[Planck rev6 EEPROM 재설정](https://cdn.discordapp.com/attachments/473506116718952450/539284620861243409/planck_rev6_default.bin)을 사용하여 EEPROM을 강제로 재설정할 수 있습니다. 이 이미지를 플래시한 후 정상적인 펌웨어를 다시 플래시하면 키보드가 정상적으로 작동할 것입니다.
[Preonic rev3 EEPROM 재설정](https://cdn.discordapp.com/attachments/473506116718952450/537849497313738762/preonic_rev3_default.bin)

부트매직이 어떤 형태로든 활성화되어 있는 경우에도 이를 수행할 수 있습니다(자세한 내용은 [부트매직 문서](features/bootmagic) 및 키보드 정보를 참조하세요).