# Rust로 생성한 WebAssembly 디버깅하기 

이 섹션은 Rust로 생성한 WebAssembly를 디버깅하는데 유용한 정보를 담고 있습니다.

## debug 심볼을 포함해서 빌드하기

> ⚡ 디버깅 할때 항상 debug 심볼을 포함하고 빌드하는지 확인해주세요!

debug 심볼이 활성화 돼 있지 않다면 `"name"` 커스텀 섹션이 컴파일된 `.wasm` 파일에 반영되지 않을 수도 있습니다. 이런 경우에는 [스택 추적 (stack trace)](https://ko.wikipedia.org/wiki/스택_추적) 의 함수 이름이 Rust 함수 이름 대신 `wasm-function[42]` 와 같이 표시됩니다. 정상적으로 활성화 된 경우에는 `wasm_game_of_life::Universe::live_neighbor_count` 와 같이 표시됩니다.

(`wasm-pack build --debug` 혹은 `cargo build` 명령어로) "debug" 빌드를 할 경우 이 심볼이 기본값으로 활성화되게 됩니다.

"release" 빌드를 할 때는 debug 심볼이 기본값으로 활성화 되지 않으니 참고해주세요. 활성화 하고 싶다면 `Cargo.toml` 파일을 열고 `[profile.release]` 섹션에 `debug = true`가 포함돼 있는지 확인해주세요.

```toml
[profile.release]
debug = true
```

## `console` API로 로깅 하기

로깅(Logging) 작업은 코드를 설계하면서 만든 가설들을 증명하고 프로그램의 버그를 해결하는데 매우 효과적입니다. 웹 환경에서 [`console.log`
함수](https://developer.mozilla.org/en-US/docs/Web/API/Console/log)를 호출하여 브라우저의 개발자 콘솔에 메세지를 로그할수 있습니다.

[`web-sys` 크레이트][web-sys]를 사용하여 `console` 로깅 함수를 사용할수 있습니다.

```rust
extern crate web_sys;

web_sys::console::log_1(&"Hello, world!".into());
```

대신 [`console.error` 함수](https://developer.mozilla.org/en-US/docs/Web/API/Console/error)을 사용해볼수도 있습니다. `console.error`는 `console.log`와 같은 타입 시그니처(signature)를 가지고 있지만 개발자 툴에서 로그 메세지와 함께 스택 추적을 찾고 표시하는데 사용할 수 있습니다.

### 레퍼런스

* `web-sys` 크레이트로 `console.log` 함수 사용하기:
  * [로그할 값들을 배열로 받는 `web_sys::console::log`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log.html)
  * [한 값만 로그하는 `web_sys::console::log_1`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_1.html)
  * [두 값을 로그하는 `web_sys::console::log_2`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_2.html)
  * 기타...

* `web-sys` 크레이트로 `console.error` 함수 사용하기:
  * [로그할 값들을 배열로 받는 `web_sys::console::error`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error.html)
  * [한 값만 로그하는 `web_sys::console::error_1`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_1.html)
  * [두 값을 로그하는 `web_sys::console::error_2`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_2.html)
  * 기타...

* [`console` 객체에 대해 설명해주는 MDN 문서](https://developer.mozilla.org/en-US/docs/Web/API/Console)
* [Firefox 개발자 도구 — 웹 콘솔](https://developer.mozilla.org/en-US/docs/Tools/Web_Console)
* [Microsoft Edge 개발자 도구 — 콘솔](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/console)
* [Console Chrome DevTools 콘솔을 사용하여 시작해보기](https://developers.google.com/web/tools/chrome-devtools/console/get-started)

## 패닉 로그하기

[`console_error_panic_hook` 크레이트는 `console.error` 함수를 사용하여 예상하지 못한 패닉들을 로그합니다.][panic-hook] 외계어같고 디버그하기 어려운 `RuntimeError: unreachable executed` 에러 메세지 대신에 Rust의 깔끔하게 포맷된 패닉 메세지를 보여줍니다.

정말 간단하게도 코드가 시작되는 함수에서 `console_error_panic_hook::set_once()`를 불러서 훅(hook)을 설정하기만 하면 됩니다.

```rust
#[wasm_bindgen]
pub fn init_panic_hook() {
    console_error_panic_hook::set_once();
}
```

[panic-hook]: https://github.com/rustwasm/console_error_panic_hook

## 디버거 사용하기

정말 아쉽게도 WebAssembly 디버깅 경험은 아직 개선할 부분이 많습니다. 대부분의 Unix 시스템에서는 [DWARF][dwarf]를 사용해서 소스 레벨에서 실행하는 프로그램을 살펴보는데 디버거가 필요로 하는 정보를 인코딩(encoding)할수 있습니다. 하지만 Windows에서는 대신 사용할수 있는 포맷이 있긴 하지만 현재로서는 WebAssembly를 직접적으로 지원하지 않습니다. 그러므로 따로 작성했던 Rust 소스 텍스트 대신 컴파일러가 출력한 WebAssembly 명령어를 그대로 확인해야 합니다.

> [W3C WebAssembly group의 디버깅 하위 조항][debugging-subcharter]도 있으므로 미래에는 이런 문제가 개선될 것으로 예상됩니다!

[debugging-subcharter]: https://github.com/WebAssembly/debugging
[dwarf]: http://dwarfstd.org/

그래도 WebAssembly를 사용하는 JavaScript 코드를 살펴보고 wasm의 상태를 직접 살펴보는데 디버거가 매우 유용하므로 참고해주세요.

### 레퍼런스

* [Firefox 개발자 도구 — 디버거](https://developer.mozilla.org/en-US/docs/Tools/Debugger)
* [Microsoft Edge 개발자 도구 — 디버거](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/debugger)
* [Chrome DevTools에서 JavaScript 디버깅 시작해보기](https://developers.google.com/web/tools/chrome-devtools/javascript/)

## 처음부터 WebAssembly 디버깅을 최소화할수 있도록 해주세요
고쳐야 할 버그가 JavaScript나 Web API 환경을 필요로 한다면 [`wasm-bindgen-test`으로 테스팅 코드를 작성해보세요][wbg-test].

그렇지 *않다면* `#[test]` 속성을 포함해서 일반적인 Rust 코드처럼 재현해보세요. 이렇게 하면 운영체제의 성숙한 네이티브 툴링을 최대한 활용하여 디버깅할 수 있습니다. [`quickcheck`][quickcheck]과 같은 테스팅 크레이트와 테스팅 코드 축소기도 사용해보면서 기계적으로 테스트 코드의 사이즈를 줄여보세요. 최종적으로는, JavaScript 환경이 요구되지 않도록 더 작은 Rust 테스트 코드로 분리시키면 버그를 찾아서 잡기가 쉬워집니다.

컴파일러와 링커 오류(linker error)없이 네이티브 `#[test]` 속성을 사용하려면, `Cargo.toml` 파일의 `[lib.crate-type]` 배열에 `rlib`가 포함돼 있는지 확인해주세요.

```toml
[lib]
crate-type ["cdylib", "rlib"]
```

[quickcheck]: https://crates.io/crates/quickcheck
[web-sys]: https://rustwasm.github.io/wasm-bindgen/web-sys/index.html
[wbg-test]: https://rustwasm.github.io/wasm-bindgen/wasm-bindgen-test/index.html
