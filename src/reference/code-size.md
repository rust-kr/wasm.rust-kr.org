# `.wasm` 파일 사이즈 줄이기

이 섹션에서는 어떻게 Rust 코드를 수정하고 `.wasm` 빌드를 최적화해야 더 작은 사이즈의 바이너리를 출력할수 있는지 살펴보겠습니다.

## 왜 출력되는 파일의 사이즈가 중요한가요?

`.wasm` 파일을 네트워크로 전송할때 파일의 사이즈가 작을수록 클라이언트에서 더 빠르게 다운로드 할수 있습니다. `.wasm` 파일이 빨리 다운로드 될수록 페이지가 더 빨리 로드되고 유저 경험이 더 나아지게 됩니다.

하지만 코드 사이즈가 어쩌면 제일 중요하게 확인해야 할 부분이 아닐수도 있습니다. 모호하고 측정하기 어렵겠지마 "페이지가 로드되고 사용할 수 있게 될 때까지 걸리는 시간"이 실제로는 더 중요하게 고려해봐야할 부분일수도 있습니다. (코드가 로드돼야 사이트가 작동하기 시작한다는 내용을 생각해 본다면) 코드 사이즈가 이 시간에 큰 영향을 미치긴 하지만 이게 유일하게 확인해야 할 부분은 아닌 것을 알수 있습니다.

WebAssembly는 보통 gzip 파일 포맷 형식으로 압축되어 전송되는데, 그러므로 유선을 통해 파일을 더 빠르게 보낼수 있도록 gzip 포맷으로 압축된 파일의 사이즈를 비교해야 합니다. 참고로 WebAssembly 바이너리 포맷은 gzip 포맷에 적합하므로 50% 이상으로 사이즈를 줄일 수 있습니다.

게다가, WebAssembly의 바이너리 포맷은 매우 빠르게 읽고 처리할수 있도록 최적화가 잘 되어있습니다. 요즘 사용되는 브라우저들은 보통 "baseline compilers" 라는 기능을 가지고 있는데, 이 기능을 통해 네트워크로 wasm 파일을 보내는 동시에 동일하게 빠른 속도로 전송받은 WebAssembly 코드를 네이티브 기계어로 컴파일할 수 있습니다. 그렇기 때문에 [`instantiateStreaming`을 사용한다면][hacks] 웹페이지가 한 번 로드 된 이후부터는 WebAssembly 모듈을 바로 사용할 수 있게 됩니다. 반면에 JavaScript 코드를 실행할 때는 주로 파싱하는 처리 외에도 코드를 빠르게 실행할수 있도록 JIT 컴파일 과정 등을 거쳐야 하기 때문에 주로 시간이 더 오래 걸리게 됩니다.

마지막으로, WebAssembly가 실행 속도 측면에서 JavaScript와 비교했을때 훨씬 최적화가 잘 돼있다는 부분을 기억해주세요. 더 확실하게 하고 싶다면 JavaScript와 WebAssembly 런타임 속도를 각각 측정해보고 코드 사이즈가 얼마나 중요한지 확인해볼수도 있습니다.

하지만 `.wasm` 파일의 사이즈가 예상보다 크더라도 바로 낙담하지는 말아주세요! 코드 사이즈는 큰 그림의 일부일 뿐입니다. JavaScript와 WebAssembly를 비교할 때 코드 사이즈만 비교하게 된다면 많은 부분을 놓치게 됩니다.

[hacks]: https://hacks.mozilla.org/2018/01/making-webassembly-even-faster-firefoxs-new-streaming-and-tiering-compiler/

## 코드 사이즈를 줄일 수 있도록 빌드 최적화하기

`rustc`에는 더 작은 `.wasm` 바이너리를 생성할 때 유용한 옵션을 몇가지 가지고 있습니다. 어떤 상황에서는, 컴파일 시간이 더 오래 걸리는 부분을 희생해서 `.wasm` 사이즈를 더 작게 줄이기도 합니다. 또 다른 경우에는, 더 빠른 런타임을 위해 `.wasm` 파일 사이즈를 포기하기도 합니다. 각 옵션의 장단점을 잘 이해하고, 프로파일링과 측정 작업을 해보면서 코드 사이즈와 런타임 속도 중 어떤 것이 우선시 되어야 하는지 잘 알고 결정하는 것이 중요합니다.

