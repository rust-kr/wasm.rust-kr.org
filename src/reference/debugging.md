# Rust로 생성한 WebAssembly 디버깅하기 

이 섹션은 Rust로 생성한 WebAssembly를 디버깅하는데 유용한 정보를 포함합니다.

## debug 심볼을 포함해서 빌드하기

> ⚡ 디버깅할 때 항상 debug 심볼을 포함하고 빌드하는지 확인해 주세요!

debug 심볼이 활성화 되어있지 않다면 `"name"` 커스텀 섹션이 컴파일된 `.wasm` 파일에 반영되지 않을 수도 있습니다. 이런 경우에는 [스택 추적 (stack trace)](https://ko.wikipedia.org/wiki/스택_추적) 을 할 때 함수 이름이 Rust 함수 이름 대신 `wasm-function[42]` 처럼 읽기 어렵게 표시됩니다. 정상적으로 이 심볼이 활성화 된 경우에는 `wasm_game_of_life::Universe::live_neighbor_count` 와 같이 표시됩니다.

(`wasm-pack build --debug` 혹은 `cargo build` 명령어로) "debug" 빌드를 할 경우 이 심볼이 기본값으로 활성화됩니다.

"release" 빌드를 할 때는 debug 심볼이 기본값으로 활성화되지 않으니 참고해주세요. 그래도 활성화를 하고 싶다면 `Cargo.toml` 파일을 열고 `[profile.release]` 섹션에 `debug = true`를 포함시켜주세요.

```toml
[profile.release]
debug = true
```

## `console` API로 로깅 하기

로깅(logging) 작업은 코드를 설계하면서 만든 가설들을 확인하고 프로그램의 버그를 잡아내는 데 매우 효과적입니다. 보통은 웹 환경에서 [`console.log`
함수](https://developer.mozilla.org/en-US/docs/Web/API/Console/log)를 호출하여 브라우저의 개발자 콘솔에 메세지를 로그할 수 있습니다.

하지만 [`web-sys` 크레이트][web-sys]를 사용하여 `console` 로깅 함수를 해볼수도 있습니다.

```rust
extern crate web_sys;

web_sys::console::log_1(&"Hello, world!".into());
```

또 다른 옵션으로는 [`console.error` 함수](https://developer.mozilla.org/en-US/docs/Web/API/Console/error)가 있습니다. `console.error`는 `console.log`와 같은 타입 시그니처(signature)를 포함하고 있지만 개발자 툴에서 로그 메세지와 함께 스택 추적을 찾고 표시하는 데 사용할 수 있습니다.

### 참조

* `web-sys` 크레이트로 `console.log` 함수 사용하기:
  * [로그할 값들을 배열로 받는 `web_sys::console::log`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log.html)
  * [한 값만 로그 하는 `web_sys::console::log_1`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_1.html)
  * [두 값을 로그 하는 `web_sys::console::log_2`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_2.html)
  * 기타...

* `web-sys` 크레이트로 `console.error` 함수 사용하기:
  * [로그할 값들을 배열로 받는 `web_sys::console::error`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error.html)
  * [한 값만 로그하는 `web_sys::console::error_1`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_1.html)
  * [두 값을 로그하는 `web_sys::console::error_2`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_2.html)
  * 기타...

* [`console` 객체에 대해 설명해 주는 MDN 문서](https://developer.mozilla.org/en-US/docs/Web/API/Console)
* [Firefox 개발자 도구 — 웹 콘솔](https://developer.mozilla.org/en-US/docs/Tools/Web_Console)
* [Microsoft Edge 개발자 도구 — 콘솔](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/console)
* [Console Chrome DevTools 콘솔을 사용하여 시작해 보기](https://developers.google.com/web/tools/chrome-devtools/console/get-started)

## 패닉 로그 하기

[`console_error_panic_hook` 크레이트는 `console.error` 함수를 사용하여 예상치 못한 패닉들을 로그 합니다.][panic-hook] 이 크레이트를 통해 외계어같이 보이고 디버깅하기 어려운 `RuntimeError: unreachable executed` 에러 메세지 대신 Rust 환경에서 로그 했던 것과 같이 깔끔하게 포맷된 패닉 메세지를 표시할 수 있습니다.

정말 간단하게도 코드가 시작되는 함수에서 `console_error_panic_hook::set_once()`를 호출해서 훅(hook)을 설정하기만 하면 됩니다.

```rust
#[wasm_bindgen]
pub fn init_panic_hook() {
    console_error_panic_hook::set_once();
}
```

[panic-hook]: https://github.com/rustwasm/console_error_panic_hook

## 디버거 사용하기

정말 아쉽게도 WebAssembly 디버깅 경험은 아직 개선돼야할 부분이 많습니다. 대부분의 Unix 시스템에서는 [DWARF][dwarf]를 사용하여 소스 레벨에서 실행하는 프로그램을 살펴보게 되는데, 이 DWARF를 사용하여 디버거가 필요로 하는 정보를 인코딩(encoding)할 수 있습니다. 반면에 Windows에서는 대신 사용할 수 있는 포맷이 있음에도 현재로서는 WebAssembly를 직접적으로 지원하지는 않습니다. 그러므로 따로 작성했던 Rust 소스 텍스트 대신 컴파일러가 출력한 WebAssembly 명령어를 그대로 확인해야 합니다.

> [W3C WebAssembly group의 디버깅 하위 조항][debugging-subcharter]도 있으므로 미래에는 이런 문제가 개선될 것으로 예상됩니다!

[debugging-subcharter]: https://github.com/WebAssembly/debugging
[dwarf]: http://dwarfstd.org/

그래도 WebAssembly와 함께 작성된 JavaScript 코드를 살펴보고 wasm의 상태를 직접 살펴보는데 디버거가 매우 유용하므로 참고해 주세요.

### 참조

* [Firefox 개발자 도구 — 디버거](https://developer.mozilla.org/en-US/docs/Tools/Debugger)
* [Microsoft Edge 개발자 도구 — 디버거](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/debugger)
* [Chrome DevTools에서 JavaScript 디버깅 시작해보기](https://developers.google.com/web/tools/chrome-devtools/javascript/)

## 처음부터 WebAssembly 디버깅을 최소화할 수 있도록 신경 써서 코드를 작성해 주세요
고쳐야 할 버그가 JavaScript나 Web API 환경을 필요로 한다면 [`wasm-bindgen-test`으로 테스팅 코드를 작성해 보세요][wbg-test].

그렇지 *않다면* `#[test]` 속성을 포함하여 일반적인 Rust 코드처럼 실행 환경을 재현해 보세요. 이런 방식으로 운영체제의 성숙한 네이티브 툴링을 최대한 활용하여 디버깅할 수 있게 됩니다. [`quickcheck`][quickcheck]과 같은 테스팅 크레이트와 테스팅 코드 축소기도  보면서 기계적으로 테스트 코드의 사이즈를 줄여볼수도 있습니다. 최종적으로는, JavaScript 환경이 요구되지 않도록 더 작은 Rust 테스트 코드로 분리시키는 식으로 버그를 더 쉽게 찾고 고칠 수 있게 됩니다.

컴파일러와 링커(linker) 오류 없이 네이티브 `#[test]` 속성을 사용하려면 `Cargo.toml` 파일의 `[lib.crate-type]` 배열에 `rlib`가 포함돼 있는지 확인해 주세요.

```toml
[lib]
crate-type ["cdylib", "rlib"]
```

[quickcheck]: https://crates.io/crates/quickcheck
[web-sys]: https://rustwasm.github.io/wasm-bindgen/web-sys/index.html
[wbg-test]: https://rustwasm.github.io/wasm-bindgen/wasm-bindgen-test/index.html
