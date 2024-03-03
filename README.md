<div align="center">

  <h1>The Rust and WebAssembly Book</h1>

  <strong>이 간결한 책은 어떻게 Rust와 WebAssembly를 함께 사용하는 방법을 설명해줍니다. 튜토리얼과 알찬 연습 예제들로 구성돼 있습니다.</strong>

  <h3>
    <a href="https://rustwasm.github.io/docs/book/">Read the Book</a>
    <a href="https://rustwasm.github.io/docs/book/">책 읽기</a>
    <span> | </span>
    <a href="https://github.com/rustwasm/book/blob/master/CONTRIBUTING.md">기여하기</a>
    <span> | </span>
    <a href="https://discordapp.com/channels/442252698964721669/443151097398296587">채팅</a>
  </h3>

  <sub><a href="https://rustwasm.github.io/">The Rust and WebAssembly Group</a>이 🦀🕸 으로 만듬.</sub>
</div>

## 소개

이 레포지토리는 어떻게 wasm 개발에 Rust와 일반적인 작업환경을 사용하여 시작하는 문서를 포함하고 있습니다.

이 레포지토리는 Rust를 wasm에 사용하는 것, 주로 사용되는 작업환경 구성, 어떻게 시작하는지, 그리고 더 깊게 들어갈수록 더 심층적인 내용을 포함합니다. 이 책은 Rust로 놀라운 것들을 할수있도록 도와주는 가이드의 역할을 합니다.

Rust와 WebAssembly를 함께 사용하는 방법을 배우고 싶다면, 책을 [여기서 온라인으로](https://rustwasm.github.io/book/game-of-life/introduction.html) 읽어보세요.

["The Rust and WebAssembly book" 개선을 위해 이슈 열기.](https://github.com/rustwasm/book/issues)

[책 이슈]: https://github.com/rustwasm/book/issues

## 책 빌드하기

이 책은 [`mdbook`][mdbook] 으로 만들어졌습니다. `mdbook`을 설치하려면 `cargo` 가 설치되어 있어야 합니다. 어떤 Rust 툴링이라도 설치되어 있지 않다면, [`rustup`][rustup]을 먼저 설치해야 합니다. 설치를 위해 웹사이트에 있는 지침을 따라주세요.

설치가 완료됐다면 다음 과정을 계속 따라가주세요:

```bash
$ cargo install mdbook
```

바이너리를 실행하기 위해 `cargo install` 디렉토리가 `$PATH` 에 있는지 확인해주세요.

이제 이 디렉토리에서 이 명령어를 실행하세요:

```bash
$ mdbook build
```

이 명렁어는 책을 빌드하고 `book` 이라는 디렉토리로 파일을 생성합니다. 이 디렉토리에서 `index.html` 파일을 찾아 브라우저로 열수 있습니다. 변경중인 내용을 보고 싶다면 다음 명령어를 실행하여 다음 명령어를 실행하여 변경된 내용을 자동으로 생성할수도 있습니다:

```bash
$ mdbook serve
```

이 명령어는 로컬 환경에서 구동하고 파일을 자동으로 생성합니다. 이렇게 `build`를 매번 실행할 필요 없이 변경된 내용을 쉽게 확인할수 있습니다. 이러한 파일들은 모두 마크다운 문법으로 작성되며, 책을 읽기 위해 파일을 생성을 하고싶지 않다면 `src` 디렉토리에서 볼수 있습니다.

[mdbook]: https://github.com/rust-lang-nursery/mdBook
[rustup]: https://github.com/rust-lang-nursery/rustup.rs/
[book]: https://rustwasm.github.io/book/game-of-life/introduction.html
