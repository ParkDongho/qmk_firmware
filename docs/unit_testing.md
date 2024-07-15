# 단위 테스트

단위 테스트가 처음이시라면 인터넷에서 많은 좋은 자료를 찾을 수 있습니다. 그러나 대부분의 자료는 여기저기 흩어져 있고, 다양한 의견이 있으므로 특정 자료를 추천하지는 않겠습니다.

대신 두 가지 스타일의 단위 테스트를 자세히 설명한 두 권의 책을 추천합니다.

* "Test Driven Development: By Example: Kent Beck"
* "Growing Object-Oriented Software, Guided By Tests: Steve Freeman, Nat Pryce"

동영상을 선호하신다면, Uncle Bob의 [Clean Coders Videos](https://cleancoders.com/)를 참고할 수 있습니다. 다만, 많은 영상을 시청하려면 비용이 꽤 들 수 있습니다. James Shore의 무료 [Let's Play](https://www.jamesshore.com/Blog/Lets-Play) 비디오 시리즈도 있습니다.

## Google Test 및 Google Mock
[Google Test](https://github.com/google/googletest)를 사용하여 코드의 단위 테스트를 작성할 수 있습니다. Google Test 프레임워크에는 테스트 목과 스텁을 작성하기 위한 또 다른 구성 요소인 "Google Mock"도 포함되어 있습니다. 실제 테스트 작성 방법에 대한 정보는 해당 사이트의 문서를 참조하십시오.

## C++ 사용

Google Test 및 모든 테스트는 C++로 작성해야 하며, QMK 코드베이스의 나머지 부분이 C로 작성되었더라도 마찬가지입니다. C++을 전혀 모른다 하더라도 필요한 C++ 기능에 대한 명확한 문서와 예제가 있어 큰 문제가 되지 않을 것입니다. 테스트 코드의 나머지 부분은 거의 일반적인 C 코드처럼 작성할 수 있습니다. 컴파일러 오류가 무섭게 보일 수 있지만, 주의 깊게 읽으면 해결할 수 있을 것입니다.

기억할 점은 모든 C 파일 포함문 주위에 `extern "C"`를 추가해야 한다는 것입니다.

## 새로운 기능 또는 기존 기능에 대한 테스트 추가

특정 기능을 단위 테스트하려면, `quantum/sequencer/tests` 폴더에 있는 기존 테스트를 확인하십시오. 그런 다음 아래 단계를 따라 유사한 구조를 만드십시오.

1. 기능이 포함된 폴더에 테스트 하위 폴더를 추가합니다(존재하지 않는 경우).
2. 해당 폴더에 `testlist.mk` 및 `rules.mk` 파일을 만듭니다.
3. 루트 폴더의 `testlist.mk` 및 `build_test.mk`에서 해당 파일을 포함합니다.
4. `testlist.mk` 파일에 테스트 그룹의 새 이름을 추가합니다. 거기 정의된 각 그룹은 별도의 실행 파일이 됩니다. 이렇게 하면 다양한 부분을 목(mock)으로 처리할 수 있습니다. 기존 테스트처럼 공통 접두사를 추가하는 것이 좋습니다. 이렇게 하면 make 명령으로 하위 문자열 필터링을 수행할 수 있어 테스트의 하위 집합을 쉽게 실행할 수 있습니다.
5. `rules.mk` 파일에 소스 파일 및 필요한 옵션을 정의합니다.
   * `_SRC`: 소스 파일
   * `_DEFS`: 추가 정의
   * `_INC`: 추가 포함 폴더
6. 테스트 폴더에 새로운 cpp 파일을 작성합니다. 해당 파일은 `rules.mk` 파일에서 포함된 파일 중 하나여야 합니다.

각기 다른 부분을 목(mock)으로 처리하는 여러 테스트가 있는 것을 알 수 있습니다. 또한 테스트에 필요한 최소한의 것만 컴파일하는 것을 알 수 있습니다. 가능한 최소한의 코드만 컴파일하도록 권장합니다. 관련 동영상은 [Matt Hargett "Advanced Unit Testing in C & C++](https://www.youtube.com/watch?v=Wmy6g-aVgZI)을 참조하십시오.

## 테스트 실행

코드베이스의 모든 테스트를 실행하려면 `make test:all`을 입력하십시오. 하위 문자열과 일치하는 테스트를 실행하려면 `make test:matchingsubstring`을 입력하십시오. `matchingsubstring`에는 더 구체적인 이름을 지정하기 위해 콜론을 포함할 수 있습니다. `make test:tap_hold_configurations`는 모든 기능에 대해 `tap_hold_configurations` 테스트를 실행하고, `make test:retro_shift:tap_hold_configurations`는 Retro Shift 기능에 대해서만 `tap_hold_configurations` 테스트를 실행합니다.

테스트는 항상 플랫폼의 기본 컴파일러로 컴파일되므로 컴퓨터에서 다른 프로그램처럼 실행됩니다.

## 테스트 디버깅

테스트에 문제가 있는 경우 `./build/test` 폴더에서 실행 파일을 찾을 수 있습니다. 이를 GDB 또는 유사한 디버거로 실행할 수 있어야 합니다.

[디버그 메시지](unit_testing#debug-api)를 `stderr`로 전달하려면 `DEBUG=1`로 테스트를 실행할 수 있습니다. 예를 들어

```
make test:all DEBUG=1
```

또는 테스트 `rules.mk`에 `CONSOLE_ENABLE=yes`를 추가하십시오.

## 전체 통합 테스트

현재 전체 펌웨어를 컴파일하고 테스트할 키맵을 정의하는 전체 통합 테스트는 불가능합니다. 그러나 그렇게 하기 위한 계획이 있으며, 이는 단위 테스트에 익숙하지 않은 사람들에게는 더 쉬울 것입니다.

이 모델에서는 입력을 에뮬레이트하고 에뮬레이트된 키보드에서 특정 출력을 기대하게 됩니다.

# 변수 추적 {#tracing-variables}

때때로 변수가 왜 변경되었는지, 어디에서 변경되었는지 궁금할 수 있으며, 디버거가 없으면 이를 추적하는 것이 상당히 까다로울 수 있습니다. 물론 수동으로 출력문을 추가하여 추적할 수 있지만 변수 추적 기능을 활성화할 수도 있습니다. 이 기능은 코드에 의해 변경된 변수와 메모리 손상으로 인해 변경된 변수 모두에 대해 작동합니다.

이 기능을 사용하려면 `make` 명령의 끝에 `VARIABLE_TRACE=x`를 추가하십시오. 여기서 `x`는 추적하려는 변수의 수를 나타내며, 일반적으로 1입니다.

그런 다음 코드의 적절한 위치에서 `ADD_TRACED_VARIABLE`을 호출하여 추적을 시작하십시오. 예를 들어 모든 레이어 변경 사항을 추적하려면 다음과 같이 할 수 있습니다.

```c
void matrix_init_user(void) {
  ADD_TRACED_VARIABLE("layer", &layer_state, sizeof(layer_state));
}
```

이렇게 하면 `layer_state`의 메모리 위치를 추적하는 "layer"라는 추적 변수가 추가됩니다(이 이름은 정보 제공용입니다). 4바이트(`layer_state`의 크기)를 추적하여 변수의 모든 수정 사항을 보고합니다. 기본적으로 4보다 큰 크기를 지정할 수 없지만 `make` 명령 줄 끝에 `MAX_VARIABLE_TRACE_SIZE=x`를 추가하여 변경할 수 있습니다.

변수 변경 사항을 실제로 감지하려면 변수를 수정하는 코드 주위에 `VERIFY_TRACED_VARIABLES`를 호출해야 합니다. 변수가 수정되면 두 `VERIFY_TRACED_VARIABLES` 호출 사이에 수정이 발생했음을 알려줍니다. 그런 다음 더 추적하기 위해 호출을 더 추가할 수 있습니다. 코드베이스에 호출을 남발하는 것은 추천하지 않습니다. 몇 개의 호출로 시작한 다음, 이진 검색 방식으로 계속 추가하는 것이 좋습니다. 더 이상 필요하지 않은 호출은 삭제할 수도 있습니다. 각 호출은 파일 이름과 줄 번호를 ROM에 저장해야 하므로 너무 많은 호출을 추가하면 메모리가 부족할 수 있습니다.

버그를 찾은 후에는 추적 코드를 모두 삭제하는 것을 잊지 마십시오. 추적 코드를 포함한 풀 리퀘스트를 만들고 싶지는 않을 것입니다.