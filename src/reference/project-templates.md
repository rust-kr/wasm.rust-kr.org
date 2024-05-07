# 프로젝트 템플릿

"The Rust and WebAssembly Working Group"은 개발자들이 빠르게 새 프로젝트를 시작하고 실행할수 있도록 여러가지 프로젝트 템플릿 목록을 만들고 관리합니다.

## `wasm-pack-template`

[`wasm-pack`][wasm-pack]으로 셋업된 [이 템플릿][wasm-pack-template]은 Rust와 WebAssembly 프로젝트를 손쉽게 시작할수 있도록 준비돼 있습니다.

`cargo generate` 명령어를 실행하여 프로젝트 템플릿을 클론해보세요:

```
cargo install cargo-generate
cargo generate --git https://github.com/rustwasm/wasm-pack-template.git
```

## `create-wasm-app`

[이 템플릿][create-wasm-app]은 [`wasm-pack`][wasm-pack]을 통해 배포된 npm 패키지를 간편하게 사용할 때 유용한 기능들을 포함합니다.

`npm init` 명령어로 사용해보세요:

```
mkdir my-project
cd my-project/
npm init wasm-app
```

이 템플릿은 주로 `wasm-pack-template`이라는 종속성괴 함께 사용되는데, `create-wasm-app`으로 프로젝트를 생성하면 `npm link` 기능으로 생성한 프로젝트와 연결하는데 사용됩니다.

## `rust-webpack-template`

[이 템플릿][rust-webpack-template]은 Rust 코드를 WebAssembly로 컴파일하고 출력된 파일들을 Webpack의 [`rust-loader`][rust-loader]를 사용해서 바로 파이프라인으로 후킹하여 연결할수 있도록 거의 모든 보일러플레이트들을 대신 설정해줍니다.

`npm init` 명령어로 사용해보세요:

```
mkdir my-project
cd my-project/
npm init rust-webpack
```

[wasm-pack]: https://github.com/rustwasm/wasm-pack
[wasm-pack-template]: https://github.com/rustwasm/wasm-pack-template
[create-wasm-app]: https://github.com/rustwasm/create-wasm-app
[rust-webpack-template]: https://github.com/rustwasm/rust-webpack-template
[rust-loader]: https://github.com/wasm-tool/rust-loader/
