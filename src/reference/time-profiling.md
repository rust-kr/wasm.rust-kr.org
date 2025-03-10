# 타임 프로파일링

이 섹션은 러스트와 웹어셈블리를 사용하여 성능을 덜 쓰면서도 지연 시간도 줄일 수 있도록 페이지를 프로파일링하는 방법에 대해 다룹니다.

> ⚡ 프로파일링을 할 때에는 항상 최적화된 빌드를 사용해주세요! `wasm-pack build` 명령어를 사용하고 있다면 이러한 최적화 옵션이 기본적으로 활성화돼있으니 참고해  주세요.

## 사용 가능한 툴들

### `window.performance.now()` 타이머

[`performance.now()` 함수][perf-now]는 웹 페이지가 로드된 시점부터 측정된 밀리초 단위의 단조 시간 (monotonic timestamp)를 반환합니다.

`performance.now`를 호출할 때에는 거의 오버헤드가 발생하지 않습니다. 그러므로 시스템 성능에 큰 영향을 주거나 측정 값에 편향을 주지 않으면서 간단하고 상세한 측정을 원활하게 할 수 있게 됩니다.

이 함수를 사용하여 프로그램의 여러 작업들이 실행되는 시간을 측정해 볼 수 있고, [`web-sys` 크레이트][web-sys]를 통해 `window.performance.now()`에 접근할 수도 있습니다:

```rust
extern crate web_sys;

fn now() -> f64 {
    web_sys::window()
        .expect("should have a Window")
        .performance()
        .expect("should have a Performance")
        .now()
}
```

* [`web_sys::window` 함수](https://rustwasm.github.io/wasm-bindgen/api/web_sys/fn.window.html)
* [`web_sys::Window::performance` 메소드](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.Window.html#method.performance)
* [`web_sys::Performance::now` 메소드](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.Performance.html#method.now)

[perf-now]: https://developer.mozilla.org/en-US/docs/Web/API/Performance/now

### 개발자 도구 내의 프로파일러

모든 웹 브라우저의 개발자 도구는 프로파일러를 포함하고 있습니다. 이러한 프로파일러들은 "call tree"나 "flame graph"와 같은 일반적인 시각 자료와 함께 어떤 함수들이 가장 많은 시간을 소비하는지 표시해 줍니다.

"name" 커스텀 섹션이 웹어셈블리 바이너리에 포함되도록 [debug 심볼을 포함해서 빌드][symbols]하면 프로파일러가 `wasm-function[123]`와 같이 복잡하고 불투명한 이름 대신 Rust 함수 이름을 대신 표시하게 됩니다.

이러한 프로파일러들은 인라인 함수를 **표시하지 않는다는** 점을 기억해 주세요. 러스트와 LLVM는 인라인 작업을 매우 무겁게 수행하기 때문에 이러한 작업을 해주더라도 결괏값이 조금은 읽기 어려울수 있습니다.

[symbols]: ./debugging.html#building-with-debug-symbols

[![러스트 심볼을 표시하는 프로파일러 스크린샷](../images/game-of-life/profiler-with-rust-names.png)](../images/game-of-life/profiler-with-rust-names.png)

#### 추가 자료

* [Firefox 개발자 도구 — 성능](https://developer.mozilla.org/en-US/docs/Tools/Performance)
* [Microsoft Edge 개발자 도구 — 성능](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/performance)
* [Chrome DevTools 자바스크립트 프로파일러](https://developers.google.com/web/tools/chrome-devtools/rendering-tools/js-execution)

### `console.time`와 `console.timeEnd` 함수

[`console.time` 과 `console.timeEnd` 함수][console-time]를 사용해서 브라우저 개발자 도구 콘솔에 명명된 작업의 타이밍을 기록할 수 있습니다. 작업이 시작될 때 `console.time("어떤 거 하는 작업")`를 호출하고, 작업이 완료될 때는 `console.timeEnd("어떤 거 하는 작업")`를 호출합니다. 참고로 작업의 이름을 지어주는 문자열 인자는 필수가 아닙니다.

[`web-sys` 크레이트][web-sys]를 통해 이러한 함수들을 직접적으로 사용할 수 있습니다:

* [`web_sys::console::time_with_label("어떤 거 하는 작업")`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.time_with_label.html)
* [`web_sys::console::time_end_with_label("어떤 거 하는 작업")`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.time_end_with_label.html)

브라우저의 콘솔에 `console.time` 로그를 표시하는 스크린샷을 확인해보세요:

[![console.time 로그 스크린샷](../images/game-of-life/console-time.png)](../images/game-of-life/console-time.png)

추가로, 다음 이미지에 보이는 것과 같이 브라우저 프로파일러의 "timeline"과 "waterfall" 뷰에 `console.time`과 `console.timeEnd`의 로그가 표시됩니다.

[![console.time 로그 스크린샷](../images/game-of-life/console-time-in-profiler.png)](../images/game-of-life/console-time-in-profiler.png)

[console-time]: https://developer.mozilla.org/en-US/docs/Web/API/Console/time

### `#[bench]` 속성으로 네이티브 환경을 사용할 때

웹 브라우저에서 `#[test]` 속성을 대신 이용하여 운영체제의 네이티브 코드로 디버깅을 했던 것과 같이, `#[bench]` 속성과 함께 함수를 작성해서 운영 체제의 네이티브 코드 프로파일링 툴을 사용해 볼 수도 있습니다.

작업 중인 크레이트의 하위 경로인 `benches`에 벤치마킹 코드를 작성해 보겠습니다. 먼저 `crate-type`이 `"rlib"`이 포함하고 있는지 확인해 주세요. 그렇지 않다면 벤치마크 바이너리가 메인 라이브러리 (main lib) 에 링크되지 못하게 됩니다.

하지만 실제로는 거의 사용하지 않는 코드에 시간을 소비하고 있을 수도 있으니, 네이티브 코드로 벤치마킹을 하기 전에 브라우저 프로파일링 툴을 먼저 확인해 보는 것도 좋습니다!

#### 추가 자료

* [Linux 환경에서 `perf` 프로파일러 사용하기](http://www.brendangregg.com/perf.html)
* [macOS 환경에서 Instruments.app 프로파일러 사용하기](https://help.apple.com/instruments/mac/current/)
* [VTune 프로파일러는 Windows와 Linux를 지원합니다.](https://software.intel.com/en-us/vtune)

[web-sys]: https://rustwasm.github.io/wasm-bindgen/web-sys/index.html
