# 어떤 크레이트들을 WebAssembly에서 바로 사용할수 있나요?

가장 간단하게 WebAssembly 환경에서 작동하지 않는 것들부터 목록으로 만들어 보겠습니다. 다음 내용에 해당하지 않는 크레이트들은 휴대성이 좋고 WebAssembly 환경에서 잘 작동한다고 생각해도 좋고, 임베디드 시스템과 `#![no_std]`를 지원한다면 보통은 WebAssembly도 지원한다고 볼수 있습니다.

## 이러한 크레이트들은 WebAssembly 환경에서 작동하지 않을수도 있습니다

### C 언어와 시스템 라이브러리 종속성이 있는 경우

wasm에는 시스템 라이브러리가 없기 떄문에 시스템 라이브러리에 바인딩된 코드가 있는 크레이트는 작동하지 않습니다.

wasm에는 언어 간 통신에 사용할수 있는 안정된 버전의 ABI와 언어 간 링킹(linking)도 없기 때문에, C 라이브러리를 사용하는 크레이트도 작동하지 않습니다. 특히 `clang` 컴파일러가 `wasm32` 타겟을 기본으로 제공하기 때문에 결국은 작동할 것으로 예상되지만, 현재로서는 완벽하지 않습니다.

### 파일 I/O

WebAssembly는 파일 시스템에 접근할 수 없습니다. &mdash; 파일 시스템을 필요로 하고 wasm에서 사용할수 있도록 코드가 마련되지 않은 크레이트는 작동하지 않습니다.

### 스레드 생성

[스레딩을 웹어셈블리에 지원하는 계획][wasm-threading]이 있긴 하지만 아직 준비가 되지 않았습니다. `wasm32-unknown-unknown` 타겟에서 스레드를 생성하려고 시도하면 wasm 트랩 (wasm trap) 이 발생하면서 코드가 패닉하게 됩니다.

[wasm-threading]: https://rustwasm.github.io/2018/10/24/multithreading-rust-and-wasm.html

## 어떤 다목적 크레이트들이 WebAssembly에서 바로 작동하는 편인가요?

### 알고리즘과 자료 구조

[A* 알고리즘](https://ko.wikipedia.org/wiki/A*_알고리즘)나 [splay trees](https://en.wikipedia.org/wiki/Splay_tree)처럼, 특정한 [알고리즘](https://crates.io/categories/algorithms)이나 [data structure](https://crates.io/categories/data-structures)의 구현을 제공하는 크레이트들은 WebAssembly와 잘 작동하는 편입니다.

### `#![no_std]`

[Rust 스탠다드 라이브러리를 사용하지 않는 크레이트들](https://crates.io/categories/no-std)도 WebAssembly와 잘 작동하는 편입니다.

### 파서 (Parser)

입력을 받고 I/O 작업을 수행하지 않는 이상, [파서들](https://crates.io/categories/parser-implementations)은 WebAssembly와 잘 작동하는 편입니다.

### 텍스트 처리

[인간 언어를 텍스트 형태로 나타냈을 때 발생하는 복잡성을 다루는 크레이트들](https://crates.io/categories/text-processing)은 WebAssembly와 잘 작동하는 편입니다.

### Rust 패턴

[Rust 프로그래밍의 특정한 상황에 쓰도록 공유되는 해결책들](https://crates.io/categories/rust-patterns)은 WebAssembly와 잘 작동하는 편입니다.