### 링크 시간 최적화 (Link Time Optimizations, LTO) 을 사용해서 컴파일하기

`Cargo.toml` 파일의 `[profile.release]` 섹션에 `lto = true`를 추가해주세요:

```toml
[profile.release]
lto = true
```

이렇게 LLVML이 사용하지 않는 코드를 더 공격적으로 제거하고 인라인 작업을 더 적극적으로 처리할 수 있도록 설정할수 있습니다. LTO를 사용하면 `.wasm` 파일의 사이즈가 작아질 뿐 아니라, 런타임을 더 빠르게 만들수도 있습니다! 하지만 컴파일 작업이 더 오래 걸린다는 단점이 있으니 참고해주세요.

### 런타임 속도 대신 코드 사이즈를 최적화하도록 LLVM 설정하기

LLVM의 최적화 작업은 기본적으로 사이즈 대신 속도 개선에 중점을 두고 진행됩니다. `[profile.release]`파일의 `[profile.release]` 섹션을 수정하여 이 설정을 변경할 수도 있습니다:

```toml
[profile.release]
opt-level = 's'
```

추가로 발생할 수 있는 속도 저하를 감수해서라도 더 공격적으로 최적화를 해볼수도 있습니다:

```toml
[profile.release]
opt-level = 'z'
```

정말 놀랍게도, 출력되는 파일의 사이즈가 `opt-level = "z"`대신 `opt-level = "s"`를 사용했을 때 더 작아지는 경우도 있습니다. 항상 잘 확인해보는걸 잊지 말아주세요!

### `wasm-opt` 툴 사용하기

[Binaryen][] 툴킷은 WebAssembly에 특화된 컴파일러 툴링을 포함합니다. 단순히 LLVM의 WebAssembly 백엔드 작업 외에도 훨씬 더 다양한 작업에 사용할 수 있고, `wasm-opt` 툴을 사용해서 LLVM이 빌드한 `.wasm` 바이너리를 최적화 하면 보통은 코드 사이즈를 15-20% 정도 더 줄일수 있게 됩니다. 런타임 속도도 같이 개선될수도 있으니 참고해주세요!

```bash
# 사이즈 최적화.
wasm-opt -Os -o output.wasm input.wasm

# 공격적인 사이즈 최적화.
wasm-opt -Oz -o output.wasm input.wasm

# 속도 최적화.
wasm-opt -O -o output.wasm input.wasm

# 공격적인 속도 최적화.
wasm-opt -O3 -o output.wasm input.wasm
```

[Binaryen]: https://github.com/WebAssembly/binaryen

### 디버그 정보에 관해 알아두면 좋은 점

wasm 바이너리 사이즈를 줄이는 데 바이너리 파일에 포함된 디버그 정보와 `names` 섹션이 정말 큰 역할을 합니다. 하지만 `wasm-pack`이 기본값으로 디버그 정보를 삭제하고, 추가로 `wasm-opt`도 명령어에 `-g`가 포함되지 않는 이상 `names` 섹션을 기본적으로 지우는 부분을 잘 기억해주세요.

이 책을 잘 따라왔다면 기본적으로는 디버그 정보나 `names` 섹션 없이 wasm파일을 빌드하게 됩니다. 하지만, 이러한 디버깅 정보가 wasm 바이너리에 포함돼야 하는 상황에서는 이 내용을 잘 참고해주세요!

## 사이즈 프로파일링하기

빌드 최적화 설정을 바꿨는데도 `.wasm` 코드 사이즈가 충분히 줄어들지 않는다면, 어떤 부분이 나머지 공간을 사용하는지 프로파일링 작업을 해보면서 알아보도록 합시다.

> ⚡ 타임 프로파일링 가이드를 따라왔던 것처럼, 사이즈 프로파일링 가이드도 한번 읽어보고 시도해봅시다. 읽어보는 것만으로도 시간을 정말 많이 아낄수 있습니다!

### `twiggy` 코드 사이즈 프로파일러

