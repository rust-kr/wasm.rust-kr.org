# 알면 좋은 툴들

Rust와 WebAssembly로 개발할 때 알면 좋은 툴들을 모아놓은 목록입니다.

## 개발, 빌드 및 워크플로우 관리 (Orchestration)

### `wasm-pack` | [레포지토리](https://github.com/rustwasm/wasm-pack)

`wasm-pack`은 Web과 Node.js 환경에서 JavaScript와 상호 운용하면서 Rust로 WebAssembly 작업을 할때 사용할수 있는 만능 툴입니다. `wasm-pack`을 사용해서 더 쉽게 WebAssembly를 빌드하고 npm 레지스트리에 배포하고, 기존에 이미 사용하는 워크플로우가 사용하는 패키지들과 같이 사용할 수 있도록 도와줍니다.

## `.wasm` 바이너리 최적화와 조작하기

### `wasm-opt` | [레포지토리](https://github.com/WebAssembly/binaryen)

`wasm-opt`는 WebAssembly을 입력으로 받고 변형하고 최적화한 다음, 원한다면 코드를 측정/분석까지 한 다음 WebAssembly 파일을 출력합니다. `rustc`가 작동하는 방식처럼, LLVM으로 생성한 `.wasm` 바이너리를 입력으로 넣고 `wasm-opt`를 실행하면 더 작고 더 빠르게 실행할수 있는 `.wasm` 바이너리 파일을 생성할 수 있습니다.

### `wasm2js` | [레포지토리](https://github.com/WebAssembly/binaryen)

`wasm2js` 툴은 WebAssembly 파일을 "거의 asm.js"처럼 컴파일 해줍니다. Internet Explorer 11처럼 WebAssembly 기능이 구현되지 않은 브라우저를 지원할 때 좋습니다. 이 툴은 `binaryen` 프로젝트의 일부입니다.

### `wasm-gc` | [레포지토리](https://github.com/alexcrichton/wasm-gc)

WebAssembly를 가비지 콜렉팅하고 필요하지 않은 익스포트, 임포트, 함수 등을 지울 때 사용할수 있는 심플한 툴입니다. WebAssembly 파일과 함께 `--gc-sections` 링커 플래그 (linker flag) 를 사용하면 더 효과적입니다.

보통은 다음과 같은 이유로 이 툴을 개발자가 "직접" 프로젝트에 포함시키지 않습니다:

1. `rustc` 컴파일러가 이제 새 버전의 `lid`를 지원하는데, 이 버전이 `--gc-sections` 라는 플래그를 지원합니다. 이 플래그는 LTO 빌드를 하는데 자동으로 활성화되게 됩니다.
2. `wasm-bindgen` CLI (커맨드 라인 인터페이스 / Command Line Interface) 툴이 자동으로 `wasm-gc` 를 실행시켜줍니다.

### `wasm-snip` | [레포지토리](https://github.com/rustwasm/wasm-snip)

`wasm-snip`는 WebAssembly 함수의 내용을 `unreachable` 명령어(instruction)으로 바꿔줍니다.

어떤 함수가 실제로 호출되지 않는걸 아는데 컴파일러가 이걸 모를때가 있나요? 일단 컴파일하고 `wasm-gc`를 실행해보세요! (런타임(runtime)에서 호출되지 않는) 다른 간접적으로 호출되는 다른 함수들도 모두 제거됩니다.

디버깅 코드가 없는 실제로 배포하는 빌드 (production build) 에서 강제로 Rust 의 패닉 인프라(infrastructure)를 제외할 때 유용합니다.

## Inspecting `.wasm` Binaries

### `twiggy` | [레포지토리](https://github.com/rustwasm/twiggy)

`twiggy`는 `.wasm` 바이너리에 사용하는 코드 프로파일러입니다. 바이너리의 호출 그래프 (call graph) 를 분석하고 다음과 같은 내용을 알려줍니다:

* 애초에 빌드할 때 어떤 함수가 왜 바이너리에 포함됐나요? 예: 어떤 익스포트한 함수들이 간접적으로 호출되고 있나요?

* 어떤 함수를 지우면 얼마나 공간을 아낄수 있나요? 예: 이 함수와 다른 사용되던 함수들까지 지우면 얼마나 공간이 절약되나요?

`twiggy`를 사용해서 바이너리를 더 가볍게 만들어보세요!

### `wasm-objdump` | [레포지토리](https://github.com/WebAssembly/wabt)

`wasm` 바이너리의 저레벨 상세 정보와 섹션들을 출력합니다. WAT 텍스트 포맷으로 디어셈블링(disassembling)할수도 있습니다. WebAssembly에 사용하는 `objdump`으로 생각해도 좋습니다. 이 툴은 WABT 프로젝트의 일부입니다.

### `wasm-nm` | [레포지토리](https://github.com/fitzgen/wasm-nm)

`.wasm` 바이너리 내의 임포트되고, 익스포트되고, 그리고 private인 함수 심볼들 (function symbols) 을 나열합니다. WebAssembly에 사용하는 `nm`으로 생각해도 좋습니다.