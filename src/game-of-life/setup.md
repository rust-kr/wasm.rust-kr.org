# 셋업하기

이 섹션은 어떻게 Rust 프로그램들을 WebAssembly로 컴파일하고 JavaScript 환경과 통합시키는지 설명해줍니다.

## Rust 툴체인

진행을 위해 `rustup`, `rustc`, `cargo`를 포함한 스탠다드 Rust 툴체인이 필요합니다.

[Rust 툴체인을 설치하려면 이 지침을 따라주세요.][rust-install]

Rust와 WebAssembly 개발 경험이 stable 버전의 러스트에 포함될 만큼 안정화 되고 있습니다! 그러므로 어떤 실험적 기능 flag도 요구되지 않습니다. 하지만 Rust 1.30이나 그 이후 버전이 요구됩니다.

## `wasm-pack`

`wasm-pack`은 Rust로 생성된 WebAssembly를 개발, 테스팅, 배포하도록 도와주는 만능 툴입니다.

[여기서 `wasm-pack` 다운로드 해보세요!][wasm-pack-install]

## `cargo-generate`

[`cargo-generate`는 기존에 존재하는 git 레포지토리를 템플릿으로 사용하면서 새 Rust 프로젝트를 시작하고 빠르게 구동할수 있도록 도와줍니다.][cargo-generate]

이 명령어로 `cargo-generate`를 설치해보세요:

```
cargo install cargo-generate
```

## `npm`

`npm`은 JavaScript와 함께 사용되는 패키지 매니저입니다. 이 책을 진행하면서 JavaScript 번들러와 개발 서버를 설치하고 구동하는데 사용될 예정입니다. 이 튜토리얼 끝에서 컴파일된 `.wasm`을 `npm` 레지스트리로 배포해봅니다.

[`npm` 을 설치하려면 이 지침을 따라주세요.][npm-install]

이미 `npm`이 설치돼 있다면, 이 명령어로 최신 버전으로 업데이트가 돼 있는지 확인해주세요:

```
npm install npm@latest -g
```

[rust-install]: https://www.rust-lang.org/tools/install
[npm-install]: https://www.npmjs.com/get-npm
[wasm-pack]: https://github.com/rustwasm/wasm-pack
[cargo-generate]: https://github.com/ashleygwilliams/cargo-generate
[wasm-pack-install]: https://rustwasm.github.io/wasm-pack/installer/
