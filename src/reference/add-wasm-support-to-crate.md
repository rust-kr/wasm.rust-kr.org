# 다목적 크레이트가 웹어셈블리를 지원하도록 코드 수정하기

이 섹션은 웹어셈블리를 지원하는 데에 관심이 있는 다목적 크레이트 저자들을 위해 작성됐습니다.

## 크레이트가 이미 웹어셈블리를 지원할 수도 있어요!

우선 [어떤 작업을 하는 다목적 크레이트가 웹어셈블리에 적합하지 **않나요**?](./which-crates-work-with-wasm.html) 의 내용을 먼저 읽어보세요. 작성하는 크레이트가 해당 사항이 없다면, 웹어셈블리를 지원할 확률이 높습니다.

추가로, 언제든 `cargo build` 명령어를 입력해서 웹어셈블리 타겟을 지원하는지 확인해 볼 수 있습니다:

```
cargo build --target wasm32-unknown-unknown
```

명령어 실행이 실패한다면 현재로서는 크레이트가 웹어셈블리를 지원하지 않는다는 의미입니다. 하지만 이 명령어가 성공하더라도 크레이트가 꼭 웹어셈블리를 지원한다는 의미는 아닙니다. 더 확실하게 확인하고 싶다면 [웹어셈블리 테스팅 코드를 추가하고 지속성 통합 (continuous integration) 환경에서 테스트를 실행해보기](#maintaining-ongoing-support-for-webassembly) 를 읽어보세요.

## 웹어셈블리 지원 추가하기

### 바로 I/O를 수행하지 말아주세요

웹 환경에서 I/O는 항상 비동기적일 뿐 아니라 파일 시스템도 존재하지 않습니다. 라이브러리의 I/O를 분리하고 유저들이 I/O를 직접 수행하고 슬라이스를 라이브러리로 대신 입력할 수 있도록 수정해 봅시다.

예를 들어서, 이 코드를 리팩토링해 주세요:

```rust
use std::fs;
use std::path::Path;

pub fn parse_thing(path: &Path) -> Result<MyThing, MyError> {
    let contents = fs::read(path)?;
    // ...
}
```

다음은 리팩토링 된 코드입니다:

```rust
pub fn parse_thing(contents: &[u8]) -> Result<MyThing, MyError> {
    // ...
}
```

### `wasm-bindgen`을 종속성으로 추가하기

(예를 들어서, 라이브러리 사용자가 상호작용을 직접 컨트롤하도록 허용하면 안 되는 경우처럼) 외부 환경과 따로 상호작용을 해야 하는 경우, (필요하다면 `js-sys`와 `web-sys`와 함께) `wasm-bindgen`을 컴파일, 타겟팅 종속성으로 추가해야 합니다:

```toml
[target.'cfg(target_arch = "wasm32")'.dependencies]
wasm-bindgen = "0.2"
js-sys = "0.3"
web-sys = "0.3"
```

### 동기적 I/O를 피해주세요

웹 환경에서는 비동기적 I/O만 수행할 수 있으므로 라이브러리에서 I/O 작업을 수행할 때는 동기적으로는 처리할 수 없습니다. [`futures` 크레이트](https://crates.io/crates/futures) 와 [`wasm-bindgen-futures` 크레이트](https://rustwasm.github.io/wasm-bindgen/api/wasm_bindgen_futures/)를 사용해서 비동기 I/O를 관리해 보세요. 라이브러리가 Future 타입 `F`를 제네릭 타입으로 사용한다면, 웹 환경의 `fetch`나 운영체제에서 제공하는 논블로킹 (non-blocking) I/O로 코드를 구현해 볼 수 있습니다.

```rust
pub fn do_stuff<F>(future: F) -> impl Future<Item = MyOtherThing>
where
    F: Future<Item = MyThing>,
{
    // ...
}
```

트레이트를 정의하고 웹어셈블리와 웹, 네이티브 타겟에서 실행할 수 있도록 구현해 볼 수도 있습니다:

```rust
trait ReadMyThing {
    type F: Future<Item = MyThing>;
    fn read(&self) -> Self::F;
}

#[cfg(target_arch = "wasm32")]
struct WebReadMyThing {
    // ...
}

#[cfg(target_arch = "wasm32")]
impl ReadMyThing for WebReadMyThing {
    // ...
}

#[cfg(not(target_arch = "wasm32"))]
struct NativeReadMyThing {
    // ...
}

#[cfg(not(target_arch = "wasm32"))]
impl ReadMyThing for NativeReadMyThing {
    // ...
}
```

### 스레드 생성을 피해주세요

웹어셈블리는 아직 스레드를 지원하지 않습니다. (하지만 [실험적으로 지원 작업이 진행되고 있긴 합니다.](https://rustwasm.github.io/2018/10/24/multithreading-rust-and-wasm.html)) 그러므로, 웹어셈블리에서 스레드를 생성하려고 시도하면 코드가 패닉하게 됩니다.

`#[cfg(..)]`를 사용하여 타겟이 웹어셈블리인지 아닌지 여부에 따라 스레드와 비스레드 코드를 따로 작성해 볼 수도 있습니다:

```rust
#![cfg(target_arch = "wasm32")]
fn do_work() {
    // 이 스레드에서만 작업을 수행합니다...
}

#![cfg(not(target_arch = "wasm32"))]
fn do_work() {
    use std::thread;

    // 헬퍼 스레드를 사용해서 작업을 확장합니다...
    thread::spawn(|| {
        // ...
    });
}
```

파일 I/O를 제거하고 유저들이 I/O 코드를 따로 가져올 수 있도록 허용하는 접근 방식과 유사하게, 스레드 생성 코드를 라이브러리에서 제외시킨 다음 유저들이 스레드 코드나 라이브러리를 따로 가져올 수 있도록 변경해 볼 수도 있습니다. 이렇게 구현했을 때 라이브러리 사용자들이 직접 스레드 풀 코드를 마련할 수 있게 되는 긍정적인 부수 효과를 보게 됩니다.

## 웹어셈블리의 지속적인 지원 유지하기

### 지속성 통합 (CI) 환경을 사용해서 `wasm32-unknown-unknown` 타겟으로 빌드하기

웹어셈블리를 타겟으로 할떄, 다음 명령어를 CI 스크립트에 포함시켜서 컴파일이 실패하지 않는 이유를 더 자세하게 확인할 수 있습니다:

```
rustup target add wasm32-unknown-unknown
cargo check --target wasm32-unknown-unknown
```

예를 들어서, Travis CI 설정 파일인 `.travis.yml` 에 다음 내용을 추가해 보겠습니다:

```yaml

matrix:
  include:
    - language: rust
      rust: stable
      name: "check wasm32 support"
      install: rustup target add wasm32-unknown-unknown
      script: cargo check --target wasm32-unknown-unknown
```

### Node.js와 [헤드리스 브라우저 (Headless Browsers)](https://ko.wikipedia.org/wiki/헤드리스_브라우저) 에서 테스팅하기

`wasm-bindgen-test`와 `wasm-pack-test` 하위 명령어 (subcommand) 를 사용해서 wasm 테스트들 Node.js나 헤드리스 브라우저에서 실행해보세요. 이러한 테스트들을 CI에 통합시켜 볼 수도 있습니다.

[여기서 웹어셈블리 테스팅에 대해 더 알아보세요.](https://rustwasm.github.io/wasm-bindgen/wasm-bindgen-test/index.html)
