<div align="center">

  <h1>The Rust and WebAssembly Book</h1>

  <strong>이 간결한 책은 Rust와 WebAssembly를 함께 사용하는 방법을 다룹니다. 튜토리얼과 알찬 연습 예제들로 구성돼 있습니다.</strong>

  <h3>
    <a href="https://cosmostellar.github.io/rust-wasm-book-ko/">책 읽기 (한국어)</a>
    <span> | </span>
    <a href="https://rustwasm.github.io/docs/book/">책 읽기 (영어)</a>
    <span> | </span>
    <a href="https://github.com/rustwasm/book/blob/master/CONTRIBUTING.md">기여하기</a>
    <span> | </span>
    <a href="https://discordapp.com/channels/442252698964721669/443151097398296587">채팅</a>
  </h3>

  <sub><a href="https://rustwasm.github.io/">The Rust and WebAssembly Group</a>이 🦀🕸 으로 만듬.</sub>
</div>

## 소개

이 레포지토리의 문서는 Rust를 wasm에 사용하는 방법이나 주로 사용되는 작업환경뿐 아니라 기초적인 부분부터 심화 내용까지 깊이 들어가게 되면서 더 심층적인 내용을 다루게 됩니다. Rust로 놀라운 작업을 할수 있도록 도와주는 가이드 정도로 생각해주세요!

Rust와 WebAssembly를 함께 사용하는 방법을 배우고 싶다면, 책을 [여기서 온라인으로](https://rustwasm.github.io/book/game-of-life/introduction.html) 읽어보세요.

["The Rust and WebAssembly book" 개선을 위해 이슈 열기.][book-issues]

[book-issues]: https://github.com/rustwasm/book/issues

## 책 빌드하기

이 책은 [`mdbook`][mdbook] 으로 만들어졌습니다. `mdbook`을 설치하려면 `cargo` 가 먼저 설치되어 있어야 하는데, 어떤 Rust 툴링이라도 설치되어 있지 않다면, [`rustup`][rustup]을 먼저 설치해야 합니다. 설치를 하려면 Rust 공식 웹사이트의 지침을 따라주세요.

설치가 완료됐다면 다음 과정을 계속 따라가주세요:

```bash
$ cargo install mdbook
```

바이너리 코드를 실행하기 위해 `cargo install` 경로가 `$PATH` 에 있는지 확인해주세요.

이제 이 경로에서 다음 명령어를 실행해주세요:

```bash
$ mdbook build
```

이 명렁어를 실행해서 책을 빌드하고 `book` 경로에 빌드된 파일들을 출력할수 있는데, 이 경로에 위치한 `index.html` 파일을 찾아 브라우저에서 확인해볼수 있습니다. 변경중인 내용을 실시간으로 확인하고 싶다면 다음 명령어를 실행하여 파일을 수정하면서 저장한 내용을 자동으로 반영할수도 있습니다:

```bash
$ mdbook serve
```

이 방법을 통해 자동으로 파일을 생성하고 로컬 환경에서 호스트할수 있습니다. 이렇게 `build`를 매번 실행할 필요 없이 변경된 내용을 쉽고 빠르게 확인할 수 있게 됩니다.

이러한 파일들은 모두 마크다운 문법으로 작성됐기 때문에, 직접 파일 생성 작업을 할 필요 없이 레포지토리의 `src` 경로에서 바로 읽어볼수도 있습니다.

[mdbook]: https://github.com/rust-lang-nursery/mdBook
[rustup]: https://github.com/rust-lang-nursery/rustup.rs/
[book]: https://rustwasm.github.io/book/game-of-life/introduction.html
