# 셋업

이 섹션에서는 Rust 프로그램들을 웹어셈블리로 컴파일하고 자바스크립트 환경과 통합시키는 방법에 대해 다루어 보겠습니다.

## 러스트 툴체인

진행을 위해 `rustup`, `rustc`, `cargo`를 포함한 스탠다드 Rust 툴체인이 필요합니다.

[러스트 툴체인을 설치하려면 이 지침을 따라주세요.][rust-install]

러스트와 웹어셈블리 개발 경험이 stable 버전의 러스트에 포함될 만큼 안정화되고 있기 때문에 어떤 실험적 기능 flag도 요구되지 않습니다. 하지만 Rust 1.30이나 그 이후 버전이 요구됩니다.

## `wasm-pack`

`wasm-pack`은 러스트로 생성한 웹어셈블리를 개발, 테스팅, 배포하도록 도와주는 만능 툴입니다.

[여기서 `wasm-pack` 다운로드 해보세요!][wasm-pack-install]

## `cargo-generate`

[`cargo-generate`를 통해 기존에 존재하는 git 레포지토리를 템플릿으로 사용하면서 새 러스트 프로젝트를 시작하고 빠르게 실행할 수 있습니다.][cargo-generate]

이 명령어로 `cargo-generate`를 설치해 보세요:

```
cargo install cargo-generate
```

## `npm`

`npm`은 자바스크립트와 함께 사용되는 패키지 매니저입니다. 이 책을 진행하면서 자바스크립트 번들러와 개발 서버를 설치하고 구동하는 데 사용될 예정입니다. 이 튜토리얼 끝에서는 컴파일된 `.wasm`을 `npm` 레지스트리에 배포해 봅니다.

[`npm` 을 설치하려면 이 지침을 따라주세요.][npm-install]

이미 `npm`이 설치돼 있다면, 이 명령어를 통해 최신 버전으로 업데이트돼 있는지 확인해 주세요:

```
npm install npm@latest -g
```

[rust-install]: https://www.rust-lang.org/tools/install
[npm-install]: https://www.npmjs.com/get-npm
[wasm-pack]: https://github.com/rustwasm/wasm-pack
[cargo-generate]: https://github.com/ashleygwilliams/cargo-generate
[wasm-pack-install]: https://rustwasm.github.io/wasm-pack/installer/
