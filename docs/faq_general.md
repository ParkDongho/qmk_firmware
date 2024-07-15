# 자주 묻는 질문 (FAQ)

## QMK란 무엇인가요?

[QMK](https://github.com/qmk)는 Quantum Mechanical Keyboard의 약자로, 커스텀 키보드를 위한 도구를 만드는 사람들의 모임입니다. 우리는 [QMK 펌웨어](https://github.com/qmk/qmk_firmware)로 시작했으며, 이는 [TMK](https://github.com/tmk/tmk_keyboard)를 많이 수정한 포크입니다.

## 어디서부터 시작해야 할지 모르겠어요!

이 경우, [Newbs Guide](newbs)부터 시작하는 것이 좋습니다. 거기에는 시작하는 데 필요한 많은 정보가 포함되어 있습니다.

만약 그것이 어려운 문제라면, [QMK 설정 도구](https://config.qmk.fm)를 사용해 보세요. 이 도구는 필요한 대부분의 작업을 처리해 줄 것입니다.

## 빌드한 펌웨어를 어떻게 플래시할 수 있나요?

먼저 [컴파일/플래싱 FAQ 페이지](faq_build)로 이동하세요. 거기에는 많은 정보가 있으며, 일반적인 문제에 대한 많은 해결책을 찾을 수 있습니다.

## 여기에서 다루지 않는 문제가 생기면 어떻게 하나요?

괜찮습니다. [GitHub의 오픈 이슈](https://github.com/qmk/qmk_firmware/issues)를 확인하여 누군가가 같은 문제를 겪고 있는지 확인하세요(유사한 문제인지, 실제로 동일한 문제인지 확인하세요).

그래도 찾을 수 없다면, [새 이슈를 열어주세요](https://github.com/qmk/qmk_firmware/issues/new)!

## 버그를 발견하면 어떻게 하나요?

그러면 [이슈](https://github.com/qmk/qmk_firmware/issues/new)를 열어주세요. 만약 해결 방법을 알고 있다면, GitHub에 수정 사항을 포함한 Pull Request를 열어주세요.

## `git`과 `GitHub`가 어렵게 느껴져요!

걱정하지 마세요, `git`과 GitHub를 쉽게 사용할 수 있도록 도와주는 [가이드라인](newbs_git_best_practices)이 있습니다.

추가적으로, `git` 및 GitHub와 관련된 링크를 [여기](newbs_learn_more_resources)에서 찾을 수 있습니다.

## 지원을 추가하고 싶은 키보드가 있어요

좋아요! Pull Request를 열어주세요. 코드를 검토하고 병합하겠습니다!

### QMK로 브랜드를 만들고 싶다면 어떻게 하나요?

멋지네요! 도와드리겠습니다!

사실, 우리는 페이지와 키보드에 QMK 브랜드를 추가하는 것에 관한 [전용 페이지](https://qmk.fm/powered/)가 있습니다. 여기에는 공식적으로 QMK를 지원하는 데 필요한 모든 지식과 이미지가 포함되어 있습니다.

이와 관련하여 궁금한 점이 있으면 이슈를 열거나 [Discord](https://discord.gg/Uq7gcHh)로 오세요.

## QMK와 TMK의 차이점은 무엇인가요?

TMK는 원래 [Jun Wako](https://github.com/tmk)에 의해 설계되고 구현되었습니다. QMK는 [Jack Humbert](https://github.com/jackhumbert)가 Planck용으로 만든 TMK의 포크로 시작했습니다. 시간이 지나면서 Jack의 포크는 TMK와 상당히 달라졌고, 2015년에 Jack은 자신의 포크를 QMK로 이름을 바꾸기로 결정했습니다.

기술적인 관점에서 QMK는 여러 가지 새로운 기능을 추가하여 TMK를 기반으로 구축되었습니다. 가장 눈에 띄는 점은 QMK가 사용할 수 있는 키코드의 수를 확장하고, 이를 통해 `S()`, `LCTL()`, `MO()`와 같은 고급 기능을 구현한다는 것입니다. 이러한 키코드의 전체 목록은 [Keycodes](keycodes)에서 볼 수 있습니다.

프로젝트 및 커뮤니티 관리 측면에서 TMK는 커뮤니티의 약간의 지원을 받으면서 공식적으로 지원되는 모든 키보드를 스스로 유지 관리합니다. 다른 키보드를 위해 별도의 커뮤니티 유지 포크가 존재하거나 생성될 수 있습니다. 기본적으로 몇 가지 키맵만 제공되므로 사용자들은 일반적으로 키맵을 서로 공유하지 않습니다. QMK는 중앙 관리 리포지토리를 통해 키보드와 키맵을 공유하는 것을 권장하며, 품질 기준을 따르는 모든 Pull Request를 수락합니다. 이는 주로 커뮤니티가 유지하지만, 필요한 경우 QMK 팀도 도움을 줍니다.

두 접근 방식 모두 장단점이 있으며, TMK와 QMK 간의 코드는 필요할 때 자유롭게 흐릅니다.