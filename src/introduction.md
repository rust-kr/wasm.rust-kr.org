# Rust 🦀 and WebAssembly 🕸
이 간결한 책은 [러스트][Rust]와 [웹어셈블리][WebAssembly]를 함께 사용하는 방법을 다룹니다.

> 번역된 버전에 이슈나 풀 리퀘스트를 생성하고자 하시나요? [번역 버전의 레포지토리](https://github.com/evasquare/rust-wasm-book-ko)를 확인해 주세요.

## 누구를 위한 책인가요?
이 책은 러스트와 웹어셈블리를 함께 사용하여 웹에서 동작하는 빠르고 안정된 코드를 작성하는 방법에 대해 관심이 있는 분들을 위해 작성되었습니다.
꼭 전문가가 돼야할 필요는 없지만, Rust를 조금이라도 알아야 하고 자바스크립트와 HTML, CSS에 익숙하면 더 좋습니다.

러스트를 아직 모르시나요? [*The Rust Programming Language* 책으로 시작해 보세요.][trpl]

자바스크립트나 HTML, CSS를 모르시나요? [MDN에서 더 알아보세요.][mdn]

## 이 책을 읽는 방법
[왜 러스트로 자바스크립트 개발을 해야 하나요?][why-rust-wasm] 섹션을 읽어보면 좋고, [배경지식][background]과 먼저 친숙해져 보는 것도 좋습니다.

이 [튜토리얼][tutorial]은 처음부터 끝까지 읽도록 작성됐습니다. 튜토리얼에 있는 코드를 작성, 컴파일하고 직접 실행해 보세요. Rust와 웹어셈블리를 같이 사용해 본 적이 없다면, 튜토리얼을 한번 활용해 보세요!

[참조 섹션][reference] 은 아무 순서로 정독해도 괜찮습니다.

> **💡 팁:** 페이지 최상단에 있는 🔍 아이콘을 누르거나 `s` 키를 눌러서 책 전체를 검색해 볼 수도 있습니다.

## 번역본

커뮤니티 [번역본](./reference/translations.md)도 있으니 한번 확인해 보세요.

## 이 책에 기여하기
이 책은 오픈소스입니다! 오타를 찾으셨나요? 누락된 부분이 있나요? [**풀 리퀘스트를 생성해 보세요!**][repo]

[Rust]: https://www.rust-lang.org
[WebAssembly]: https://webassembly.org/
[trpl]: https://doc.rust-lang.org/book/
[mdn]: https://developer.mozilla.org/en-US/docs/Learn
[why-rust-wasm]: ./why-rust-and-webassembly.html
[background]: ./background-and-concepts.html
[tutorial]: ./game-of-life/introduction.html
[reference]: ./reference/index.html
[repo]: https://github.com/rustwasm/book
[wat2wasm demo]: https://webassembly.github.io/wabt/demo/wat2wasm/