[`twiggy`][twiggy]는 WebAssembly를 입력으로 받는 코드 사이즈 프로파일러입니다. 바이너리의 [호출 그래프 (call graph)](https://en.wikipedia.org/wiki/Call_graph) 를 분석하고 다음곽 같은 내용을 알려줍니다:

* 어떤 함수들이 애초에 왜 바이너리에 포함되는건가요?

* 이 함수에 *사용되는 공간의 크기*가 어떻게 되나요? 예: 이 함수와 내부에서 호출해서 사용하게 되는 함수들까지 제거한다면 얼마나 공간을 아낄수 있나요?

<style>
/* 알수 없는 이유로 mdbook의 기본 폰트가 box-drawing character와 호환이 되지 않아서 직접 스타일을 따로 생성했습니다. */
pre, code {
  font-family: "SFMono-Regular",Consolas,"Liberation Mono",Menlo,Courier,monospace;
}
</style>

```text
$ twiggy top -n 20 pkg/wasm_game_of_life_bg.wasm
 Shallow Bytes │ Shallow % │ Item
───────────────┼───────────┼────────────────────────────────────────────────────────────────────────────────────────
          9158 ┊    19.65% ┊ "function names" subsection
          3251 ┊     6.98% ┊ dlmalloc::dlmalloc::Dlmalloc::malloc::h632d10c184fef6e8
          2510 ┊     5.39% ┊ <str as core::fmt::Debug>::fmt::he0d87479d1c208ea
          1737 ┊     3.73% ┊ data[0]
          1574 ┊     3.38% ┊ data[3]
          1524 ┊     3.27% ┊ core::fmt::Formatter::pad::h6825605b326ea2c5
          1413 ┊     3.03% ┊ std::panicking::rust_panic_with_hook::h1d3660f2e339513d
          1200 ┊     2.57% ┊ core::fmt::Formatter::pad_integral::h06996c5859a57ced
          1131 ┊     2.43% ┊ core::str::slice_error_fail::h6da90c14857ae01b
          1051 ┊     2.26% ┊ core::fmt::write::h03ff8c7a2f3a9605
           931 ┊     2.00% ┊ data[4]
           864 ┊     1.85% ┊ dlmalloc::dlmalloc::Dlmalloc::free::h27b781e3b06bdb05
           841 ┊     1.80% ┊ <char as core::fmt::Debug>::fmt::h07742d9f4a8c56f2
           813 ┊     1.74% ┊ __rust_realloc
           708 ┊     1.52% ┊ core::slice::memchr::memchr::h6243a1b2885fdb85
           678 ┊     1.45% ┊ <core::fmt::builders::PadAdapter<'a> as core::fmt::Write>::write_str::h96b72fb7457d3062
           631 ┊     1.35% ┊ universe_tick
           631 ┊     1.35% ┊ dlmalloc::dlmalloc::Dlmalloc::dispose_chunk::hae6c5c8634e575b8
           514 ┊     1.10% ┊ std::panicking::default_hook::{{closure}}::hfae0c204085471d5
           503 ┊     1.08% ┊ <&'a T as core::fmt::Debug>::fmt::hba207e4f7abaece6
```

[twiggy]: https://github.com/rustwasm/twiggy

### LLVM-IR 직접 살펴보기

LLMV-IR은 LLVM이 WebAssembly를 출력하기 전에 거치게 되는 최종 중간 표현 (final intermediate representation) 입니다. 이 최종 중간 표현은 최종적으로 출력하는 WebAssembly 바이너리와 매우 유사하게 생겼습니다. LLVM-IR의 크기가 클수록 출력되는 `.wasm` 파일의 크기도 커지게 되고, 함수들이 LLVM-IR 크기의 25%까지 차지하게 된다면 `.wasm` 파일에서도 함수들이 마찬가지로 25%를 차지하게 됩니다. 이러한 수치들이 보통은 일치하지만, LLVM-IR은 `.wasm` 파일이 (DWARF와 같은 디버깅 정보 처럼) 가지고 있지 않는 다른 중요한 정보들도 가지고 있는 점 또한 잘 참고해주세요. 이러한 하위 루틴들은 해당 함수의 위치에 인라인됩니다.

`cargo` 명령어를 실행해서 LLVM-IR 파일을 직접 생성해보세요:

```
cargo rustc --release -- --emit llvm-ir
```

그 다음 `find` 명령어를 사용해서 `.ll` 파일을 검색해보겠습니다. 이 LLVM-IR 파일은 `cargo`의 `target` 경로에 위치하게 됩니다:

```
find target/release -type f -name '*.ll'
```

#### 레퍼런스

* [LLVM 언어 레퍼런스 메뉴얼](https://llvm.org/docs/LangRef.html)

## 툴과 테크닉을 활용하여 더 깊게 파고들어서 최적화하기

`.wasm` 바이너리 사이즈를 줄이는 설정은 보통 자동화가 돼 있습니다. 하지만 추가로 불필요한 코드를 제거하고 최적화를 해줘야 하는 경우에는 더 깊게 들어가서 수정해줘야 할 때가 있습니다. 이 섹션에서는 코드 사이즈를 줄일때 사용해볼수 있는 투박한 방법들에 대해 알아보겠습니다.

### 문자열 포맷 피하기

`format!`이나 `to_string` 과 같은 함수/매크로들을 사용하면 출력되는 바이너리의 사이즈가 불필요하게 커질수도 있습니다. 가능하면 문자열 포맷은 디버그 모드에서만 사용하고, 배포 버전을 빌드할 때에는 정적 문자열 (static string) 을 사용해보세요.

### 코드 패닉 피하기

말처럼 쉽지는 않겠지만, `twiggy` 와 같은 툴을 사용하거나 LLVM-IR 파일을 살펴보면서 어떤 함수들이 패닉하는지 살펴볼 수 있습니다.

패닉이 항상 `panic!()` 매크로의 형식으로 나타나지는 않고 보통은 다음과 같은 여러가지 다양한 이유로 발생할 수 있습니다:

* 슬라이스 사이즈 범위를 벗어난 인덱스의 요소에 접근하고자 시도할 때 (out of bounds) : `my_slice[i]`

* 나머지 연산자를 사용할때 나누는 수가 0인 경우: `나눠지는 수 / 나누는 수`

* `Option`이나 `Result`의 값을 `unwrap()`를 사용하여 값에 접근할 때: `opt.unwrap()` 또는 `res.unwrap()`

처음 두 방법 대신 사용해볼수 있는 더 안전한 방법들도 있습니다. `my_slice[i]` 처럼 인덱스를 통해 직접 접근하지 않고 `my_slice.get(i)` 를 사용하여 `Option` 타입의 값에 접근해볼수도 있고, `check_div` 함수를 불러서 값을 나눌수 있는지 확인해볼수도 있습니다. 이렇게 3번쨰 경우만 집중적으로 신경쓸수 있게 됐습니다.

추가로, 코드를 패닉시키지 않으면서 `Option`이나 `Result` 타입의 값을 "안전한 방법과 불안전한 방법", 두 가지 방법으로 처리해볼수도 있습니다.

안전한 방법부터 한번 살펴보도록 합시다. `None`이나 `Error` 값을 반환받았을때 코드를 패닉시키는 대신 `abort` 함수를 사용해보겠습니다:

```rust
#[inline]
pub fn unwrap_abort<T>(o: Option<T>) -> T {
    use std::process;
    match o {
        Some(t) => t,
        None => process::abort(),
    }
}
```

최종적으로는 패닉 코드가 `wasm32-unknown-unknown` 타겟의 abort 명령어로 옮겨지기 때문에, 이런 식으로 코드를 작성하면서 불필요한 코드를 지울수 있게 됩니다.

다른 시도해볼수 있는 방법으로는, [`unreachable` 크레이트][unreachable]가 있습니다. 이 크레이트는 `Option`과 `Result` 타입의 값들과 함께 사용할수 있도록 불안전한 [`unchecked_unwrap` 확장 크레이트들][unchecked_unwrap]을 제공하는데, 컴파일러가 `Option`을 `Some`으로, `Result`를 `Ok`로 *추측*해서 옮길수 있도록 도와줍니다. 하지만 이런 추측이 맞아떨어지지 않으면 정의하지 않은 동작 (undefined behavior) 이 발생하게 되므로 코드가 잘 작동한다고 확신하지만 컴파일러가 잘 모르고 있는 상황을 *잘 이해하고 있을 때만* 이 방법을 사용해주세요. 일반적으로는 이 방법을 배포 버전에서만 적용하고 그 외에는 컴파일러가 확인할수 있도록 디버그 빌드를 따로 설정하길 권장합니다.

[unreachable]: https://crates.io/crates/unreachable
[unchecked_unwrap]: https://docs.rs/unreachable/1.0.0/unreachable/trait.UncheckedOptionExt.html#tymethod.unchecked_unwrap

### 할당을 피하거나 `wee_alloc`을 대신 사용해보세요.

Rust는 기본값으로 `dlmalloc`라는 할당자를 이식해서 사용합니다. 이 할당자는 10 KB 정도의 사이즈를 차지하게 되는데, 동적 할당을 사용하지 않아도 괜찮다면 이 사이즈를 절약해볼수도 있습니다.

완전히 동적 할당 없이 작업하기가 사실은 쉽지는 않은 편인데, 코드의 [핫 스팟 (hot spot)](https://en.wikipedia.org/wiki/Hot_spot_(computer_programming)) 에서 할당을 없애는 작업은 대부분은 훨씬 쉬운 편입니다. (보통은 이 작업을 통해 핫 스팟인 코드들을 훨씬 빠르게 만들수도 있습니다.) 이러한 상황에서 [전역 할당자 대신 `wee_alloc`를 사용하면][wee_alloc] (전부는 아니지만) 이 10 KB 만큼 차지되는 공간의 대부분을 절약할 수 있습니다. `wee_alloc`는 할당자를 필요로 하지만 굳이 아주 빠를 필요가 없고, 실행 속도가 느려지는 대신 코드 사이즈를 줄여도 괜찮을때 사용하도록 설계됐습니다.

[wee_alloc]: https://github.com/rustwasm/wee_alloc

### 제네릭 타입 매개변수 대신 트레이트 객체를 사용해보세요

다음 예시와 같이 타입 매개변수를 사용하는 제네릭 함수를 작성한다고 가정해봅시다:

```rust
fn whatever<T: MyTrait>(t: T) { ... }
```

`rustc`와 LLVM는 제네릭 함수가 호출될 때 사용된 타입에 해당하는 바이너리 코드를 각각 따로 생성합니다. 이런 접근은 어떤 `T` 타입을 컴파일러가 처리하는지에 따라 컴파일러 최적화에 유용할 수도 있습니다. 하지만 동시에 코드 사이즈가 빠르게 늘어날 수도 있다는 단점도 가지고 있습니다.

다음과 같이 타입 매개변수 대신 트레이트 객체를 사용해보겠습니다:

```rust
fn whatever(t: Box<MyTrait>) { ... }
// or
fn whatever(t: &MyTrait) { ... }
// etc...
```

이 코드를 컴파일할때 가상 환경에서 함수들을 동적 디스패치 (dynamic dispatch) 라는 메커니즘으로 처리를 하게 되는데, 한 가지 버전의 함수만 `.wasm` 파일로 처리되게 됩니다. 이러한 처리를 하면서 컴파일러 최적화 측면에서 손실이 있을 수 있고, 간접적이고 동적으로 디스패치된 함수를 부르는데 추가적인 비용이 들수도 있다는 단점이 있습니다.

### `wasm-snip` 툴을 사용해보세요
[`wasm-snip`은 WebAssembly 함수의 코드를 `unreachable` Assembly 명령어로 교체해줍니다.][snip] 하지만 잘 생각해보면 못 하나를 박는다고 아주 거대한 망치를 가져와서 열심히 두드리는 것 같은 느낌이 조금씩 듭니다.

런타임에서 사용하지 않는 함수들과 이 함수들이 간접적으로 호출하는 다른 함수들을 제거하고  싶은데, 컴파일러가 어떤 함수를 사용하지 않는지 모른다면 어떻게 해야 할까요? 우선은 코드를 빌드해보고 `wasm-opt`를 `--dce` 플래그를 포함해서 다시 실행해보세요! 간접적으로 호출되는 (런타임에서 부르지 않는) 함수들까지 지워버릴수 있습니다.

패닉 코드가 이런 문제들로 종종 이어질 수 있기 때문에, 패닉 인프라 (panicking infrastructure) 를 지울때 이 툴이 유용하게 사용될수도 있습니다.

[snip]: https://github.com/fitzgen/wasm-snip
