# 알면 좋은 크레이트들

Rust와 WebAssembly로 개발할 때 알면 정말 좋은 크레이트들을 모아놓은 목록입니다.

[creates.io 웹사이트에서 WebAssembly 카테고리로 등록된 크레이트들을 필터링해서 볼수도 있습니다.][wasm-category]

## JavaScript과 DOM 조작

### `wasm-bindgen` | [crates.io](https://crates.io/crates/wasm-bindgen) | [레포지토리](https://github.com/rustwasm/wasm-bindgen)

`wasm-bindgen`은 Rust와 JavaScript 사이의 고레벨 상호작용을 도와주는 크레이트입니다. JavaScript와 Rust를 넘나드는 임포트를 할수 있도록 도와줍니다.

### `wasm-bindgen-futures` | [crates.io](https://crates.io/crates/wasm-bindgen-futures) | [레포지토리](https://github.com/rustwasm/wasm-bindgen/tree/master/crates/futures)

`wasm-bindgen-futures`는 JavaScript의 `Promise`와 Rust의 `Future`을 연결해주는 다리 역할을 하는 크레이트입니다. Rust와 JavaScript 사이에서 양방향으로 변환이 가능하고 Rust에서 비동기(asynchronous) 작업을 수행할 때 유용합니다. DOM 이벤트 및 I/O 작업(operation)과 상호작용할 수 있도록 해줍니다.

### `js-sys` | [crates.io](https://crates.io/crates/js-sys) | [레포지토리](https://github.com/rustwasm/wasm-bindgen/tree/master/crates/js-sys)

`wasm-bindgen`으로 작성된 `js-sys` 크레이트는 `Object`, `Function`, `eval` 등의 JavaScript 전역 타입과 메소드를 전부 임포트합니다. 이러한 API들은 모든 웹 뿐만 아니라 Node.js를 포함한 모든 ECMAScript 환경에 이식해서 사용할 수 있습니다.

### `web-sys` | [crates.io](https://crates.io/crates/web-sys) | [레포지토리](https://github.com/rustwasm/wasm-bindgen/tree/master/crates/web-sys)

DOM 조작, `setTimeout`, Web GL, Web Audio 등과 같은 모든 웹 API들을 `wasm-bindgen`으로 작성한 크레이트입니다.

## 오류 보고 및 로깅

### `console_error_panic_hook` | [crates.io](https://crates.io/crates/console_error_panic_hook) | [repository](https://github.com/rustwasm/console_error_panic_hook)

`console.error`로 메세지를 패닉 메세지를 넘겨주는 패킹 훅(hook)과 함께, `wasm32-unknown-unknown` 타겟으로 패닉들을 디버깅 할수 있도록 도와주는 크레이트입니다.

### `console_log` | [crates.io](https://crates.io/crates/console_log) | [repository](https://github.com/iamcodemaker/console_log)

[`log` 크레이트](https://crates.io/crates/log)의 백엔드(backend)를 제공해주는 크레이트입니다. 로그된 메세지들을 개발자 도구(devtools) 콘솔로 넘겨줍니다.

## 동적 할당

### `wee_alloc` | [crates.io](https://crates.io/crates/wee_alloc) | [repository](https://github.com/rustwasm/wee_alloc)

이 크레이트의 이름은 **W**asm-**E**nabled, **E**lfin Allocator 에서 유래됐는데, (1000 bytes 이하 사이즈의 압축되지 않은 `.wasm` 바이너리로 구성된) 코드 사이즈가 할당 성능보다 더 중요할 때 사용할만한 할당자 구현 (allocator implementation) 코드입니다.

## `.wasm` 바이너리를 파싱하고(parsing) 생성하기

### `parity-wasm` | [crates.io](https://crates.io/crates/parity-wasm) | [repository](https://github.com/paritytech/parity-wasm)

직렬화(serializing)과 역직렬화(deserializing), 그리고 `.wasm` 바이너리를 빌드하는데 사용하는 저레벨 WebAssembly 포맷입니다. "names" 및 "reloc.WHATEVER"과 같이 잘 알려진 섹션들이 잘 지원돼 있습니다.

### `wasmparser` | [crates.io](https://crates.io/crates/wasmparser) | [repository](https://github.com/yurydelendik/wasmparser.rs)

WebAssembly 바이너리 파일을 읽는데 사용하는 간단한 이벤트 기반(event-driven) 라이브러리입니다. 각각 파싱한 내용의 바이트 오프셋 (byte offset)을 제공하는데, 예를 들어 reloc을 읽는 작업 등을 할때 필요합니다.

## WebAssembly 컴파일하고 인터프리팅(Interpreting)하기

### `wasmi` | [crates.io](https://crates.io/crates/wasmi) | [repository](https://github.com/paritytech/wasmi)

Parity 라는 회사에서 만든 임베딩 할수 있는 WebAssembly 인터프리터입니다.

### `cranelift-wasm` | [crates.io](https://crates.io/crates/cranelift-wasm) | [repository](https://github.com/bytecodealliance/wasmtime/tree/master/cranelift)

WebAssembly 코드를 네이티브 호스트의 기계어(machine code)로 컴파일해주는 크레이트. Cranelift (né Cretonne) 코드 생성기 프로젝트의 일부입니다.

[wasm-category]: https://crates.io/categories/wasm
