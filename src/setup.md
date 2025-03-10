# 시작하기

러스트로 웹어셈블리 개발을 하기 전에 개발 환경을 먼저 설정해야 합니다. 여러 버전의 Rust 컴파일러를 설치하고 관리하는데 필요한 [rustup][] (공식 툴) 을 설치하지 않았다면 어서 설치해보세요. 해당 웹사이트에 있는 지침을 따라서 컴퓨터에 설치할 수 있습니다. 지금으로서는 웹어셈블리 개발을 위해서 nightly 버전의 러스트가 필요합니다.

```bash
$ rustup default nightly
```

설치가 완료됐다면 `wasm32-unknown-unknown` 툴체인이 필요합니다.

```bash
$ rustup target add wasm32-unknown-unknown --toolchain nightly
```

다음으로, 더 작은 사이즈의 웹어셈블리 바이너리를 생성하는데 관심이 있다면 [wasm-gc][wasm-gc] 툴을 설치해서 컴파일러 툴체인으로 더 작은 바이너리를 생성하고 버그를 잡아보세요.

```bash
$ cargo install wasm-gc
```

그리고 마지막으로 작은 웹어셈블리 바이너리를 만드는데 **정말** 관심이 있다면, [binaryen toolkit][binaryen]의 `wasm-opt` 을 설치해보세요.

[rustup]: https://www.rustup.rs/
[binaryen]: https://github.com/WebAssembly/binaryen
[wasm-gc]: https://github.com/alexcrichton/wasm-gc
