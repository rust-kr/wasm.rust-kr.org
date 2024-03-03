# 튜토리얼: Conway's Game of Life

이 튜토리얼은 [Conway's Game of Life][gol]를 Rust와 WebAssembly로 구현하는 내용을 담고 있습니다.

[gol]: https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life

## 누구를 위한 튜토리얼인가요?

이 튜토리얼은 이미 기초적인 Rust와 JavaScript를 배웠고, Rust와 WebAssembly, JavaScript를 같이 사용하고 싶어하는 사람들을 위해 작성됐습니다.

기초적인 Rust, JavaScript, HTML 코드를 문제 없이 작성할 수 있어야 합니다. 하지만 전문가가 돼야 할 필요는 전혀 없습니다.

## 무엇을 배우게 되나요?

* WebAssembly 컴파일링을 할수 있도록 Rust 툴체인 셋업하는 법.

* Rust, WebAssembly, JavaScript, HTML, CSS으로 만든 다언어 프로그램 개발을 위한 워크플로우.

* Rust와 WebAssembly, 그리고 JavaScript의 강점을 모두 살리는 API를 디자인하는 방법.

* Rust 코드에서 컴파일된 WebAssembly 모듈 디버깅하는 법.

* Rust와 WebAssembly 프로그램을 더 빠르게 만들기 위해 타임 프로파일링 하는 법.

* `.wasm` 바이너리를 네트워크에서 다운로드 할때 더 작고 더 빠르게 만들기 위해 Rust와 WebAssembly 프로그램을 사이즈 프로파일링 하는 방법.