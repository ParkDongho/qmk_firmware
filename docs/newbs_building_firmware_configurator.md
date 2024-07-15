# QMK 설정 도구

[![QMK 설정 도구 스크린샷](https://i.imgur.com/anw9cOL.png)](https://config.qmk.fm/)

[QMK 설정 도구](https://config.qmk.fm)는 QMK 펌웨어 `.hex` 또는 `.bin` 파일을 생성하는 온라인 그래픽 사용자 인터페이스입니다.

설정 도구는 설계된 것과 다른 컨트롤러를 사용하는 키보드에 대한 펌웨어를 생성할 수 없다는 점을 유의해야 합니다. 예를 들어, pro micro용으로 설계된 보드에 RP2040 컨트롤러를 사용하는 경우에는 사용할 수 없습니다. 이 경우 명령줄 [변환기](feature_converters#supported-converters)를 사용해야 합니다.

[비디오 튜토리얼](https://www.youtube.com/watch?v=-imgglzDMdY)을 시청하십시오. 많은 사람들이 이 비디오만으로도 자신만의 키보드를 프로그래밍하는 데 충분한 정보를 얻을 수 있습니다.

QMK 설정 도구는 Chrome이나 Firefox에서 가장 잘 작동합니다.

::: warning
**참고: Keyboard Layout Editor (KLE) 또는 kbfirmware와 같은 다른 도구에서 생성된 파일은 QMK 설정 도구와 호환되지 않습니다. 해당 파일을 로드하거나 가져오지 마십시오. QMK 설정 도구는 다른 도구입니다.**
:::

[QMK 설정 도구: 단계별 가이드](configurator_step_by_step)를 참조하십시오.